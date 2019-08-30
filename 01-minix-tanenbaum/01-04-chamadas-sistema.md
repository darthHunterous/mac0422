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