# Visão Geral da Camada de Enlace de Dados: Teoria e Prática (Parte 2)

## Introdução

Fala rapaziada, firmeza total? Seguinte, agora que vimos o básico da teoria, vamos começar a praticar e ver a camada de enlace funcionando, a princípio a ideia é bem simples, será um único conteúdo abordando os principais assuntos relacionados à Ethernet, LAN e Spanning-Tree, fique à vontade para ler e praticar com o lab disponibilizado. Por uma preferência pessoal, utilizei o PNETLAB, mas o lab é compatível com o EVE-NG.

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/00-topology.png">

## Antes de começarmos, precisamos falar sobre o Switch

O switch, ou comutador (em tradução livre), na teoria é um dispositivo que opera na camada 2, conforme você vai ver em muitas literaturas, mas eu não levo isso como uma verdade absoluta, eu prefiro seguir a linha de que o switch é um dispositivo que possui diversas capacidades e desempenha diversas funções, uma delas é fazer o encaminhamento de quadros, portanto essa função está localizada na camada 2.

Esse dispositivo, quando executando as funções previstas na camada 2, encaminha os quadros baseado no endereço MAC de destino e somente isso, portanto, o switch não faz o encaminhamento baseado em endereçamento IP.

> Uma coisa que passou batida e eu quase ia me esquecendo, é que a camada 2 (camada de enlace) se refere a um enlace, por mais óbvio que seja, acho importante falar que o encaminhamento de quadros tem escopo local, dentro da mesma LAN, rede ou VLAN, nesse momento não vamos falar dos tipos de virtualização para a camada 2, mas saiba que existem algumas formas de "enganar" os equipamentos, mas esse é um assunto para o Vinícius do futuro tratar.

Prosseguindo, ao preparar um quadro para realizar o envio, o dispositivo observa a sua tabela MAC para verificar se conhece o endereço MAC de destino para determinado endereço IP, se ele não conhecer, vai precisar utilizar o protocolo ARP para descobrir o endereço MAC de destino, o ARP é um protocolo auxiliar do protocolo IP.

### Protocolo ARP

A intenção é ser breve, para mais detalhes, consulte a [RFC 826](https://datatracker.ietf.org/doc/html/rfc826), falando de forma bem resumida, esse protocolo auxilia no funcionamento do protocolo IP. Para encaminharmos um quadro em uma rede local, os dispositivos devem conhecer o endereço físico dos equipamentos de destino, mas nem sempre temos essa informação, geralmente possuímos o endereço IP e é aqui que o *ARP* (Address Resolution Protocol) entra em ação.

Antes que eu me esqueça, é importante dar contexto e falarmos sobre as formas de envio dos quadros:

- Unicast: O quadro é enviado a um único destino, quando o dispositivo conhece o endereço do destino
- Multicast: O quadro é enviado a um grupo, vamos ver o seu funcionamento ao falar de alguns protocolos de roteamento em um futuro próximo
- Broadcast: O quadro é enviado para todas as interfaces, exceto a que o quadro foi recebido
  - *Unknown Unicast*: Na prática, o funcionamento é idêntico ao broadcast, mas é o nome técnico do quadro que o ARP envia para descobrir o endereço físico, através do seu endereço IP

Para entendermos melhor o funcionamento do ARP, vamos imaginar que estamos no primeiro dia de aula e que ninguém se conhece, o professor possui uma lista com os nomes dos alunos, mas ainda não os conhece e nem sabe em que lugar estão sentados.

Nesse exemplo, a sala de aula é a LAN, o professor e os alunos são dispositivos.

Partindo do ponto de vista do professor, vamos explicar o funcionamento do protocolo ARP:

1. O professor ao realizar a chamada, diz o nome do aluno, Pedro, para toda a sala de aula (ARP Request - Unknown Unicast)
2. A mensagem chega a todos os alunos presentes na sala de aula, correto? Então, os alunos ignoram a mensagem, exceto um... (Os quadros são descartados pelos dispositivos que não possuem o endereço IP contigo no ARP Request)
3. Pedro responde, com isso o professor sabe quem é e onde ele está (ARP Reply - Unicast)

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/01-arp-protocol.png">

#### ARP Gratuito

Não poderia deixar de falar do ARP gratuito, é um mecanismo fundamental para evitar a duplicação de endereços IP, ao conectar um dispositivo na rede, ele envia automaticamente um ARP Request para saber se o seu endereço IP está ocupado.

## Finalmente chegou a parte prática

Vamos relembrar a topologia apresentada no início, digamos que o PC1 precisa se comunicar com o PC2.

Para o nosso entendimento de como os quadros são encaminhados, vamos realizar um teste simples de ping, perceba que o protocolo ARP vai entrar em ação para descobrir o endereço físico do PC2, o quadro chega na porta e1/3 do SWT-ACC1 com a seguinte estrutura.

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/02-pc1-wants-to-send-a-msg-to-pc2.png">

Depois do ARP Request passar pela LAN e encontrar o destino, ele vai retornar com o endereço físico para o dispositivo que originou a solicitação, simples assim, poderia ser, mas não era para ser, abaixo vamos entender melhor, firmeza?

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/03-arp-request.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/04-arp-reply.png">

Vamos entender como que o ARP trafega na LAN, primeiro de tudo, eu sei que escolhi uma topologia complexa demais para explicar o que vai acontecer, peço que me deixe saber se eu deveria ter simplificado mais para explicar os conceitos. Quando você envia o pacote pela rede, ao ser encapsulado, o dispositivo de origem vai inserir o endereço MAC (FF:FF:FF:FF:FF:FF), ou seja, como ele não sabe o MAC, ele usa esse endereço especial para descobrir o endereço MAC do dispositivo (ARP Request).
Observe que o quadro contendo o ARP Request entra na interface e1/3 do SWT-ACC1, certo?
O SWT-ACC1 vai fazer a sua parte, lembra? Vai encaminhar o quadro para todas as portas, exceto a que recebeu.

> Os outros equipamentos vão repetir exatamente o mesmo comportamento.

Bom, falamos sobre os frames de broadcast e unknown unicast, mas você já parou para pensar que na nossa topologia eles podem vagar eternamente? Temos inúmeros caminhos redundantes, cada switch vai receber uma cópia e reencaminhar o frame novamente por todas as outras portas infinitamente... Amigos, esse negócio se chama loop de camada 2! Nesse ponto, não há nada a ser feito, os quadros vão ficar vagando eternamente pela rede local, causando instabilidade na tabela MAC e aumentando consumo de recursos até que algum switch não consiga encaminhar mais nenhum quadro e trave completamente!

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/05-arp-stp-need.gif">

Mas afinal, como resolver esse problema?
Bom, para isso foi desenvolvido um protocolo de rede chamado Spanning-Tree Protocol (STP) e o que ele faz? Indo direto ao ponto:

> O STP bloqueia um ou mais caminhos redundantes, criando assim, um caminho sem loops.

O mais legal de tudo é que o cenário que eu mencionei é totalmente hipotético, já que o spanning-tree vem habilitado por padrão em switches! Espera um pouco, mas como que ficou a LAN depois do STP eliminar os loops?

Vamos aplicar o comando abaixo em todos os switches, analisar as saídas do comando e redesenhar a topologia sem loops:

```cisco
show spanning-tree
```

### Vamos analisar a topologia por partes

Primeiro de tudo, vamos analisar as portas do equipamento SWT-CORE1:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-1.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-1.png">

Agora, vamos analisar o switch SWT-CORE2:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-2.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-2.png">

SWT-DIST1:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-3.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-3.png">

SWT-DIST2:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-4.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-4.png">

SWT-ACC1:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-5.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-5.png">

Finalmente, vamos analisar o último equipamento, o SWT-ACC2:

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-show-stp-6.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-show-stp-6.png">

Estado da rede após o STP estabilizar:
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/08-loop-free-topology-1.png">

Vamos fazer uma pequena reflexão agora, geralmente os switches utilizados na camada CORE são os que possuem maior capacidade, certo? Porém na nossa topologia, o switch eleito como *root*, foi o SWT-DIST1. Mas o que é o switch root!?

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/09-stp-tunning-1.png">

SWT-CORE1:

```cisco
enable
configure terminal
spanning-tree vlan 1 priority 0
end
write
```

SWT-CORE2:

```cisco
enable
configure terminal
spanning-tree vlan 1 priority 4096
end
write
```

```cisco
show spanning-tree
```

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/09-stp-tunning-2.png">

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/10-loop-free-topology-2.png">



### Referências

- Gustavo Kalau

- https://community.cisco.com/t5/artigos-routing-switching/spanning-tree-protocolo-stp/ta-p/5209086

- https://www.howtonetwork.com.br/forum/switching/etherchannel-e-os-seus-metodos-de-load-balancing