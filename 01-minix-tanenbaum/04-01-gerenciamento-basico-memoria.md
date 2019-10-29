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

