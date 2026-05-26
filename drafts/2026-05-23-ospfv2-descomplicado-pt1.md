# OSPF Descomplicado

## Introdução

Olá, tudo bem? Era para ser somente um artigo abordando um assunto, mas acabou que, conforme eu ia escrevendo, o texto foi ficando cada vez mais extenso. No meio do caminho, resolvi mudar o foco e separar em partes para não ficar cansativo demais. Para começar, escolhi este título por ser *fanboy* da [*LinuxTips*](https://linuxtips.io/) e, embora o artigo traga bastante conteúdo, recomendo o curso [*Descomplicando o OSPF*](https://gustavokalau.com.br/), do professor Gustavo Kalau, para um aprofundamento um pouco maior.
Antes de falarmos sobre o OSPF propriamente dito, vale recapitular o papel de um protocolo de roteamento dinâmico. São conceitos simples, mas que preparam o terreno para o que vem a seguir.

As principais funções de um protocolo de roteamento incluem:

- **Descoberta de vizinhos e redes:** identificação automática (ou manual) de outros roteadores no mesmo segmento, eliminando o mapeamento manual de rotas a cada expansão.
- **Cálculo e seleção do melhor caminho (*path selection*):** uso de algoritmos que analisam a topologia (largura de banda, atraso, saltos etc.) para escolher a melhor rota por prefixo.
- **Prevenção de loops:** mecanismos que impedem pacotes de circular indefinidamente. Em protocolos *link-state*, que é o caso do OSPF, todos os nós calculam rotas a partir de um mapa topológico livre de loops.
- **Tolerância a falhas:** reação a quedas de link ou equipamento, recalculando rotas alternativas.
- **Roteamento sem classe (*classless*):** suporte a VLSM e CIDR, transportando máscaras nas atualizações de roteamento.

## Conceitos Fundamentais

Neste momento, vamos focar no OSPF versão 2. O OSPFv2 (*Open Shortest Path First version 2*) é um protocolo de roteamento dinâmico interno (IGP) do tipo *link-state*, padronizado pela IETF na RFC 2328. Para entender o que isso realmente significa, vamos comparar com protocolos mais antigos, como o RIP (*distance vector*):

- **Distance vector:** o roteamento usa como parâmetro a quantidade de saltos (quanto menos, melhor). É como dirigir olhando só para as placas de distância: você confia no número, mas não sabe como é a estrada, se há congestionamento ou como as vias se conectam. Na prática, as interfaces operam em velocidades diferentes, o caminho com menos saltos pode ser o mais lento, de forma paradoxal, o melhor caminho pode ser justamente o pior.
- **Link-state:** os roteadores trocam informações, como se fossem peças de um quebra-cabeça (os LSAs, *Link-State Advertisements*). Reunidas na LSDB (*Link-State Database*), que ao ser sincronizada podemos considerar que o quebra-cabeças está completo, **cada roteador monta um mapa topológico idêntico e completo da área**: todos os nós, links, status (Up/Down) e o "pedágio" (custo) de cada trecho.

O ciclo de vida do OSPF em uma área resume-se a três fases:

1. Descoberta de vizinhos e formação de adjacências
2. Construção e sincronização da LSDB
3. Execução do algoritmo SPF (*Dijkstra*)

### Descoberta de Vizinhos e Formação de Adjacências

Descobrir um vizinho (fase 1) é só o começo. Mas, atenção: antes de começarem a trocar informações e passarem pela máquina de estados, os roteadores precisam concordar num verdadeiro "acordo de cavalheiros". Através dos pacotes Hello, eles validam um checklist. Se algo estiver fora do lugar, o processo de criar uma vizinhança nem começa!

Para que o processo de formação de adjacência seja iniciado, as seguintes configurações precisam ser exatamente iguais em ambos os lados do link:

- **ID da Área:** Se um roteador diz pertencer à Área 0 e o outro à Área 1 no mesmo link, eles ignoram-se.
- **Sub-rede e Máscara:** Os IPs das interfaces conectadas têm de pertencer rigorosamente à mesma rede.
- **Hello e Dead Timers:** O ritmo da conversa tem de bater (geralmente 10s para o Hello e 40s para o Dead Timer).
- **Autenticação:** Se houver uma senha configurada, ela precisa ser idêntica.
- **Flags de Área Especial (Stub/NSSA):** Ambos precisam concordar sobre o tipo de área em que estão inseridos.

Além destas, existem duas regras para garantir que o processo não trave mais adiante, durante a troca da base de dados:

- **Router IDs Únicos:** Cada roteador precisa do seu próprio documento de identidade (RID). Se houver duplicidade na rede, a matriz de rotas entra em colapso.
- **MTU (Maximum Transmission Unit) Igual:** O tamanho máximo de pacote suportado nas interfaces precisa ser o mesmo. Se um lado tiver um MTU de 1500 bytes e o outro 1400, eles começam a conversar, mas a adjacência "trava" a meio do caminho. Importante lembrar que isso ocorre porque o OSPF não suporta fragmentação.

Com tudo isso previamente alinhado, para sincronizar a LSDB, os roteadores OSPF passam por uma máquina de estados de sete passos. A comunicação ocorre diretamente sobre IP (protocol ID 89), com cinco tipos de pacote, **Hello**, **DBD**, **LSR**, **LSU** e **LSAck**, que aparecem conforme o avanço dos estados:

1. **Down (inativo):** estado inicial. O processo OSPF está ativo na interface, mas nenhum *Hello* foi recebido. O roteador envia *Hello* para o multicast **224.0.0.5** (*All OSPF Routers*) e aguarda resposta.
2. **Init (inicialização):** chegou um *Hello* de um vizinho, mas a comunicação ainda é unidirecional. O RID local não aparece no campo de vizinhos ativos do pacote recebido.
3. **2-Way (bidirecional):** o *Hello* lista o próprio *Router ID (RID)*. A comunicação bidirecional está confirmada. O protocolo avalia o **tipo de rede (*Network Type*)** da interface e decide o próximo passo. Neste material, vamos falar somente sobre os dois tipos mais comuns:
3.1. **Point-to-Point:** conexões entre dois nós. A eleição de DR/BDR é ignorada e o processo segue para os estados seguintes.
3.2. **Broadcast (multiacesso):** padrão em Ethernet. Com vários dispositivos no segmento, o OSPF elege um *Designated Router* (DR) e um *Backup Designated Router* (BDR) para centralizar a troca de LSAs, os demais (*DROTHERs*) mantêm adjacência **2-Way** entre si e só avançam com DR e BDR. Vamos detalhar posteriormente, mas isso acontece para evitar problemas de escalabilidade.
4. **ExStart:** a comunicação básica está pronta, falta apenas sincronizar a LSDB. Os pares elegem *Master* e *Slave* (maior RID vence) para definir a sequência inicial dos pacotes **DBD** (*Database Description*).
5. **Exchange:** *Master* e *Slave* trocam DBDs, o "índice" da LSDB, com cabeçalhos de LSA para comparação, ainda sem o conteúdo completo das rotas.
6. **Loading:** após os DBDs, o roteador pede LSAs ausentes ou mais recentes via **LSR** (*Link-State Request*). O vizinho responde com **LSU** (*Link-State Update*) e o solicitante confirma com **LSAck** (*Link-State Acknowledgment*).
7. **Full:** as LSDBs estão idênticas e sincronizadas na área, agora pode tirar o grito ***É TETRA*** da garganta! Finalmente o roteador executa o SPF e instala as melhores rotas na tabela de roteamento (RIB).

> Nota: As fases 2 e 3 ocorrem nos estados **Exchange**, **Loading** e **Full**: a LSDB fica sincronizada e, em seguida, cada roteador roda o SPF de forma independente para popular a RIB.

## SPF e Métrica

Com o mapa da rede (LSDB) em mãos, o OSPF traça o melhor caminho com o algoritmo **SPF (*Shortest Path First*)**, de Edsger W. Dijkstra. O cálculo segue quatro passos:

**1. Ponto de partida:** cada roteador executa o SPF de forma independente, colocando **a si mesmo** como raiz da *Shortest Path Tree* (SPT).

**2. Métrica:** o critério de melhor caminho é o **custo**, inversamente proporcional à largura de banda do link, links mais rápidos, pedágio mais barato.

> **Custo = referência de banda / largura de banda da interface.** Por padrão, a referência é 100 Mbps, o que faz FastEthernet e GigabitEthernet terem custo 1; em redes mais modernas isso pode ser um problema, use **auto-cost reference-bandwidth** para não distorcer o cálculo.

**3. Custo cumulativo:** o roteador soma o custo de saída de cada interface ao longo do caminho até cada destino.

**4. Instalação na RIB:** para a mesma sub-rede, vence o menor custo cumulativo, o perdedor permanece só na LSDB. A rota vencedora vai para a **RIB**. Em caso de empate, o OSPF instala ambos os caminhos e faz balanceamento (ECMP, *Equal-Cost Multipathing*).

## Características e Capacidades

- **Áreas:** a topologia se divide em áreas para isolar falhas e reduzir execuções de SPF. A área 0 é o *backbone*, as demais conectam-se a ela (fisicamente ou via *virtual link*).
- **Router ID:** identidade única de 32 bits por roteador (formato de IPv4, mas não é um endereço utilizável para tráfego de usuário).
- **DR e BDR:** em segmentos broadcast, o OSPF elege um *Designated Router* e um *Backup Designated Router* para evitar adjacências completas entre todos os roteadores. Os *DROTHERs* reportam mudanças ao DR, que replica a informação no segmento.
- **Sumarização:** nos limites de área (ABRs) ou na redistribuição (ASBRs), condensando prefixos para estabilizar a LSDB. Vamos detalhar essas funções no próximo artigo da série.
- **Tipos de área (Stub, Totally Stubby, NSSA):** áreas que não precisam de rotas externas completas, recebendo rota padrão e economizando CPU e memória.
- **Autenticação:** *clear-text* ou criptografia (MD5, HMAC-SHA) para impedir roteadores não autorizados no domínio.

## Conclusão: O Fim do Início

Finalmente chegamos ao fim desta primeira etapa. Como pudemos ver, o OSPF não é apenas um protocolo que sai anunciando rotas sem critério algum. Ele é metódico, exige um "match" perfeito (quase um *Tinder* dos roteadores) com os pacotes Hello e constrói um mapa completo de toda a rede antes de tomar qualquer decisão, usando o algoritmo de Dijkstra.

É meio chato ser repetitivo e falar o que todo mundo fala, mas entender esses fundamentos teóricos, desde as regras para formar vizinhança, passando pela máquina de estados, até chegar ao cálculo do custo, é exatamente o que separa um mero "digitador de comandos" de um verdadeiro Engenheiro de Redes. Posso te afirmar com base na minha experiência, quando algo der errado, é essa base que vai te ajudar!

Mas convenhamos... teoria sem prática é igual a Buchecha sem Claudinho, Piu-piu sem Frajola e etc. Apenas me perdoem pelas referências! Agora que já sabemos como algumas coisas acontecem por baixo do capô, está chegando a hora de aplicar alguns comandos e ver as coisas funcionando de forma empírica.

Na Parte 2 desta série, vamos focar 100% na implementação. Vamos abrir a CLI, configurar o processo OSPF do zero, seja no bom e velho modelo clássico (`network statement`) ou por interface, configurar Router IDs (ou deixar o roteador escolher), forçar eleições de DR/BDR e ver aquelas mensagens de log deliciosas pipocando na tela avisando que a adjacência chegou a FULL e está tudo funcionando como deveria.

Então, pode escolher o seu emulador favorito (Packet Tracer, GNS3, EVE-NG, PNETLab) e até o próximo artigo da série. Só para adiantar, eu utilizo o PNETLab, vou disponibilizar o arquivos iniciais na próxima parte, vale lembrar que o arquivo é totalmente compatível com o EVE-NG e que já vai estar com as interfaces pré-configuradas, hein!

Vamos dominar juntos o OSPF e, como diriam por aí: VAAAAAAAAI!
