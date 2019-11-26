# 5.3 - Implementação do sistema de arquivos

## 5.3.1 - Layout do sistema de arquivos

* Sistema de arquivos armazenados em discos
* **Layout de disco**
  * Setor zero do disco: **MBR (Master boot record)** (para o boot do computador)
    * Final do MBR contém tabela de partições (uma pode ser marcada como ativa)
    * BIOS lê e executa o código no MBR
      * Procura a partição ativa, lê o primeiro bloco (**bloco de boot**) e executa, carregando o SO contido na partição
      * Toda partição começa com um bloco de boot mesmo se não conter um SO
  * MBR pode ser chamado também de **IPL (Initial Program Loader)**, **Volume Boot Code** ou **masterboot**
  * Em sistemas compatíveis com PC não pode haver mais de quatro **partições primárias** (espaço para array de 4 eleemtnos de descritores de partição entre MBR e o fim do primeiro setor de 512 bytes)
  * Uma entrada na tabela de partição pode ser uma **partição estendida** (aponta para lista ligada de **partições lógicas**)
    * BIOS não pode inicializar da partição lógica, primeiro uma partição primária precisa carregar código para gerenciá-las
  * MINIX 3: **tabela de subpartições**, contida dentro de outra partição
    * Vantagem: código que gerencia uma partição primária também pode gerenciar uma tabela de subpartição
    * Subpartições podem ser usadas para componentes difernetes como swap, root device, binários do sistemas e arquivos do usuário. Problemas do SO não propagam entre subpartições e fica fácil atualizar o SO trocando o conteúdo de apenas algumas subpartições
* **Exemplo de sistema de arquivos**
  * Partes de todo o disco
    * MBR
    * Tabela de partições
    * Várias partições de disco
      * Bloco de boot
      * Super bloco
        * Contém parâmetros chave sobre o sistema de arquivos e é colocado na memória ao inicializar o computador ou sistema de arquivos
      * Gerenciamento de espaço livre
      * i-nodes
        * Array estrutura de dados (uma por arquivo) descrevendo o arquivo e os blocos em que se encontra
      * Diretório raiz
      * Arquivos e diretórios

## 5.3.2 - Implementando Arquivos

### Alocação Contígua

* Cada arquivo como sequência contígua de blocos de disco (com blocos de 1KB, alocado em 50 blocos consecutivos)
* **Vantagens**
  * Implementação simples: fácil de localizar os blocos do arquivo (necessário apenas o endereço em disco do primeiro block e a quantidade de blocos presente no arquivo)
  * Boa performance de leitura: apenas um seek necessário (encontrar primeiro bloco), depois sem mais atrasos rotacionais, vindo na velocidade completa do disco
* **Desvantagem**
  * Fragmentação do disco com o tempo, necessitando recompactar ou procurar buracos livres que caibam no tamanho requisitado
* Alocação contígua era usada em fitas magnéticas por ser simples e rápida, mas foi descontinuado pelo incômodo de ter que declarar o tamanho total do arquivo em sua criação. Porém voltou com mídias em disco como o CD
  * Mostra a importância de estudar sistemas antigas pois podem voltar a ser implementados em sistemas futuros

### Alocação em lista ligada

* Primeira palavra do bloco aponta para o bloco físico seguinte do arquivo, o resto são dados
* Todo bloco de disco pode ser usado evitando fragmentação (ocorre fragmentação apenas no último bloco de cada arquivo)
* A entrada do diretório só precisa armazenar o endereço em disco do primeiro bloco
* **Desvantagens**:
  * Acesso aleatório é lento: para acessar o bloco `n`, SO tem que começar no começo do arquivo e ler os `n-1` blocos anteriores
  * Armazenamento no bloco não é mais uma potência de dois pois o ponteiro ocupa alguns bytes

### Lista ligada com tabela na memória

* Elimina as desvantagens da alocação em lista ligada convencional
* **FAT (File Allocation Table)**: cadeia de blocas na tabela, terminados por `-1`
  * Exemplo: Arquivo nos blocos `4, 7, 2, 10, 12`
    * Entrada no índice 4 da tabela contém o número `7`. Entrada de índice 7 contém 2, assim vai até entrada no índice 12 contendo `-1`, encerrando a cadeia do arquivo
  * Elimina a desvantagem do bloco precisar conter ponteiros, pois neste caso o bloco é todo disponível para dados. Também facilita o acesso aleatório pois a corrente está toda na memória e pode ser seguida sem fazer referências de disco
  * Ainda suficiente a entrada no diretório manter apenas um inteiro (número do bloco inicial do arquivo) e ser capaz de localizar todos blocos
  * **Desvantagem**: a tabela inteira precisa estar na memória todo tempo
    * Com 20 GB de disco e bloco de 1 KB, a tabela precisa de 20 milhões de entrada (cada entrada contendo). A 4 bytes por entrada, ocupa 80 MB de memória
    * A tabela pode ser paginada na memória, mas ainda ocuparia memória virtual, espaço em disco e gerando tráfego de paginação
  * Usado no MS-DOS e Windows 98 (suporte nas versões mais recentes do Windows)

### I-nodes

* Estrutura **i-node (nó-índice)**: lista os atributos e endereços de disco dos blocos do arquivo
* Tendo o i-node é possível encontrar todos blocos do arquivo
* **Vantegem**: i-node presente na memória apenas quando o arquivo correspondente está aberto
  * Com `n` bytes por i-node e um máximo de `k` arquivos abertos de uma vez, apenas `kn` bytes precisam ser alocados com antecedência
* **Problema**: se o tamanho do arquivo crescer além do número fixado de endereços de disco por i-node
  * Os últimos endereços de disco podem ser reservados para apontar para **blocos indiretos**, que contém mais endereços de blocos de disco. Tais blocos indiretos podem ser estendidos para **blocos indiretos duplos** e **blocos indiretos triplos**

## 5.3.3 - Implementando Diretórios

* Arquivo precisa ser aberto para ser lido
* Ao abrir, o SO usa o path para localizar a entrada de diretório (necessita que o diretório raiz esteja localizado - local fixo relativo ao começo da partição)
* Diretório raiz pode ser obtido do superblock no sistema de arquivos do UNIX, que contém informação sobre o tamanho das estruturas de dados do sistema de arquivos que precedem a área dos dados
  * Possível encontrar a localização dos i-nodes: primeiro aponta para o diretório raiz
* No Windows XP, informação do setor de boot localiza o **MFT (Master File Table)** (localiza outras partes do sistema de arquivos)
* Ao localizar o diretório raiz, procura-se a entrada de diretório do arquivo através da árvore de diretórios que provê informação para encontrar os blocos de disco do arquivo (endereço do arquivo, primeiro bloco ou do i-node)
* **Sistema de diretório**: mapeia o nome ascii do arquivo na informação necessária para localizar os dados
* Armazenamento dos atributos dos arquivos: podem ser armazenados na respectiva entrada do diretório ou nos i-nodes em sistemas que os usam

### Arquivos compartilhados





