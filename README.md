

# Projeto 3.1 - Serialização/Desserialização 

O objetivo principal deste projeto é a compreensão de como uma mensagem é serializada/desserializada nos níveis mais baixos de uma comunicação.
Para a realização do projeto, foi utilizado o protocolo UART e dois Arduino Due.


 
## Protocolo UART

O protocolo UART possui como valor inicial 1 (sinal HIGH) e a recepção é iniciada quando é enviado um START BIT, de valor 0 (LOW). Com o START BIT enviado, a transmissão dos dados é iniciada.
O payload é um caractere de uma palavra arbitrária a ser transmitida e cada caractere é um conjunto de 8 bits (1 byte). Após  

## Requisitos 
1. Transmissor
     - serializar dados via uart embarcado no arduino
     
2. Receptor
     - desserializar dados via uart embarcado no arduino
          
3. Documentação
     - Explicar a comunicação UART
     - Exibir a forma de onda gerada pela implementação (usando o analog discovery)
     - Explicar o código

## Itens extras

- Implementar um protocolo próprio de transmissão e recepção de dados.

## Validação

- Em sala de aula, abrir as duas aplicações em computadores distintos e transmitir uma frase entre eles via o pipeline desenvolvido anteriormente.

## Rubricas

| Nota | Descritivo                                                |
|------|-----------------------------------------------------------|
| A    | - Entregue no prazo                                       |
|      | - Implementado um item do extras                          |
| B    | - Entregue no prazo                                       |
|      | - Implementado requisitos necessários                     |
| C    | - Entregue fora do prazo                                  |
|      | - Implementando requisitos necessários                    |
| D    | - Nem todos os requisitos necessários foram implementados |
| I    | - Não entregue                                            |



