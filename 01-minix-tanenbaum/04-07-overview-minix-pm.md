# 4.7 - Overview do Gerenciador de Processos do MINIX 3

* Gerenciamento de memória em MINIX 3: paginação e swapping **NÃO** são usados
* **Gerenciador de processos (PM)**: servidor no espaço de usuário
  * Gerencia chamadas de sistema relacionadas a gerenciamento de processos, algumas envolvidas com gerenciamento de memória (`fork`, `exec` e `brk`)
  * Outras chamadas de gerenciamento de processos são relacionadas a sinais, propriedades de processo como permissões e uso de CPU
  * PM também gerencia o relógio de tempo real
* Em MINIX 3 o gerenciador de memória está embutido no gerenciador de processos
* **PM** mantém uma lista com os buracos na memória, ordenados por endereço de memória
  * Quando memória se faz necessária (`fork` ou `exec`), procura-se na lista de buracos usando **first fit**
  * O processo permanece sempre no mesmo lugar pois não há swap e sua área alocada também não cresce ou diminui
* Razões para MINIX 3 não possuir swapping ou paginação:
  * Fácil de entender (fins didáticos)
  * Arquitetura do Intel 8088 no IBM PC original
  * Fácil de portar para outros hardwares
* A implementação do gerenciamento de memória no MINIX 3 difere de outros SOs pois PM não é parte do kernel
  * PM é processo que roda no espaço do usuário e se comunica com o kernel através de mensagens
  * Exemplo da **separação** entre **política** e **mecanismo**:
    * **Política**: colocação de cada processo na memória é decidida pelo PM
    * **Mecanismo**: configurar os mapas de memória para os processos é feito pela system task no kernel
    * Fácil de modificar a política (algoritmos, etc) sem modificar as camadas base do SO

## 4.7.1 - Layout de memória