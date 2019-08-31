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

* `fd = creat(name, mode)`: jeito obsoleto de criar arquivos

  * Recebe o nome do arquivo e o modo de proteção em octal
    * `0751`: zero indica octal em C, 7 `rwx` para o dono, 5 `r-x` para o grupo e 1 `--x` para outros
  * Cria o arquivo e o abre para escrita
  * Se usado num arquivo existente, trunca para comprimento zero
  * Obsoleto, `open` pode criar novos arquivos

* `fd = mknod(name, mode, addr)`: cria um i-node normal, especial ou de diretório

  * `fd = mknod("/dev/ttyc2", 020744, 0x0402)`
    * Cria arquivo para o console 2, modo 020744 (`rwxr--r--`), com major device no byte maior (04) e minor device no byte menor (02)
  * Só funciona no modo superuser

* `fd = open(file, how, ...)`: abre um arquivo para alterações ou leitura

  * Especifica nome do arquivo a ser aberto e o modo: `O_RDONLY`, `O_WRONLY`, `O_RDWR`

* `s = close(fd)`: fecha um arquivo

* `n = read(fd, buffer, nbytes)`: escreve dados de um arquivo no buffer

* `n = write(fd, buffer, nbytes)`: escreve dados do buffer no arquivo

* `pos = lseek(fd, offset, whence)`: move o ponteiro que indica a posição atual no arquivo

  * Parâmetros: descritor do arquivo, posição no arquivo e de onde contar: começo, atual ou final do arquivo
  * Retorna a posição absoluta no arquivo após a alteração

* `s = stat(name, &buf)`: informações sobre o arquivo

  * `&buf`: ponteiro para uma estrutura onde se colocar a informação

  * ```c
    struc stat {
        short st_dev;  /* device onde i-node pertence */
        unsigned short st_ino;  /* número do i-node */
        unsigned short st_mode;  /* modo da palavra */
        short st_nlink;  /* número de links */
        short st_uid;  /* id do usuário */
        short st_gid;  /* id do grupo */
        short st_rdev;  /* major e minor device para arquivos especiais */
        long st_size;  /* tamanho do arquivo */
        long st_atime;  /* horário do último acesso */
        long st_mtime;  /* horário da última modificação */
        long st_ctime;  /* horário da última alteração ao i-node */
    }
    ```

  * 

* `s = fstat(fd, &buf)`: informações sobre o arquivo

  * `&buf`: ponteiro para uma estrutura onde se colocar a informação

* `fd = dup(fd)`: aloca um novo descritor de arquivo

  * `dup2(fd, fd2);` faz `fd2` se referir a `fd`
  * Sempre retorna o menor descrito de arquivos disponível

* `s = pipe(&fd[0])`: cria um pipe

  * Retorna dois `fd`, um para leitura (`fd[0]`) e outro para escrita (`fd[1]`)

  * Geralmente, um `fork` a seguir e o pai e filho fecham os descritores de escrita e leitura deles para que possa se escrever e ler do pipe

  * ```c
    #define STD_INPUT 0  /* descritor de arquivo para entrada padrão */
    #define STD_OUTPUT 1 /* descritor de arquivo para saída padrão */
    
    char *process1, *process2;  /* ponteiros para o nome dos programas */
    pipeline(process1, process2) {
        int fd[2];
        
        pipe(&fd[0]);  /* cria o pipe */
        if (fork() != 0) {
            /* Código do processo pai */
            close(fd[0]);  /* processo 1 não precisa ler do pipe */
            close(STD_OUTPUT);  /* prepara para uma nova saída padrão */
            dup(fd[1]);  /* configura a saída padrão para fd[1], escrita do pipe */
            close(fd[1]);  /* descritor não mais necessário */
            execl(process1, process1, 0);
            
        }
        else {
            /* Código do processo filho */
            close(fd[1]);  /* processo 2 não precisa escrever no pipe */
            close(STD_INPUT);  /* prepara para uma nova entrada padrão */
            dup(fd[0]);  /* configura a entrada padrão para fd[0], leitura do pipe */
            close(fd[0]);  /* descritor não mais necessário */
            execl(process2, process2, 0);
        }
    }
     
    ```

* `s = ioctl(fd, request, argp)`: executa operações especiais em um arquivo

  * Funções de biblioteca `tcgetattr` e `tcsetattr` usam `ioctl` para mudar caracteres para corrigir erros de digitação no terminal, mudar o modo do terminal, etc
  * **Modos do terminal**: cooked, raw, cbreak
    * **Modo cooked (processado)**: modo normal, caracteres de apagamento e eliminação funcionam normalmente
      * `CTRL-S` e `CTRL-Q` param e iniciam o output do terminal
      * `CTRL-D` é o fim do arquivo
      * `CTRL-C` gera sinal de interrupção
      * `CTRL-\` gera sinal de saída forçando  um core dump
      * Em POSIX, **modo canônico**
    * **Modo bruto (raw)**: todas funções desabilitadas, portanto cada caracter é passado ao programa sem processamento especial
    * **Modo cbreak**: meio termo. apagamento, eliminação de caracteres e CTRL-D desabilitados, mas o resto funciona
  * Chamada para `tcsetattr` para configurar parâmetros no terminal
    * `ioctl(fd, TCSETS, &termios)`
      * Parâmetros: arquivo, segundo como operação e terceiro, estrutura POSIX com flags e array com caracteres de controle

* `s = access(name, amode)`: checa se um arquivo é acessável

* `s = rename(old, new)`: novo nome para um arquivo

* `s = fcntl(fd, cmd, ...)`: travar arquivo e outras operações



### 1.4.4 - Chamadas para Gerenciamento de Arquivo

* `s = mkdir(name, mode)`: cria novo diretório
* `s = rmdir(name)`: remove um diretório vazio
* `s = link(name1, name2)`: cria uma nova entrada `name2` apontando para `name1`
  * Cada arquivo em UNIX tem um número exclusivo (i-number), índice numa tabela de **i-nodes**, com informações sobre esse arquivo
  * Um diretório é um arquivo contendo um conjunto de pares (i-number, nome ASCII)
* `s = unlink(name)`: remove uma entrada do diretório
  * Se nenhuma entrada aponta para um determinado arquivo (campo no i-node indicando a quantidade de entrada em um diretório que aponta para um dado arquivo), o arquivo é removido do disco

* `s = mount(special, name, flag)`: monta um sistema de arquivos
  * Exemplo de CD-ROM: `mount("/dev/cdrom0", "/mnt", 0)`
    * Parâmetros: arquivo especial de bloco para o CD-ROM, segundo é o lugar na árvore a ser montado e o terceiro indica leitura e escrit ou apenas leitura
  * Porções de HD também podem ser montadas (**partitions** ou **minor devices**)
* `s = unmount(special)`: desmonta sistema de arquivos
* `s = sync()`: transfere os **blocos do cache** para o disco
  * MINIX 3 mantém um **cache de blocos** recentemente usados para evitar ficar lendo do disco, porém se o sistema falar antes do bloco ser escrito para o disco, o sistema de arquivos é danificado
  * Programa `update` do MINIX efetua um `sync` a cada 30 segundos
* `s = chdir(dirname)`: muda o diretório atual
* `s = chroot(dirname)`: muda o diretório raiz
  * Apenas superusers
  * **FTP** e **HTTP** usam para que usuários remotos só acessem porções do sistema de arquivos abaixo da nova raiz



### 1.4.5 - Chamadas para Proteção

* 11 bits para proteção de arquivo: 9 são o **rwx** e os outros 2 são `SETGID` e `SETUID`
  * `SETGID`: `02000`
  * `SETUID`: `04000`
    * Quando setado, o ID de um usuário que executa certo programa se torna o ID do dono deste arquivo enquanto durar a execução
    * Usado para permite que usuários executam programas que façam uso de funções de superusuário, como criar diretórios
      * Exemplo: se `mkdir` recebe modo `04755` (apenas dono do arquivo consegue executar, que é o superusuário), usuários comuns tornam-se capazes de criar diretórios, porém de maneira restringida
* `s = chmod(name, mode)`: altera os bits de proteção de um arquivo
  * `chmod("file", 0644)`: apenas leitura para todos exceto o dono
* `uid = getuid()`: retorna o uid real do processo chamador
* `uid = geteuid()`: retorna o uid efetivo do processo chamador (pode ter sido alterado por bits como `SETUID`)
*  `gid = getgid()`: retorna o gid real do processo chamador
* `gid = getegid()`: retorna o gid efetivo do processo chamador
* Usuários comuns não podem alterar seu UID a não ser executando programas com `SETUID` configurado, porém o superusuário pode:
  * `s = setuid(uid)`: configura o uid do chamador
    * Tanto o efetivo quanto real
  * `s = setgid(gid)`: configura o gid do chamador
    * Tanto efetivo quanto real
* `s = chown(name, owner, group)`: altera o dono e grupo de um arquivo
  * Apenas superuser pode executar
* `oldmask = umask(complmode)`: altera a máscara do modo
  * Exemplo: `umask(022);`, depois se o modo de um arquivo for configurado para `0777`, na verdade estaremos setando o modo para `0755` devido à máscara
  * Usuário comum pode invocar essa função
  * Bit mask herdade por processos filhos, então a shell pode proteger que arquivos acidentalmente sejam criados com permissão de escrita



### 1.4.6 - Chamadas para Gerenciamento de Tempo

* `seconds = time(&seconds)`: tempo passado desde 01/01/1970
* `s = stime(tp)`: define o tempo passado desde 01/01/1970 (data atual)
* `s = utime(file, timep)`: define o último acesso de um arquivo
* `s = times(buffer)`: retorna o tempo de CPU usado pelo usuário e pelo sistema (gasto em syscalls)