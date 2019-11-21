# 4.4 - Algoritmos de Substituição de Página

* **Page fault**: SO escolhe uma página para remover abrindo espaço para a nova página
  * Página removida modificada -> reescrever no disco
* Performance do algoritmo de substituição é melhor se manter as páginas mais usadas na memória, evitando substituições frequentes

## 4.4.1 - Algoritmo ótimo de substituição de página

* Fácil, porém impossível de implementar
* Remove a instrução que demorará mais tempo a ser usada
* SO não tem como saber isso com certeza
* Pode ser implementado na segunda execução após armazenar informações obtidas durante a primeira execução
  * Não útil em sistemas reais pois os parâmetros medidos se referem apenas a um programa específico e ainda por cima um input específico

## 4.4.2 - Algoritmo de substituição de página não usada recentemente

* Bits de status para coletar estatísticas de uso sobre as páginas
  * **R**: quando a página é referenciada (escrita ou leitura)
  * **M**: quando a página é modificada (escrita)
* Contidos em cada entrada da tabela de páginas
* Precisam ser atualizados a cada referência da memória, portanto são setados pelo hardware. Quando se tornam `1`, ficam assim até o SO resetar para `0` via software
* Se os bits não existem em hardware podem ser simulados:
  1. Processo inicia -> todas entradas na tabela de página são marcadas como fora da memória
  2. Página referenciada -> page fault
  3. SO seta o bit `R` (nas suas tabelas internas), muda a entrada da tabela de páginas para a página correta com modo de **apenas leitura** e reinicia instrução
  4. Se mais tarde tentar escrever na página, ocorre page fault novamente e o SO seta o bit `M` mudando o modo para **leitura/escrita**
* **Algoritmo simples**
  * Processo inicia, SO seta ambos bits para zero
  * Periodicamente (a cada interrupção do clock, por exemplo), o bit `R` é limpado para distinguir as páginas referenciadas recentemente
  * Quando ocorre page fault, SO divide todas páginas em 4 categorias de acordo com seus bits `R` e `M`
    1. **Classe 0**: sem referência e sem modificação
    2. **Classe 1**: sem referência e com modificação
       * Não é impossível: pode ocorrer quando uma página de classe 3 tem seu bit `R` limpo por interrupção de clock
    3. **Classe 2**: com referência e sem modificação
    4. **Classe 3**: com referência e com modificação
  * **Algoritmo NRU (Not recently used)**: remove uma página aleatória da menor classe
    * Fica implícito que é melhor remover uma página modificada que não referenciada dentro de um tique do relógio (cerca de 20ms) [classe 1] do que uma página limpa com uso pesado [classe 2]

## 4.4.3 - Algoritmo de substituição de página FIFO (First in, first out)

* SO mantém uma lista com todas as páginas na memória: a página na cabeça é a mais antiga e na cauda a mais recente
* **Page fault**: a página na cabeça é removida e a nova página adicionada à cauda
* Pode remover páginas importantes e muito usadas, deixando de ser eficiente, portanto FIFO é pouco usado

## 4.4.4 - Algoritmo de substituição de página de segunda chance

* Melhora FIFO para evitar jogar fora uma página muito usada
* **Algoritmo segunda chance**
  * Inspeciona o bit `R` da página mais antiga
    * Se zero: substitui pois a página é antiga e não usada
    * Se um: bit recebe zero e a página é colocada no final da lista, como se tivesse chegado agora. A busca continua por um bit R igual a zero

## 4.4.5 - Algoritmo de substituição de página do Relógio

* **Second chance** se torna ineficiente por mover páginas ao redor da lista
  * Melhor utilizar uma lista circular, com um ponteiro para a página mais antiga
* Quando ocorre **page fault**: se a página apontada (mais antiga) tem bit `R` zero, remove a página, senão zera R e avança o ponteiro

## 4.4.6 - Algoritmo de substituição de página LRU (Least recently used)

* Aproximação do algoritmo ótimo: páginas usadas nas últimas instruções serão provavelmente usadas novamente nas próximas
* **Paginação LRU**: quando ocorrer page fault, jogar fora a página que não tem sido usada há mais tempo
* Implementar lista ligada com todas páginas da memória, mais recentes na frente e menos recentes atrás
* **Dificuldade**: atualizar a lista a cada referência de memória
  * Encontrar a página na lista, deletar e mover para frente é custoso mesmo com hardware especializado
* **LRU com hardware especial**: contador de 64 bits, `C`, incrementado a cada instrução
  * Cada entrada na tabela de páginas precisa ter um campo grande o suficiente para esse contador
  * A cada referência de memória, o valor de `C` é salvo na entrada da tabela de páginas
  * Quando ocorre page fault, o SO examina os contadores de todas entradas e encontra o que tem menor valor (LRU)
* **Segunda implementação de LRU**
  * Matrix `n` por `n`, zerada (máquina com `n` page frames)
  * Quando o page frame `k` é referenciado, o hardware faz toda fileira `k` valer 1 e toda coluna `k` valer zero (a intersecção entre fileira e coluna fica com valor zero, pois é setado depois)
  * A fileira com menor valor binário é a menos recentemente usada, a ser removida

## 4.4.7 - Simulação de LRU em software

* Algoritmo **NFU (not frequently used)**
  * Contador em software associado a cada página (iniciado com zero)
  * A cada interrupção de relógio, cada página tem somada ao seu contador o valor do bit `R`
  * Quando ocorre **page fault**, remove a de menor contador
* **Problema**: NFU não esquece valores de passagens anteriores
  * Páginas muito usadas em uma passagem não serão removidas em passagens subsequentes, então o SO removerá páginas úteis
* **Técnica de envelhecimento (aging)**
  * Permite NFU simular LRU bem
  * Antes de somar `R`, os contadores sofrem um shift de 1 bit a direita, e `R` é somado no bit mais a esquerda
  * Quando ocorre page fault, a página com menor contador é removida
* **Diferenças com LRU**
  * Na figura 4.17e, páginas 03 e 05 não foram referidas por dois tiques, mas ambas foram referidas antes. Porém não temos como saber qual foi referida por último de fato, apenas que foram referenciadas entre o tique 01 e 02. Ao tomar apenas um bit por intervalo, não é possível distinguir referências no começo do intervalo de clock daquelas acontecendo no final
  * Outra diferença é que no raging, os contadores têm um número finito de bits (no exemplo são 8). Se duas páginas tiverem contador zero, uma deve ser escolhida de forma aleatória mesmo que uma tenha sido referenciada há 9 tiques e a outra há 1000 tiques
    * Porém com 8 bits e um tique de relógio de 20 ms, se uma página não foi referenciada em 160 ms provavelmente não é importante e pode ser removida apesar disso