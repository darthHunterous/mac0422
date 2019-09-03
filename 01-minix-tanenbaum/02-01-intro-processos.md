# Capítulo 02 - Processos

* **Processo**: abstração de um programa em execução

  

## 2.1 - Introdução a Processos

* Na multiprogramação, a CPU troca de programa a cada algumas centenas de milisegundos
* A cada instante, a CPU só executa um programa, mas ao longo de 1 segundo, pode trabalhar em vários programas, dando aos usuários a ilusão de paralelismo (**pseudoparalelismo**)
* Paralelismo verdadeiro vem de sistemas **multiprocessadores** (duas ou mais CPUs compartilhando a mesma memória física)



### 2.1.1 - Modelo de Processo

* Todo software executável em um computador é organizado em um número de **processos sequenciais** (**processos**)
* **Multiprogramação**: rápida troca entre processos feita pela CPU, pseudoparalelismo
* Existe apenas um contador de programa físico, porém cada processo armazena a informação do seu respectivo contador de programa antes de ser trocado por outro processo
* Analogia do confeiteiro e do bolo para entender a distinção entre processo e programa: a receita do bolo é o programa (algoritmo), o confeiteiro é o processador e os ingredientes do bolo são a entrada. O processo consiste na atividade de ler a receita, pegar os ingredientes e fazer o bolo.
  * Porém se o filho do confeiteiro chega chorando, necessitando um curativo, o confeiteiro irá interromper o processo de fazer o bolo, lembrando onde ele parou (por exemplo, lendo qual parte da receita), pegará a caixa de primeiros socorros, um livro sobre curativos, terminará o curativo de seu filho (que tem maior prioridade) e depois retornará exatamente ao ponto em que parou na confecção do bolo
* Um único processador pode ser compartilhado entre vários processos usando um algoritmo de escalonamento para determinar quando deve-se interromper um processo



### 2.1.2 - Criação de Processo

* Eventos que causam criação de processos
  * Inicialização do sistema
  * Execução da chamada de criação de processo por um em execução
  * Requisição de usuário por um novo processo
  * Início de tarefa em lote
* **Daemons**: processos que ficam em segundo plano para lidar com alguma atividade
* `ps`: no MINIX lista os processos em execução
* Única chamada para criação de processos é o `fork`
  * Clone do processo que invoca, com a mesma imagem de memória. Processo filho deve executat `execve` para executar um novo programa
  * Porém, pai e filho tem espaços de memórias distintos, alterações em um não alteram o outro



### 2.1.3 - Término de Processo

* Condições de término
  * Saída normal (voluntária)
    * Termina por executar sua tarefa, chamada de `exit`
  * Saída por erro (voluntária)
    * Exemplo de passar um arquivo que não existe para um compilador
  * Erro fatal (involuntário)
    * Bug no programa: divisão por zero, referência a endereço que não existe
  * Morto por outro processo (involuntário)
    * Chamada de `kill`



### 2.1.4 - Hierarquia de Processos

* Grupo de processos: um processo, seus filhos e demais descendentes
* Dois processo especiais na inicialização de MINIX
  * **Servidor de reencarnação**: (re)inicia drivers e servidores
  * **Init**: executa o script em `/etc/rc` que gera comandos para o servidor de reencarnação iniciar drivers e servidores não presentes na imagem de boot. Dessa maneira, tais drivers e servidores tornam-se filhos do servidor de reencarnação, então se eles forem encerrados, o servidor de reencarnação será informado e irá reiniciá-los
    * Permite que MINIX 3 tolere falha de driver ou servidor, porque um novo será iniciado automaticamente
    * Após a execução de `/etc/rc`, `init` lê o arquivo de configuração `/etc/ttytab` para checar terminais existentes, fazendo `fork` do processo `getty` para cada um, mostrando prompt de login e aguardando entrada
    * Ao digitar um nome, `getty` dá `exec` em um processo de login com o mesmo nome como argumento
      * Sucedendo o login, `login` dá `exec` na shell do usuário, que se torna filha de `init`
    * Exemplo acima de como árvores de processos são criadas



### 2.1.5 - Estados de Processo

