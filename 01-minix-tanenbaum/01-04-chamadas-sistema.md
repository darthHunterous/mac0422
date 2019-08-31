# 1.4 - Chamadas de Sistema

* Computadores com uma CPU só podem executar uma instrução por vez, portanto um processo precisa ser interrompido para executar uma chamada de sistema, transferindo o controle ao SO

* Programas devem checar os resultados de uma chamada de sistema caso tenham ocorrido erros
  * `count = read(fd, buffer, nbytes);`
    * `fd` é o arquivo a ser lido
    * `nbytes` é o número de bytes a ser lido
    * A chamada retorna o número de bytes efetivamente lido ou `-1` em caso de erro, colocando o número do erro na variável global `errno`
* MINIX 3 conta com **53** chamadas de sistema divididas em **6** categorias

### 1.4.1 - Chamadas de Gerenciamento de Processos

* `pid = fork()`: única maneira de criar processos em MINIX 3

  * Cria um processo filho idêntico ao pai (duplicata), após a criação passam a ser independentes
  * Todas variáveis idênticas na criação mas mudanças subsequentes não afetam um ao outro
  * Retorna **0** no processo filho e o valor do identificador processo filho no pai (**PID**)

* `pid = waitpid(pid, &statloc, opts)`: aguarda que os processos filhos terminem

  * Pode aguardar um filho específico ou por qualquer filho através do parâmetro `-1`
  * Ao completar a espera, `statloc` aponta para o status de saída do filho
  * Opções como terceiro parâmetro
  * Forma atualizada da chamada `wait`

* `s = wait(&status)`: versão antiga de `waitpid`

* Ao digitar um comando na shell, ela faz um fork de um novo processo que executa tal comando com a chamada `execve`

  * `s = execve(name, argv, envp)`: substitui a imagem do núcleo de um processo
    * Primeiro parâmetro: nome do arquivo a ser executado
    * Segundo: ponteiro para o array com argumentos
    * Terceiro: ponteiro para o array do ambiente

* **Exemplo de uma shell simplificada**

  * ```c
    #define TRUE 1
    
    while(TRUE) {  /* executar para sempre */
        type_prompt();  /* mostrar prompt na tela */
        read_command(command, parameters);  /* le input do terminal */
        
        if(fork() != 0) {  /* fork do processo filho */
        	/* Código do pai */
            waitpid(-1, &status, 0);  /* aguarda a saída do filho */
        }
        else {
            /* Código do filho */
            execve(command, parameters, 0);  /* executar comando */
        }
    }
    ```

* A main de um programa contém `main(argc, argv, envp)`
  * Exemplo de `cp file01 file02`
    * `argc` é a contagem de itens incluindo o nome do programa (`3`)
    * `argv` é o ponteiro para o array de strings da linha de comando
      * `argv[0]` aponta para `cp`
    * `envp` ponteiro para o ambiente,  array de strings da forma `name=value`, passando o tipo de terminal e nome do diretório base do programa
* `exit(status)`: encerra a execução do processo e retorna o status
  * `status` vai de 0 a 255 e é retornado ao pai via `statloc` na chamada `waitpid`
    * O byte menor de `status` contém o status do término (zero sendo normal)
    * O byte maior contém o status de saída do filho
* Processos divididos em **três segmentos**
  * **Segmento de texto**: código do programa
  * **Segmento de dados**: variáveis
  * **Segmento da pilha**
  * Os dados crescem para cima e a pilha cresce para baixo. Entre eles há um vão de espaço não usado no qual a pilha cresce automaticamente
    * Os valores de endereçamento crescem de baixo para cima
* `size = brk(addr)`: configura onde termina o segmento de dados (valor de endereço maior que o atual indica crescimento e vice-versa). Tal parâmetro precisa ser menor que o stack pointer, senão os dois segmentos iriam se cruzar, o que é proibido
  * rotina `sbrk` recebe como parâmetro a variação que o segmento de dados deve sofrer
* `pid = getpid()`: retorna o id do processo que a invocou
* `pid = getpgrp()`: retorna o id do grupo do processo que invocou
* `pid = setsid()`: cria uma nova sessão e retorna o PID do grupo  do processo
  * Sessões são relacionados ao **job control** do POSIX, não presentes em MINIX 3
* `l = ptrace(req, pid, addr, data)`: usado no debug de um processo, lendo a memória dele e controlando



### 1.4.2 - Chamadas de Sinais

* Quando um processo não preparado recebe um sinal, ele é encerrado diretamente
  * Para anunciar que está preparado para receber algum sinal: `s = sigaction(sig, &act, &oldact)`
    * Define a ação ao receber sinais, endereço do procedimento para lidar com o sinal  e um local para armazenar o atual procedimento
    * Ao receber o sinal, o estado do processo é armazenado em sua própria pilha e então o procedimento para lidar com o sinal é chamado. Ao término deste, chama-se `s = sigreturn(&context)` para continuar de onde parou
    * `sigaction` substitui a chamada `signal` mantida como procedimento de uma biblioteca para compatibilidade

* Sinais podem ser bloqueados e nesse caso ficam segurados como pendentes até serem desbloqueados
  * `sigprocmask` permite um processo definir um conjunto de sinais bloqueados ao apresentar um bitmap para o kernel
  * `s = sigprocmask(how, &set, &old)`: examinar ou alterar a máscara do sinal
* `s = sigpending(set)`: retorna o conjunto de sinais bloqueados (pendentes) como bitmap
* `s = sigsuspend(sigmask)`: troca a máscara do sinal e suspende o processo
* Constantes `SIG_IGN` para todos sinais subsequentes de um dado tipo serem ignoraros e `SIG_DFL` parar restaurar a ação padrão do sinal (matar ou ignorar)
  * Para evitar que `SIGINT` (`CTRL+C`) mate um processo em background na shell, ela executa `sigaction(SIGINT, SIG_IGN, NULL);` e o mesmo para `SIGQUIT`(`CTRL+\`)
* `s = kill(pid, sig)`: envia um sinal para um processo
  * `SIGKILL`, sinal 9, enviado para um processo em background não pode ser capturado ou ignorado
* `residual = alarm(seconds)`: configura o temporizador para o qual quando expirado o processo recebe um `SIGALRM`
* `s = pause()`: suspende o processo que chama até receber o próximo sinal, evitando gastar tempo de CPU à toa



### 1.4.3 - Chamadas para Gerenciamento de Arquivos

$$
x^2
$$

* Teste de equação, deletar