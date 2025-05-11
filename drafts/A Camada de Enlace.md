#### Introdução

A **camada de enlace de dados** é fundamental no modelo OSI e TCP/IP, sendo a segunda camada no processo de comunicação entre dispositivos em rede. Ela é responsável por garantir que as unidades de informação, chamadas **quadros**, sejam transferidas de maneira confiável entre dispositivos fisicamente conectados por meio de um canal de comunicação, como cabos, linhas telefônicas ou canais sem fio.

Neste artigo, abordaremos os principais tópicos relacionados à camada de enlace de dados, discutiremos serviços oferecidos, enquadramento, controle de erros e outros aspectos.

#### Funções Básicas da Camada de Enlace de Dados

A camada de enlace de dados utiliza os serviços da camada física para a transmissão de bits, garantindo que os dados cheguem à máquina de destino. Entre as funções principais desta camada estão:

1. **Interface de Serviço**: Proporcionar uma interface definida para a camada de rede, facilitando a comunicação entre camadas superiores e inferiores.
2. **Enquadramento**: Organizar bytes em quadros para transmissão eficiente e integrada.
3. **Controle de Erros**: Detectar e corrigir possíveis erros durante a transmissão de dados.
4. **Controle de Fluxo**: Regular a taxa de transmissão de dados para evitar sobrecarregar receptores mais lentos.

##### Serviços

Os serviços da camada de enlace de dados variam de acordo com o protocolo, mas podemos categorizá-los em três tipos principais:

- **Serviço não orientado a conexões sem confirmação**: Os quadros são enviados sem qualquer confirmação de recebimento. A Ethernet é um exemplo clássico desse serviço, sendo usado em ambientes onde a taxa de erro é baixa e a recuperação de dados é feita por camadas superiores.
- **Serviço não orientado a conexões com confirmação**: Cada quadro enviado é confirmado individualmente, o que permite o retransmissão de quadros perdidos. O padrão **802.11 (WiFi)** adota essa abordagem para garantir confiabilidade em redes sem fio.
- **Serviço orientado a conexões com confirmação**: Neste serviço, uma conexão lógica é estabelecida entre as máquinas antes do envio dos dados. Cada quadro é numerado e a confirmação garante a entrega.
##### Enquadramento

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

#### 2. Controle de Erros e de Fluxo

Os protocolos da camada de enlace de dados empregam diferentes mecanismos para controlar erros e fluxo, garantindo a integridade e eficiência da transmissão de dados. Entre os principais mecanismos estão:

- **Detecção de Erros**: Métodos como o checksum e CRC (Cyclic Redundancy Check) são amplamente utilizados para identificar falhas na transmissão de quadros.
- **Correção de Erros**: Em alguns casos, a camada de enlace pode corrigir automaticamente pequenos erros ou solicitar a retransmissão do quadro defeituoso.
- **Controle de Fluxo**: Protocolos como o **Windowing** (controle de janela) e **ACK/NACK** (acknowledgement/negative acknowledgement) regulam o fluxo de dados entre dispositivos, evitando que um transmissor rápido sobrecarregue um receptor mais lento.

