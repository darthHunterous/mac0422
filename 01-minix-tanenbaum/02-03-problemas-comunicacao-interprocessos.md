# 2.3 - Problemas de IPC

### 2.3.1 - Janta dos Filósofos

* Dijkstra, 1965

* Cinco filósofos ao redor de uma mesa circular

  * Cada filósofo tem um prato de macarrão
  * O macarrão precisa ser comido com dois garfos por ser escorregadio
  * Há cinco garfos (um entre cada prato)
  * Cada filósofo alterna entre estar comendo e pensando
    * Ao ficar faminto, um filósofo tenta pegar o garfo à sua esquerda e outro à sua direita. Se conseguir, come por um tempo, coloca os garfos na mesa e volta a pensar

* **Complicações**

  * Se todos filósofos tentarem pegar seus garfos à esquerda simultaneamente, nenhum conseguirá pegar o garfo à direito e haverá **deadlock**
  * Se modificar para checar se o garfo à direita está disponível logo após pegar o da esquerda e, em caso negativo, colocar de volta na mesa, pode ocorrer **starvation** (**inanição**) se todos filósofos entrarem simultaneamente nessa verificação, pois ficarão apenas pegando e colocando o garfo na mesa sem que o processo faça progresso
  * Outra possibilidade: filósofo esperar um tempo aleatório após falhar em pegar o garfo à direita, tornando o starvation improvável
    * Problema: é uma solução que pode falhar de acordo com uma sequência improvável de número aleatórios
  * Mutex (semáforo) na manipulação dos garfos: garante a corretude porém apenas um filósofo é capaz de comer a cada instante
    * Com 5 garfos, dois filósofos deveriam poder comer ao mesmo tempo

* **Código - Solução**

  * Eficiente (paralelismo) e sem deadlock

  * ```c
    #define N 5
    #define LEFT (i+N-1) % N
    #define RIGHT (i+1) % N
    #define THINKING 0
    #define HUNGRY 1 /* filósofo tentando pegar os garfos */
    #define EATING 2
    typedef int semaphore;
    int state[N]; /* estado de cada filósofo de 0 a 2 */
    semaphore mutex = 1;
    semaphore s[N]; /* um semáforo por filósofo */
    
    /* número do filósofo de 0 a N-1 */
    void philosopher(int i) {
        while(1) {
            think();
            take_forks(i); /* pega dois garfos ou bloqueia o filósofo */
            eat();
            put_forks();
        }
    }
    
    void take_forks(int i) {
        down(&mutex);
        state[i] = HUNGRY;
        test(i); /* tenta pegar dois garfos */
        up(&mutex);
        down(&s[i]); /* bloqueia se não adquiriu garfos */
    }
    
    void put_forks(int i) {
        down(&mutex);
        state[i] = THINKING;
        test(LEFT); /* checa se o vizinho esquerdo estava esperando para comer */
        test(RIGHT); /* checa se o vizinho esquerdo estava esperando para comer */
        up(&mutex);
    }
    
    void test(i) {
        if (state[i] == HUNGRY && state[LEFT] != EATING && state[RIGHT] != EATING) {
            state[i] = EATING;
            up(&s[i]);
        }
    }
    ```

### 2.3.2 - Problema dos Leitores e Escritores

* Janta dos filósofos: modela processos competindo para acesso a recursos limitados (dispositivos de entrada, por exemplo)

* Problema dos leitores e escritores: modela acesso a um banco de dados

  * Se um processo está atualizando o banco, nenhum outro pode acessar, mesmo para leitura

* **Código - Solução**

  * ```c
    typedef int semaphore;
    semaphore mutex = 1; /* controla acesso a rc */
    semaphore db = 1; /* controla acesso ao banco */
    int rc = 0; /* processos lendo ou esperando para ler */
    
    void reader() {
        while(1) {
        	down(&mutex); /* acesso exclusivo a rc */
        	rc = rc + 1;
        	if (rc == 1)
            	down(&db);
        	up(&mutex); /* libera acesso a rc */
        	read_data_base();
        	down(&mutex); /* acesso exclusivo a rc */
        	rc = rc - 1;
        	if (rc == 0)
            	up(&db);
        	up(&mutex); /* libera acesso a rc */
        	use_data_read();
        }
    }
    
    void writer() {
        while(1) {
            think_up_data();
            down(&db); /* acesso exclusivo ao bd */
            write_data_base();
            up(&db);
        }
    }
    ```