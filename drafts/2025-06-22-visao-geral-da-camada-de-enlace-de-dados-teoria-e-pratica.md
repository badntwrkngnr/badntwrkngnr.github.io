# Introdução e Base Teórica

A **camada de enlace de dados** é fundamental no modelo OSI e TCP/IP, sendo a segunda camada no processo de comunicação entre dispositivos em rede. Ela é responsável por garantir que as unidades de informação, chamadas **quadros**, sejam transferidas de maneira confiável entre dispositivos fisicamente conectados por meio de um canal de comunicação, sejam eles guiados, como cabos de rede e fibras ópticas, ou não-guiados, como as redes sem fio.

Em todo canto, você vai ver que para profissionais de redes de computadores é necessário ter uma base sólida, por isso, uma das coisas fundamentais é conhecer a camada de enlace, pois ela é essencial para entender como switches, endereços MAC e VLANs funcionam na prática. Neste artigo, exploraremos alguns dos conceitos teóricos e depois aplicaremos esse conhecimento em cenários reais de configuração.

## Funções Básicas da Camada de Enlace de Dados

A camada de enlace de dados utiliza os serviços da camada física para a transmissão de bits, garantindo que os dados cheguem à máquina de destino. Entre as funções principais desta camada estão:

1. **Interface de Serviço**: Proporcionar uma interface definida para a camada de rede, facilitando a comunicação entre camadas superiores e inferiores.
2. **Enquadramento**: Organizar bytes em quadros para transmissão eficiente e integrada.
3. **Controle de Erros**: Detectar e corrigir possíveis erros durante a transmissão de dados.
4. **Controle de Fluxo**: Regular a taxa de transmissão de dados para evitar sobrecarregar receptores mais lentos.

Além das funções citadas, a camada de enlace também é responsável pelo **endereçamento físico**, utilizando endereços MAC (Media Access Control) para identificar dispositivos em uma rede local (LAN), esse conceito é fundamental para entender como switches encaminham quadros.

### Serviços

Os serviços da camada de enlace de dados variam de acordo com o protocolo, mas podemos categorizá-los em três tipos principais:

- **Serviço não orientado a conexões sem confirmação**: Os quadros são enviados sem qualquer confirmação de recebimento. A Ethernet é um exemplo clássico desse serviço, sendo usado em ambientes onde a taxa de erro é baixa e a recuperação de dados é feita por camadas superiores.
- **Serviço não orientado a conexões com confirmação**: Cada quadro enviado é confirmado individualmente, o que permite o retransmissão de quadros perdidos. O padrão **802.11 (WiFi)** adota essa abordagem para garantir confiabilidade em redes sem fio.
- **Serviço orientado a conexões com confirmação**: Neste serviço, uma conexão lógica é estabelecida entre as máquinas antes do envio dos dados. Cada quadro é numerado e a confirmação garante a entrega.

Nas certificações de fabricantes de equipamentos, sobretudo o CCNA 200-301, o foco geralmente recai sobre o **Ethernet (IEEE 802.3)**, que utiliza um serviço não orientado a conexão sem confirmação. A título de curiosidade, podemos falar que o **WiFi (IEEE 802.11)** usa confirmações devido à natureza propensa a erros das redes sem fio, esse padrão também é cobrado no blueprint do CCNA 220-301, mas não será abordado nesse artigo.

### Enquadramento

Para garantir que os quadros sejam transmitidos de forma correta, a camada de enlace de dados deve organizar o fluxo de bits brutos provenientes da camada física em quadros. Esse processo é chamado de **enquadramento** e envolve:

- Dividir o fluxo contínuo de bits em quadros.
- Adicionar um **checksum** (soma de verificação) a cada quadro para detectar erros.
- Recalcular o checksum no destino e verificar se o valor corresponde ao valor transmitido.

Existem várias estratégias de enquadramento, como:

1. **Contagem de caracteres**: Um campo de contagem define o tamanho do quadro.
2. **Bytes de flag com inserção de bytes**: Utiliza flags especiais para marcar o início e o fim dos quadros, inserindo bytes adicionais quando necessário.
3. **Bits de flag com inserção de bits**: Semelhante ao método de bytes, mas opera a nível de bits.
4. **Violações de codificação da camada física**: Utiliza violações propositalmente criadas nas regras de codificação da camada física para indicar o início e o fim dos quadros.

A Ethernet, por exemplo, utiliza um preâmbulo (sequência de bits de sincronização) seguido de um campo de comprimento para marcar o início e o final dos quadros.

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/ETH-HEADER-TRAILER.png">

Campos do Cabeçalho Ethernet e Trailer (IEEE 802.3)

| Campo                       | Bytes   | Descrição                                                                                                                                                                                              |
| --------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Preâmbulo                   | 7       | Sincronização                                                                                                                                                                                          |
| SFD (Start Frame Delimiter) | 1       | Sinaliza que o próximo byte inicia o campo de MAC Address de destino                                                                                                                                   |
| Destino                     | 6       | Identifica o MAC Address de destino da mensagem                                                                                                                                                        |
| Origem                      | 6       | Identifica o MAC Address de origem da mensagem                                                                                                                                                         |
| Tipo                        | 2       | Identifica o tipo de protocolo que está dentro do quadro, os mais comuns são os protocolos IPv4 e IPv6                                                                                                 |
| Dados e Preenchimento       | 46~1500 | Nesse campo estão os dados de camadas superiores, o L3PDU ou pacote. Caso os dados não atendam aos requerimentos mínimos de comprimento dos dados (46 bytes), a origem adiciona dados de preenchimento |
| FCS (Frame Check Sequence)  | 4       | A NIC (Network Interface Card) de destino utiliza esse campo para saber se foram experimentados erros na transmissão de dados                                                                          |

### Controle de Erros e de Fluxo

Os protocolos da camada de enlace de dados empregam diferentes mecanismos para controlar erros e fluxo, garantindo a integridade e eficiência da transmissão de dados. Entre os principais mecanismos estão:

- **Detecção de Erros**: Métodos como o checksum e CRC (Cyclic Redundancy Check) são amplamente utilizados para identificar falhas na transmissão de quadros.
- **Correção de Erros**: Em alguns casos, a camada de enlace pode corrigir automaticamente pequenos erros ou solicitar a retransmissão do quadro defeituoso.
- **Controle de Fluxo**: Protocolos como o **Windowing** (controle de janela) e **ACK/NACK** (acknowledgement/negative acknowledgement) regulam o fluxo de dados entre dispositivos, evitando que um transmissor rápido sobrecarregue um receptor mais lento.

### Métodos de Encaminhamento de Quadros

- Store-and-Forward: o switch recebe o quadro inteiro antes de encaminhá-lo, fornecendo maior confiabilidade e, consequentemente, maior latência.
- Cut-Through: o switch começa a encaminhar o quadro assim que detecta o endereço MAC de destino, 6 bytes logo após o campo SFD, e não faz a verificação de erros, a latência tende a ser muito baixa, porém a chance de encaminhar quadros corrompidos é alta, sendo ideal para redes com baixa taxa de erros.
  - Fast-Forward: é uma variação do Cut-Through, onde o switch introduz um pequeno atraso (delay) antes de encaminhar, reduzindo colisões. O switch aguarda os primeiros 64 bytes, tamanho mínimo de um quadro ethernet, antes de encaminhar, se o quadro for menor, é descartado (runt frame). É um método menos comum, é mais observado em switches mais antigos.
  - Fragment-Free: é um híbrido entre o Cut-Through e o Store-and-Forward. O switch verifica os primeiros 64 bytes, que é onde ocorrem a maioria dos erros de transmissão, antes de encaminhar, se não houver erro, ele encaminha o restante do quadro sem fazer verificação de erros. Faz o balanceamento entre velocidade e confiabilidade mínima.

Abaixo segue uma tabela que faz uma comparação entre os modos de encaminhamento de quadros:

| Modo              | Latência | Verificação de Erros | Uso típico                      |
| ----------------- | -------- | -------------------- | ------------------------------- |
| Store-and-Forward | Alta     | Completa (CRC/FCS)   | Redes modernas                  |
| Cut-Through       | Mínima   | Nenhuma              | Data Centers / Low-Latency      |
| Fragment-Free     | Moderada | Primeiros 64 bytes   | Redes com histórico de colisões |

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

1. Você deseja encaminhar um pacote a um dispositivo que você conhece o endereço IP, mas não conhece o endereço físico (MAC);
2. Quando você envia o pacote pela rede, ao ser encapsulado, o dispositivo de origem vai inserir o endereço MAC (FF:FF:FF:FF:FF:FF), ou seja, como ele não sabe o MAC, ele usa esse endereço especial para descobrir o endereço MAC do dispositivo (ARP Request);
3. O quadro viaja pela rede e ao chegar no destino incorreto, é descartado, até que ao chegar ao destino correto, é reconhecido e então envia uma resposta diretamente a quem fez a solicitação (ARP Reply);
4. Ao receber o ARP Reply, o dispositivo de origem ao enviar novos pacotes ao dispositivo de destino não precisará mais fazer o uso do ARP, pelo menos enquanto o temporizador não acabar.

### Referências

- Gustavo Kalau

- https://community.cisco.com/t5/artigos-routing-switching/spanning-tree-protocolo-stp/ta-p/5209086

- https://www.howtonetwork.com.br/forum/switching/etherchannel-e-os-seus-metodos-de-load-balancing