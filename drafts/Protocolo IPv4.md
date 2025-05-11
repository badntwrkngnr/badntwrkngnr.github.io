## Introdução

Esse é um assunto que merecia a dedicação de um artigo exclusivo por conta de alguns pequenos detalhes, que não são utilizados no cotidiano.

Se você está chegando agora e não possui conhecimento prévio, recomendo que você leia o artigo introdutório que escrevi.

O texto utiliza a RFC791 como base devido à sua importância histórica, ao longo desse artigo foram adicionadas as atualizações aplicadas ao longo do tempo.

Antes de começarmos, é importante informar que apesar da utilização do IPv4 ser muito difundida, o protocolo é obsoleto e, portanto, considerado legado.

O novo padrão para a internet é o IPv6 e falaremos dele no futuro.

Agora, vamos continuar de onde havíamos parado.

## O Cabeçalho IP

O cabeçalho IP é formado por palavras de 32 bits (4 bytes por palavra), possui uma parte fixa de 20 bytes e uma parte opcional de tamanho variável, conforme podemos observar na imagem abaixo, cada uma dessas palavras é representada por uma “linha”:

![[Untitled 83.png|Untitled 83.png]]

Os dados são transmitidos da esquerda para a direita e de cima para baixo, com o bit mais significativo localizado no campo _Versão_, essa ordenação de bytes é conhecida como _big-endian_.

> _Endianness (literal: extremidade, em computação: ordem dos bytes)_, descreve como os computadores organizam os bytes de dados na memória de acordo com a sua grandeza.  
>   
>   
> _Big-endian_, é a ordenação dos bytes partindo do byte mais significativo para o menos significativo, essa ordenação também é conhecida como ordem de bytes de rede.  
>   
>   
> _Little-endian_, é o oposto da big-endian, a ordenação dos bytes é feita do byte menos significativo para o mais significativo.

Vamos analisar o cabeçalho na ordem da transmissão dos dados, começando pelo campo **_Versão_**.

![[Untitled 1 47.png|Untitled 1 47.png]]

### Versão

![[Untitled 2 37.png|Untitled 2 37.png]]

Apesar da sua extrema importância, é muito simples de entender o seu funcionamento, nesse ponto é definido como os bits subsequentes serão interpretados.

O campo **_Versão_** possui 4 bits de comprimento e indica o formato do cabeçalho IP, para o escopo desse artigo, estamos observando a versão 4.

Abaixo segue o exemplo de um pacote, demonstrando a atribuição do campo para a versão 4 _(notação binária 0100)_:

![[1e6466d8-8f32-42cb-9a8b-69281a693196.png]]

### Comprimento do Cabeçalho de Internet

![[Untitled 3 23.png|Untitled 3 23.png]]

O campo Comprimento do Cabeçalho de Internet possui 4 bits de comprimento e informa quantas palavras de 32 bits serão utilizadas na composição do cabeçalho, para o cabeçalho ser considerado válido, ele possui um valor mínimo de 5 e um valor máximo de 15 _(0101 e 1111, respectivamente na notação binária)_.

Conforme demonstrado no exemplo abaixo, foi observado um valor de 20 bytes, que corresponde a 5 “palavras”:

![[a7c1ccbc-39d8-4b22-ad57-e5b99d342d68.png]]

### Tipo de Serviço

![[Untitled 4 20.png|Untitled 4 20.png]]

O campo Tipo de Serviço, possui 8 bits de tamanho e tem a função de diferenciar as classes de serviços.

Vamos pensar que para aplicações de voz, a entrega rápida é mais importante que a garantia de entrega, para o caso de transferência de arquivos, evitar a transmissão com erros é mais importante que a velocidade.

Em resumo, o campo Tipo de Serviço fornecia 3 bits para prioridade de sinal e 3 bits para informar o que era mais importante para o host, se era o atraso, throughput ou confiabilidade.

Abaixo podemos observar em detalhes os parâmetros utilizados para classificar os serviços:

Bits 0-2: Precedência

Bit 3: Delay, 0 = Normal e 1 = Baixo Delay

Bit 4: Throughput, 0 = Normal e 1 = Alto Throughput

Bit 5: Confiabilidade, 0 = Normal e 1 = Alta Confiabilidade

Bits 6-7: Reservados para uso futuro

> Em redes de computadores, o termo **_throughput_** se refere à quantidade de dados que podem ser transmitidos com sucesso através da rede em um determinado período de tempo.  
> Em termos simples, é a medida da eficiência da rede em transmitir dados.  

Precedência:

111 - Controle de Rede

110 - Controle Entre Redes

101 - Crítico/ECP

100 - Flash Override

011 - Flash

010 - Imediato

001 - Prioridade

000 - Rotina

![[Untitled 5 17.png|Untitled 5 17.png]]

Como ninguém sabia utilizar esses parâmetros nos roteadores, acabou que esse campo não era muito utilizado, foi quando a IETF aproveitou e usou para implementar os Serviços Diferenciados (RFC2474), por mais que o cerne da função não tenha mudado.

![[Untitled 6 12.png|Untitled 6 12.png]]

![[Untitled 7 11.png|Untitled 7 11.png]]

### Comprimento Total

![[Untitled 8 9.png|Untitled 8 9.png]]

indica o comprimento do datagrama, incluindo o cabeçalho IP e os dados.

### Identificação

![[Untitled 9 8.png|Untitled 9 8.png]]

um valor identificador atribuído pelo remetente para auxiliar na montagem dos fragmentos de um datagrama.

### Flag

![[Untitled 10 5.png|Untitled 10 5.png]]

indica informações sobre a fragmentação dos pacotes.

### Deslocamento de Fragmento

![[Untitled 11 4.png|Untitled 11 4.png]]

indica a localização do fragmento dentro do datagrama.

### Tempo de vida

indica a quantidade de saltos um datagrama pode dar até ser descartado.

### Protocolo

indica o protocolo utilizado na camada de transporte.

### Soma de Verificação do Cabeçalho
como alguns campos do cabeçalho mudam, como por exemplo TTL, é feita uma verificação em cada nó que o cabeçalho IP é processado.

## Materiais de apoio e referências:

[Definição de Endianness - Mozilla](https://developer.mozilla.org/pt-BR/docs/Glossary/Endianness)  

[Extremidade ou ordenação - Wikipedia](https://pt.wikipedia.org/wiki/Extremidade_(ordena%C3%A7%C3%A3o))
