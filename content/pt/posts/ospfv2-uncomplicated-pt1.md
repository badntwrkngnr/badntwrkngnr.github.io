---
title: "OSPF Descomplicado - Parte 1"
slug: "ospfv2-uncomplicated-pt1" 
date: 2026-05-23
translationKey: "ospfv2-uncomplicated-pt1"
categories: ["networking"]
math: true
draft: false
---

## Introdução

Olá, tudo bem contigo? Era para ser somente um artigo completo abordando um assunto do começo ao fim, mas acabou que, conforme eu ia escrevendo, o texto ia ficando cada vez mais extenso, isso acontece porque eu tenho uma dificuldade colossal de identificar o momento correto de encerrar alguns assuntos. No meio do caminho, achei melhor mudar a abordagem e separar em partes para não ficar cansativo demais. Para começar, escolhi este título por ser *fanboy* da [*LinuxTips*](https://linuxtips.io/) e, embora o artigo traga bastante conteúdo, aproveito para recomendar o curso [*Descomplicando o OSPF*](https://gustavokalau.com.br/), do professor Gustavo Kalau, para um aprofundamento um pouco maior.
Antes de falarmos sobre o OSPF propriamente dito, vale recapitular o papel de um protocolo de roteamento dinâmico. São conceitos simples, mas que são fundamentais e vão preparar o terreno para o que vem a seguir.

As principais funções de um protocolo de roteamento incluem:

- **Descoberta de vizinhos e redes:** identificação automática (ou manual) de outros roteadores no mesmo segmento, eliminando o mapeamento manual de rotas a cada expansão.
- **Cálculo e seleção do melhor caminho (*path selection*):** uso de algoritmos que analisam a topologia (largura de banda, atraso, saltos etc.) para escolher a melhor rota por prefixo.
- **Prevenção de loops:** mecanismos que impedem pacotes de circular indefinidamente. Em protocolos *link-state*, que é o caso do OSPF, todos os nós calculam rotas a partir de um mapa topológico livre de loops.
- **Tolerância a falhas:** reação a quedas de link ou equipamento, recalculando rotas alternativas.
- **Roteamento sem classe (*classless*):** suporte a VLSM e CIDR, transportando máscaras nas atualizações de roteamento.

## Conceitos Fundamentais

Neste momento, vamos começar a falar sobre o protocolo OSPF na versão 2. O OSPFv2 (*Open Shortest Path First version 2*) é um protocolo de roteamento dinâmico interno (IGP) do tipo *link-state*, padronizado pela IETF na RFC 2328. Para entender o que isso realmente significa, vamos comparar com protocolos mais antigos, como o RIP (*distance vector*):

- **Distance vector:** o roteamento usa como parâmetro a quantidade de saltos (quanto menos, melhor). É como dirigir olhando só para as placas de distância: você confia no número, mas não sabe como é a estrada, se há congestionamento ou como as vias se conectam. Na prática, as interfaces operam em velocidades diferentes, o caminho com menos saltos pode ser o mais lento e, paradoxalmente, o melhor caminho pode ser justamente o pior.
- **Link-state:** os roteadores trocam informações, como se fossem peças de um quebra-cabeça (no caso do OSPF, essas mensagens são os LSAs, *Link-State Advertisements*). Reunidas na LSDB (*Link-State Database*), quando sincronizada, o quebra-cabeça está completo e **cada roteador monta um mapa topológico idêntico e completo da área**: todos os nós, links, status (Up/Down) e o "pedágio" (custo) de cada trecho.

O ciclo de vida do OSPF em uma área resume-se a três fases:

1. Descoberta de vizinhos e formação de adjacências
2. Construção e sincronização da LSDB
3. Execução do algoritmo SPF (*Dijkstra*)

### SPF e Métrica

Com o mapa da rede (LSDB) em mãos, o OSPF traça o melhor caminho com o algoritmo **SPF (*Shortest Path First*)**, de Edsger W. Dijkstra. O cálculo segue quatro passos:

**1. Ponto de partida:** cada roteador executa o SPF de forma independente, colocando **a si mesmo** como raiz da *Shortest Path Tree* (SPT), .

**2. Métrica:** o critério de melhor caminho é o **custo**, inversamente proporcional à largura de banda do link, links mais rápidos, pedágio mais barato.

![OSPF Cost Formula](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-11.png)

O OSPF considera as condições do caminho, mas preste atenção em uma coisa extremamente importante: ele considera apenas a **largura de banda** como parâmetro para calcular a sua métrica. Outro detalhe importante: por padrão, a banda de referência é de 100 Mbps no Cisco IOS, o que faz portas FastEthernet e GigabitEthernet terem o mesmo custo **1**, isso pode ser um problema, para corrigir, utilize o comando **auto-cost reference-bandwidth** em todos os dispositivos do domínio OSPF para não distorcer o cálculo, no material de troubleshooting que será produzido, vamos demonstrar na prática. Apenas como curiosidade e para efeitos de comparação, o protocolo EIGRP pode considerar a largura de banda, atraso, carga e confiabilidade.

**3. Custo cumulativo:** o roteador soma o custo de saída de cada interface ao longo do caminho até cada destino.

**4. Instalação na RIB:** para a mesma sub-rede, vence o menor custo cumulativo, o perdedor permanece só na LSDB. A rota vencedora vai para a **RIB**. Em caso de empate, o OSPF instala ambos os caminhos e faz balanceamento (ECMP, *Equal-Cost Multipathing*).

Para demonstrar o comportamento entre os dois tipos de algoritmos mencionados acima, vamos utilizar a topologia abaixo:

![Topologia RIP e OSPF](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1.png)

Vamos exemplificar primeiramente o funcionamento de um protocolo que utiliza o algoritmo *Distance Vector*, o RIP:

- Como ponto de partida, vamos dizer que as conexões em azul são *Links Ethernet* e as conexões em amarelo são *Links Seriais*.
- Agora, imagine que um pacote precisa sair do roteador RT2 e chegar ao RT7.
- Baseado na topologia que temos, o caminho mais curto seria RT2 > RT4 > RT7.
- Com base na velocidade dos links, podemos dizer que o "melhor caminho" talvez não seja o mais eficiente (rápido) para chegar ao destino.

![Caminho RIP na topologia](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-1.png)

Agora, vamos exemplificar o funcionamento básico do OSPF, que utiliza a lógica *link-state*. Com a fórmula de cálculo fornecida, podemos definir os custos conforme a topologia abaixo:

![Custos OSPF na topologia](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-2.png)

Agora, com o valor do custo acumulado da origem até chegar no destino calculado pelo OSPF, vamos observar o comportamento do encaminhamento de pacotes na prática!
A topologia já está com OSPF habilitado, explicaremos as duas formas de ativação na seção de adjacência. O arquivo dos labs está disponível para [*download*](https://drive.google.com/drive/folders/1PFSQtfXWONkJiUJ5dYLB7eKL-c4M5bcl?usp=drive_link), você pode baixar todos eles e praticar o quanto quiser, lembrando que utilizo o PNETLab, portanto o arquivo também é compatível com o EVE-NG!
Ao examinarmos a tabela de roteamento, podemos ver que estamos recebendo a rota de destino da rede que está localizada no roteador RT7 com um custo de 8. Você pode estar se perguntando como, já que se somarmos os custos das interfaces de saída até o destino, o valor obtido é 7. Aqui reside um detalhe minúsculo que pode confundir a linha de raciocínio durante uma análise da tabela de roteamento. Conforme definido na RFC do OSPF e implementado pelo Cisco IOS, a interface loopback é tratada como *stub host* e sempre vai receber um custo fixo e automático de 1. Me vejo forçado a falar sobre o termo *stub* antes da hora, mas para o entendimento do conceito, preciso que você imagine que no contexto de redes é como se fosse o "fim da linha" ou podemos entender como um "beco sem saída". É um dispositivo que o tráfego pode entrar e sair normalmente, porém nunca vai chegar diretamente a um terceiro destino porque tem um único caminho para o tráfego chegar e sair. Não confunda com *stub area*, que é outro conceito, com o natural aprofundamento do tema, falaremos disso nos próximos artigos.

```cisco
RT2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override

Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O        1.1.1.1 [110/2] via 10.1.12.1, 00:00:16, Ethernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
C        2.2.2.2 is directly connected, Loopback0
      3.0.0.0/32 is subnetted, 1 subnets
O        3.3.3.3 [110/3] via 10.1.12.1, 00:00:16, Ethernet0/0
      4.0.0.0/32 is subnetted, 1 subnets
O        4.4.4.4 [110/4] via 10.1.12.1, 00:00:06, Ethernet0/0
      5.0.0.0/32 is subnetted, 1 subnets
O        5.5.5.5 [110/7] via 10.1.12.1, 00:00:06, Ethernet0/0
      6.0.0.0/32 is subnetted, 1 subnets
O        6.6.6.6 [110/5] via 10.1.12.1, 00:00:06, Ethernet0/0
      7.0.0.0/32 is subnetted, 1 subnets
O        7.7.7.7 [110/8] via 10.1.12.1, 00:00:06, Ethernet0/0
      8.0.0.0/32 is subnetted, 1 subnets
O        8.8.8.8 [110/6] via 10.1.12.1, 00:00:06, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 11 subnets, 2 masks
C        10.1.12.0/24 is directly connected, Ethernet0/0
L        10.1.12.2/32 is directly connected, Ethernet0/0
O        10.1.13.0/24 [110/2] via 10.1.12.1, 00:00:16, Ethernet0/0
C        10.1.24.0/24 is directly connected, Serial1/0
L        10.1.24.2/32 is directly connected, Serial1/0
O        10.1.34.0/24 [110/3] via 10.1.12.1, 00:00:16, Ethernet0/0
O        10.1.46.0/24 [110/4] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.47.0/24 [110/67] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.58.0/24 [110/6] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.68.0/24 [110/5] via 10.1.12.1, 00:00:06, Ethernet0/0
O        10.1.75.0/24 [110/7] via 10.1.12.1, 00:00:06, Ethernet0/0
```

Segue a representação gráfica do caminho percorrido pelos pacotes até o destino:

![Caminho percorrido pelos pacotes até o destino](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-4.png)

Abaixo podemos observar, através do comando *traceroute*, o caminho que os pacotes percorrem até chegar à rede de destino:

```cisco
RT2#traceroute 7.7.7.7
Type escape sequence to abort.
Tracing the route to 7.7.7.7
VRF info: (vrf in name/id, vrf out name/id)
  1 10.1.12.1 1 msec 1 msec 1 msec
  2 10.1.13.3 1 msec 1 msec 0 msec
  3 10.1.34.4 1 msec 2 msec 1 msec
  4 10.1.46.6 1 msec 2 msec 1 msec
  5 10.1.68.8 2 msec 2 msec 2 msec
  6 10.1.58.5 1 msec 2 msec 1 msec
  7 10.1.75.7 2 msec 2 msec *
RT2#
````

### Descoberta de Vizinhos e Formação de Adjacências

Descobrir um vizinho (fase 1) é só o começo. Mas, atenção: antes de começarem a trocar informações e passarem pela máquina de estados, os roteadores precisam concordar num verdadeiro "acordo de cavalheiros". Através dos pacotes Hello, eles validam um checklist. Se algo estiver fora do lugar, o processo de criar uma vizinhança nem começa!

Para que o processo de formação de adjacência seja iniciado, as seguintes configurações precisam ser exatamente iguais em ambos os lados do link:

- **ID da Área:** Se um roteador diz pertencer à Área 0 e o outro à Área 1 no mesmo link, eles não irão seguir com o processo de adjacência.
- **Sub-rede e Máscara:** Os endereços IP das interfaces conectadas têm de pertencer exatamente  à mesma rede.
- **Hello e Dead Timers:** O ritmo da conversa tem de bater (geralmente 10s para o Hello e 40s para o Dead Timer).
- **Autenticação:** Se houver uma senha configurada, ela precisa ser idêntica.
- **Flags de Área Especial (Stub/NSSA):** Ambos precisam concordar sobre o tipo de área em que estão inseridos.

Além destas, existem duas regras para garantir que o processo não trave mais adiante, durante a troca da base de dados:

- **Router IDs Únicos:** Cada roteador precisa do seu próprio documento de identidade (RID). Se houver duplicidade na rede, a matriz de rotas entra em colapso.
- **MTU (Maximum Transmission Unit) Igual:** O tamanho máximo de pacote suportado nas interfaces precisa ser o mesmo. Se um lado tiver um MTU de 1500 bytes e o outro 1400, eles começam a conversar, mas a adjacência "trava" a meio do caminho. Importante lembrar que isso ocorre porque o OSPF não suporta fragmentação.

> O número do processo OSPF não precisa ser idêntico entre roteadores para formar adjacência.

Para exemplificar melhor, foi criada a topologia abaixo:

![Topologia de adjacência OSPF](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-6.png)

Com tudo isso previamente alinhado, para sincronizar a LSDB, os roteadores OSPF passam por uma máquina de estados de sete passos. A comunicação ocorre diretamente sobre IP (protocol ID 89), com cinco tipos de pacote, **Hello**, **DBD**, **LSR**, **LSU** e **LSAck**, que aparecem conforme o avanço dos estados:

1. **Down (inativo):** estado inicial. O processo OSPF está ativo na interface, mas nenhum *Hello* foi recebido. O roteador envia *Hello* para o multicast **224.0.0.5** (*All OSPF Routers*) e aguarda resposta.
2. **Init (inicialização):** chegou um *Hello* de um vizinho, mas a comunicação ainda é unidirecional. O RID local não aparece no campo de vizinhos ativos do pacote recebido.
3. **2-Way (bidirecional):** o *Hello* lista o próprio *Router ID (RID)*. A comunicação bidirecional está confirmada. O protocolo avalia o **tipo de rede (*Network Type*)** da interface e decide o próximo passo. Neste material, vamos falar somente sobre os dois tipos mais comuns: **Point-to-Point**, conexões entre dois nós. A eleição de DR/BDR é ignorada e o processo segue para os estados seguintes. **Broadcast (multiacesso)**, o padrão em Ethernet. Com vários dispositivos no segmento, o OSPF elege um *Designated Router* (DR) e um *Backup Designated Router* (BDR) para centralizar a troca de LSAs, os demais (*DROTHERs*) mantêm adjacência **2-Way** entre si e só avançam para o estado **Full** com DR e BDR. Vamos detalhar isso na seção de Características mais adiante, mas adianto que isso acontece para evitar problemas de escalabilidade.
4. **ExStart:** a comunicação básica está pronta, falta apenas sincronizar a LSDB. Os pares elegem *Master* e *Slave* (maior RID vence) para definir a sequência inicial dos pacotes **DBD** (*Database Description*).
5. **Exchange:** *Master* e *Slave* trocam DBDs, o "índice" da LSDB, com cabeçalhos de LSA para comparação, ainda sem o conteúdo completo das rotas.
6. **Loading:** após os DBDs, o roteador pede LSAs ausentes ou mais recentes via **LSR** (*Link-State Request*). O vizinho responde com **LSU** (*Link-State Update*) e o solicitante confirma com **LSAck** (*Link-State Acknowledgment*).
7. **Full:** as LSDBs estão idênticas e sincronizadas na área, agora pode tirar o grito ***É TETRA*** da garganta! Finalmente o roteador executa o SPF e instala as melhores rotas na tabela de roteamento (RIB).

> Nota: As fases 2 e 3 ocorrem nos estados **Exchange**, **Loading** e **Full**: a LSDB fica sincronizada e, em seguida, cada roteador roda o SPF de forma independente para popular a RIB.

### Ativando o OSPF no Cisco IOS

Falamos muita teoria já, mas para que o entendimento seja claro, a prática é fundamental. Antes de qualquer pacote *Hello* sair pela rede, precisamos habilitar o OSPF nos roteadores. No Cisco IOS, existem duas formas de fazer isso:

#### 1. Network statement (configuração global no *router ospf*)

É o modelo mais tradicional. Você entra no modo de configuração do processo OSPF e informa quais redes (e, por consequência, quais interfaces) participarão do protocolo, a configuração abaixo foi aplicada no roteador *RT1*:

```cisco
router ospf 1
 network 10.0.0.1 0.0.0.0 area 0
```

Muita gente confunde o network statement com um comando de anúncio de rotas, mas o funcionamento é um pouco diferente, ele age como um seletor de interface. O processo é simples:

- Você informa um endereço e uma máscara wildcard.
- O roteador percorre as interfaces configuradas.
- Se o endereço IP da interface 'casar' com a máscara, o OSPF passa a operar nela.

Simples assim!

No exemplo do endereço IP 10.0.0.1 com a máscara wildcard 0.0.0.0, estamos sendo específicos, apenas a interface com esse IP exato participará do OSPF. Se o endereço não for encontrado, a interface não entra no processo e, consequentemente, não há anúncio de rota. Entender esse conceito é fundamental e pode te poupar tempo ao analisar o comportamento de uma rede.

#### 2. Configuração por interface

A alternativa é anexar a interface diretamente ao processo OSPF, sem a necessidade de aplicar o *network statement*, conforme foi realizado no roteador *RT2*:

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf 2 area 0
```

Ambas as formas ativam o OSPF na interface. A diferença está apenas no **local** você configura: no processo (genérico/centralizado) ou na interface (específico).

> No Cisco IOS, configurações **específicas** prevalecem sobre as **genéricas**, isso não seria diferente para as configurações do OSPF. Se houver conflito, por exemplo, um *network* colocando a interface na Área 0 e um *ip ospf 2 area 1* na mesma porta, o que está na interface vence. Vamos demonstrar isso no artigo sobre troubleshooting que virá posteriormente, ok? Preciso me manter focado para não deixar esse material maior do que já está!

##### Primeiro refinamento na configuração dos equipamentos

Vale lembrar também o comportamento padrão do OSPF baseado no tipo de rede. Na nossa topologia, devemos aplicar o comando *ip ospf network point-to-point* na interface, por se tratar de uma conexão ponto-a-ponto, para sobrescrever o comportamento padrão para redes *broadcast* que o Ethernet possui, com isso, não haverá a eleição desnecessária de DR e BDR. Vamos observar isso na prática logo adiante.

#### Retornando ao assunto principal desse tópico

Agora, vamos ver o processo de formação de vizinhança e adjacência funcionando na prática:

- Ao configurarmos o OSPF, o roteador envia os pacotes Hello, como podemos observar nas capturas de pacotes realizadas com o WireShark conforme abaixo:

![Captura de pacote Hello OSPF](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-8.png)

![Detalhe do pacote Hello OSPF](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-9.png)

![Pacote Hello OSPF na captura](/assets/images/networking/ospfv2-uncomplicated/ospfv2-uncomplicated-pt1-10.png)

Foi aplicado o comando abaixo para avaliar os protocolos de roteamento ativos no roteador *RT1*:

```cisco
RT1#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.0.0.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.1 0.0.0.0 area 0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
```

Após o processo de adjacência ser concluído, o comando *show ip ospf neighbor* deve apresentar a saída abaixo:

```cisco
RT1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2          1   FULL/DR         00:00:38    10.0.0.2        Ethernet0/0
```

Agora observaremos a saída do comando *show ip protocols* para o roteador *RT2*:

```cisco
RT2#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 2"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 10.0.0.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
  Routing on Interfaces Configured Explicitly (Area 0):
    Ethernet0/0
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 110)
```

Agora, podemos observar o *RT2* após a conclusão do processo de adjacência:

```cisco
RT2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.1          1   FULL/BDR        00:00:36    10.0.0.1        Ethernet0/0
```

Podemos observar as diferenças entre as formas que o OSPF foi habilitado nos dois roteadores, se você prestar atenção às saídas do comando *show ip protocols*, podemos descobrir como o OSPF foi habilitado:

- **RT1** via *network statement*

```cisco
Routing for Networks:
  10.0.0.1 0.0.0.0 area 0
```

- **RT2** via *interface*

```cisco
Routing on Interfaces Configured Explicitly (Area 0):
  Ethernet0/0
```

Com o *debug ip ospf adj* habilitado, dá para acompanhar a transição de estados no **RT1**:

```cisco
*Jun  7 01:41:48.306: OSPF-1 ADJ   Et0/0: Interface going Up
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: end of Wait on interface
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect BDR 0.0.0.0
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:42:28.313: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: none
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: 2 Way Communication to 10.0.0.2, state 2WAY
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Prepare dbase exchange
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0x2376 opt 0x52 flag 0x7 len 32
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Neighbor change event
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0xD30 opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: NBR Negotiation Done. We are the SLAVE
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Summary list built, size 1
*Jun  7 01:44:54.415: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0xD30 opt 0x52 flag 0x2 len 52
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0xD31 opt 0x52 flag 0x1 len 32  mtu 1500 state EXCHANGE
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Exchange Done with 10.0.0.2
*Jun  7 01:44:54.416: OSPF-1 ADJ   Et0/0: Synchronized with 10.0.0.2, state FULL
*Jun  7 01:44:54.416: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on Ethernet0/0 from LOADING to FULL, Loading Done
```

E no **RT2**:

```cisco
*Jun  7 01:44:54.413: OSPF-2 ADJ   Et0/0: Interface going Up
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: 2 Way Communication to 10.0.0.1, state 2WAY
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Backup seen event before WAIT timer
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: DR/BDR election
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect BDR 10.0.0.2
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Elect DR 10.0.0.1
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: DR: 10.0.0.1 (Id)   BDR: 10.0.0.2 (Id)
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Prepare dbase exchange
*Jun  7 01:44:54.414: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0xD30 opt 0x52 flag 0x7 len 32
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x2376 opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: First DBD and we are not SLAVE
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xD30 opt 0x52 flag 0x2 len 52  mtu 1500 state EXSTART
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: NBR Negotiation Done. We are the MASTER
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Summary list built, size 0
*Jun  7 01:44:54.415: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0xD31 opt 0x52 flag 0x1 len 32
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xD31 opt 0x52 flag 0x0 len 32  mtu 1500 state EXCHANGE
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Exchange Done with 10.0.0.1
*Jun  7 01:44:54.416: OSPF-2 ADJ   Et0/0: Send LS REQ to 10.0.0.1 length 36 LSA count 1
*Jun  7 01:44:54.417: OSPF-2 ADJ   Et0/0: Rcv LS UPD from 10.0.0.1 length 64 LSA count 1
*Jun  7 01:44:54.417: OSPF-2 ADJ   Et0/0: Synchronized with 10.0.0.1, state FULL
RT2(config-router)#
*Jun  7 01:44:54.417: %OSPF-5-ADJCHG: Process 2, Nbr 10.0.0.1 on Ethernet0/0 from LOADING to FULL, Loading Done
```

Até aqui vimos o comportamento em rede **broadcast** , com eleição de DR/BDR. Para que não ocorra a eleição de DR/BDR sem necessidade, vamos mudar o tipo de rede para *point-to-point* direto na interface e ver como o processo muda:

- **RT1**

```cisco
interface Ethernet0/0
 ip address 10.0.0.1 255.255.255.252
 ip ospf network point-to-point
```

- **RT2**

```cisco
interface Ethernet0/0
 ip address 10.0.0.2 255.255.255.252
 ip ospf network point-to-point
```

Logs do **RT1** após a mudança:

```cisco
*Jun  7 02:35:37.050: OSPF-1 ADJ   Et0/0: Interface going Up
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: 2 Way Communication to 10.0.0.2, state 2WAY
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Prepare dbase exchange
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0xDFC opt 0x52 flag 0x7 len 32
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0x92C opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: NBR Negotiation Done. We are the SLAVE
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Nbr 10.0.0.2: Summary list built, size 1
*Jun  7 02:36:01.279: OSPF-1 ADJ   Et0/0: Send DBD to 10.0.0.2 seq 0x92C opt 0x52 flag 0x2 len 52
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Rcv DBD from 10.0.0.2 seq 0x92D opt 0x52 flag 0x1 len 32  mtu 1500 state EXCHANGE
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Exchange Done with 10.0.0.2
*Jun  7 02:36:01.280: OSPF-1 ADJ   Et0/0: Synchronized with 10.0.0.2, state FULL
*Jun  7 02:36:01.280: %OSPF-5-ADJCHG: Process 1, Nbr 10.0.0.2 on Ethernet0/0 from LOADING to FULL, Loading Done
```

Logs do **RT2**:

```cisco
*Jun  7 02:36:01.278: OSPF-2 ADJ   Et0/0: Interface going Up
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: 2 Way Communication to 10.0.0.1, state 2WAY
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Prepare dbase exchange
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0x92C opt 0x52 flag 0x7 len 32
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0xDFC opt 0x52 flag 0x7 len 32  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: First DBD and we are not SLAVE
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x92C opt 0x52 flag 0x2 len 52  mtu 1500 state EXSTART
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: NBR Negotiation Done. We are the MASTER
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Nbr 10.0.0.1: Summary list built, size 0
*Jun  7 02:36:01.279: OSPF-2 ADJ   Et0/0: Send DBD to 10.0.0.1 seq 0x92D opt 0x52 flag 0x1 len 32
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Rcv DBD from 10.0.0.1 seq 0x92D opt 0x52 flag 0x0 len 32  mtu 1500 state EXCHANGE
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Exchange Done with 10.0.0.1
*Jun  7 02:36:01.280: OSPF-2 ADJ   Et0/0: Send LS REQ to 10.0.0.1 length 36 LSA count 1
*Jun  7 02:36:01.281: OSPF-2 ADJ   Et0/0: Rcv LS UPD from 10.0.0.1 length 64 LSA count 1
*Jun  7 02:36:01.281: OSPF-2 ADJ   Et0/0: Synchronized with 10.0.0.1, state FULL
```

Observe que agora não há mais um DR/BDR:

- **RT1**

```cisco
RT1#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.2          0   FULL/  -        00:00:35    10.0.0.2        Ethernet0/0
```

- **RT2**

```cisco
RT2#show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
10.0.0.1          0   FULL/  -        00:00:38    10.0.0.1        Ethernet0/0
```

## Características e Capacidades

- **Áreas:** a topologia se divide em áreas para isolar falhas e reduzir execuções de SPF. A área 0 é o *backbone*, as demais conectam-se a ela para conseguirem falar entre si (seja fisicamente ou via *virtual link*).
- **Router ID:** identidade única de 32 bits por roteador (possui o mesmo formato de um endereço IPv4, mas não é um endereço utilizável para tráfego de usuário).
- **DR e BDR:** em segmentos broadcast, o OSPF elege um *Designated Router* e um *Backup Designated Router* para evitar adjacências completas entre todos os roteadores. Os *DROTHERs* reportam mudanças ao DR, que replica a informação no segmento.
- **Sumarização:** nos limites de área (ABRs) ou na redistribuição (ASBRs), condensando prefixos para estabilizar a LSDB. Vamos detalhar essas funções no próximo artigo da série.
- **Tipos de área (Stub, Totally Stubby, NSSA):** áreas que não precisam de rotas externas completas, recebendo rota padrão e economizando CPU e memória. Lembra o *stub host* da loopback? Pois é, aqui o conceito é outro, estamos falando de um tipo de área OSPF, não de um host isolado.
- **Autenticação:** *clear-text* ou criptografia (MD5, HMAC-SHA) para impedir roteadores não autorizados no domínio.

### Aplicando a segunda camada de refinamento

Observamos ao longo desse material que, o Neighbor ID é o mesmo endereço da interface que o OSPF foi ativado. A verdade é que para o OSPF, pouco importa se o seu equipamento tem um hostname criativo ou não. O protocolo não entende nomes, ele identifica cada dispositivo através de um Router ID (RID), que é um identificador de 32 bits (com o mesmo formato de um endereço IP) e que tem de ser absolutamente único no domínio OSPF.
Para assumir este identificador, o processo OSPF segue uma hierarquia de eleição bastante específica:

1. **Configuração Manual**: Se o administrador for à CLI e configurar o router-id, o protocolo obedece de imediato e ignora o resto.
2. **Maior endereço IP de uma Interface Loopback**: Na falta da configuração manual, o OSPF verifica todas as Loopbacks ativas e escolhe a que tiver o maior endereço IP.
3. **Maior endereço IP de uma Interface Física**: Se falharem as opções anteriores, ele procura a interface física ativa com o maior IP. Atenção: isto não é recomendado! Se essa interface cair, o processo torna-se instável.

Configurar manualmente o RID previne dores de cabeça futuras e garante a estabilidade. A seguir seguem os comandos aplicados nos roteadores RT1 e RT2 para aplicar o Router-ID manualmente:

```cisco
RT1(config)# router ospf 1
RT1(config-router)# router-id 1.1.1.1
```

```cisco
RT2(config)# router ospf 2
RT2(config-router)# router-id 2.2.2.2
```

## Conclusão: O Fim do Início

Finalmente chegamos ao fim desta primeira etapa. Como pudemos ver, o OSPF não é apenas um protocolo que sai anunciando rotas sem critério algum. Ele é metódico, exige um "match" (quase um *Tinder* dos roteadores) com os pacotes Hello e constrói um mapa completo de toda a rede antes de tomar qualquer decisão, usando o algoritmo de Dijkstra.

É meio chato ser repetitivo e falar o que todo mundo fala, mas entender esses fundamentos teóricos, desde as regras para formar vizinhança, passando pela máquina de estados, até chegar ao cálculo do custo, é exatamente o que separa um mero "digitador de comandos" de um verdadeiro Engenheiro de Redes. Posso te afirmar, com base na minha experiência: quando algo der errado, é o conhecimento de base que vai te salvar!

Nesta Parte 1 já colocamos a mão na massa com labs, capturas, *traceroute*, configs e aqueles logs deliciosos pipocando na tela até chegar em FULL. Mas convenhamos... Ainda dá para ir muito mais longe. Teoria e prática precisam andar juntas, como Buchecha e Claudinho, Piu-piu e Frajola...

Na Parte 2 desta série, vamos subir o nível: pacotes Hello, tipos de LSA, topologias mais complexas para exemplificar os conceitos, áreas stub/NSSA, sumarização, troubleshooting de verdade e cenários que se aproximam mais do dia a dia. No material de troubleshooting, também vamos demonstrar na prática o ajuste do *auto-cost reference-bandwidth* para corrigir o problema do custo igual entre FastEthernet e GigabitEthernet, e o comportamento de precedência quando há conflito entre o *network statement* e o *ip ospf* configurado diretamente na interface. Espero que a base que construímos aqui, adjacência, custo, SPF, te ajude em um futuro próximo.

Então, pode escolher o seu emulador favorito (Packet Tracer, GNS3, EVE-NG, PNETLab) e até o próximo artigo da série. Só para relembrar, eu utilizo o PNETLab, vale lembrar que o arquivo é totalmente compatível com o EVE-NG e que já vai estar com as interfaces pré-configuradas, hein!

Vamos dominar juntos o OSPF e, como diriam por aí: VAAAAAAAAI!
