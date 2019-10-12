# 2.6 - Implementação Processos MINIX 3



## 2.6.10 - Escalonamento MINIX 3

* Algoritmo de escalonamento multinível
* Prioridades iniciais aos processos que podem ser trocadas durante execução
  * Maior prioridade da camada 01 à 04
  * Dentro de cada camada as prioridades podem ser diferentes
  * Todos processos de usuário inicialmente iguais, comando `nice` pode aumentar ou diminuir

### Fila de processos na inicialização

* No momento do término da inicialização e começo da execução (linha 7252 em `kernel/main.c`)
* Array `rdy_head`: uma entrada por fila apontando para o processo encabeçando ela
* Array `rdy_tail`: uma entrada por fila apontando para o último processo de cada fila
* Arrays definidos com o macro `EXTERN` (`kernel/proc.h`: 5595 e 5596)
  * `EXTERN` é macro para `extern`: evita que variáveis declaradas em `.h` sejam armazenadas várias vezes (uma vez para cada ver que o arquivo for linkado)
* A fila inicial é definida durante a inicialização pela tabela `image` em `table.c` (6095 a 6109)
  * Tabela que lista todos programas que pertencem à imagem de boot
* 16 filas, uma por nível de prioridade
  * ![](zzz-001-02-06-priority-queue.png)

* Escalonamento por rodízio: após consumir o quantum, o processo vai para o fim da fila e recebe novo quantum
* Quando um processo bloqueado acorda ele é colocado na cabeça da fila se tinha quantum restante
* Processos bloqueados são removidos da fila, onde ficam só os processos executáveis
* Algoritmo: encontrar a fila de maior prioridade (sendo a maior, zero) não vazia e escolher o processo na cabeça
* Processo `IDLE` sempre pronto e na fila de menor prioridade, se todas outras estão vazias, ele executa

### Funções

* `enqueue`: chamada com um ponteiro para uma tabela de processo como argumento (linha 7787)
  * Chama `sched` para determinar em qual fila o processo deve ser colocado e se na cabeça ou cauda da fila
  * Insere de acordo com uma lista ligada com cabeça e cauda
  * `pick_proc` é camada para determinar o próximo processo a executar
* `dequeue`: chamada com um ponteiro para tabela de processo como argumento (linha 7823)
  * Processo precisa estar rodando para bloquear
    * Provavelmente na cabeça da fila
  * Se um sinal foi enviado para matar processo que não executava, a fila deve ser percorrida para encontrá-lo
    * Ajustar todos ponteiros de acordo
  * Se o processo estava em execução, chamar `pick_proc` para executar um novo
  * No começo, testar se o processo removido opera no espaço do kernel (checando a integridade das áreas de pilha compartilhadas entre tarefas que rodam no kernel, pois compartilham a mesma área de pilha definida por hardware)
    * Em caso positivo checar se o padrão no final da área da pilha do processo não foi sobreescrito (7835 a 7838)
* `sched`: seleciona em qual fila colocar um processo e se colocar no começo ou final
  * Tabela de processos: quantum, tempo restante de quantum, prioridade e prioridade máxima
  * De 7880 a 7885 checa se o quantum foi usado inteiramente
    * Caso não, reinicia com o que faltava
    * Se foi usado checa se rodou duas vezes seguidas
      * Caso sim, recebe penalidade de `+1` na prioridade para evitar loop infinito
      * Se outro processo rodou, penalidade `-1`
  * Dois processo alternando em loop juntos é um problema em aberto
  * Todos processos começam com sua prioridade máxima, portanto penalidades negativas não alteram em nada até se tornarem positivas
  * Limite inferior é que processos nunca podem ser colocados na fila de `IDLE`
* `pick_proc` (linha 7910): configura `next_ptr`
  * Sempre que um processo bloqueia, `pick_proc` é chamado para reescalonar a CPU
  * Testa cada fila. Se um processo na primeira fila está pronto configura `proc_ptr` e já retorna, senão vai checando cada fila até `IDLE_Q`
  * Ponteiro `bill_ptr` é alterado para rastrear o tempo de CPU dado a um processo na linha 7694

### Procedimentos restantes em `proc.c`

* `lock_send`, `lock_enqueue` e `lock_dequeue`
  * Dão acesso às funções básicas usando `lock` e `unlock` da mesma maneira que `lock_notify`

### Resumo

* Múltiplas filas de prioridade
* Primeiro processo na fila de maior prioridade sempre roda a seguir
* Clock task monitra o tempo usado pelos processos
* Se usa todo o quantum é colocado no final de sua fila
* Task, drivers e servidores são esperados a rodar até bloquear, recebendo um grande quantum, mas se demorarem pode ser preemptados
  * Previne que processos de alta prioridade travem o sistema
  * Um processo que previne outros de serem rodados podem ser movidos para uma fila de menor prioridade temporariamente

### 2.6.11 - Suporte de Kernel dependente do hardware