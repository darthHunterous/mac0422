# 4.6 - Segmentação

* Ter mais de um espaço de endereço virtual pode solucionar alguns problemas
  * Exemplo de compilador com várias tabelas (fonte, tabela de símbolos, tabela de constantes, árvore de análise sintática [parse tree] e pilha para procedimentos dentro do compilador)
    * Os 4 primeiros crescem continuamente e a pilha cresce e diminui de maneira imprevisível, portanto durante a compilação o espaço reservado para cada seção pode ser insuficiente ou interferir nos outros
* **Solução**: vários espaços de endereço independentes (**segmentos**)
  * Podem ter comprimentos diferentes
  * Comprimento pode mudar durante execução, de forma independente uns dos outros
* Especificar endereço na memória segmentada (bidimensional): endereço de duas partes, um número de segmento e um endereço dentro do segmento
* **Vantagens:**
  * Linkar procedimentos compilados separadamente é mais fácil, pois uma chamada ao procedimento no segmento `n` usa o endereço `(n, 0)` para endereçar o ponto de entrada (palavra `0`)
  * Sem vários espaços de endereços diferentes, mudar o tamanho de um procedimento pode alterar o endereço de início de outro, custoso pois precisa-se alterar qualquer outro procedimento que chame os alterados para incorporar as mudanças de endereço
  * Mais fácil de compartilhar bibliotecas que podem ser colocadas em um segmento e compartilhadas entre processos, não precisando constaram no espaço de endereço de todos processos
* Segmentos contêm apenas um tipo de objetos, formando uma entidade lógica que o programador tem ciência da existência, portanto proteção faz sentido e pode-se ter a apropriada para cada tipo

### Comparação Paginação vs. Segmentação

* **Programador ciente da existência**
  * Não vs. Sim
* **Quantidade de espaços de endereço lineares**
  * 1 vs. vários
* **Espaço de endereço total pode exceder a memória física?**
  * Sim vs. Sim
* **Distinção de procedimentos e dados, proteção separada**
  * Não vs. Sim
* **Tabelas podem ser realocadas facilmente?**
  * Não vs. Sim
* **Fácil compartilhamento de procedimentos**
  * Não vs. Sim
* **Razão da invenção da técnica**
  * Paginação: conseguir um grande espaço linear de endereços sem precisar de mais memória física
  * Segmentação: permitir programas e dados serem quebrados em espaços de endereço logicamente independentes e permitir compartilhamento e proteção

## 4.6.1 - Implementação de segmentação pura

* Segmentos não possuem tamanho fixo, enquanto páginas sim
* Com o tempo, a memória vai sofrendo de **checkerboarding** (**fragmentação externa**) conforme segmentos vão sendo realocados e sobrando buracos de memória não usada entre eles
  * Pode ser resolvido com **compactação**

## 4.6.2 - Segmentação com paginação: Intel Pentium

* Até 16K segmentos com até 2^32 bytes de espaço de endereçamento virtual
* Pode ser configurado pelo SO para usar uma das técnicas ou ambas
* **Memória virtual do Pentium**
  * **LDT (Local descriptor table)**
    * Cada programa tem a sua
    * Descreve segmentos locais ao programa (código, dados, pilha)
  * **GDT (Global descriptor table)**
    * Uma para todos programas
    * Segmentos do sistema, incluindo o próprio SO
* **Funcionamento**
  * Programa Pentium carrega um seletor para acessar um segmento em um dos seis registradores de segmento
    * Registrador CS: guarda o seletor do segmento de código
    * Registrador DS: guarda o seletor do segmento de dados
  * **Seletores**
    * Número de 16 bits
      * 1 bit para indicar LDT ou GDT (1 para LDT)
      * 13 bits para o número da entrada na LDT ou GDT (ou seja cada tabela pode armazenar 8K descritores de segmento)
      * 2 bits relacionados à proteção (nível de privilégio de 0 a 3)
    * Descritor `0` é proibido (causa trap), se carregado em um registrador de segmento indica indisponibilidade
  * Após carregar o seletor, o descritor é pego da tabela adequada e armazenado em registradores de microprograma para acesso rápido
  * **Descritor**
    * 32 bits
    * **Estrutura**
      * Base 0 a 15
        * Base 24 a 31
        * Bit G (0 para limite em bytes e 1 para em páginas)
        * Bit D (0 para segmento de 16 bits e 1 para 32)
        * `0`
        * Buraco
        * Limite 16 a 19
      * Limite 0 a 15
        * Bit P (0 segmento ausente da memória e 1 para presente)
        * 2 bits DPL (nível de privilégo 0 a 3)
        * Bit S (0 para sistema e 1 para aplicação)
        * Tipo e proteção do segmento
        * Base 16 a 23
  * **Diretório de páginas**: cada programa tem um
    * Contém 1024 entradas de 32 bits
    * Cada entrada aponta para uma tabela de páginas contendo 1024 entradas de 32 bits
    * As entradas da tabela de páginas apontam para page frames
  * **Endereço linear**
    * 10 bits `dir`
      * Indexa o page directory para um ponteiro para a tabela de página adequada
    * 10 bits `page`
      * Índice da tabela de páginas para encontrar o endereço físico do page frame
    * 12 bits `offset`
      * Adicionado ao endereço do page frame para conseguir o endereço físico do byte ou palavra necessária
  * **Entradas da tabela de páginas**: 32 bits
    * 20 bits para o número do page frame
    * Bits restantes para acesso e controle e de proteção
* Cada tabela de página tem 1024 page frames de 4KB (4 MB de memória), portanto um segmento com menos de 4 MB tem um diretório de páginas com única entrada (ponteiro para sua única tabela de páginas)
* **Evitar referências repetidas**: Pentium possui TLB que mapeia as combinações `dir-page` mais recentes usadas para o endereço físico do page frame diretamente, sem ficar fazendo as indexações acima do page directory para page table, etc
* **Proteção**: 4 níveis (2 bits)
  * Níveis
    * Nível 0: kernel
    * Nível 1: syscalls
    * Nível 2: bibliotecas compartilhadas
    * Nível 3: programas de usuário
  * Tentativas de acessar dado em um nível maior são permitidos e em níveis menores são ilegais
  * Chamadas de procedimentos de outros níveis são possíveis mas controladas através da instrução `CALL` contendo um seletor em vez de endereço
    * Seletor designa um descritor chamada **call gate** (endereço do procedimento a ser chamado)
      * Evita pula no meio de um código qualquer, apenas usando pontos de entradas oficiais