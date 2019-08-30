# 1.2 - História de SOs

* Máquina analítica de Babbage, sem SO. Porém Babbage não conseguiu na precisão que precisava por limitações de engenharia
  * Ada Lovelace contratada para programar para sua máquina

### 1.2.1 - Primeira Geração: válvulas e painéis

* 1945 a 1955
* Gigantescos e lentos
* Programação em linguagem de máquina, cabeamento no painel de controle para controlar as funções básicas
* No começo dos 50s, surgem cartões perfurados, porém o procedimento continuava o mesmo

### 1.2.2 - Segunda Geração: transistores e sistemas de lote

* 1955 a 1965
* Transistor em meados dos 50s, confiabilidade ao computador pois não queimavam como as válvulas
* **Mainframes** operados por pessoas especializadas, custando milhões
  * **Jobs** executados através de cartões perfurados
  * Tempo gasto com operadores transitando pela sala
* Para evitar tempo gasto, **sistemas de lotes**:
  * Na sala de input, cópia dos cartões para fita magnética (IBM 1401, barato)
  * Operador traz essa fita magnética para computadores mais poderosos que faziam cálculos (IBM 7094, caro)
  * Outro operador pega a saída e leva a outro computador barato para imprimir a saída (IBM 1401)
* Computadores dessa geração eram usados em cálculos científicos e de engenharia (equações diferencias parciais), com FORTRAN e Assembly. OSs como FMS (Fortran Monitor System) e IBSYS (da IBM)

### 1.2.3 - Terceira Geração: circuito integrado, multiprogramação

* Fabricantes com duas linhas de computadores: uma voltada a cálculos científicos (caros) e outro mais barato que se baseavam em caracteres para ordenação e impressão
  * Arquiteturas diferentes, caro para manter e incompatibilidade entre si	

* Série 360 da IBM, com mesma arquitetura e conjunto de instruções, variando em preço e performance. Primeira linha a implementar circuito integrado
  * Técnica de **multiprogramação**: CPU não fica inativa esperando operações de input acabarem
    * Em questões científicas o overhead de entrada é pequeno, porém para uso de processamento de dados comerciais, poderia compor 90% do tempo de uso
    * Solução de particionar a memória com jobs diferentes em cada partição, pois enquanto aguarda o final de input, outro job usa a CPU
  * Técnica de **Spooling** (Simultaneous Peripheral Operation On Line): ler jobs de cartões imediatamente, que ficavam esperando o término de outros jobs para ocupar a partição de memória do job que terminou
* Técnica de **Timesharing**: cada usuário tem um terminal online e enquanto eles aguardam, a CPU é alocada entre os jobs que desejam ser executados
  * Sistema **MULTICS**
    * www.multicians.org
* Computadores em rede (PC ou **workstation**) conectados em rede via **LAN** a um **file server** onde todos programas e dados são armazenados
  * **Middleware**: fornece o meio entre usuários locais e dados nos servidores remotos, fazendo computadores em rede parecerem locais ao usuários de cada PC, apresentando uma interface consistente
    * Middlewares aparentam ser o OS de um **sistema distribuído** (na prática, não são)
* **UNIX** origina no Bell Labs após Ken Thompson encontrar um PDP-7 sem uso e escreveu uma versão simplificada, de um usuário, do MULTICS
  * Várias empresas desenvolvem versões do UNIX incompatíveis entre si (**System V** da AT&T e **BSD** na Universidade de Berkeley) até que IEEE desenvolve o padrão **POSIX** com uma interface de chamadas de sistema bem definidas
    * Definições POSIX disponíveis em www.unix.org

### 1.2.4 - Quarta Geração: computadores pessoais

* De 1980 até agora
* Era do **microprocessador** com o desenvolvimento de circuitos LSI (Large Scale Integration), colocando milhares de transistores em uma pequena área
* Minicomputadores fizeram que empresas e universidade pudessem ter um, enquanto que **microcomputadores** tornaram acessível o uso individual
* 1974: Intel 8080, 8 bits concorrente do Motorola 6800
  * **CP/M** como SO
* Começo dos 80s: Intel 8086, 16 bits
  * Interpretador de BASIC e **DOS** (Disk Operating System) renomeado para **MS-DOS** 
  * **GUI (Graphical User Interface)** , de pronúncia "gooey", inventado por Doug Engelbart em Stanford. Antes apenas sistema de linha de comando.
    * Steve Jobs viu como **user-friendly** para anunciar o Apple Macintosh em 1984 (Motorola 68000 de 16 bits e 64 KB de **ROM** - Read Only Memory para rodar a GUI)
* 2001: **Mac OS X**, nova versão da Macintosh GUI em cima do UNIX da Berkeley
* 2005: Apple muda de processadores Motorola para Intel
* Microsoft inventa Windows para concorrer com o Macintosh. No começo, interface gráfica em cima do MS-DOS de 16 bits. Hoje em dia, Windows descendem do Windows NT, de 32 bits
* Programadores experientes preferem linha de comandos porém UNIX possuem o **X Window** para o gerenciamento básico de janelas, podendo-se instalar uma GUI completa como **Motif** 

* A partir de meados dos 80s, cresce PCs rodando **sistemas operacionais em rede** e **sistemas operacionais distribuídos**

  * Em rede: máquinas independentes rodando SO localmente e possuindo seus próprios usuários locais, usuários conseguem enxergar múltiplos computadores, logar remotamente e copiar arquivos entre máquinas
* Distribuído: usuários não sabem onde seus programas estão executando ou onde os arquivos estão

### 1.2.5 - História do MINIX 3

* 1987: lançado MINIX
* 1997: MINIX 2 e segunda edição do livro
* Diferenças do 1 para 2: de 16 bit modo real em 8088 com disquetes para 32 bits em modo protegido em 386 com HD
* MINIX 3 redesenha o sistema, reestruturando o kernel, reduzindo o tamanho e ampliando modularidade e confiabilidade
  * Menos de 4000 linha de código
  * Tamanho pequeno de kernel para conter menos bugs (estudo revelou que 1000 linhas de código podem conter de 6 a 16 bugs conhecidos, obviamente sem contar os bugs não descobertos)
  * Estudo mostra que em média 6% dos arquivos contém bugs e que após várias versões, o número se estabiliza ao invés de ir para zero
  * Em MINIX 3 os drivers ficam fora do kernel com o intuito de causar menos dano assim
* Linus Torvalds era um estudante universitário quando estudou o código do MINIX e de acordo com a necessidade ler grupos da USENET no seu próprio PC, começou a modificar porções do MINIX até que o desenvolvimento gerou um kernel primitivo em Agosto de 1991. Outras pessoas o ajudaram e em Março de 1994, Linux 1.0 é lançado
* Ferramentas **open-source** **LAMP**: Linux, Apacha, MySQL e Perl/PHP