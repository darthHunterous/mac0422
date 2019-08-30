# 1.3 - Conceitos de Sistemas Operacionais

* **Chamadas de sistema**: interface entre o SO e os programas de usuário, fornecidas pelo SO
  * No MINIX 3 podem ser que lidam com processos ou com o sistema de arquivos

### 1.3.1 - Processos

* **Processo**: programa em execução
  * Cada processo tem seu **espaço de endereçamento**, lista de locais de memória em que o processo pode ler e escrever
  * Periodicamente, o SO para de executar um processo e começa a executar outro, salvando todas informações relevantes sobre o estado atual de execução
    * Cada processo salva as informações sobre ele em uma **tabela de processos**
  * Um processo suspenso consiste da sua **imagem do núcleo** (espaço de endereço) e da sua entrada na tabela de processos
  * Um exemplo de processo é o **interpretador de comandos** ou **shell**, que lê comandos do terminal. Se o usuário pedir a compilação de um programa, a shell cria um novo processo para executar o compilador. Quando a compilação termina, esse processo faz uma chamada de sistema para se encerrar.
  * **Processos filho**: criados por outro processo (estruturados em árvore)
    * **Comunicação interprocessos**: processos cooperando a uma tarefa precisam se comunicar e sincronizar atividades

### 1.3.2 - Arquivos

* Chamadas de sistema para abrir e fechar arquivos
* **Diretório**: agrupa arquivos juntos
  * Entradas no diretório podem ser arquivos ou diretório (origina a hierarquia do sistema de arquivos)
* Arquivos podem ser especificados a partir do seu **path name** em relação ao **diretório raiz**
  * Colocar `/` no começo do path indica valor absoluto, ou seja, iniciando da raiz. Sem a barra, estaríamos no referindo a diretórios a partir do atual
    * No Windows usa-se a contrabarra `\` como separador
* Todo processo tem um **diretório de trabalho**, a partir do qual se procuram paths não absolutos. Esse diretório pode ser alterado com  chamadas de sistema
* Proteção de arquivos e diretórios em 11 bits, 3 campos de 3 bits cada + 2 bits a serem discutidos depois:
  * Um campo para o dono, outro para o grupo do dono e o terceiro para todo o resto
  * Cada campo tem um bit para leitura, escrita e execução (**rwx bits**)
  * `rwxr-x--x`: dono por ler, escrever e executar. Grupo só não pode escrever e todo o resto só pode executar
    * Em diretórios, `x` indica permissão de busca
* Antes de abrir um arquivo, checa-se sua permissão e o sistema retorna um **file descriptor (descritor de arquivo)** inteiro. Retorna código ``-1` se o acesso é proibido
* MINIX 3 permite montar sistema de arquivos a partir de unidades removíveis, que se inserem junto à arvore principal
  * Antes da chamada de montagem, o **sistema de arquivos raiz** e o segundo sistema de arquivos não se relacionam

* **Arquivo especial**: faz dispositivos de I/O se comportarem como arquivos, podendo ser lidos e escritos com as mesmas chamadas de arquivos
  * Dois tipos: **arquivos especiais de bloco** e **arquivos especiais de caracteres**
  * Mantidos no diretório `/dev`, `/dev/lp` pode se referir à impressora de linha
  * **Arquivos especiais de bloco**: modelam dispositivos que colecionam blocos endereçáveis randômicos, como discos
  * **Arquivos especiais de caracter**: modelam dispositivos que aceitam ou têm como saída um fluxo de caracteres
* **Pipe**: pseudoarquivo para conectar processos
  * Processo A envia dados para o processo B, escrevendo como se fosse um arquivo de saída. O processo B lê os dados como se fosse um arquivo de entrada
  * O processo só consegue identificar que se trata de um pipe através de uma chamada de sistema, do contrário pensará ser um arquivo

### 1.3.3 - A Shell

* Interface entre o usuário e o SO
* Iniciada quando o usuário loga, tem o terminal como entrada e saída padrão
* Imprime o **prompt**, caracter para indicar que a shell aguarda comandos
* Capaz de redirecionar a entrada e saída: `sort < file01 > file02`
  * Toma input de `file01` e escreve a saída em `file02`
* Após um comando, a shell cria um processo filho e espera ele terminar, dando novamente o prompt para o comando seguinte
  * Usando `&`, a shell não espera o processo terminar, rodando como tarefa em segundo plano



 