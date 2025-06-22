## Introdução

A **camada de enlace de dados** é fundamental no modelo OSI e TCP/IP, sendo a segunda camada no processo de comunicação entre dispositivos em rede. Ela é responsável por garantir que as unidades de informação, chamadas **quadros**, sejam transferidas de maneira confiável entre dispositivos fisicamente conectados por meio de um canal de comunicação, como cabos, linhas telefônicas ou canais sem fio.

Para profissionais de redes de computadores, para termos uma base sólida é fundamental conhecer a Camada de Enlace, pois ela é essencial para entender como switches, endereços MAC e VLANs funcionam na prática. Neste artigo, exploraremos os conceitos teóricos e depois aplicaremos esse conhecimento em cenários reais de configuração.
### Funções Básicas da Camada de Enlace de Dados

A camada de enlace de dados utiliza os serviços da camada física para a transmissão de bits, garantindo que os dados cheguem à máquina de destino. Entre as funções principais desta camada estão:

1. **Interface de Serviço**: Proporcionar uma interface definida para a camada de rede, facilitando a comunicação entre camadas superiores e inferiores.
2. **Enquadramento**: Organizar bytes em quadros para transmissão eficiente e integrada.
3. **Controle de Erros**: Detectar e corrigir possíveis erros durante a transmissão de dados.
4. **Controle de Fluxo**: Regular a taxa de transmissão de dados para evitar sobrecarregar receptores mais lentos.

Além das funções citadas, a camada de enlace também é responsável pelo **endereçamento físico**, utilizando endereços MAC (Media Access Control) para identificar dispositivos em uma rede local (LAN), esse conceito é fundamental para entender como switches encaminham quadros.

O
#### Serviços

Os serviços da camada de enlace de dados variam de acordo com o protocolo, mas podemos categorizá-los em três tipos principais:

- **Serviço não orientado a conexões sem confirmação**: Os quadros são enviados sem qualquer confirmação de recebimento. A Ethernet é um exemplo clássico desse serviço, sendo usado em ambientes onde a taxa de erro é baixa e a recuperação de dados é feita por camadas superiores.
- **Serviço não orientado a conexões com confirmação**: Cada quadro enviado é confirmado individualmente, o que permite o retransmissão de quadros perdidos. O padrão **802.11 (WiFi)** adota essa abordagem para garantir confiabilidade em redes sem fio.
- **Serviço orientado a conexões com confirmação**: Neste serviço, uma conexão lógica é estabelecida entre as máquinas antes do envio dos dados. Cada quadro é numerado e a confirmação garante a entrega.

Nas certificações de fabricantes de equipamentos, o foco geralmente recai sobre o **Ethernet (IEEE 802.3)**, que utiliza um serviço não orientado a conexão sem confirmação. Já o **WiFi (IEEE 802.11)** usa confirmações devido à natureza propensa a erros das redes sem fio.
#### Enquadramento

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

![[Pasted image 20250420134628.png]]
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
#### Controle de Erros e de Fluxo

Os protocolos da camada de enlace de dados empregam diferentes mecanismos para controlar erros e fluxo, garantindo a integridade e eficiência da transmissão de dados. Entre os principais mecanismos estão:

- **Detecção de Erros**: Métodos como o checksum e CRC (Cyclic Redundancy Check) são amplamente utilizados para identificar falhas na transmissão de quadros.
- **Correção de Erros**: Em alguns casos, a camada de enlace pode corrigir automaticamente pequenos erros ou solicitar a retransmissão do quadro defeituoso.
- **Controle de Fluxo**: Protocolos como o **Windowing** (controle de janela) e **ACK/NACK** (acknowledgement/negative acknowledgement) regulam o fluxo de dados entre dispositivos, evitando que um transmissor rápido sobrecarregue um receptor mais lento.

#### Métodos de Encaminhamento de Quadros

* Store-and-Forward: o switch recebe o quadro inteiro antes de encaminhá-lo, fornecendo maior confiabilidade e, consequentemente, maior latência.
* Cut-Through: o switch começa a encaminhar o quadro assim que detecta o endereço MAC de destino, 6 bytes logo após o campo SFD, e não faz a verificação de erros, a latência tende a ser muito baixa, porém a chance de encaminhar quadros corrompidos é alta, sendo ideal para redes com baixa taxa de erros.
	* Fast-Forward: é uma variação do Cut-Through, onde o switch introduz um pequeno atraso (delay) antes de encaminhar, reduzindo colisões. O switch aguarda os primeiros 64 bytes, tamanho mínimo de um quadro ethernet, antes de encaminhar, se o quadro for menor, é descartado (runt frame). É um método menos comum, é mais observado em switches mais antigos.
	* Fragment-Free: é um híbrido entre o Cut-Through e o Store-and-Forward. O switch verifica os primeiros 64 bytes, que é onde ocorrem a maioria dos erros de transmissão, antes de encaminhar, se não houver erro, ele encaminha o restante do quadro sem fazer verificação de erros. Faz o balanceamento entre velocidade e confiabilidade mínima.

Abaixo segue uma tabela que faz uma comparação entre os modos de encaminhamento de quadros:

| Modo              | Latência | Verificação de Erros | Uso típico                      |
| ----------------- | -------- | -------------------- | ------------------------------- |
| Store-and-Forward | Alta     | Completa (CRC/FCS)   | Redes modernas                  |
| Cut-Through       | Mínima   | Nenhuma              | Data Centers / Low-Latency      |
| Fragment-Free     | Moderada | Primeiros 64 bytes   | Redes com histórico de colisões |

Ao estudar para o CCNA, o padrão para redes locais (LANs) é o Ethernet, que nada mais é que um conjunto de padrões e regras a serem seguidas para que os dispositivos possam se comunicar.
Comecei a escrever esse texto com a intenção de criar um guia para revisar rapidamente o conteúdo que já estudei.