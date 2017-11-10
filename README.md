

# Projeto 3.1 - Serialização/Desserialização 

O objetivo principal deste projeto é a compreensão de como uma mensagem é serializada/desserializada nos níveis mais baixos de uma comunicação.
Para a realização do projeto, foi utilizado o protocolo UART e dois Arduino Due.


 
## Protocolo UART

O protocolo UART possui como valor inicial 1 (sinal HIGH) e a recepção é iniciada quando é enviado um START BIT, de valor 0 (LOW). Com o START BIT enviado, a transmissão dos dados é iniciada.
Cada payload é um caractere de uma frase arbitrária a ser transmitida e cada caractere retorna um conjunto de 8 bits (1 byte). Após o envio do payload é enviado o bit de paridade, que tem por função checar se o pacote enviado está ou não corrompido.
Por fim o STOP BIT, de valor 1 (HIGH), é enviado, retornando o protocolo ao seu estado inicial.

A imagem a seguir representa um pacote transmitido pelo protocolo UART.

![UART](doc/uart.png) 

## Resultados no WaveForms

O gráfico a seguir foi obtido utilizando um Arduino Due rodando o código de transmissão um Analog Discovery. O pino TX1 do Arduino foi conectado no pino 7 do Analog e ambos compontentes foram conectados no terra.
 
![Grafico do WaveForms](doc/transmissao.png) 

No gráfico é possivel identificar os bits (considerando o pacote h4C):

OBSERVAÇÃO: na imagem, o payload está invertido devido ao shift realizado no código (será explicado na seção seguinte).

 - Start bit (0)
 - Caractere L (01001100)
 - Parity bit (1)
 - Stop bit (1)

## Descrição do TX
### Enviar START BIT

Envia o valor 0 (LOW) para o pino TX


```
digitalWrite(uart -> pin_tx, LOW);
_sw_uart_wait_T(uart);
  }
 ```
     
### Enviar START BIT

O código percorre bit por bit de cada caractere e realiza um bit shift para a direita e também realizando um and com o byte 0x01. Desta forma, todos os bits são zerados, com excessão do mais significativo. A cada iteração, o bit shift faz com que o bit significativo seja alterado e enviado ao pino tx do arduino.
 
 ```
for(int i = 0; i <= 7 ; i++) {
     int pyl = data >> i & 0x01;
     digitalWrite(uart -> pin_tx, pyl);
     _sw_uart_wait_T(uart);
   } 
 ```
          
### Parity bit

**Cálculo do parity bit**: Inicialmente, o código realiza o mesmo processo que o código do envio do START BIT. Contudo, ao invés de enviar o bit mais significativo para o pino de transmissão, este é mantido em uma variável. Cada iteração faz com que o bit significativo seja somado aos bits significativos anteriores.

Após a soma dos bits é posta uma condição pois deseja-se obter o bit de paridade IMPAR. Desta forma, se a divisão do valor da soma dos bits significativos for não divisível por 2, o código retorna 0 e caso contrário, retorna 1. 

 ```
int calc_even_parity(char data) {
  int sum = 0;
  for(int i = 0; i<=7; i++){
    sum += (data >> i) & 0x01;
  }

  if (sum % 2 == 1)
   {
    return 0;
  }
  else
   {
    return 1;
  }

} 
 ```

**Envio do parity bit**: Inicialmente, a variável parity é definida como 0. O valor é atualizado de acordo com o resultado da função descrita no item de cálculo do parity bit. 

 ```
  int parity = 0;

  if(uart->paritybit == SW_UART_EVEN_PARITY) {
     parity = calc_even_parity(data);
  } else if(uart->paritybit == SW_UART_ODD_PARITY) {
     parity = !calc_even_parity(data);
  }
 ```
 
 Por fim, o valor já atualizado de acordo com os dados passados na função de cálculo é enviado para o pino tx do Arduíno.
 
 ```
  if(uart->paritybit != SW_UART_NO_PARITY) {
    digitalWrite(uart -> pin_tx, parity);
    _sw_uart_wait_T(uart);
  }
 ```

### Enviar STOP BIT

O envio de um pacote é finalizado com o envio do sinal 1 (HIGH) para o pino tx, fazendo com que o canal de transmissão volte ao seu estado inicial.

```
  for(int i = 0; i < uart->stopbits; i++) {
    digitalWrite(uart -> pin_tx, HIGH);
    _sw_uart_wait_T(uart);
  }
}
```

## Descrição do RX

### Recebe START BIT

O código faz um loop aguardando a recepção do valor 0 (LOW) no pino RX do arduino. Esta recepção é feita em duas partes para evitar que alguma ruído possa vir a ser reconhecido como start bit.

```
  bool aguardante = false;
  while (aguardante == false){

  // Confirma start BIT
    if (digitalRead(uart -> pin_rx) == 0 ){
      _sw_uart_wait_half_T(uart);
      
  // checa se bit ainda é 0
      if(digitalRead(uart -> pin_rx) == 0 ){
        aguardante = true;
      }
    }
  }

  _sw_uart_wait_T(uart);
  ```
  
### Recebe dados

A recepção de dados é semelhante à transmissão de dados do TX. A diferença entre os códigos é a maneira que ocorre o bit shift. No caso da recepção, o bit shift é para a esquerda e é utilizado um or.

```
  for(int i = 0; i<=7; i++){
    nchar = (digitalRead(uart -> pin_rx) << i) | nchar;
    _sw_uart_wait_T(uart);
  }
  ```
 
### Recebe parity

O cálculo de paridade do código do TX e do RX deve ser igual para que ocorra o recebimento dos dados. 
Após o recebimento de cada bit do payload, espera-se um tempo T para que seja iniciado o recebimento do próximo bit.

```
  rx_parity = digitalRead(uart -> pin_rx);
  _sw_uart_wait_T(uart);  
```

### Recebe STOP BIT

O final da recepção de um pacote é marcado pelo STOP BIT, possibilitando que a recepção de um outro pacote seja iniciada.

```
  recebe stop bit
  int stopBit = digitalRead(uart -> pin_rx);
```
