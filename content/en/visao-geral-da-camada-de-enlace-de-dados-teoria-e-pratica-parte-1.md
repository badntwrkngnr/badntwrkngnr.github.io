---
title: "x"
slug: "x" 
date: 2026-05-23
translationKey: "x"
categories: ["nx"]
math: true
draft: true
---

## Introdução e Base Teórica

Este artigo é baseado em anotações que fiz durante minha preparação para o exame CCNA (Cisco Certified Network Associate) 200-301, onde pude aprofundar meus conhecimentos sobre a camada de enlace de dados.

A **camada de enlace de dados** é fundamental no modelo OSI e TCP/IP, sendo a segunda camada no processo de comunicação entre dispositivos em rede. Ela é responsável por garantir que as unidades de informação, chamadas **quadros**, sejam transferidas de maneira confiável entre dispositivos fisicamente conectados por meio de um canal de comunicação, sejam eles guiados, como cabos de rede e fibras ópticas, ou não-guiados, como as redes sem fio.

## Uma Reflexão Pessoal sobre a Importância da Camada 2

Durante minha preparação para o exame CCNA 200-301, confesso que **subestimei significativamente a importância da camada de enlace de dados**. Minha experiência profissional sempre esteve mais focada em protocolos de camada 3, trabalhando na maior parte do tempo com backbones e serviços de VPNs L2/L3 em ambientes de service providers. Essa bagagem me levou a acreditar que dominar a camada 2 seria extremamente simples.

**Admito que eu estava completamente enganado!** Devido ao CCNA ser focado em redes enterprise, percebi lacunas importantes no meu conhecimento sobre como switches realmente funcionam, como endereços MAC são utilizados, e como VLANs impactam diretamente a arquitetura de redes corporativas. A diferença entre o ambiente de service provider (onde a camada 2 é muitas vezes "transparente" para os serviços) e o ambiente enterprise (onde a camada 2 é o alicerce de tudo) é muito mais significativa do que eu imaginava.

Esta experiência me ensinou que, independentemente do seu background em redes, **a camada de enlace de dados merece atenção especial e estudo dedicado**. Para profissionais que, como eu, têm mais experiência com protocolos de camada superior, entender profundamente como a camada 2 funciona é fundamental para uma preparação completa para certificações como o CCNA.

Em todo lugar, você vai observar comentários dizendo que para profissionais de redes de computadores é necessário ter uma base sólida, por isso, uma das coisas fundamentais é conhecer a camada de enlace, pois ela é essencial para entender como switches, endereços MAC e VLANs funcionam na prática. Neste artigo, exploraremos alguns dos conceitos teóricos e depois, nos próximos artigos, aplicaremos esse conhecimento em cenários reais de configuração.

## Funções Básicas da Camada de Enlace de Dados

A camada de enlace de dados utiliza os serviços da camada física para a transmissão de bits, garantindo que os dados cheguem à máquina de destino. Entre as funções principais desta camada estão:

1. **Interface de Serviço**: Proporcionar uma interface definida para a camada de rede, facilitando a comunicação entre camadas superiores e inferiores.
2. **Enquadramento**: Organizar bytes em quadros para transmissão eficiente e integrada.
3. **Controle de Erros**: Detectar e corrigir possíveis erros durante a transmissão de dados.
4. **Controle de Fluxo**: Regular a taxa de transmissão de dados para evitar sobrecarregar receptores mais lentos.

Além das funções citadas, a camada de enlace também é responsável pelo **endereçamento físico**, utilizando endereços MAC (Media Access Control) para identificar dispositivos em uma rede local (LAN).

O endereço MAC possui **6 bytes (48 bits)** e é dividido em duas partes:

1. **OUI (Organizationally Unique Identifier)**: 3 bytes (24 bits) - Identifica o fabricante do dispositivo
2. **Vendor Assigned**: 3 bytes (24 bits) - Identificador único atribuído pelo fabricante

<img src="/assets/images/networking/2025-06-22-visao-geral-da-camada-de-enlace-de-dados-teoria-e-pratica/26-mac-address-structure.png">

O endereço MAC é representado em formato hexadecimal, separado por dois pontos ou hífens. Por exemplo: `00:1A:2B:3C:4D:5E` ou `00-1A-2B-3C-4D-5E`. Apenas de curiosidade, o primeiro byte do OUI também contém informações importantes:

- **Bit 0 (LSB)**: Indica se o endereço é unicast (0) ou multicast (1)
- **Bit 1**: Indica se o endereço é globalmente administrado (0) ou localmente administrado (1)
### Serviços

Os serviços da camada de enlace de dados variam de acordo com o protocolo, mas podemos categorizá-los em três tipos principais:

- **Serviço não orientado a conexões sem confirmação**: Os quadros são enviados sem qualquer confirmação de recebimento. A Ethernet é um exemplo clássico desse serviço, sendo usado em ambientes onde a taxa de erro é baixa e a recuperação de dados é feita por camadas superiores.
- **Serviço não orientado a conexões com confirmação**: Cada quadro enviado é confirmado individualmente, o que permite a retransmissão de quadros perdidos. O padrão **802.11 (WiFi)** adota essa abordagem para garantir confiabilidade em redes sem fio.
- **Serviço orientado a conexões com confirmação**: Neste serviço, uma conexão lógica é estabelecida entre as máquinas antes do envio dos dados. Cada quadro é numerado e a confirmação garante a entrega.

Nas certificações de fabricantes de equipamentos, sobretudo o CCNA 200-301, o foco geralmente recai sobre o **Ethernet (IEEE 802.3)**, que utiliza um serviço não orientado a conexão sem confirmação. A título de curiosidade, podemos falar que o **WiFi (IEEE 802.11)** usa confirmações devido à natureza propensa a erros das redes sem fio, esse padrão também é cobrado no blueprint do CCNA 200-301, mas não será abordado nesse artigo.

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
| FCS (Frame Check Sequence)  | 4       | A NIC (Network Interface Card) de destino utiliza esse campo para saber se foram detectados erros na transmissão de dados                                                                          |

### Controle de Erros e de Fluxo

Os protocolos da camada de enlace de dados empregam diferentes mecanismos para controlar erros e fluxo, garantindo a integridade e eficiência da transmissão de dados. Entre os principais mecanismos estão:

- **Detecção de Erros**: Métodos como o checksum e CRC (Cyclic Redundancy Check) são amplamente utilizados para identificar falhas na transmissão de quadros.
- **Correção de Erros**: Em alguns casos, a camada de enlace pode corrigir automaticamente pequenos erros ou solicitar a retransmissão do quadro defeituoso.
- **Controle de Fluxo**: Protocolos como o **Windowing** (controle de janela) e **ACK/NACK** (acknowledgement/negative acknowledgement) regulam o fluxo de dados entre dispositivos, evitando que um transmissor rápido sobrecarregue um receptor mais lento.

### Métodos de Encaminhamento de Quadros

- Store-and-Forward: o switch recebe o quadro inteiro antes de encaminhá-lo, fornecendo maior confiabilidade e, consequentemente, maior latência.
- Cut-Through: o switch começa a encaminhar o quadro assim que detecta o endereço MAC de destino, 6 bytes logo após o campo SFD, e não faz a verificação de erros, a latência tende a ser muito baixa, porém a chance de encaminhar quadros corrompidos é alta, sendo ideal para redes com baixa taxa de erros.
  - Fast-Forward: é uma variação do Cut-Through, onde o switch introduz um pequeno atraso (delay) antes de encaminhar, reduzindo colisões. O switch aguarda os primeiros 64 bytes, tamanho mínimo de um quadro ethernet, antes de encaminhar, se o quadro for menor, é descartado (runt frame). É um método menos comum, é mais observado em switches mais antigos.
  - Fragment-Free: é um híbrido entre o Cut-Through e o Store-and-Forward. O switch verifica os primeiros 64 bytes, onde ocorrem a maioria dos erros de transmissão, antes de encaminhar, se não houver erro, ele encaminha o restante do quadro sem fazer verificação de erros. Faz o balanceamento entre velocidade e confiabilidade mínima.

Abaixo segue uma tabela que faz uma comparação entre os modos de encaminhamento de quadros:

| Modo              | Latência | Verificação de Erros | Uso típico                      |
| ----------------- | -------- | -------------------- | ------------------------------- |
| Store-and-Forward | Alta     | Completa (CRC/FCS)   | Redes modernas                  |
| Cut-Through       | Mínima   | Nenhuma              | Data Centers / Low-Latency      |
| Fragment-Free     | Moderada | Primeiros 64 bytes   | Redes com histórico de colisões |

## Conclusão

Neste primeiro artigo, exploramos os fundamentos teóricos da camada de enlace de dados, abordando suas funções essenciais, serviços, métodos de enquadramento e controle de erros e fluxo. Compreendemos como os diferentes modos de encaminhamento de quadros impactam o desempenho e a confiabilidade das redes.

Esta base teórica sólida é fundamental para profissionais de redes, especialmente aqueles que buscam certificações como o CCNA 200-301, onde o conhecimento prático da camada de enlace é essencial para configurar switches, gerenciar VLANs e entender o funcionamento dos endereços MAC.

**Para profissionais com background em service providers** (como eu), que estão acostumados a trabalhar com protocolos de camada superior como MPLS e VPNs, este estudo da camada 2 representa uma mudança significativa de perspectiva. O que antes era "transparente" nos serviços de telecomunicações, agora se torna o **alicerce fundamental** para entender como as redes enterprise realmente funcionam.

Nos próximos artigos desta série, aprofundaremos esses conceitos através de **configurações práticas em equipamentos reais**, onde implementaremos cenários de laboratório que demonstram:

- Configuração e gerenciamento de switches
- Implementação e troubleshooting de VLANs
- Análise de tráfego de rede com ferramentas de captura
- Resolução de problemas comuns em redes locais
- Otimização de desempenho baseada nos diferentes modos de encaminhamento

O conhecimento teórico apresentado aqui servirá como alicerce para as práticas avançadas que virão, permitindo uma compreensão mais profunda dos mecanismos internos dos equipamentos de rede e suas aplicações no mundo real. **Para aqueles que, como eu, subestimaram inicialmente a importância da camada 2, este estudo representa um investimento valioso no desenvolvimento de uma visão mais completa e integrada das redes de computadores.**

