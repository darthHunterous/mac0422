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