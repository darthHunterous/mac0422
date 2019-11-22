# 4.5 - Problemas de Design (projeto) para paginação

## 4.5.1 - Modelo do conjunto de trabalho

* **Paginação por demanda**: processos iniciados sem suas páginas na memória
  * Na primeira instrução já recebe page fault, com várias outras vindo em sequência. Depois de um tempo estabiliza e executa com poucos page faults
* **Localidade de referência**: durante qualquer fase de sua execução, um processo faz referência a uma pequena fração de suas páginas
* **Conjunto de trabalho (working set)**: conjunto de páginas que o processo está usando no momento
  * Se o conjunto de trabalho inteiro está na memória, o processo roda sem muitas page faults até a próxima fase de execução
  * Se a memória disponível não é suficiene para todo o conjunto de trabalho, o processo causará várias page faults e executará lentamente
    * **Ultrapaginação (thrasing)**: programa que causa page faults a cada poucas instruções
* **Modelo do conjunto de trabalho**: tentar definir qual é o conjunto de trabalho do processo e garantir esteja na memória antes da execução
  * Em sistemas multiprogramados, processos são movidos para o disco e suas páginas removidas da memória, porém quando ele volta a execução mais tarde, sofre várias page faults até se estabilizar novamente
  * **Pré-paginação**:  carregar páginas antes de deixar o processo executar
    * O conjunto de trabalho varia lentamente com o tempo, portanto pode-se fazer uma estimativa razoável de quais páginas serão necessárias ao reiniciar o processo
  * Uma forma de implementar o modelo do conjunto de trabalho é monitorar quais páginas estão no conjunto de trabalho. Pode ser feito com o algoritmo de envelhecimento, obtendo um `n` experimental tal que qualquer página que contenha um bit `1` dentre seus `n` primeiros bits é considerado parte do conjunto
* **Algoritmo wsclock (working set clock)**: algoritmo do relógio porém se o ponteiro aponta para uma página de bit R igual a zero mas que pertença ao conjunto de trabalho, ela é poupada de ser removida da memória

## 4.5.2 - Políticas de alocação: Local versus global

* Algoritmo de substituição de página **local**: remove a página menos recentemente usada apenas dentro do processo que gerou a page fault
  * Corresponde a alocar para cada processo uma fração fixa da memória
* Algoritmo de substituição de página **global**: aloca page frames dinâmicos entre os processos executáveis
  * Número de page frames por processo varia
* Globais funcionam melhor especialmente quando o conjunto de trabalho pode variar com o tempo
  * Se usando algoritmo local e o conjunto de trabalho cresce, ocorre ultrapaginação mesmo com vários page frames livres. Se o conjunto diminui, desperdiça memória
* Pode-se ter um algoritmo para alocar page frames a processos
  * Uma opção é dividir igualmente entre todos processos (exemplo: 12416 page frames para 10 processos, 1241 para cada sobrando 6 para lidar com page faults)
  * Outra opção é alocar proporcionalmente ao tamanho de cada processo (um de 300 KB recebe 30 vezes mais que um de 10 KB)
    * Garantindo que cada processo receba também um número mínimo aceitável para que a execução seja possível (uma instrução com dois operandos precisa de seis páginas pelo menos)
* Com um algoritmo global pode-se inicializar cada processo com uma quantidade de páginas proporcional ao tamanho do processo mas a alocação precisa ser atualizada dinamicamente
  * Algoritmo **PFF (Page fault frequency)**: diz quando aumentar ou diminuir a alocação de página de um processo, mas não qual página substituir em uma page fault
* **PFF** assume que a taxa de faults diminui quanto mais page frames são atribuídos
  * Tenta manter a taxa de page faults dentre limites aceitáveis: uma taxa muito alta implica que o processo precisa de mais page frames para funcionar melhor e uma taxa muito baixa indica que o processo ocupa memória demais
  * Se a taxa muito alta não puder ser diminuída, alguns processos são removidos da memória (**controle de carga**) de forma a liberar mais page frames para page faults subsequentes
    * Mostra que mesmo com paginação, swap ainda é necessário para reduzir a demanda por memória

## 4.5.3 - Tamanho de página

### Dois motivos para um tamanho pequeno de página

1. Em média, metade da página final fica vazia para um segmento de texto, dados ou pilha, desperdiçando espaço (**fragmentação interna**)
   * Desperdício de `n * p / 2` bytes com `n` segmentos na memória e tamanho de página de `p` bytes
2. Uma página maior faz com que uma porção maior de programa não usada esteja na memória do que com um tamanho de página menor
   * Exemplo de programa com 8 fases sequenciais de 4 KB. Com 32 KB de tamanho de página, o programa ocupará 32 KB a todo instante, com 16 KB de página, idem. Mas com uma página de 4 KB ou menor ocupará apenas 4 KB a qualquer instnate

### Desvantagem tamanho pequeno de página

* Páginas pequenas implicam que os programas precisarão de várias páginas, resultando numa grande tabela de páginas
* Transferência ao disco são uma página por vez e a maior parte do tempo é gasto na busca (seek) e no atraso rotacional (rotational delay), logo a transferência de uma página pequena ou de uma grande leva praticamente o mesmo tempo
  * Exemplo: leva `64 * 10ms` para carregar 64 páginas de 512 bytes mas apenas `4 * 10.1ms` para carregar 4 páginas de 8 KB
* O espaço ocupado pela tabela de páginas aumenta quando o tamanho da página diminui
  * Suposições
    * Tamanho médio de procesos: `s` bytes
    * Tamanho da página: `p` bytes
    * Cada entrada de página precisa de `e` bytes
  * Consequências
    * Número de páginas por processo: `s/p` bytes
    * Espaço ocupado pela tabela é `s * e / p` bytes
    * Memória gasta na última página do processo é `p/2`
  * Overhead
    * `overhead == s * e / p +  p / 2`
      * O tamanho da tabela é grande quando o tamanho da página é pequeno
      * A fragmentação interna é grande quando o tamanho da página é grande
    * Ponto ótimo em algum lugar no meio, tomando derivada em relação a `p` e igualando a zero
      * `-s * e / p**2 + 1/2 == 0`
      * Segue o tamanho ótimo da página dado por `p == sqrt(2 * s * e)`
        * Com `s == 1 MB` e `e == 8 bytes` o tamanho ótimo da página é 4 KB
  * Com maiores tamanhos de memória, a tendência é maiores tamanho de página

## 4.5.4 - Interface de memória virtual

* Em alguns sistemas programadores podem ter controle sobre o mapa de memória para que dois ou mais processos possam compartilhar a mesma memória: compartilhamento com alta largura de banda, um processo escreve na memória e o outro lê
* Permite sistema de passagem de mensagem de alta performance
  * Comumente ao passar mensagens os dados são copiados de um espaço de endereço para outro, de forma custosa
  * Se um processo controla seu mapeamento de páginas, o processo que envia pode desmapear a página que contém a mensagem e enviar o nome da página ao remetente, que mapeia essa página, evitando ter que transmitir os dados em si
* **Memória compartilhada distribuída**: vários processos em rede compartilhando um conjunto de páginas
  * Ao referenciar uma página não mapeada, o processo recebe page fault. O gerenciador de page faults no kernel ou espaço do usuário localiza em qual máquina está a página faltando e envia mensagem pedindo para ser desmapeada e enviada através da rede. Quando chega é mapeada e a instrução faltosa reiniciada