# 4.3 - Memória Virtual

* Programas grandes demais para a memória: divisão em **overlays**
  * Começo do zero
  * Ao terminar, chama o próximo
  * Armazenados no disco, swap pelo SO
  * Trabalho de criar os overlays pelo programador e não SO
* **Memória virtual**: trabalho de dividir o programa delegado ao SO
  * Tamanho do programa, dados e pilha podem exceder a memória física: partes em uso na memória principal e o resto em disco
  * Multiprogramação: ao aguardar outro pedaço ser trazido do disco, o programa aguarda por I/O e o tempo de CPU pode ser dado a outro

## 4.3.1 - Paginação

* **Endereços virtuais** são gerados por programas e formam o **espaço de endereço virtual**
* **Sem memória virtual**, a palavra da memória física de mesmo endereço que o endereço virtual é lida (endereço virtual diretamente no barramento de memória)
* **Com memória virtual**: endereço virtual para **MMU (Memory management unit)**, endereços virtuais para endereços físicos
* **Páginas**: divisões do espaço de endereços virtuais
  * Unidades correspondentes na memória física são os **page frames**
  * Páginas e page frames têm mesmo tamanho
* **Exemplo de funcionamento da MMU**
  * Figura 4.8
  * Instrução `MOV REG, 0` envia o endereço virtual zero para a MMU
  * A MMU checa que tal endereço na página 0 (de 0 a 4095) está mapeada no page frame 2 (8192 a 12287), transformando o endereço zero em 8192 para o barramento de memória
  * O **bit presente/ausente** indica quais páginas estão na memória física
  * Se a página não estiver presente na memória física, a CPU interrompe o SO (**trap** chamada **page fault** - **falta de página**), remove o page frame menos usado da física e escreve a página referenciada necessária, alterando o mapa
* **Funcionamento interno MMU**
  * Figura 4.9
  * Exemplo de mapeamento do endereço virtual `8196` (`0010000000000100`)
    * 16 bits compostos por 4 para o número da página e 12 de offset (16 páginas e 4096 bytes por página)
    * O número da página é usado como índice na **tabela de páginas**, se o bit de presença for zero, gera interrupção. Senão, os 3 bits do número do page frame são copiados para o registrador de saída junto com os 12 de offset formando um endereço físico de 15 bits

## 4.3.2 - Tabela de páginas

* Tabela de páginas mapeia páginas virtuais em page frames
  * Função com o número da página virtual como argumento e o número do frame físico como resultado
* **Problemas**
  1. A tabela de páginas pode ser muito grande
     * Com página de 4KB e espaço de endereço de 32 bis, há 1 milhão de páginas no espaço de endereço virtual (1 milhão de entradas na tabela de páginas) e cada processo precisa da sua própria tabela de páginas (têm seu próprio espaço de endereços virtual)
  2. O mapeamento precisa ser rápido
     * Pois o mapeamento é feito em toda referência à memória
* **Design mais simples conceitualmente**
  * Única tabela de páginas (array de registradores)
  * Uma entrada para cada página virtual
  * SO carrega os registradores com a tabela de páginas (retirada de uma cópia na memória principal) quando o processo inicia
  * **Vantagens**: simples e não precisa de referência de memória durante o mapeamento
  * **Desvantagens**: caro (se a tabela de páginas é muito grande) e carregar toda a tabela a cada context switch pode afetar a performance
* **Toda tabela de páginas na memória**
  * Único registrador para apontar para o começo da tabela na memória principal
  * Permite alterar o mapa de memória em um context switch apenas alterando o registrador
  * **Desvantagem**: necessita referências de memória para ler as entradas da tabela

### Tabelas de página multinível

* Evita armazenar tabelas gigantes na memória

* **Exemplo**

  * Endereço de 32 bits: campo `PT1`com 10 bits, campo `PT2` com 10 bits e `offset` com 12 bits (por conta desse offset, páginas tem 4 KB, com um total de `2^20` delas)
  
  * Exemplo de processo que necessita 12 MB: 4 MB embaixo para texto do programa, 4 MB para dados e no topo 4 MB para pilha (buraco entre o topo dos dados e fundo da pilha)
  
  * **Tabela de páginas de nível superior**: 1024 entradas (10 bits do `PT1`)
    * MMU extrai o campo `PT1` quando recebe um endereço virtual e usa como índice na tabela de nível superior
    * Cada uma das 1024 entradas representam 4 MB pois divide o espaço de endereços virtual de 4 GB (32 bits, `2^32`) em 1024 pedaços
    
  * A entrada da tabela de nível superior gera um endereço para uma **tabela de páginas de segundo nível**
  
  * Entrada `0` da tabela de nível superior aponta para a tabela de páginas do texto do programa, entrada `1` para a tabela dos dados e entrada `1023` para a tabela da pilha
  
  * `PT2` é usado como índice na **tabela de página de segundo nível** para localizar o respectivo **page frame**
  
  * **Exemplo do endereço `0x00403004`**
    * Em binário: `0000000001 0000000011 000000000100`
      * `PT1 == 0d01`
      * `PT2 == 0d02`
      * `Offset == 0d04`
    * Em decimal: `0d4206596` (`12292` bytes dentro dos dados)
    * `MMU` usa `PT1` para obter a entrada `1` dentro da tabela de nível superior (endereços de 4M a 8M). Depois usa `PT2` como índice na tabela de segundo nível para obter a entrada `3` (`12288` a `16383` dentro do bloco de 4M, endereços absolutos `4206592` a `4210687`), que contém o número do **page frame** para o dado endereço. Se o bit de presença na entrada da tabela de páginas for zero, ocorre page fault. Caso contrário, combina-se o número do page frame na tabela de segundo nível com o offset para montar o endereço físico, colocado no barramento e enviado à memória
    
  * O espaço de endereços virtuais contém mais de um milhão de páginas, mas apenas 4 tabelas são necessários: nível superior, e tabelas de segundo nível de 0 a 4M, 4M a 8M (embaixo, entradas 0 e 1) e os 4M superiores (entrada 1023)

  
### Estrutura de uma entrada na tabela de páginas

* Layout da entrada varia de máquina a máquina, mas o tipo de informação é aproximadamente o mesmo
* Tamanho comum da entrada: 32 bits
* **Componentes**
  * Espaço vazio
  * Uso de cache desativado
    * Permite desativar o uso de cache
    * Importante para páginas que mapeiam em registradores ao invés da memória
    * Se o SO aguarda pela resposta de um dispositivo I/O, é essencial que o hardware continue pegando a palavra do dispositivo e não uma cópia cacheada
    * Máquinas com espaço separado para I/O e que não I/O mapeado na memória não precisam desse bit
  * Referência
    * Indica se a página está sendo usada (escrita ou leitura)
    * Ajuda o SO a decidir qual página remover da memória quando ocorre page fault, de preferência uma sem uso corrente
  * Modificação
    * **Dirty bit**, reflete o estado da página
    * Se a página foi modifica, recebe um e precisa ser escrita de volta no disco, do contrário pode ser apenas abandonada por ser igual a que está em disco
  * Proteção
    * Indica os tipos de acessos permitidos
    * Com um bit: 0 para leitura e escrita e 1 para apenas leitura
    * Com 3 bits, cada um habilita leitura, escrita e execução
  * Bit de presença/ausência
    * Indica se a página virtual encontra-se na memória
  * Número do page frame
    * Localiza o endereço na memória principal
* A tabela de páginas armazena apenas informação necessária para a tradução de um endereço virtual em físico. O endereço em disco que armazena a página quando não está na memória fica em tabelas dentro do SO pois o hardware não precisa dele

## 4.3.3 - Translation Lookaside Buffers (TLB)

* **Localidade de referência**: grande número de referências a um número pequeno de páginas

  * Pequena porção das entradas na tabela de páginas são muito lidas, o resto acaba sendo pouco usado

* **TLB (Translation Lookaside Buffer)** ou **Memória associativa**: dispositivo de hardware para mapear rapidamente endereços virtuais para endereços físicos sem passar pela tabela de páginas

  * Geralmente dentro da **MMU**
  * Pequeno número de entradas (de 8 a 64)
  * Cada entrada contém campos como:
    * Válido
    * Página virtual
    * Modificação
    * Proteção
    * Page frame

* **Funcionamento TLB**: endereço virtual apresentado para a MMU é checado em paralelo para ver se o seu número da página virtual está presente no TLB

  * Se é encontrado e o acesso não viola a proteção. a page frame é tomado diretamente do TLB, sem passar pela tabela de páginas
  * Se é encontrado no TLB mas o acesso violaria a proteção, gera um **protection fault** (erro de proteção)
  * Se a página virtual não está no TLB, o MMU procura na tabela de páginas, remove uma das entradas do TLB e coloca a nova entrada, recém procurada
  * Ao remover uma entrada do TLB, o bit de modificação é copiado para a respectiva entrada na tabela de páginas na memória

  

### Gerenciamento do TLB por software

* Com gerenciamento de páginas por softwarem as entradas da TLB são explicitamente carregadas pelo SO
* Para reduzir as faltas na TLB, o SO pode tentar adivinhar quais páginas são mais prováveis a serem usadas a seguir, carregando previamente
* Processar um TLB miss significa ir na tabela de páginas e localizar a página faltante, porém as páginas que comportam a tabela de páginas podem não estar na TLB, gerando novos TLB faults. Para evitar isso pode-se manter um cache grande (4KB ou maior) de entradas TLB em um local fixo cuja página do cache seja sempre mantida na TLB. Olhando primeiro o cache de software, o SO reduz o número de TLB miss

### 4.3.4 - Tabela de páginas invertida

* Em espaços de endereço virtuais de 32 bits (2^32 bytes, com 4096 bytes por página), são necessários mais de um milhão de entradas. Resulta em uma tabela de páginas de 4 MB
* Em computadores de 64 bits, o espaço de endereços tem 2^64 bytes, com páginas de 4 KB resulta em uma tabela de 2^52 entradas. Cada entrada a 8 bytes, tomaria mais de 30 milhões de GBs para a tabela
* **Tabela de páginas invertida**: uma entrada por page frame na memória real ao invés de uma entrada por página do espaço de endereços virtual
  * Endereços virtuais de 64 bits, com páginas de 4KB e 256 MB de RAM necessitaria 65536 entradas na tabela invertida
  * A entrada armazena qual (processo, página virtual) está localizada no page frame
* **Vantagem**: economiza muito espaço na memória
* **Desvantagem**: tradução do endereço virtual para físico se torna mais difícil
  * Processo `n` ao fazer referência à página virtual `p`: hardware precisa procurar em toda a tabela invertida por uma entrada `(n, p)`
  * Busca necessária em toda referência à memória, não apenas em page faults. Demorado percorrer uma tabela de 64K a cada referência de memória
    * Solução: usar TLB para armazenar as páginas mais usadas. Porém ainda precisa-se pesquisar na tabela quando dos TLB misses. Neste caso, uma hash table pode ajudar a acelerar a busca

  

  