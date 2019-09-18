# 2.4 - Escalonamento

* **Multiprogramação**: vários processos competindo pela CPU ao mesmo tempo

* **Escalonador**: decide qual dos vários processos prontos será executado a seguir

  

## 2.4.1 - Introdução ao Escalonamento

* Escalonamento em sistema de lotes: apenas ler a próxima tarefa na fita magnética

### Comportamento de processos

* Processos alternam entre períodos de computação com requisição de entrada/saída
  * Computação é quando a CPU está em uso
  * Entrada/saída é quando um processo é bloqueado para aguardar um dispositivo externo
* Tipos de processos
  * **Limitado por processamento** (**CPU-bound**): longas rajadas de uso da CPU e poucas esperas por entrada/saída
  * **Limitado por entrada/saída** (**I/O bound**): pequenas rajadas de processamento e vários esperas por entrada/saída
    * Fator chave para definir é o pouco uso de CPU e não as longas esperas

### Quando escalonar

* Passível de escalonamento:
  * Escalonamento necessário (processo não mais apto a executar)
    1. Saída de um processo
    2. Processo bloqueado em entrada/saída ou semáforo
  * Escalonamento comumente feito, mas não obrigatório
    1. Novo processo criado
       * O pai pode requisitar prioridade diferente para seu filho
    2. Interrupção de entrada/saída
       * Geralmente significa que o dispositivo de entrada/saída completou seu trabalho e o processo bloqueado aguardando pode ser executado
    3. Interrupção de relógio
       * Algoritmo **não-preemptivo**: permite um processo rodar até bloquear ou voluntariamente liberar a CPU
       * Algoritmo **preemptivo**: permite um processo rodar por um tempo máximo fixado
         * Requer uma interrupção de relógio para retornar o controle à CPU

### Categorias de algoritmos de escalonamento

* Ambientes diferentes, escalonamento diferente
  1. Lote
     * Sem usuários esperando, períodos longos para cada processo são aceitáveis
     * Trocando menos entre processos melhora a performance
  2. Interativo
     * Preempção essencial para evitar que um processo monopolize a CPU, negando serviço a outros usuários interagindo
  3. Tempo real
     * Em sistemas com  restrições de tempo real, preempção pode não ser necessária pois os processos sabem que não podem executar por um lonog período, fazendo seu trabalho e bloqueando rapidamente
     * Executam apenas programas necessários a uma dada aplicação

### Objetivos de Algoritmo de Escalonamento

* Objetivos por circunstância
  * **Todos Sistemas**
    * Imparcialidade: dar a cada processo uma porção justa da CPU
    * Imposição da política: garantir que a política declarada é executada
    * Equilíbrio: manter todas partes do sistema ocupadas
  * **Sistemas de Lote**
    * Taxa de saída: maximar tarefas por hora
    * Tempo de retorno: minimizar tempo entre envio e término
    * Utilização de CPU: manter a CPU ocupada
  * **Sistemas Interativos**
    * Tempo de resposta: atender requisições rapidamente
    * Proporcionalidade: satisfazer o que usuários esperam
  * **Sistemas em tempo real**
    * Cumprir prazos: evitar perda de dados
    * Previsibilidade: evitar degradação de qualidade
* Imparcialidade é impr