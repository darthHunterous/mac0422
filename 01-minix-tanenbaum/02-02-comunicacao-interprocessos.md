# 2.2 - Comunicação Interprocessos

* Necessidade de comunição entre processos de maneira bem estruturada, sem usar interrupções
* Três problemas
  * Passar informação
  * Garantir que dois processos não interfiram entre si em atividades críticas (exemplo, pegar o último mega de memória)
  * Sequenciamento de dependências: processo A produz dados e B os imprime, então B precisa esperar que A tenha produzido algum dado
* Passar informação é simples para threadas por compartilharem o mesmo espaço de endereçamento

### 2.2.1 - Condições de Corrida

* **Condições de corrida**: quando dois ou mais processos estão lendo ou escrevendo em alguns dados compartilhados e o resultado final depende da ordem de quem executa

### 2.2.2 - Seções Críticas

* **Exclusão mútua**: evitar que mais de um processo leia e escreva em dado compartilhados ao mesmo tempo
* **Seção crítica**: parte em que a memória compartilhada é acessada
* Quatro condições para cooperação de processos com dados compartilhados
  1. Dois processos não podem estar simultaneamente na seção crítica
  2. Sem suposições sobre a velocidade e número de processadores
  3. Nenhum processo fora da seção crítica pode bloquear outros processos
  4. Nenhum processo deve esperar para sempre para entrar na sua seção crítica

### 2.2.3 - Exclusão mútua com espera ativa (busy waiting)

* **Desativar interrupções**

  * Solução mais simples
  * Desabilitar interrupções assim que entrar na seção crítica, evitando que a CPU troque para outro processo, examinando e atualizando a memória compartilhada sem que outro processo intervenha
  * Ruim ao dar o poder de processos de usuário desativar interrupções, pois se não for retornado, acaba-se com o gerenciamento do sistema
  * Não funciona para sistemas com vários processadores
  * Conclusão: útil para questões do próprio SO mas inapropriado para exclusão mútua de processos de usuário

* **Variáveis de trava**

  * Variável compartilhada de trava inicializada como zero. Quando o processo quer entrar na seção crítica, testa a trava e se for zero, entra e a configura com valor `1`
  * Mesmo problema de condição de corida, pois se um processo ler a trava e ver que vale zero mas antes que insira o valor `1`, seja trocado por outro processo, este outro poderá configurar a trava a `1` e quando o processo original retornar, este também colocará valor `1`, sendo assim haverá dois processos na seção crítica simultaneamente

* **Alternância Estrita**

  * Usa-se uma variável para indicar de quem é o turno atual, enquanto cada processo fica checando continuamente para ver se já chegou sua vez (**espera ativa** ou **busy waiting**, que é testar continuamente uma variável até certo valor aparecer)
  * Gasta tempo de CPU, portanto usado quando a espera será curta
  * Trava com busy waiting chama-se **spin lock** (**trava giratória**)
  * Ruim quando um dos processos é muito mais devagar, pois o mais lento ficará mais tempo na região não crítica e outro processo mais rápido terá que esperar o lento sair da região não crítica, checar que ainda é sua vez, executar a região crítica e aí sim alterar o turno para o do processo mais rápido
    * Viola a condição 03: o processo lento, fora da região crítica está bloqueando o processo rápida
  * Evita condições de corrida, mas viola a condição 3

* **Solução de Peterson**

  * ```c
    int turno;	/* vez de qual processo */
    int interessado[2];	/* inicializado como zero */
    
    void entrar_regiao(int processo) {	/* processo 0 ou 1 */
      int outro;	/* numero do outro processo */
      outro = 1 - processo;	/* oposto do processo atual */
      
      interessado[processo] = 1;	/* mostra que o processo atual tem interesse em entrar */
      turno = processo;	/* configura a flag */
      while (turno == processo ** interessado[outro] == 1)	/* trava giratoria */;
    }
    
    void deixar_regiao(int processo) {	/* processo: que esta saindo */
      interessado[processo] = 0;	/* indica a saida da regiao critica */
    }
    ```

    * Inicialmente nenhum está na região crítica
    * Processo `0` indica seu interesse e determina seu turno
    * Processo `1` ainda não está interessado então a trava giratória se encerra imediatamente
    * Se o processo `1` tentar entrar na região, ficará esperando até que `0` deixe de estar interessado, que ocorre quando o processo `0` chamar a função para deixar a região crítica
    * Se ambos tentarem entrar na região crítica ao mesmo tempo, um deles armazenará seu valor na variável do turno por último, retendo este valor. Quando ambos chegarem na trava giratória, o outro processo executará ela zero vezes, entrando na região crítica, enquanto o outro espera na trava

* **Instrução TSL**

  * Ajuda de hardware, presente em computadores desenhados com vários processadores

  * Test and Set Lock (Testar e configurar trava)

    * `TSL RX, LOCK`: lê o contéudo da palavra da memória `LOCK` e armazena no registrador `RX`, armazenando não zero em `LOCK`

  * Ler a palavra e armazenar nela são garantidas como indivisíveis (nenhum outro processador pode acessar a palavra da memória até a instrução terminar)

    * CPU bloqueia o barramento de memória para garantir isso

  * `LOCK` pode ser usada para coordenar o acesso à memória compartilhada: quando `0`, qualquer processo pode configurar como `1` usando `TSL` e depois ler ou escrever na memória compartilhada. Por fim, o processo configura `LOCK` de volta a `0` com a instrução `move`

  * Código em Pseudo Assembly

    * ```assembly
      enter_region:
      	TSL reg, lock		# copia lock para registrador e configura lock como 1
      	CMP reg, 0		# checa se lock era zero
      	JNE enter_region		# se lock nao era zero, a trava estava configurada, entao fica em loop
      	RET		# retorna para quem fez a chamada, entrando na regiao critica
      
      leave_region:
      	MOVE lock, 0		# coloca zero no lock
      	RET		# retorna para quem fez a chamada
      ```

      * Quando a trava está configurada, fica em trava giratória. Eventualmente a trava se tornará zero (pois o outro processo sairá da região crítica) e o processo atual retorna da primeira subrotina, entrando na seção crítica

### 2.2.4 - Sleep e Wakeup

* **Problema da inversão de prioridade**: busy waiting não só gasta CPU esperando mas também pode causar situações inesperadas como para processos de diferentes prioridades: se um processo de baixa prioridade estiver em sua seção crítica e outro de alta prioridade se torna pronto para execução, o escalonador transicionará entre ambos, porém o de processo de alta prioridade ficará preso em loop para sempre pois o de baixa prioridade foi interrompido na seção crítica e jamais será escalonado enquanto o outro não terminar sua execução

* `sleep` e `wakeup` como solução

  * `sleep` faz o processo que chamou ser bloqueado até outro processo acordá-lo
  * `wakeup` recebe um parâmetro: o processo que deve acordar

* **Problema do Produtor-Consumidor**

  * Também **problema do buffer limitado**

  * Processos compartilham um buffer: produtor coloca informação no buffer enquanto o consumidor retira

  * Quando o buffer está cheio e o produtor tenta colocar um novo item, ele deve ser bloqueado e só acordado quando o consumidor tiver retirado algo do buffer. Analogamente, se o consumidor tentar tirar algo do buffer vazio, deve ser bloqueado também

  * Problemas de condição de corrida sobre uma variável de contagem do buffer: producer testa se `count` é `N` (número máximo do buffer), se é, dorme, senão adicionar um item e incrementa `count`. O consumidor é similar, checando `count` igual a zero

  * **Código**

    * ```c
      #define N 100  /* tamanho do buffer */
      int count = 0;  /* itens no buffer */
      
      void produtor() {
        int item;
        
        while (1) {
          item = produzir_item(); /* gera proximo item */
          if (count == N) /* dorme com buffer cheio */
            sleep();
          inserir_item(item); /* coloca item no buffer */
          count++;
          if (count == 1) /* se ha itens para consumir, acorda consumidor */
            wakeup(consumidor);
        }
      }
      
      void consumidor() {
        int item;
        
        while (1) {
          if (count == 0) /* dorme com buffer vazio */
            sleep();
          item = remove_item(); /* remove item do buffer */
          count--;
          if (count == N-1)
            wakeup(produtor);
          consumir_item(item);
        }
      }
      ```

      * Condição de corrida: buffer vazio, consumidor acaba de ler `count` para efetuar a comparação com zero quando o escalonador para a execução do consumidor para executar o produtor. Após o produtor adicionar um item ao buffer, vê que `count` era zero e agora é `1` e envia um sinal de `wakeup` para o consumidor, mas como este não chegou a executar `sleep` antes de ser interrompido pelo escalonador, esse sinal é perdido. Quando o consumidor executa pelo escalonador, ele continua o teste de `count` sendo zero e executa `sleep`, com isso, o produtor eventualmente irá encher o buffer e executar `sleep` e ambos processos dormirão para sempre.
      * A solução é um **bit de espera por wakeup**: se um `wakeup` é enviado para um processo ainda acordado, esse bit recebe `1`. Depois, se o processo tentar dormir e o bit vale `1`, coloca-se `0` no bit e o processo permanece acordado

### 2.2.5 - Semáforos

* **Semáforo**: valor `0` se nenhum `wakeup` foi salvo e um valor positivo para a quantidade de `wakeups` pendentes

* Duas operações: `down` e `up`

  * `down`: checa se o valor do semáforo é maior que zero. Se for, diminui em um o número de `wakeups` armazenados. Se for `0`, o processo dorme sem completar o `down` por enquanto. Toda essa operação é feita de forma indivisível, como **ação atômica**
  * `up`: incrementa o valor do semáforo. Se há processos dormindo no semáforo, esperando o término de uma operação `down`, escolhe-se um deles aleatoriamente para completar o `down`
    * Após um `up` com processos dormindo no semáforo, o valor ainda será `0` mas haverá um processo dormindo a menos
    * Também indivisível

* Garante-se que nenhum outro processo pode acessar o semáforo até que sua operação tenha sido completa ou bloqueada, sendo a atomicidade essencial para resolver sincronização e evitar condições de corrida

* **Problema produtor-consumidor com semáforos**

  * Resolvido o problema de `wakeups` perdidos

  *  Implementar `up` e `down` como chamadas de sistema, desabilitando interrupções enquanto as operações do semáforo executam

  * Com vários processadores, cada semáforos deve ser protegido com uma trava, usando a instrução TSL para que apenas um processador por vez examine o semáforo

    * Mais eficiente que a espera ativa (busy waiting) pois as operações no semáforo são breves e bem definidas

  * Solução com `3` semáforos:

    * `full`: número de slots cheios

    * `empty`: número de slots vazios

    * `mutex`: evita produtor e consumidor acessarem o buffer simultaneamente

      * **Semáforos binários**: inicializados com `1` e usado por vários processos para garantir que apenas um entre na seção crítica por vez

    * **Código**

      * ```c
        #define N 100 /* tamanho do buffer */
        typedef int semaphore; /* semaforos sao tipos especiais de int */
        semaphore mutex = 1; /* controla acesso a regiao critica */
        semaphore empty = N; /* espacos vazios no buffer */
        semaphore full = 0; /* espacos ocupados no buffer */
        
        void producer() {
          int item;
          
          while (1) {
            item = produce_item();
            down(&empty);	/* diminui quantidade vazia no buffer */
            down(&mutex); /* entra na secao critica */
            insert_item(item);
            up(&mutex); /* sai da secao critica */
            up(&full); /* incrementa quantidade de elementos no buffer */
          }
        }
        
        void consumer() {
          int item;
          
          while (1) {
            down(&full);
            down(&mutex);
            item = remove_item(); /* remove do buffer */
            up(&mutex);
            up(&empty);
            consume_item(item); /* faz algo com o item ja retirado */
          }
        }
        ```

        * Semáforos usados de duas maneiras:
          1. Exclusão mútua (para que apenas um processo por vez possa acessar o buffer)
          2. **Sincronização** (`full` e `empty` para garantir que certos eventos ocorram na sequência correta, como produtor parar de executar com o buffer cheio e o consumidor parar com o buffer vazio)

* Semáforos na sequência de interrupção da figura 2.5

  * Para ocultar interrupções pode-se inicializar um semáforo em `0` em cada dispositivo de input e logo que cada um destes é inicializado, o processo gerenciador executa `down` nesse semáforo, bloqueando.
  * Ao ocorrer a interrupção, executa-se `up` nesse semáforo, tonando o processo passível de execução novamente
  * Etapa 06 da figura 2.5: "função de passagem de mensagem marca o destinário que aguarda tal mensagem como pronto" equivale a uma operação `up` no semáforo do dispositivo para que na etapa 07 o escalonador escolha executar o gerenciador de dispositivo

### 2.2.6 - Mutex

* Gerencia exclusão mútua como um semáforo, mas sem contagem
* Variável com dois estados: liberado ou travado
* Pode ser representada com um bit, mas usa-se um inteiro, onde `0` é travado e todos outros valores, liberado
* `mutex_lock` para acessar a região crítica. 
  * Se o mutex está liberado (seção crítica disponível), a thread que chamou pode entrar. 
  * Se estiver bloqueado, o processo é suspenso até que o processo atualmente na seção crítica termine e chame `mutex_lock` novamente
  * Se vários processos estão aguardando, um é escolhido aleatoriamente

### 2.2.7 - Monitores

* Se o `mutex` no código do produtor-consumidor com semáforo sofresse `down` antes do `empty`, como buffer cheio, o produtor seria bloqueado com o `mutex` zerado. O consumidor ao tentar acessar o buffer seria bloqueado ao executar `down` no `mutex` zerado.

  * Ambos ficariam bloqueados para sempre (**deadlock**)

* **Monitor**: agrupamento de procedimentos, variáveis e estruturas de dados em um pacote

  * Processos podem acessar os procedimentos quando quiserem mas não podem acessar diretamente as estruturas de dados a partir de procedimentos fora do monitor

* Garantia de exclusão mútua:  apenas um processo ativo em um monitor a qualquer instante

  * Ao chamar um procedimento do monitor, checa-se se há outro processo ativo dentro dele. Se houver, o processo que chama fica suspenso até que o outro deixe o monitor
  * Responsabilidade do compilador implementar a exclusão mútua nas entradas do monitor
  * Transformar todas regiões críticas em procedimentos de monitor garante que dois processos não irão executar na região crítica simultaneamente

* Só exclusão mútua com monitores não basta, pois é necessário uma forma de bloquear processos: **variáveis de condição** com operações `wait` e `signal`

  * Quando o procedimento do monitor descobrir que não pode prosseguir (produtor encontra o buffer cheio), executa `wait` na variável de condição `full`, fazendo processo que chama bloquear
  * O consumidor pode acordar outro processo dormindo executando `signal` na variável de condição que o outro aguarda. Para evitar dois processos ativos no monitor, convenção após a execução de `signal`: quem executa o `signal` precisa sair do monitor imediatamente, portanto `signal` só pode aparecer como instrução final em uma rotina do monitor
    * Se vários processos aguardam um sinal numa dada variável de condição, o escalonador decide quem acorda

* Variáveis de condição não acumulam sinais, portanto se não houver alguém esperando por um, esse sinal é perdido, portanto o `wait` deve vir antes do `signal`

* A exclusão mútua automática do monitor garante que um processo produtor encontrando o buffer cheio possa concluir a operação `wait` sem se preocupar que o escalonador troque para o consumidor antes que o `wait` seja completado, pois o consumidor não poderá entrar no monitor até que `wait` termine e o produtor seja marcado como não mais executável

* **Código Produtor-Consumidor com monitores**

  * ```pascal
    monitor ProducerConsumer
    	condition full, empty;
    	integer count;
    	
    	procedure insert(item: integer);
    	begin
    		if count = N then wait(full);
    		insert_item(item);
    		count := count + 1;
    		if count = 1 then signal(empty)
    	end;
    	
    	function remove: integer;
    	begin
    		if count = 0 then wait(empty);
    		remove = remove_item;
    		count := count - 1;
    		if count = N - 1 then signal(full)
    	end;
    	
    	count := 0;
    end monitor;
    
    procedure producer;
    begin
    	while true do
    	begin
    		item = produce_item;
    		ProducerConsumer.insert(item)
    	end
    end;
    
    procedure consumer;
    begin
    	while true do
    	begin
    		item = ProducerConsumer.remove;
    		consume_item(item)
    	end
    end;
    ```

* Java possui threads a nível de usuário e provê a keyword `synchronized` para métodos, garantindo que quando uma thread execute este método, nenhuma outra possa executar outro método `synchronized` dessa classe

  * Não possui variáveis de condição como em monitores, no lugar, tem `wait` e `notify`, equivalentes a `sleep` e `wakeup`, que dentro de métodos sincronizados não estão sujeitos a condições de corrida

* Monitores são um conceito de linguagem de programação, portanto é responsabilidade do compilador definir as regras de exclusão mútua

  * C não possui monitores

* C também não possui semáforos, mas podem ser implementados através da adição das rotinas `up` e `down`, independentes do compilador, mas o SO deve já prever o uso de semáforos

* Para sistemas distribuídos, com várias CPUs, cada qual com sua memória privada, semáforos se tornam muito baixo nível para evitar corridas

  * Usa-se **passagem de mensagens**

### 2.2.8 - Passagem de Mensagens

* Duas primitivas `send` e `receive` como chamadas de sistemas

* Podem ser colocadas como funções de biblioteca

  * `send(destination, &message)`
  * `receive(source, &message)` 

* **Problemas de projeto para Passagem de Mensagens**

  * Mensagens podem ser perdidas pela rede
    * O destinatário deve enviar uma **confirmação** que recebeu a mensagem, pois do contrário o o remetente deve enviar novamente após algum tempo
  * A confirmação pode ser perdida pela rede
    * O destinatário deve ser capaz de distinguir entre uma nova mensagem e uma retransmissão
  * O processo especificado em `send` e `receive` não pode ser ambíguo, o sistema de mensagens deve lidar com como os processos são nomeados
  * **Autenticação**: garantia de estar comunicando com o servidor de arquivos legítimo
  * Passar mensagem na mesma máquina: mais lento que semáforos e monitores. Uma opção é passar mensagens através de registradores

* **Produtor-Consumidor com passagem de mensagens**

  * Suposições:

    * Mensagem de mesmo tamanho
    * Mensagens enviadas e não recebidas são armazenadas em buffer pelo SO

  * Consumidor envia `N` mensagens vazias e o produtor retorna elas com conteúdo sempre que tem um item a ser enviado

  * Se o produto trabalha muito rápido, todas mensagens ficam cheias aguardando o consumidor e então o produtor é bloqueado aguardando o retorno de uma mensagem vazia

    * O inverso também ocorre

  * **Código**

    * ```c
      #define N 100 /* tamanho do buffer */
      
      void producer() {
        int item;
        message m; /* buffer de mensagens */
        
        while (1) {
          item = produce_item(); /* gera algo para por no buffer */
          receive(consumer, &m); /* aguarda uma mensagem vazia */
          build_message(&m, item); /* constroi a mensagem a ser enviada */
          send(consumer, &m); /* envia um item para o consumidor */
        }
      }
      
      void consumer() {
        int item, i;
        message m;
        
        for (i = 0; i < N; i++)
        	send(producer, &m); /* envia N mensagens vazias */
        while (1) {
          receive(producer, &m); /* recebe mensagem contendo item */
          item = extract_item(&m); /* extrai item da mensagem */
          send(producer, &m); /* envia de volta a resposta vazia */
          consume_item(item);
        }
      }
      ```

* Maneiras de endereçamento
  * Atribuir a cada processo um endereço e endereçar mensagens a eles
  * Estrutura de dados **caixa de correio**: buffer de mensagens para qual são endereçadas ao invés de processos
    * Ao tentar enviar para um caixa de correio cheia, o processo é suspenso até que uma mensagem seja removida
    * Tanto o consumidor e o produtor tem uma caixa de correio para `N` mensagens, deixando claro o mecanismo de buffer
  * Também pode-se remover o buffer (estratégia de **rendezvous**), então se `send` é feito antes do `receive` o processo que envia é bloqueado até ocorrer o `receive`, copiando a mensagem diretamente do remetente para o destinatário
    * Com `receive` antes do `send`, o receptor fica travado até que o `send` ocorra
* Em MINIX 3, usa-se o rendezvous para comunicação entre processos que compõe o SO
  * Processos de usuário se comunicam com o SO por rendezvous também
  * Comunicação entre processo de usuário se dá via pipes (caixa de correio)
    * Diferença entre pipe e caixa de correio: pipe não preserva limites de mensagens. Se são enviadas 10 mensagens de 100 bytes e outro processo ler 1000 bytes do pipe, receberá as 10 de uma vez
    * Em um sistema de mensagens, cada `read` deve retornar apenas uma mensagem
* **MPI - Message Passing Interface**: comumente usado em sistemas de programação paralela
  * Computação científica