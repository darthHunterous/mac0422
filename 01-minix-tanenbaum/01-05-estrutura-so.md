# 1.5 - Estrutura do Sistema Operacional

### 1.5.1 - Sistemas Monolíticos

* Bagunçado, não há estrutura
* Coleção de funções que podem chamar uns aos outros sem restrições
* Não há modularização e pacotes definindo os pontos de entrada corretos
* Chamadas de sistema são requisitadas colocando parâmetros em lugares pré definidos (registradores ou na pilha) e depois executando uma instrução de interrupção especial (**chamada de núcleo** ou **chamada de supervisor**), que transfere o controle para o SO, trocando de modo usuário para modo kernel
  * No modo kernel todas instruções são permitidas
  * No modo usuário, I/O e certas instruções não são permitidas
* A preparação para um syscall coloca os parâmetro na pilha (ordem reversa, pois historicamente C precisava que o primeiro parâmetro de printf, a string de formatação, ficasse no topo da pilha)
* Estrutura básica do SO monolítico
  1. Programa principal invoca o procedimento do serviço requisitado
  2. Um conjunto de funções de serviço continuam com as chamadas de sistema
  3. Um conjunto de funções utilitárias que auxiliam os funções de serviço



### 1.5.2 - Sistemas em Camada

* Hierarquia de camadas, cada uma construída sobre as abaixo dela. Sistema THE (1968):
  * **Camada 5 - Operador**
  * **Camada 4 - Programas de Usuário**
  * **Camada 3 - Gerenciamento de Entrada e Saída**
  * **Camada 2 - Comunicação Operador-Processo**
  * **Camada 1 - Gerenciamento de Memória e Tambor**
  * **Camada 0 - Alocação de processador e multiprogramação**



### 1.5.3 - Máquinas Virtuais

* Um sistema com tempo compartilhado fornece: multiprogramação e uma máquina estendida com uma interface mais conveniente que o hardware básico
* O **monitor de máquina virtual** roda no hardware básico e executa a multiprogramação no caso do VM/370, provendo várias máquinas virtuais para a próxima camada acima, porém não são máquinas estendidas, mas cópias idênticas do hardware básico
  * Cada máquina virtual pode rodar um SO diferente
  * Podem rodar o **CMS** (Conversational Monitor System) , sistema interativa e de único usuário, para usuário de tempo compartilhado
* A ideia de máquina virtual hoje em dia é por compatibilidade: rodar programas antigos (MS-DOS em Pentium por exemplo). No Pentium, a Intel provê uma máquina virtual para o antigo 8086
  * Tais programs rodam no hardware básico ao executar instruções normais, mas ao tentar interromper o SO para uma chamada de sistema ou tentar fazer I/O protegida, ocorre uma interrupção para o monitor da máquina virtual
* MINIX 3 pode ser instalado em uma máquina virtual como VMWare sem correr riscos de corromper ou apagar os dados em disco durante experimentações de alunos
* **JVM (Java Virtual Machine)**: compilador Java produz código para a JVM, endo como vantagem a fácil compatibilidade com qualquer máquina que execute o interpretador da JVM



### 1.5.4 - Exokernels

* Sistema que dá aos usuários clone do computador em si, mas com um subconjunto dos recursos
* o **exokernel** roda em modo kernel e tem como trabalho alocar recursos para as máquinas virtuais e checar se nenhuma está tentando usar os recursos de outra
* Elimina a necessidade de mapeamento de recursos pelo monitor de máquina virtual, pois o exokernel só precisa tomar nota de qual máquina virtual foi alocada para cada recurso



### 1.5.5 - Modelo Cliente-Servidor

* Tendência de SOs modernos de deixar um **kernel** mínimo, movendo código para camadas mais acima
  * Implementação da maioria das funções do sistema através de processos de usuário
  * Requisição de um serviço através de um **processo cliente** que requisita para um **processo servidor**, que faz o trabalho e devolve a resposta
* Em MINIX 3, drivers estão no espaço do usuário e usam chamadas especiais do kernel para requistar leituras e escritas de registradores de I/O ou acessar informação do kernel