## Vamos para a parte prática?

Agora que vimos o básico da teoria, vamos praticar e ver a camada de enlace funcionando, a princípio a ideia é bem simples, será um único conteúdo abordando os principais assuntos relacionados à Ethernet, LAN e Spanning-Tree, posso te adiantar que a tendência é que o texto fique um pouco cansativo, fique à vontade para ler aos poucos e praticar com o lab disponibilizado. Por uma preferência pessoal, utilizei o PNETLAB, mas o lab é compatível com o EVE-NG.

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/00-topology.png">

### Antes de começarmos, precisamos falar sobre o switch

O switch, ou comutador, em tese é um dispositivo que opera na camada 2, conforme você vai ver em muitas literaturas, mas eu não levo isso como uma verdade absoluta e escrita em pedra, eu prefiro seguir a linha de que o switch é um dispositivo que desempenha diversas funções, uma delas é fazer o encaminhamento de quadros, portanto essa função está localizada na camada 2.

Esse dispositivo, quando executando as funções previstas na camada 2, encaminha os quadros baseado no endereço MAC de destino e somente isso, ele não faz o encaminhamento baseado em endereçamento IP.

> Uma coisa que passou batida e eu quase ia me esquecendo, é que a camada 2 (camada de enlace) se refere a um enlace, por mais óbvio que seja, acho importante falar que o encaminhamento de quadros tem escopo local, dentro da mesma LAN, rede ou VLAN, nesse momento não vamos falar dos tipos de virtualização para a camada 2, mas saiba que existem formas de enganar os equipamentos.

Prosseguindo, ao receber um quadro, o switch observa a sua tabela MAC e verifica se conhece o endereço de destino, se ele não conhece, vai fazer a utilização protocolo ARP, que é um protocolo auxiliar do protocolo IP.

### O início

Bom, eu fiz uma pequena alteração na topologia, igual pode ser observado na imagem abaixo, e liguei todos os dispositivos do lab e para se ter uma noção de como o encaminhamento de quadros funciona:

- <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/01-topology-with-pcs.png">

Podemos observar que o PC1 não sabe o endereço físico do PC2, e vice-versa, mas sabemos o endereço IP do PC1 (10.0.0.1/30) e do PC2 (10.0.0.2/30), porque é o endereço que foi configurado:

- <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/02-pc1-show-arp-inicial.png">
- <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/03-pc2-show-arp-inicial.png">

Para observarmos o comportamento do protocolo *ARP*, foi aplicado o comando de ping no PC1 em direção ao endereço IP do PC2, para isso, abri o wireshark em todas as interfaces do SWT-ACC1:

- Interface 1/3:
  - Apesar de estar escrito "Broadcast", o comportamento é conhecido como "Unknown Unicast":
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/04-unknown-unicast-01.png">
  - Aqui podemos ver no detalhe do quadro de unknown unicast:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/05-unknown-unicast-02.png">
  - Aqui podemos ver a resposta para o endereço do PC1 retornando:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/06-unknown-unicast-03.png">

- Interface 0/0:
  - Olha o unknown unicast aparecendo mais uma vez:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/07-unknown-unicast-04.png">
  - Novamente, o detalhe do unknown unicast:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/08-unknown-unicast-05.png">
  - Essa interface retornou o quadro para o SWT-ACC1 encaminhar para o PC1:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/09-unknown-unicast-06.png">

- Interface 0/1:
  - Essa interface enviou o unknown unicast, porém não recebeu a resposta do ARP, vamos falar sobre isso mais adiante, guarde essa informação:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/10-unknown-unicast-07.png">
  - Só para manter o padrão:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/11-unknown-unicast-08.png">

Vamos voltar as nossas atenções ao PC1 e ao PC2...

- PC1:
  - Após recebermos a resposta do comando *ping*, se liga, a tabela ARP está com o endereço MAC do PC2:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/12-ping-pc1-pc2.png">
  - Antes de seguirmos em frente, rapidinho, tem uma coisa muito importante acontecendo, essa informação do endereço MAC do PC2 vai expirar e com isso, quando o PC1 tiver que enviar qualquer tipo de comunicação ao PC2, ele fará uso do ARP novamente.

- PC2:
  - Após enviar a resposta do *ping* para o PC1, o PC2 instalou o endereço MAC do PC1 na sua tabela ARP:
    - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/13-pc2-show-arp-depois.png">
  - Sim, você não perguntou absolutamente nada, mas o mesmo vale para o PC2, o endereço MAC vai expirar também.

### Vamos voltar nossa atenção para a teoria novamente

Gostaria de dizer que talvez fosse mais interessante falar toda a teoria primeiro e depois praticar tudo de uma vez, mas eu preciso genuinamente da sua ajuda, se gostar desse formato de falar da teoria e da prática de forma cadenciada, me deixe saber e se não gostar, peço que não minta, que eu corrijo para os próximos artigos, ok?

Vimos algumas palavras que não foram mencionadas e agora precisamos esclarecer para fazer sentido o que foi observado no exemplo acima, vou tentar ser o mais objetivo possível para não deixar esse texto maior do que deveria:

1. Protocolo ARP
2. Broadcast
3. Unknown Unicast

### Protocolo ARP

A intenção é ser breve, para mais detalhes, consulte a [RFC 826](https://datatracker.ietf.org/doc/html/rfc826), falando de forma bem resumida, esse protocolo auxilia no funcionamento do protocolo IP, é sabido que para encaminharmos um quadro em uma rede local, os dispositivos devem conhecer o endereço físico dos equipamentos de destino, mas nem sempre temos essa informação, geralmente possuímos o endereço IP e é aqui que o *ARP* (Address Resolution Protocol) entra em ação...

Primeiro de tudo e antes de mais nada, é importante dar contexto e falarmos sobre as formas de envio dos quadros:

- Unicast: O quadro é enviado a um único destino, quando o dispositivo conhece o endereço do destino
- Multicast: O quadro é enviado a um grupo, vamos ver o seu funcionamento ao falar de alguns protocolos de roteamento em um futuro próximo, assim eu espero
- Broadcast: O quadro é enviado para todas as interfaces, exceto a que o quadro foi recebido

Mas você deve estar se perguntando, o que raios é *Unknown Unicast*? Na prática, se comporta como o broadcast, mas é o nome técnico do quadro que o ARP envia para descobrir o endereço físico, através do seu endereço IP.

Vamos lá, imagine que você esteja no primeiro dia de aula, o professor possui uma lista com os nomes dos alunos, mas não os conhece e nem sabe em que lugar estão sentados...

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/14-arp-protocol.png">

Simplificando:

1. O professor pergunta o nome do aluno (ARP Request)
2. O aluno responde, com isso o professor sabe quem é e onde ele está (ARP Reply)

Agora traduzindo:

- Você deseja encaminhar um pacote a um dispositivo que você conhece o endereço IP, mas não conhece o endereço físico (MAC)

  - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/15-arp-request.png">

- Quando você envia o pacote pela rede, ao ser encapsulado, o dispositivo de origem vai inserir o endereço MAC (FF:FF:FF:FF:FF:FF), ou seja, como ele não sabe o MAC, ele usa esse endereço especial para descobrir o endereço MAC do dispositivo (ARP Request)
  - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/16-arp-request-flooding.png">
  - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/17-arp-request-flooding 1.png">
- O quadro viaja pela rede e ao chegar no destino incorreto, é descartado, vamos fingir que o endereço IP do PC2 não é 10.0.0.2
  - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/18-arp-request-imagine.png">
- O frame vai trafegar pela rede até que ao chegar ao destino correto, é reconhecido e então envia uma resposta diretamente a quem fez a solicitação (ARP Reply)
  - <img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/19-arp-request-reply.png">
- Ao receber o ARP Reply, o dispositivo de origem ao enviar novos pacotes ao dispositivo de destino não precisará mais fazer o uso do ARP, pelo menos enquanto o temporizador não acabar.

### Surge a necessidade de evitar que "quadros fantasmas" fiquem vagando pela rede

Bom, falamos sobre os frames de broadcast e unknown unicast, mas você já parou para pensar que na nossa topologia eles podem vagar eternamente como fantasmas?

Temos inúmeros caminhos redundantes, cada switch vai receber uma cópia e reencaminhar o frame novamente por todas as outras portas, vou usar uma topologia mais simples para exemplificar...
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/20-stp-01.png">

Ao enviar um quadro unknown unicast, o switch faz uma cópia e envia para todas as portas, exceto pela que recebeu...
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/21-stp-02.png">

O próximo switch vai fazer a mesma coisa...
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/22-stp-03.png">

O unknown unicast vai chegar ao seu destino, mas o comportamento dos outros switches vai permanecer o mesmo...
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/23-stp-04.png">

Ao receber o frame, vai encaminhar uma cópia para todas as portas...
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/24-stp-05.png">

Nesse ponto, não há nada a ser feito, os quadros vão ficar vagando eternamente pela rede local igual uma alma penada, causando instabilidade na tabela MAC e consumo de CPU até que algum switch não consiga encaminhar mais nenhum quadro e trave completamente!
<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/25-stp-06.png">

Mas afinal, como resolver esse problema?
Bom, para isso foi desenvolvido um protocolo de rede chamado Spanning-Tree Protocol (STP) e o que ele faz? Indo direto ao ponto:
> O STP bloqueia um dos caminhos redundantes, evitando que ocorra o loop.

### Referências

- Gustavo Kalau

- https://community.cisco.com/t5/artigos-routing-switching/spanning-tree-protocolo-stp/ta-p/5209086

- https://www.howtonetwork.com.br/forum/switching/etherchannel-e-os-seus-metodos-de-load-balancing