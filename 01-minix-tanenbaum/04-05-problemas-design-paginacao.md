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

