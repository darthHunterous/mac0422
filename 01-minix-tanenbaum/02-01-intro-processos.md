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

* Necessidade de comunicação entre processos
  * Em um caso de pipe como `cat chapter1 chapter2 chapter3 | grep tree` pode ocorrer que o `grep` esteja pronto para executar mas que a input fornecida pelo output do `cat` ainda não esteja, portanto precisa ser **bloqueado**
* Três estados de um processo
  * Executando (usando a CPU no instante em questão)
  * Pronto (passível de execução, parado para execução de outro processo)
  * Bloqueado (indisponível até certo evento externo ocorrer)
* Transições entre estados
  * Executando para Bloqueado: processo bloqueia aguardando input
    * Ocorre quando o processo descobre que não consegue continuar
  * Executando para Pronto: escalonador escolhe outro process
  * Pronto para Executando: escolhido pelo escalonador
  * Bloqueado para disponível: input torna-se disponível
    * Se nenhum outro processo estiver rodando, pode-se transicionar direto para o estado de Execução
* O nível mais baixo do SO é o escalonador, com vários processos em cima. Todas interrupções e detalhes de execução do processo estão escondidos nele

### 2.1.6 - Implementação de Processos

* **Tabela de processos**: mantida pelo SO, array de estruturas. Uma entrada por processo (entradas são **blocos de controle de processo**)
  * Entradas contém informações sobre o estado do processo, o program counter, stack pinter, alocação de memória, estado dos arquivos abertos, informações de contabilidade e escalonamento, alarmes, sinais, e tudo mais que precisa ser salvo referente ao processo quando ele transiciona de *executando* para *pronto*, permitindo que seja reiniciado depois como se nunca tivesse parado
* Em MINIX 3, comunicação entre processos, gerenciamento de memória e gerenciamento de arquivos são gerenciados por módulos separados dentro do sistema, particionadno a tabela de processo, onde cada módulo mantém o campo necessários a ele
* **Campos da tabela de processos por módulo**
  * **Kernel** (campos ligados a esse capítulo)
    * Registradores
    * Program Counter
    * Palavra de status do programa
    * Stack pointer
    * Estado do processo
    * Prioridade de escalonamento atual
    * Prioridade de escalonamento
    * Tiques de escalonamento restantes
    * Tamanho do quantum
    * Tempo de CPU usado
    * Ponteiros da fila de mensagens
    * Bits de sinais pendentes
    * Bits de flags
    * Nome do processo
  * **Gerenciamento de processos**
    * Ponteiro para segmento de texto
    * Ponteiro para segmento de dados
    * Ponteiro para segmento bss
    * Status de saída
    * Status de sinal
    * ID do processo
    * Processo pai
    * Grupo do processo
    * Tempo de CPU dos filhos
    * UID real
    * UID efetivo
    * GID real
    * GID efetivo
    * Informação de arquivo para compartilhamento de texto
    * Mapas de bits de sinais
    * Bits de flag
    * Nome do processo
  * **Gerenciamento de arquivo**
    * Máscara UMASK
    * Diretório raiz
    * Diretório de trabalho
    * Descritores de arquivo
    * ID real
    * UID efetivo
    * GID real
    * GID efetivo
    * tty de controle
    * Área de salvamento para leitura e escrita
    * Parâmetros da chamada de sistema
    * Bits de flag
* **Tabela de descritores de interrupção**: estrutura de dados associada a cada classe de dispositivo de entrada ou saída
  * Cada entrada nessa tabela possui um **vetor de interrupção**: contém o endereço do procedimento de tratamento do serviço de interrupção
    * Um exemplo é se o processo de usuário 23 está executando e ocorre uma interrupção de disco. O program counter, palavra de status do programa e registradores necessários são empurrados para a stackk atual pelo hardware de interrupção. O computador então salta para o endereço especificado no vetor de interrupção de disco e daí em diante o resto é gerido por software
    * O procedimento de tratamento da interrupção salva todos registradores na entrada da tabela de processos referente ao processo atual, sendo que o número de processo atual e um ponteiro para sua entrada na tabela de processos já estã salvos em variáves globais para rápido acesso
    * Depois, a informação depositada pela interrupção é removida da pilha, e o ponteiro da pilha é configurado a uma pilha temporária usada pela rotina de tratamento de processos
    * Ações como salvar registradores e configurar o ponteiro da pilha não podem ser expressas em linguagens de alto nível, portanto são executadas por uma rotina em Assembly. Quando essa rotina se encerra, chamada uma rotina em C para lidar com o resto do trabalho da interrupção
* No MINIX 3, processos se comunicam por mensagens
  * Portanto, o próximo passo na interrupção é construir uma mensagem para o processo do disco, que estará bloqueado aguardando por essa mensagem
  * Tal mensagem informa que uma interrupção ocorreu, diferenciando de mensagens de processos de usuários que requisitem acesso à leitura de blocos do disco
  * O estado do processo de disco se altera de *bloqueado* para *pronto* e o escalonador é chamado
* No MINIX 3, processos podem ter prioridades diferentes, para melhor suprir dispositivos de entrada e saída que processos de usuários, como um exemplo
  * Após a invocação do escalonador quando o processo do disco torna-se pronto, se ele tem a maior prioridade, será executado. Se o processo inicialmente interrompido tem a mesma ou mais prioridade, o processo de disco irá aguardar
* Após tudo isso, finalmente, a rotina em Assembly carrega os registradores e o mapa da memória para o processo atual e o executa
* **Resumo de uma interrupção pelo nível mais baixo do SO**
  1. Hardware empilha o program counter
  2. Harware carrega o novo program counter a partir do vetor de interrupção
  3. Rotina assembly salva os registradores
  4. Rotina assembly configura a nova pilha
  5. O serviço de interrupção em C constrói e envia a mensagem
  6. Código de passagem de mensagem mark como o destinário aguardando a mensagem como pronto
  7. O escalonador decidade qual processo rodar a seguir
  8. A rotina em C retorna para o código em Assembly
  9. A rotina em Assembly inicia o novo processo a ser executado

### 2.1.7 - Threads

* Cada processo tem um espaço de endereçamento e uma única thread de controle. Em alguns casos pode ser desejável ter múltiplas threads de controle no mesmo espaço de endereçamento rodando quase em paralelo, como se fossem processos separados
  * A tais fluxos de controles chamam-se **threads** ou **processos leves**
* Um processo é uma forma de agrupar recursos relacionados juntos
  * Possui espaço de endereçamento contendo texto do programa e dados, como outros recursos (arquivos abertos, processos filhos, alarm, rotinas de tratamento de sinais, etc)
* Outro conceito de um processo é o de seu fluxo de controle (**thread**)
  * A thread possui um program counter que guarda a próxima instrução a ser executada, possui registradores para suas variáveis de trabalho atuais. Possui uma pilha, com o histórico de execução, com um frame para cada procedimento chamado não retornado ainda
* São conceitos diferentes embora uma thread precise ser executada em um processo
  * Procesos são usada para agrupar recursos
  * Threadas são entidades programadas para execução na CPU
* Threads adicionam ao modelo de processos a capacidade de várias execuções no mesmo ambiente de processo, de forma independente umas das outras
* Comparação entre 3 processos, cada qual com sua thread e 1 processo possuindo 3 threads
  * Embora em ambos tenhamos 3 threads, no primeiro caso cada uma opera em um espaço de endereçamento diferente enquanto que no segundo caso compartilham o mesmo espaço
* Exemplo de uso de várias threads é na requisição de várias images por um browser. Para cada imagem, o browser precisa requisitar ao servidor, configurando uma conexão e efetuando a requisição. Muito tempo se gasta para estabelecer e encerrar tais conexões, portanto com várias threads em um processo, várias imagens podem ser requisitidas ao mesmo tempo, acelerando a performance, pois o fator limitante é o tempo de estabelecimento de conexão e não a velocidade de transmissão
* Alguns dos campos na tabela de processos do MINIX na verdade são por threads, portanto faz-se necessária uma tabela de threads separada, com uma entrada para armazenar as informações de cada thread
  * **Informações sobre itens compartilhados por threads em um processo**
    * Espaço de endereçamento
    * Variáveis globais
    * Arquivos abertos
    * Processos filhos
    * Alarmes pendentes
    * Sinais e rotinas de tratamento de sinais
    * Informações de contabilização
  * **Campos de informações sobre itens privados a cada thread**
    * Program counter
      * Necessário por thread, pois cada uma pode ser suspensa e resumida assim como processos
    * Registradores
      * Necessários por thread pois quando uma é suspensa, seus registradores precisam ser salvos nesses campos da tabela
    * Pilha
    * Estado
* Threads como processos podem estar em *execução*, *pronta* ou *bloqueadas*
* Em alguns SOs, threads são gerenciadas no espaço do usuário e não pelo SO em si. Quando a thread vai ser bloqueada, ela escolhe seu sucessor antes de parar
  * Exemplo de **POSIX P-threads** e **Mach C-threads**
* Quando o SO gerencia as threads por procesos, cabe ao escalonador decidir qual thread será executada após uma ser bloqueada. Para isso o kernel possui uma tabela de threads análoga à tabela de processos
* Trocar threads é mais rápido quando elas são gerenciadas no espaço do usuário, pois evitam-se chamadas de sistema. Porém quando uma thread é bloqueada por aguardar entrada/saída ou tratamento de um erro, o kernel bloqueia todo o processo pois desconhece a existência das threads
* Threads desvinculadas do sistema causam uma série de problemas, como se um `fork` deve replicar as várias threads do pai em um filho. Caso não sejam replicadas, o processo pode não funcionar corretamente, mas sendo replicadas no filho, o que ocorreria se uma thread bloqueasse aguardando input do teclado? Tanto o pai quanto o filho recebem a entrada?
  * Se uma thread faz um chamada de sistema, o status da chamada é colocado na variável global `errno`, mas o que ocorre se outra thread fazer uma outra chamada antes que a primeira tenha sido capaz de ler o valor retornado?
  * Tais problemas indicam que introduzir threads em um sistema sem pensar em um redesenhamento dele irá causar muitos problemas