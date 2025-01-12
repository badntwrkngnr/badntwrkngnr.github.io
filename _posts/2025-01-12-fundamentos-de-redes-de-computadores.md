---
Descrição: Fundamentos de Redes de Computadores
Layout: post
Categories: networking
---

# Fundamentos de Redes de Computadores
## Introdução

Este artigo representa um dos trabalhos mais cansativos que já realizei. Foram diversas revisões e reestruturações, decidi refazer, quase que do zero, e abordar o tema de maneira simples, direta e acessível. Organizei o conteúdo em tópicos-chave, buscando criar um resumo prático e fácil de assimilar. Essa abordagem reflete a forma como eu gostaria de ter aprendido, e espero que ela seja útil para você também.
## Objetivos de uma rede

- Acesso a dados e serviços: As redes permitem que os usuários acessem dados e serviços de outros dispositivos conectados à rede.
- Compartilhamento de recursos: Os recursos, como impressoras e armazenamento, podem ser compartilhados entre os dispositivos da rede.
- Administração centralizada: Uma rede pode ser administrada centralmente, facilitando o gerenciamento e a manutenção dos dispositivos e serviços.

## Camadas de Rede

As camadas de rede organizam as funções necessárias para a comunicação entre dispositivos. Cada modelo divide essas funções em grupos lógicos (camadas), que trabalham juntas para garantir a transmissão de dados.
### Modelo OSI (7 camadas)

O modelo OSI (**Open Systems Interconnection**) é um modelo teórico, usado como referência para entender e projetar redes de computadores. Ele divide a comunicação em **7 camadas**, cada uma com funções específicas:

1. **Física**: Transmite bits pelo meio físico (cabos, Wi-Fi).
2. **Enlace de Dados**: Garante a comunicação confiável entre dois dispositivos conectados.
3. **Rede**: Determina a rota que os dados devem seguir (endereços IP).
4. **Transporte**: Garante a entrega confiável ou rápida dos dados trocados entre dispositivos (TCP/UDP).
5. **Sessão**: Gerencia conexões entre aplicativos.
6. **Apresentação**: Traduz os dados para um formato compreensível, como criptografia ou compressão.
7. **Aplicação**: Interage com o usuário final (navegadores, e-mail).
### Modelo TCP/IP (4 camadas)

O modelo TCP/IP é mais simples e prático. Ele é amplamente utilizado para guiar o funcionamento da internet. Possui **4 camadas**:

1. **Acesso à Rede**: Cuida da transmissão física e do controle de acesso ao meio.
2. **Internet**: Trata do endereçamento e do roteamento dos dados (protocolo IP).
3. **Transporte**: Garante a entrega confiável dos dados (TCP) ou comunicação rápida (UDP).
4. **Aplicação**: Abrange tudo o que o usuário vê e interage, como navegadores e aplicativos.
### Modelo Híbrido (5 camadas)

Esse modelo é uma simplificação que combina elementos do OSI e do TCP/IP, muito usado na literatura e pelos fabricantes de equipamentos. Ele tem **5 camadas**:

1. **Física**: Lida com os aspectos físicos da transmissão (cabos, sinais).
2. **Enlace**: Gerencia a comunicação entre dispositivos conectados diretamente.
3. **Rede**: Define o roteamento e endereçamento (IP).
4. **Transporte**: Garante a entrega confiável ou rápida dos dados (TCP/UDP).
5. **Aplicação**: Focado nos aplicativos usados pelo usuário (HTTP, FTP, etc.).
## Abrangência das redes

A infraestrutura de rede pode variar em termos de tamanho da área de cobertura, número de usuários, quantidade e tipos de serviços fornecidos e área de responsabilidade. Os principais tipos de redes são:

- PAN (Personal Area Network): É uma rede de área pessoal que conecta dispositivos em torno de uma pessoa
- LAN (Local Area Network): É uma rede local que cobre uma área limitada, como um escritório ou um prédio.
- MAN (Metropolitan Area Network): É uma rede metropolitana que abrange uma cidade ou uma área geográfica maior.
- WAN (Wide Area Network): É uma rede de longa distância que pode abranger uma grande área geográfica, como um país ou até mesmo globalmente.
- SAN (Storage Area Network): É uma rede de armazenamento dedicada para conectar dispositivos de armazenamento, como discos rígidos e unidades de fita.

### Como a internet funciona?

De um modo bem simples, e até grosseiro, a internet é um conjunto de várias redes interconectadas globalmente, permitindo a comunicação e o compartilhamento de informações e serviços em escala global (chora terraplanista).

### Intranets e Extranets

- Intranet: É uma rede privada de LANs e/ou WANs pertencente a uma organização. Ela é acessível apenas aos funcionários e membros autorizados da organização.
- Extranet: É a parte da rede que pode ser acessada por usuários externos à organização, fornecendo acesso controlado a determinados serviços ou informações.

## Arquitetura de Redes

Existem quatro características básicas que os arquitetos de rede devem considerar para atender às expectativas dos usuários:

1. Tolerância a falhas
    - Uma rede deve ser projetada para ser resiliente a falhas, de modo que, mesmo se um componente ou conexão falhar, a rede possa continuar funcionando sem interrupções significativas.
2. Escalabilidade
    - A arquitetura da rede deve ser escalável, ou seja, capaz de lidar com o aumento do número de dispositivos e usuários sem comprometer o desempenho.
3. Qualidade de Serviços (QoS)
    - A QoS refere-se à capacidade da rede de fornecer serviços com diferentes requisitos de desempenho, como largura de banda, latência e confiabilidade, de acordo com as necessidades dos aplicativos e usuários.
4. Segurança:
    - A segurança é uma consideração essencial na arquitetura de rede para proteger os dados e os recursos da rede contra acesso não autorizado, ataques e outras ameaças.

## Topologias de Rede

Topologias de rede descrevem como os dispositivos (computadores, roteadores, etc.) estão conectados e organizados em uma rede.

### Barramento

Na topologia de barramento, todos os dispositivos estão conectados a um único cabo principal, chamado de "barramento".

<img src="/assets/images/networking/2025-01-12-fundamentos-de-redes-de-computadores/bus-network.svg" height="300px"/>

### Anel

Na topologia de anel, os dispositivos estão conectados em círculo, formando um laço fechado. Os dados circulam em uma única direção (ou às vezes em ambas).

<img src="/assets/images/networking/2025-01-12-fundamentos-de-redes-de-computadores/ring-network.svg" height="300px"/>

### Estrela

Na topologia em estrela, todos os dispositivos estão conectados a um dispositivo central, como um switch ou roteador.

<img src="/assets/images/networking/2025-01-12-fundamentos-de-redes-de-computadores/star-network.svg" height="300px"/>

### Mesh

Na topologia mesh, cada dispositivo está conectado a vários outros dispositivos, criando múltiplos caminhos para os dados.

<img src="/assets/images/networking/2025-01-12-fundamentos-de-redes-de-computadores/mesh-network.svg" height="300px"/>

*Isso não é um pentagrama, ok?*
## Modelos de Redes de Computadores

### Centralizada (Mainframe x Terminais)

Neste modelo, há um computador central muito potente, o **mainframe**, que realiza o processamento dos dados. Os **terminais**, que são dispositivos simples, apenas enviam e recebem informações do mainframe, sem processar dados de forma independente. Imagine uma sala de evidências, onde todos os itens apreendidos são armazenados. A sala funciona como o **mainframe**, concentrando todas as informações e recursos. Quando um investigador (terminal) precisa consultar ou adicionar algo, ele deve acessar a sala, mas não pode processar as evidências por conta própria. Todo o controle, organização e processamento das informações acontecem dentro da sala.

### Cliente/Servidor

No modelo **cliente/servidor**, o servidor é um computador central que oferece serviços ou recursos, como arquivos, websites ou aplicativos. Os **clientes** são dispositivos que solicitam esses serviços ao servidor. Imagine uma cafeteria. O **balcão de atendimento** é o servidor, enquanto os clientes são... bem, os **clientes**.

1. O cliente vai até o balcão (servidor) e faz o pedido, como um café, lanche ou salgado.
2. O atendente no balcão prepara o pedido (processa a solicitação) e entrega o item pronto ao cliente.
3. O cliente consome o que recebeu e, se precisar de algo mais, volta ao balcão para fazer outra solicitação.

### Peer-to-peer

No modelo **peer-to-peer** (P2P), todos os dispositivos na rede (os **pares**) têm as mesmas funções, podendo tanto fornecer quanto receber recursos. Não há um servidor centralizado. Imagine um grupo de amigos que formaram um clube do livro.

1. Cada pessoa traz livros para compartilhar e também pode pegar emprestado livros de outros membros.
2. Não existe uma biblioteca ou responsável central para organizar os empréstimos; cada um é tanto "fornecedor" quanto "consumidor".
3. Por exemplo, se você quiser ler um livro específico, pode pedir para um dos colegas caso algum deles tenha o exemplar. Da mesma forma, outros membros podem pedir livros emprestados à você.
## Serviços

### Tipos de Serviços

Basicamente, existem dois tipos de serviços na internet: **orientados à conexão** e **não orientados à conexão**.

- **Comutação de circuitos**: Aqui, existe um canal dedicado e uma rota pré-definida para a comunicação. Imagine que você mora em Bauru/SP e precisa viajar de ônibus até São Paulo/SP. A rota já foi estabelecida pela empresa de ônibus: o veículo sai da rodoviária de Bauru, segue pela rodovia Marechal Rondon até Botucatu, pega a rodovia Castelinho, depois a Castello Branco, atravessa um pequeno trecho da Marginal Tietê e, finalmente, chega à rodoviária da Barra Funda. Mesmo que ocorram congestionamentos ou acidentes no trajeto, o ônibus seguirá o roteiro definido.
    
- **Comutação de pacotes**: Nesse caso, os dados são enviados em blocos discretos (chamados de pacotes), que podem tomar caminhos diferentes até chegar ao destino. Agora imagine que você faz o mesmo trajeto de carro, usando um aplicativo de navegação. Se houver um acidente ou congestionamento no trajeto, o aplicativo recalcula a rota e te direciona por um caminho alternativo mais rápido, evitando o incidente.

> Datagramas são unidades de transferência básica, associadas a redes de comutação de pacotes. Eles são usados em redes que fornecem um serviço de comunicação sem conexão, sem a necessidade de entrega garantida, hora de chegada ou ordem de chegada.

### Exemplos de Serviços

Email: é um serviço de comunicação eletrônica que permite o envio e recebimento de mensagens assíncronas por meio da internet, podemos fazer analogia às cartas enviadas pelos serviços postais.

Terminal:

- Telnet: É um serviço que permite a conexão a um dispositivo remoto para acessar recursos e executar comandos, sem criptografia
- SSH: É um serviço similar ao Telnet, mas com criptografia, garantindo maior segurança nas comunicações.

Nuvem:

- Nuvem pública: É uma infraestrutura de computação em nuvem fornecida por provedores de serviços para uso público geral.
- Nuvem privada: É uma infraestrutura de computação em nuvem que é dedicada a uma única organização e não é compartilhada com outras organizações.