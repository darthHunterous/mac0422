# 4.1 - Gerenciamento Básico de Memória

* **Classes de sistemas de gerenciamento**

  1. Sistemas que movem processos entre a memória e o disco durante execução (swap e paginação)

  2. Sistemas que não movem processos (mais simples)

## 4.1.1 - Monoprogramação sem swapping ou paginação

* **Mais simples**: rodar apenas um programa e compartilhar a memória entre esse programa e o SO	
  * **Três modelos**
    1. SO na RAM no fundo da memória
       * Mainframes e minicomputadores
    2. SO na ROM (Read Only) no topo da memória
       * Palmtops e sistemas embarcados
    3. Drivers de dispositivo no topo na ROM e resto do SO na RAM no fundo
       * Computadores pessoais iniciais (MS-DOS), porção do sistema na ROM é a **BIOS** (Basic Input Output System)

## 4.1.2 - Multiprogramação com partições fixas

* Sistemas modernos permitem vários processos ao mesmo tempo. Quando um bloqueia por I/O, outro pode usar a CPU, aumento a utilização dela
* **Conseguir multiprogramação**: dividir a memória em `n` partições (possivelmente desiguais)
  1. Partições fixas de memória com filas de entrada separados para cada uma
     * Coloca-se um processo na fila de entrada da menor partição capaz de armazenar
     * Com partições fixas, espaço não utilizado é desperdiçado
     * **Desvantagem**: filas de entrada vazias para partições grandes e cheias para partições pequenas. Tarefas pequenas esperam para entrar na memória mesmo com memória suficiente livre
  2. Partições fixas de memória com única fila de entrada
     * Quando a partição é liberada procura-se por toda a fila de entrada a maior tarefa que cabe nesta partição
       * Discrimina contra tarefas pequenas. Solução é ter pelo menos uma partição pequena ou definir que uma tarefa não pode ser pulada mais de `k` vezes

## 4.1.3 - Realocação e proteção

### Realocação

* Ligador deve saber o endereço que o programa começa na memória
* **Exemplo**: primeira instrução chama procedimento no endereço absoluto 100
  * Se o programa estiver no endereço 100K, pulará para memória dentro do SO (endereço 100), precisa-se acessar 100K + 100 na verdade
* **Solução**: modificar instruções conforme os programas são carregados na memória
  * Ligador inclui no binário um bitmap explicitando quais palavras do programa são endereços a serem realocados e quais são outros componentes que não devem ser

### Proteção

* Ao usar endereços absolutos não há como evitar que um programa manipule qualquer palavra na memória
* **Solução IBM**: dividir a memória em blocos de 2KB com um código de 4 bits de proteção e uma chave PSW (Program Status Word) também de 4 bits
  * Hardware impede que um processo acesse memória que o código de proteção seja diferente da chave PSW
  * Apenas o SO pode alterar as chaves e códigos

### Solução alternativa: `base` e `limit`

* Registradores
* **`base`**: registrador com o endereço do início da partição
* **`limit`**: comprimento da partição
* Todo endereço gerado tem o valor do registrador base adicionado: `CALL 100` vira `CALL 100K + 100` sem modificar a instrução em si
* Endereços são checados em relação a `limit` também
* **Desvantagem**: adição e comparação extra a cada referência à memória