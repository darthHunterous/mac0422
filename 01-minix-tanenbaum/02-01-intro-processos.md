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