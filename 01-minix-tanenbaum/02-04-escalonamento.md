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
      * Processos comparáveis devem receber serviço comparável
    * Imposição da política: garantir que a política declarada é executada
    * Equilíbrio: manter todas partes do sistema ocupadas
      * Misturar processos limitados por processamento e por entrada/saída para evitar que se concentrem em disputar o mesmo recurso
  * **Sistemas de Lote**
    * Taxa de saída: maximar tarefas por hora
    * Tempo de retorno: minimizar tempo entre envio da tarefa e seu término
      * Um algoritmo de escalonamento que maximiza a taxa de saída não garante um baixo tempo de retorno, pois se ele sempre rodar tarefas curtas e elas continuarem chegando sem parar, as tarefas longas terão tempo de retorno infinito
    * Utilização de CPU: manter a CPU ocupada
  * **Sistemas Interativos**
    
    * Tempo de resposta: atender requisições rapidamente
      * Mais importante a minimizar: tempo entre um comando e o resultado
    * Proporcionalidade: satisfazer o que usuários esperam
  * **Sistemas em tempo real**
    * Cumprir prazos: evitar perda de dados
    
      * Caso de um computador controlando um dispositivo que produz dados em uma taxa constante
    
    * Previsibilidade: evitar degradação de qualidade
    
      * Escalonamento previsível e regular: processo de áudio executando erraticamente deteriora a qualidade de som (o mesmo para vídeo)
    
        
## 2.4.2 - Escalonamento em sistemas de lote

### Primeiro a chegar, primeiro atendido

* Não preemptivo
* Fila de processos prontos, executam até bloquear
  * Quando um processo bloqueado fica pronto, volta para o fim da fila
* Lista ligada: remove da frente da fila para executar e adicionar um novo processo se dá por acoplar ao fim da fila
* Não otimiza alternância entre processos limitados por processamento e processos limitados por entrada/saída

### Tarefa mais curta primeiro

* Não preemptivo
* Requer conhecimento dos tempo de execução
* Reduz o tempo médio de retorno
  * Como primeira tarefa termina no tempo `a`, segunda em `a+b`, terceira em `a+b+c` e quarta em `a+b+c+d`, a média é dada por `(4a + 3b + 2c + d) / 4`
    * `a` contribui mais para média, portanto deve ser a tarefa mais curta para minimizar o tempo médio de retorno
* Apenas ótimo quando todas tarefas estão disponíveis simultaneamente

### Menor tempo de execução restante

* Versão preemptiva do escalonamento da tarefa mais curta primeiro
* Sempre escolhe o processo com menor tempo restante de execução, trocando se uma nova tarefa demanda menos tempo que a atual em execução

### Escalonamento de três níveis

* Tarefas que chegam são colocadas em uma fila de entrada
* **Escalonador de admissão** (primeiro nível de escalonamento) decide quais tarefas ingressam no sistema (memória principal)
  * Tipicamente misturando tarefas limitadas por processamento e por entrada/saída
* Se o número de processos for grande, podem não caber na memória, necessitando armazenar no disco (**escalonador de memória**, segundo nível)
  * Disco é lento, não deve ser revisado com muita frequência
  * Escalonador de memória deve decidir quantos processos quer na memória (**nível de multiprogramação**)
  * Aproximação: se uma certa classe de processos computa cerca de 20% do tempo, manter cinco por perto é por volta do número correto para manter a CPU ocupada
* Critérios para o escalonador de memória trazer processos do disco para a memória
  1. Quanto tempo passou desde que o processo foi trocado?
  2. Quanto tempo de CPU o processo teve recentemente?
  3. Quão grande é o processo?
  4. Quão importante é o processo?
* **Escalonador da CPU**: terceiro nível
  * Pega processos prontos na memória e executa a seguir

## 2.4.3 - Escalonamento em Sistemas Interativos

* Escalonamento em 3 níveis não é possível, apenas dois (memória e CPU)

### Escalonamento rodízio

* Cada processo recebe um **quantum** (intervalo de tempo que pode ser executado)
  * Se estiver executando quando acabar seu quantum, a preempção da CPU é feita e dada para outro processo
* Escalonador mantém uma lista de processos executáveis, onde um processo é colocado no final ao gastar seu quantum
* O problema é o tamanho do quantum por conta do tempo gasto para **troca de processo** ou **troca de contexto**
  * Se a troca leva 1ms e o quantum é 4ms, então a CPU faz trabalho útil por 4ms e gastar 1ms na troca entre eles (20% de overhead administrativo)
  * Com quantum 100ms, o tempo desperdiçado na troca é só 1% mas se 10 usuários interativos apertaram a tecla `Enter` simultaneamente, 10 novos processos serão colocados na lista de executáveis: o décimo processo levará 1 segundo até rodar totalmente, dando a impressão de uma resposta lenta ao usuário
  * Configurar o quantum maior que a rajada média da CPU torna a preempção rara, com as trocas acontecendo apenas quando são logicamente necessários (por bloqueio do processo)
* Quantum de 20 a 50 ms é geralmente razoável

### Escalonamento por prioridade

* Rodízio prevê que os processos são igualmente importantes
* Com prioridades, o processo executável de manior prioridade tem sua execução permitida
* Para reduzir a prioridade de um processos, pode-se reduzir a cada interrupção de relógio e se a prioridade cair abaixo da maior prioridade seguinte, troca-se o processo
* Cada processo pode receber um quantum, que esgotado troca-se para o próximo processo de maior prioridade
* UNIX tem o comando `nice` para o usuário reduzir a prioridade do seu processo
* Prioridade dinâmica para processos limitados por entrada/saída: definir prioridade `1/f` onde `f` é a fração que o processo usou do seu último quantum
* Outra maneira é agrupar classes de processos de mesma prioridade, fazendo rodízio de um quantum dentre a classe de maior prioridade
  * Se as prioridades não são ajustadas de vez em quando, as classes de menor prioridade podem não terem acesso à CPU
* MINIX 3
  * 16 classes de prioridade
  * Maior prioridade: tarefas (drives de entrada/saída) e servidores (gerenciador de memória, sistema de arquivo, rede)
  * Processos de usuário geralmente têm menor prioridade que componentes do sistema (que rodam como processos)
  * Prioridades podem se alterar durante a execução

### Múltiplas filas

* CTSS (1962)
  * Apenas um processo na memória: lenta troca por trocar com o disco
  * Quantum largo para diminuir trocas. Para manter responsividade definiram classes de prioridade
    * Maior classe era executada por um quantum, a segunda maior por dois quanta, a terceira por quatro quanta e assim vai
  * Um processo de 100 quanta executaria por: `1, 2, 4, 8, 16, 32, 64` tendo feito apenas 7 trocas ao invés de 100 em rodízio de 1 quantum
    * A cada troca afunda nas filas de prioridade, sendo rodado com menos frequência, permitindo processos curtos interativos tenham maior prioridade
  * Para evitar punir um processo que se tornasse interativo após vários quanta, se o usuário digitar `Enter` no terminal, o processo era movido para a classe de maior prioridade
    * Pode ser explorado pelo usuário
* XDS 940 (1968)
  * Quatro classes: terminal, I/O, quantum curto e quantum longo
  * Processo aguardando input do terminal ficando pronto vai para classe mais alta
  * Processo que aguarda um bloco do disco vai para segunda classe
  * Um processo que gastou seu quantum, terceira classe
  * Se usou todo o quantum várias vezes sem bloquear, movido para a classe mais baixa
  * Favorece processos interativos em relação aos que rodam no plano de fundo

### Processo mais curto a seguir

* Estimativas com base no comportamento passado do processo, executando o de menor tempo estimado de execução
* Técnica de **envelhecimento**: tempo estimado até agora como `T_0` e o tempo seguinte como `T_1`
  * Atualização da estimativa com a ponderada: `a * T_0 + (1-a) * T_1`
  * Com `a == 1/2`, o peso de `T_0` cai para `1/8` do original

### Escalonamento garantido

* Fazer promessas realistas aos usuários e cumpri-las
  * Com `n` usuários, cada um recebe `1/n` de tempo de CPU
  * Com `n` processos, cada um recebe `1/n` ciclos de CPU

### Escalonamento por sorteio

* Sortear o processo que tomará conta da CPU
* Pode ser 20 ms de CPU para o processo sorteado
* Processos mais importantes podem receber chance maior de ser sorteado para que a longo prazo tenham mais tempo de CPU

### Escalonamento com compartilhamento imparcial

* Prevenir que um usuário tenha mais tempo de CPU simplesmente por estar executando mais processos

## 2.4.4 - Escalonamento em Sistemas de tempo real

* Sistema em tempo real recebe dados de um dispositivo físico externo
* Ter a resposta certa de maneira atrasada é tão ruim quanto não ter
* **Tempo real rígido** (**hard real time**): prazos finais absolutos
* **Tempo real relaxado** (**soft real time**): perder um prazo final é indesejável mas tolerável
* Eventos que o sistema em tempo real precisa responder
  * **Periódico**: em intervalos regulares
  * **Aperiódico**: de maneira imprevisível
* **Sistema em tempo real escalonável**:
  * Se `sigma[i=1..m](C_i / P_i) <= 1`
  * `m` eventos periódicos, em que o i-ésimo tem periódo `P_i` e precisa de `C_i` segundos da CPU
* Tipos de escalonamento para sistemas em tempo real
  * **Estático**: decisões de escalonamento antes do sistema estar rodando
  * **Dinâmico**: decisões de escalonamento em tempo de execução

## 2.4.5 - Política versus mecanismo

* **Problema**: alguns escalonadores não aceitam informações diretamente dos processos sobre decisões de escalonamento, sendo que o processo principal muitas vezes faz ideia de qual dos seus filhos é mais importante (ou os mais críticos em relação ao tempo)
* **Solução**: separar o **mecanismo de escalonamento** da **política de escalonamento**
  * Parametrizar o algoritmo de escalonamento, com os processos de usuários preenchendo tais parâmetros
* Exemplo do kernel usar um algoritmo de escalonamento de prioridade mas fornecer uma chamada para que um processo possa alterar a prioridade de seus filhos, permitindo que o pai controle como seus filhos são escalonados
  * Mecanismo no kernel mas a política é configurada pelo processo

## 2.4.6 - Escalonamento de threads

* Vários processos com múltiplas threads
  * Dois níveis de paralelismo: processos e threads
* Escalonamento diferente para threads a nível de usuário e a nível de Kernel

### Threads a nível de usuário

* Kernel não sabe da existência de threads, pega um processo e deixa rodar seu quantum
* O escalonador de threads decide qual rodar, podendo ela executar quanto quiser e consumir todo o quantum do processo. Quando o processo volta a ser escalonado, a mesma threads continua rodando, consumindo todo o tempo do processo até que a thread termine
* **Exemplo**: processo com 3 threads, cada um com `5ms` de trabalho e o processo com `50ms` de quantum
  * Sem conhecimento a nível de Kernel sobre as threads, elas podem acabar sofrendo rodízio de **A1** a **A3**, com 10 trocas
  * Se as threads fossem de conhecimento do sistema, o rodízio poderia ser feito com as threads de outro processo, **B**

### Threads a nível de kernel

* Cada thread recebe um quantum
* Permite rodízio intercalando threads de cada processo

### Diferenças

* **Performance**: trocar de thread a nível de usuário é mais rápido que trocar de thread a nível de kernel, pois requer uma troca de contexto completa, mudança de mapa de memória e invalidação do cache
  * Pode ser levado em conta para o escalonamento: com duas threads de mesma prioridade, é melhor priorizar aquela que pertence ao mesmo processo
* Com threads a nível de kernel, uma thread bloquear por entrada/saída não suspende todo o processo
* Com threads a nível de usuário, pode-se aplicar um escalonador de threads específico para uma dada aplicação, capazes de ajustar uma aplicação melhor do que o kernel

