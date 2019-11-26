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

