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

