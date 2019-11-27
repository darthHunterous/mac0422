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

* i-nodes tornam fácil compartilhar arquivos com **links**

  * Várias entradas de diretório podem apontar para um mesmo i-node, que contém um contador de links, quando zera o i-node e os dados são deletados (**hard link**)

* **Limitação de hard link**

  * Diretórios e i-nodes são estruturas de um único sistema de arquivos, não podem apontar para i-node em outro sistema
  * Arquivo com apenas um dono e um conjunto de permissões: se o dono deleta a entrada no seu diretório, outro usuário pode ficar com o arquivo preso no diretório dele se não for permitida a deleção

* **Segundo tipo de link**

  * **Link simbólico** (UNIX), **atalho** (Windows) e **alias** (Mac)

  * Os dados são o path para outro arquivo (funciona entre vários sistemas de arquivos montados)

  * Pode ser usados em sistemas com atributos armazenados dentro das entradas de diretório

  * **Desvantagem**: link se torna órfão quando o arquivo é deletado ou renomeado

### Diretórios no Windows 98

* **Entrada base do diretório**
  * Nome base: 8 bytes
  * Extensão: 3 bytes
  * Atritbutos: 1 byte
  * NT: 1 byte
  * Segundos: 1 byte
  * Data e hora de criação: 4 bytes
  * Último acesso: 2 bytes
  * Primeiros 16 bits do bloco de início: 2 bytes
  * Data e hora da última escrita: 4 bytes
  * Últimos 16 bits do bloco de início: 2 bytes
  * Tamanho do arquivo: 4 bytes
* 10 bytes começando com NT (NT, segundos, data de criação, último acesso e 16 bits superiores do bloco de início) são adições da estrutura do Windows 95
  * Aumenta a quantidade de bits para apontar para o bloco inicial de 16 para 32 bits (máximo tamanho do sistema de arquivos vai de 2^16 blocos para 2^32)
* Nomes de arquivo com 8 caracteres bases mais 3 de extensão (compatibilidade com os sistemas antigos)
* **Nomes de arquivo maiores**: outra tipo de entrada alternativa
  * **Entrada**
    * Sequência: 1 byte
    * 5 caracteres: 10 bytes
    * Atributos: 1 byte
    * 0: 1 byte
    * Checksum (soma de verificação): 1 byte
    * 6 caracteres: 12 bytes
    * 0: 2 bytes
    * 2 caracteres: 4 bytes
  * Várias entradas dessa podem ser usadas para conter o nome, conforme necessário, colocada antes da entrada base do arquivo, em ordem reversa
  * Campo `Atributos`: contém valor `0x0F`, impossível para sistemas de arquivos antigos, sendo ignorados
  * Campo `Sequência` diz qual é a última entrada

### Diretórios no UNIX

* Estrutura simples
  * Número do i-node: 2 bytes
  * Nome do arquivo: 14 bytes
* Informações no i-node
* Abrir um arquivo significa pegar o nome e localizar os blocos de disco
  * Exemplo `/usr/ast/mbox`
    * Localiza o diretório raiz (primeira entrada no array de i-nodes, localizado com as informações do superbloco)
    * Procura o primeiro componente `usr` no diretório raiz para encontrar i-node do arquivo `/usr/`, que diz em qual bloco está o arquivo
    * Procura no bloco a entrada do arquivo seguinte `ast` para achar seu i-node, que irá dizer em qual bloco está a entrada de diretório para `/usr/ast/`, sendo então possível localizar o i-node do arquivo `mbox`
* Paths relativos funcionam da mesma forma, mas começam do diretório de trabalho (atual) em vez do raiz
  * Entradas `.` e `..` são colocadas quando o diretório é criado (são strings ASCII como qualquer outro nome de arquivo ou diretório)

### Diretórios em NTFS

* **NTFS (New Technology File System)**

  * Permite nomes de arquivos até 255 caracteres
* Nome de path até 32767 caracteres
  * Segundo nome no formato 8 + 3 caracteres para que sistemas mais antigos leiam arquivos NTFS via rede
* Unicode para nome de arquivos (16 bits por caracter)
  
  * Problemas de especificidades de cada linguagem: NTFS resolve com um atributo para o arquivo que define as convenções de caixa (alta ou baixa) para a linguagem definida no nome do arquivo
  * Em UNIX arquivos são sequência de bytes, em NTFS é uma coleção de atributos e atributos são fluxos de bytes
* **MFT (Master File Table)**: estrutura básica NTFS, 16 atributos (cada até 1 KB dentor da MFT)
    * **Atributo não residente**: um atributo na MFT pode apontar para um arquivo adicional com extensão dos valores do atributo
    * MFT é um arquivo, possui uma entrada para cada arquivo e diretório no sistema de arquivos
      * Pode crescer muito, criado com 12.5% do espaço da partição (cresce sem fragmentar). Se o espaço inteiro for usado outro grande espaço é reservado
        * A fragmentção da MFT consiste de poucos fragmentos muito grandes
    * Dados são um atributo (fluxo de dados)
    * **Arquivo imediato**: alguns poucos bytes no header do atributo para representar arquivos pequenos

## 5.3.4 - Gerenciamento de espaço em disco

* Armazenar um arquivo: em bytes consecutivos ou alocados em blocos do disco
  * Em bytes consecutivos, se o arquivo crescer pode ter que se mover para outro local do disco (demorado para disco)

### Tamanho do bloco

* Tamanho do setor, trilha ou cilindro são candidatos a tamanho de bloco (porém variam entre dispositivos)
* Tradeoff: unidade de alocação grande gasta espaço e pequena é lenta (seek e atraso rotacional)
* Mediana de arquivos no UNIX é cerca de 2 KB (mediana é melhor que a média, pois evita que um pequeno número de arquivos afeta o valor significativamente), supondo todos arquivos com esse tamanho e que o tempo de leitura de um bloco é a soma do seek, atraso rotacional e tempo de transferência, temos as curvas da figura **5.17** (porcentagem de utilização efetiva do disco e taxa de dados)
  * Se cruzam por volta do tamanho de bloco perto de 4 KB, onde a porcentagem de utliização do disco é cerca de 40% e a taxa de dados é 400 KB/s, sendo essa uma possível boa escolha
* No UNIX o tamanho comum é 1 KB e no MS-DOS varia de 512 bytes a 32 KB (determinado pelo tamanho do disco pois o número máximo de blocos é 2^16, forçando blocos grandes em discos grandes)

### Monitorando blocos livres

* Dois métodos: lista de regiões livres em uma lista ligada ou bitmap
* **Lista ligada de blocos de disco**
  * Cada bloco armazena quantos números de blocos de discos livres quanto couberem (a 1 KB de bloco e número de bloco de disco de 32 bits, cada bloco da lista pode armazenar 255 blocos livres + 1 espaço para o apontador do próximo bloco da lista)
  * Exemplo: disco de 256 GB precisaria de no máximo 1052689 blocos para armazenar todos os números de seus 2^28 blocos de disco
* **Bitmap**
  * Disco com `n` blocos precisa de bitmap com `n` bits
  * Bit zero representa blocos alocados e 1 representa lvires
  * Exemplo: 256 GB tem 2^28 blocos de 1 KB e precisa de 2^28 bits, ou seja, 32768 blocos para o bitmap
  * Requer apenas 1 bit por bloco, comparado com os 32 por bloco da lista ligada
* Com o método da lista ligada, apenas um bloco de ponteiros é mantido na memória
  * Ao criar arquivo, os blocos são pegos do bloco de ponteiros. Ao acabar, um novo bloco de ponteiros é lido do disco
  * Ao deletar arquivo, seus blocos são adicionados ao bloco de ponteiros na memória principal. Se encher, são escritos no disco

## 5.3.5 - Confiabilidade do sistema de arquivos

* Disquetes geralmente são perfeitos ao saírem da fábrica, mas tendem a desenvolver bad blocks (blocos defeituosos) com o uso. A poeira que entra no drive de disquete também pode danificar um disquete
* HDs possuem bad blocks de início (caro demais fabricar sem), por isso possuem setores sobrassalentes nas trilhas. Um ponto defeituoso pode ser pulado deixando um vão entre dois setores consecutivos. O controlador faz esse remapeamento automaticamente, sem conhecimento do usuário
  * Discos SCSI podem fornecer um erro ao remapear um bloco, alertando o usuário para trocar o HD eventualmetne
  * Solução de software: sistema de arquivos constrói um arquivo contendo todos bad blocks, removendo-os da lista de blocos livres

### Backups

* **Lixeira**: diretório especial para permitir que usuário restaure arquivos que outrora queria deletar mas mudou de ideia
* Backups levam muito tempo e espaço
  * Não há necessidade de backup de binários, arquivos temporários e especiais (como `/dev/`)
* **Backups incrementais**: backup apenas dos arquivos que foram modificados desde o último
  * Restauração mais complicada: primeiro restaurar o último backup completo depois os incrementais em ordem reversa
* Os dados podem ser comprimidos antes do backup, mas pontos defeituosos podem prejudicar o algoritmo de descompressão, tornando dados ilegíveis
* É difícil fazer backup de um sistema de arquivos ativo, pois modificações vão ocorrendo em tempo real. Algoritmos de backup podem tirar snapshots do sistema atual e ao fim do backup requerer modificações em cima disso
* Backups também geram problemas de segurança físicos: informações sensíveis precisam ser protegidas tanto quanto o original, mas devem estar protegidas de eventos que destruam não só o original quanto o backup (incêndios)

### Copiar disco em fita

* **Duas estratégias**:
  * **Cópia física**: começa no bloco zero do disco e escreve todos os blocos em ordem na fita de saída
    * Não há necessidade de copiar blocos livres, porém acessar a estrutura de blocos livres e pulá-los significa que não haverá correlação um pra um dos blocos no disco e na fita, sendo necessário armazenar o número de cada bloco em seu início
    * Os bad blocks precisam ser evitados também
    * **Vantagens**: simples e rápida
    * **Desvantagens**: não é possível evitar o backup de certo diretórios, fazer cópias incrementais e restaurar arquivos individuais
  * **Cópia lógica**: começa em um diretório e copia todos arquivos recursivamente que foram alterados desde uma data específica
    * Para restaurar: toda informação para recriar o path do arquivo precisa ser salva, incluindo todos diretórios, até mesmo os não modificados
    * **Problemas**:
      * A lista de blocos livres não é copiada e precisa ser reconstruída após uma restauração
      * Se um arquivo é linkado a vários diretórios, deve ser restaurado apenas uma vez
      * Arquivos em UNIX podem conter buracos (função seek para pular um offset) e não devem ser copiados e restaurados
    * Arquivos especiais nunca devem ser copiados

### Consistência dos sistema de arquivos

* Se o sistema crashar antes que blocos modificados sejam escritos de volta no disco, pode gerar inconsistência
* SOs possuem utilitários para checar a consistência: UNIX com `fsck` e Windows com `chkdsk` (`scandisk` nas versões mais antigas)
* **Dois tipos de checagem de consistência**
  * **Consistência de blocos**: duas tabelas com contadores para cada bloco, inicializados em zero
    * Primeira tabela: quantas vezes cada bloco está presente em um arquivo
    * Segunda tabela: quantas vezes cada bloco está presente na lista de blocos livres
    * Se o sistema de arquivos está consistente, cada bloco nas duas tabelas terá valor `1` após o programa checar todos i-nodes e a lista de blocos vazios
    * **Inconsistências**
      * Blocos zerados após a checagem são **blocos ausentes**, que desperdiçam espaço, portanto o verificador os adiciona de volta na lista de blocos livres
      * Blocos duplicados na lista de blocos livres: necessário reconstruir a lista
      * Bloco duplicado na lista de arquivos: pior cenário. Se um dos arquivos for removido, o bloco duplicado estará ao mesmo tempo em ambas tabelas. Se ambos arquivos forem removidos, constará duas vezes na tabela de blocos livres
        * Solução: copiar o conteúdo do bloco duplicado em um novo que estava livre e inserir este novo em um dos dois arquivos substituindo o duplicado. Um dos dois arquivos estará com problemas de consistência, mas o sistema de arquivos estará consertado e consistente. (Reportar o usuário sobre o problemas nos arquivos)
  * **Consistência de diretórios**: tabela de contadores para cada arquivo
    * Varre toda a árvore aumentando o contador para cada arquivo encontrado
      * Hard links podem fazer com que um arquivo apareça em mais de um diretório (soft links não interferem)
      * Ao finalizar a varredura, há uma lista indexada pelo número de i-node contando quantos diretórios contém cada arquivo, que pode ser comparada com a contagem de links no próprio i-node
      * **Dois tipos de erros**: contagem maior ou menor
        * Links maior que o número de entradas de diretório: se todos arquivos forem removidos, a contagem ainda será diferente de zero e o i-node não é deletado, gastando espaço em disco. Corrigido ajustando a contagem no i-node para o valor correto
        * Links menor que o número de entradas de diretório: catastrófico
          * Ao remover uma entrada de diretório deste arquivo, a contagem de i-node pode chegar a zero ainda havendo pelo menos uma entrada de diretório, que apontará para um i-node não usado (que pode até ser atribuído a outro arquivo em breve)
          * Solução: forçar a contagem de i-nodes igualar o número de entradas de diretório
  * As duas checagens podem ser integradas por eficiência (varrer os i-nodes apenas uma vez)

## 5.3.6 - Performance do sistema de arquivos

* A memória é muito mais rápida que o disco, que precisa de otimizações de performance

### Uso de cache

* **Cache de bloco** ou **cache de buffer**: blocos pertencentes ao disco que são mantidos na memória
* Algoritmos para gerenciar o cache: checar requisições de leitura e ver se o bloco está no cache. Se sim, evita o acesso a disco. Caso contrário, o bloco é lido do disco para o cache e aí utilizado
  * Para determinar se o bloco está presente no cache pode-se usar uma tabela de hash
  * Com cache cheio, um bloco tem de ser removido e escrito de volta para o disco se foi modificado. (Similar a paginação, podendo usar os mesmos algoritmos. A diferença é que referência do cache do disco é menos frequente que em paginação, portanto é fácil manter os blocos em LRU na lista ligada)
  * A lista na tabela de hash é bidirecional e ordenada do menor recentemente usado para o mais no final
  * LRU agora é alcançável mas não desejável pois se um i-node for modificado no cache, demorará até ser escrito de volta no disco (pois é colocado no final e só será removido ao chegar na frente da lista). Com o sistema crashando, isso causará inconsistência no sistema de arquivo
  * O ideal é modificar o LRU para que blocos que não serão necessários tão cedo vão na frente da lista para seus buffers serem reusados e blocos que podem ser necessários novamente vão no final da lista
  * Se o bloco é essencial para a consistência do sistema de arquivos (tudo exceto blocos de dados), deve ser escrito de volta ao disco imediatamente, reduzindo a chance de um crash quebar o sistema de arquivos
* Para evitar que blocos de dados fiquem no cache por muito tempo, há a chamada de sistema `sync` no UNIX, que escreve todos blocos do cache com modificações para o disco
  * Programa `update` automaticamente faz chamadas de `sync` a cada 30 segundos
* No Windows há os **cache de escrita direta** (**write-through**), onde todo bloco modificado é escrito para o disco imediatamente
* Essa diferença entre Windows e UNIX resulta que remover um disco removível no UNIX sem executar um `sync` resultará em dados perdidos e um sistema de arquivos corrompido, enquanto que no Windows não

### Leitura de bloco antecipada

* Ao ler o bloco `k` de um arquivo, o sistema de arquivos pode já ir pegando o bloco `k+1` para antecipar essa leitura seguinte
  * Só funciona se a leitura for sequencial, pois em acesso aleatório pode prejudicar a performance, ocupando largura de transferência do disco. A solução é o bit analisado pelo sistema de arquivos que indica se o arquivo está em modo de leitura sequencial ou acesso aleatório

### Reduzindo o movimento do braço do disco

* Usando bitmap para blocos livres, fica fácil escolher blocos livres próximos uns aos outros (preferência no mesmo cilindro)
* Outro gargalo é que ler um arquivo requer acesso ao i-node e também ao bloco
  * Colocando todos i-nodes sequencialmente, a distância entre o i-node e seus blocos vai ser de metade do número dos cilindros, tendo grandes seeks
    * Soluções:
      * Colocar i-nodes no meio do disco
      * Dividir o disco em grupos de cilindros, buscando alocar i-nodes, blocos e lista de blocos livres próximos uns aos outros

## 5.3.7 - Sistemas de arquivos estruturados em log

* Gargalo principal é tempo de busca em disco, apesar das melhorias em CPUs e crescimento do tamanho das memórias
  * Solução: **LFS (Log Structure file system)**
* Boa parte do trabalho do disco no UNIX são muitas escritas de arquivos pequenos, cujo tempo de execução são dominados pelo atraso
  * LFS estrutura o disco como um log e periodicamente o cache na memória é juntado em um segmento e escritos para o disco como um único segmento no fim do log
  * No começo do segmento há um resumo sobre ele
  * Se o segmento ocupar 1 MB, pode-se utilizar toda a banda do disco
  * i-nodes agora ficam espalhados pleo disco, mais difícil de encontrar, necessitando de um mapa de i-nodes (mantido no disco e no cache)
  * Quando o disco enche (pelo log ocupar todo disco), vários segmentos conterão blocos não mais necessários
    * Resolvido com uma **thread limpadora** que fica varrendo o log e compactando
    * O disco se torna um grande buffer circula, com a thread que escreve adicionando novos segmentos na frente e a thread limpadora removendo dos fundos

