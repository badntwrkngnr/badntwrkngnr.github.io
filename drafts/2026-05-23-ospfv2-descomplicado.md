# OSPFv2 Descomplicado

## Introdução

Olá, tudo bem? Escolhi este título em homenagem à LinuxTips e, embora o artigo traga bastante conteúdo, para mais detalhes, recomendo também o curso *Descomplicando o OSPF*, do professor Gustavo Kalau. Antes de mergulhar no OSPFv2, vale recapitular o papel de um protocolo de roteamento dinâmico, são conceitos simples, mas que preparam o terreno para o que vem em seguida.

As principais funções de um protocolo de roteamento incluem:

- **Descoberta de vizinhos e redes:** identificação automática (ou manual) de outros roteadores no mesmo segmento, eliminando o mapeamento manual de rotas a cada expansão.
- **Cálculo e seleção do melhor caminho (*path selection*):** uso de algoritmos que analisam a topologia (largura de banda, atraso, saltos etc.) para escolher a melhor rota por prefixo.
- **Prevenção de loops:** mecanismos que impedem pacotes de circular indefinidamente; em protocolos *link-state*, todos os nós calculam rotas a partir de um mapa topológico livre de loops.
- **Tolerância a falhas:** reação a quedas de link ou equipamento, recalculando rotas alternativas de forma autônoma.
- **Roteamento sem classe (*classless*):** suporte a VLSM e CIDR, transportando máscaras nas atualizações de roteamento.

## Conceitos fundamentais

O OSPFv2 (*Open Shortest Path First version 2*) é um protocolo de roteamento dinâmico interno (IGP) do tipo *link-state*, padronizado pela IETF na RFC 2328. Para entender o que isso significa na prática, vamos comparar com protocolos mais antigos, como o RIP (*distance vector*):

- **Distance vector:** o roteamento usa a quantidade de saltos, quanto menos, melhor. É como dirigir olhando só placas de distância: você confia no número, mas não sabe como é a estrada, se há congestionamento ou como as vias se conectam. Com interfaces em velocidades diferentes, o caminho com menos saltos pode ser o mais lento.
- **Link-state:** os roteadores trocam peças de um quebra-cabeça (os LSAs — *Link-State Advertisements*). Reunidas na LSDB (*Link-State Database*), **cada roteador monta um mapa topológico idêntico e completo da área**: todos os nós, links, status (Up/Down) e o "pedágio" (custo) de cada trecho.

O ciclo de vida do OSPF em uma área resume-se em três fases:

1. Descoberta de vizinhos e formação de adjacências
2. Construção e sincronização da LSDB
3. Execução do algoritmo SPF (*Dijkstra*)

## Adjacência na prática

Para demonstrar a **fase 1**, vamos observar o comportamento da topologia abaixo, começando pela área 0:

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-topologia-explicacao.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado.png]]

Descobrir um vizinho é só o começo. Para sincronizar a LSDB, dois roteadores OSPF passam por uma máquina de estados de sete passos. A comunicação ocorre diretamente sobre IP (protocolo 89), com cinco tipos de pacote, **Hello**, **DBD**, **LSR**, **LSU** e **LSAck**, que aparecem conforme o avanço dos estados:

1. **Down (inativo):** estado inicial. O processo OSPF está ativo na interface, mas nenhum *Hello* foi recebido. O roteador envia *Hello* para o multicast **224.0.0.5** (*All OSPF Routers*) e aguarda resposta.
2. **Init (inicialização):** chegou um *Hello* de um vizinho, mas a comunicação ainda é unidirecional. O RID local não aparece no campo de vizinhos ativos do pacote recebido.
3. **2-Way (bidirecional):** o *Hello* lista o próprio *Router ID (RID)*. A comunicação bidirecional está confirmada. O protocolo avalia o **tipo de rede (*Network Type*)** da interface e decide o próximo passo. Neste artigo, vamos falar somente sobre os dois tipos mais comuns:

	- **Point-to-Point:** conexões entre dois nós. A eleição de DR/BDR é ignorada e o processo segue para os estados seguintes.
	- **Broadcast (multiacesso):** padrão em Ethernet. Com vários dispositivos no segmento, o OSPF elege um *Designated Router* (DR) e um *Backup Designated Router* (BDR) para centralizar a troca de LSAs; os demais (*DROTHERs*) mantêm adjacência **2-Way** entre si e só avançam com DR e BDR.

4. **ExStart:** a comunicação básica está pronta; falta sincronizar a LSDB. Os pares elegem *Master* e *Slave* (maior RID vence) para definir a sequência inicial dos pacotes **DBD** (*Database Description*).
5. **Exchange:** *Master* e *Slave* trocam DBDs, o "índice" da LSDB, com cabeçalhos de LSA para comparação, ainda sem o conteúdo completo das rotas.
6. **Loading:** após os DBDs, o roteador pede LSAs ausentes ou mais recentes via **LSR** (*Link-State Request*). O vizinho responde com **LSU** (*Link-State Update*) e o solicitante confirma com **LSAck** (*Link-State Acknowledgment*).
7. **Full:** as LSDBs estão idênticas e sincronizadas na área — pode gritar ***É TETRA!*** Só então o roteador executa o SPF e instala as melhores rotas na tabela de roteamento (RIB).

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-1.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-2.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-3.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-4.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-5.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-6.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-7.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-8.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-9.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-10.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-11.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-12.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-13.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-14.png]]

![[../assets/images/networking/2026-05-23-ospfv2-descomplicado/2026-05-23-ospfv2-descomplicado-15.png]]

As fases 2 e 3 ocorrem nos estados **Exchange**, **Loading** e **Full**: a LSDB fica sincronizada e, em seguida, cada roteador roda o SPF de forma independente para popular a RIB.

## SPF e métrica

Com o mapa da rede (LSDB) em mãos, o OSPF traça o melhor caminho com o algoritmo **SPF (*Shortest Path First*)**, de Edsger W. Dijkstra. O cálculo segue quatro passos:

**1. Ponto de partida:** cada roteador executa o SPF de forma independente, colocando **a si mesmo** como raiz da *Shortest Path Tree* (SPT).

**2. Métrica:** o critério de melhor caminho é o **custo**, inversamente proporcional à largura de banda do link — links mais rápidos, pedágio mais barato.

> **Custo = referência de banda / largura de banda da interface.** Por padrão, a referência é 100 Mbps, o que faz FastEthernet e GigabitEthernet terem custo 1; em redes modernas, use **auto-cost reference-bandwidth** para não distorcer o cálculo.

**3. Custo cumulativo:** o roteador soma o custo de saída de cada interface ao longo do caminho até cada destino.

**4. Instalação na RIB:** para a mesma sub-rede, vence o menor custo cumulativo; o perdedor permanece só na LSDB. A rota vencedora vai para a **RIB**. Em caso de empate, o OSPF instala ambos os caminhos e faz balanceamento (ECMP, *Equal-Cost Multipathing*).

## Características e capacidades

- **Áreas:** a topologia se divide em áreas para isolar falhas e reduzir execuções de SPF. A área 0 é o *backbone*; as demais conectam-se a ela (fisicamente ou via *virtual link*).
- **Router ID:** identidade única de 32 bits por roteador (formato de IPv4, mas não é um endereço utilizável para tráfego de usuário).
- **DR e BDR:** em segmentos broadcast, o OSPF elege um *Designated Router* e um *Backup Designated Router* para evitar adjacências completas entre todos os roteadores. Os *DROTHERs* reportam mudanças ao DR, que replica a informação no segmento.
- **Sumarização:** nos limites de área (ABRs) ou na redistribuição (ASBRs), condensando prefixos para estabilizar a LSDB.
- **Tipos de área (Stub, Totally Stubby, NSSA):** áreas que não precisam de rotas externas completas, recebendo rota padrão e economizando CPU e memória.
- **Autenticação:** *clear-text* ou criptografia (MD5, HMAC-SHA) para impedir roteadores não autorizados no domínio.

## Troubleshooting

Nesta fase, a abordagem muda da configuração para a análise do *control-plane*. O isolamento de falhas OSPF costuma dividir-se em: falha na formação de vizinhança e problemas com LSAs ou tabelas de roteamento.

### Comandos de Verificação

Durante auditorias ou exames práticos, o isolamento de anomalias depende dos comandos listados abaixo para diagnosticar as estruturas internas do OSPF:

- `show ip ospf neighbor` — A primeira linha de defesa. Analise o *State* (deve estar *Full* ou *2-Way*). Qualquer estado intermediário prolongado (Init, ExStart) denuncia falhas de negociação.
- `show ip ospf interface [brief]` — Audita as redes participantes, verifica os *Timers* (Hello/Dead), a máscara de rede embutida na interface e o *Network Type* ativado na porta.
- `show ip route ospf` — Inspeciona se as rotas processadas pelo algoritmo de Dijkstra chegaram efetivamente à RIB e valida a preferência de caminhos (O, O IA, O E1, O E2).
- `show ip ospf database` — Permite abrir o "cérebro" do protocolo. Mostra a LSDB, expondo quem originou cada LSA (Router Link, Net Link, Summary Net Link) para rastrear o caminho exato e identificar filtros inesperados.
- `show ip ospf` — Fornece estatísticas cruciais do processo global, revelando a quantidade de execuções do algoritmo SPF, que aponta se a rede está instável (flapping).
- `clear ip ospf process` — Executa um reset pesado do protocolo, limpando a LSDB e renegociando vizinhos do zero; essencial após alterar variáveis amarradas ao processo como o Router ID.
- `debug ip ospf adj` / `debug ip ospf hello` — Para dissecção em tempo real. Essencial para ler nas entrelinhas falhas silenciosas de MTU, senhas incompatíveis ou falhas de *flags* nas mensagens de descoberta.

### Problemas Comuns e Localização de Falhas

**1. Vizinhança Travada (Problemas de Control-Plane):**

- **Travado em INIT:** Indica comunicação unidirecional. Um roteador ouve os Hellos, mas seu Router ID não consta na lista de vizinhos ativos do pacote recebido. Causas: ACLs bloqueando multicast (224.0.0.5), falhas L2 ou divergências rigorosas de parâmetros.

- **Mismatched Timers / Subnets:** Hellos, Dead intervals e a subnet mask devem ser rigorosamente idênticos. O teste prático para isso é o `debug ip ospf hello`, que fará *flash* na tela alertando sobre parâmetros incompatíveis.

- **Travado em EXSTART / EXCHANGE (MTU Mismatch):** Um dos problemas operacionais mais clássicos. O OSPF recusa-se a prosseguir na negociação de banco de dados se o tamanho do payload L2 não coincidir entre os pares. O processo trava na tentativa de enviar as mensagens DBD. Teste e valide sempre verificando o tamanho do IP MTU em ambas as pontas da interface física ou túnel.

- **Falha de Area Type (Stub Flag):** Se um roteador tem a área configurada como *Stub* e o outro como *Normal*, a flag nos pacotes Hello diverge, e a adjacência é descartada imediatamente.

**2. Manipulação de Rotas e Distribuição (Problemas de Data-Plane / LSA):**

- **Roteamento Assimétrico em Redistribuição:** Redistribuir informações (de EIGRP ou BGP para o OSPF) exige extrema atenção à `seed-metric` e aos LSAs Tipo 5 ou 7. A injeção dupla sem ajustes de custo leva o algoritmo a rotear por caminhos subótimos. As rotas O E1 (somam métricas internas) e O E2 (métrica estática inalterada) possuem forte hierarquia na seleção de caminhos.

- **Rotas Fantasmas ou Discontiguous Area 0:** Uma interrupção lógica na Área 0 particiona a rede, impedindo que LSAs Tipo 3 cruzem de um lado para o outro. Testes de falha geralmente apontam rotas Inter-area sumindo misteriosamente da RIB. A solução emergencial, mas provisória, é configurar um *Virtual Link* transitando por uma área de suporte.

- **Supressão de Rotas (Filtering):** Falhas causadas pelo uso de comandos como `area range not-advertise` em um ABR, que engolem silenciosamente os LSAs Tipo 3. O troubleshooting deve começar analisando a `show ip ospf database summary` no ABR originador.

O diagnóstico reativo via linha de comando deve ser sistemático, mas a inteligência operacional se alcança com monitoramento proativo. Exportar o estado das adjacências OSPF, contadores de SPF runs e mudanças bruscas de métricas para ferramentas de observabilidade — alimentando dashboards de alta visibilidade no Grafana via Zabbix — retira o analista da postura apagadora de incêndios e permite a detecção de *flaps* no OSPF antes que virem um incidente perceptível de conectividade.
