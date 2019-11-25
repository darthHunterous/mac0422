# 5.2 - Diretórios

* **Diretórios** ou **pastas** ordenam arquivos, sendo eles mesmo arquivos

## 5.2.1 - Diretórios simples

* Cada diretório contém uma entrada para cada arquivo dentro dele
  * Duas possibilidades
    1. Cada entrada contém o nome, atributos e endereços no disco
    2. Cada entrada guarda o nome e um apontador para outra estrutura de dados com os atributos e endereços no disco
* Ao abrir um arquivo, o SO procura no diretório deste arquivo até encontrar o arquivo a ser aberto, extraindo atributos e endereços de disco, colocados na memória principal. Referências subsequentes usam a informação na memória

### Tipos de sistemas de arquivos

1. Único diretório compartilhado por todos usuários
   * Problema: se dois usuários criarem um arquivo de mesmo nome, o mais recente sobrescreverá o mais antigo
2. Um diretório por usuário
   * Usuário só pode acessar arquivos no seu próprio diretório
   * Requer procedimento de login para diferir usuários

## 5.2.2 - Sistemas de diretório hierárquicos

* Permite uma árvore arbitrária de diretórios, podendo o usuário organizar arquivos de maneira mais natural

## 5.2.3 - Nomes do caminho

* Dois métodos para especificar nomes de arquivo em um sistema de arquivos organizado como árvore de diretórios
  1. **Nome do caminho absoluto**: caminho do diretório raiz até o arquivo
     * `/usr/ast/mailbox`: diretório usr dentro do raiz. usr contém um diretório ast onde está o arquivo mailbox
     * Separador de diretório
       * UNIX: `/`
       * Windows: `\`
     * Se o primeiro caracter do path é o separador, então o path é absoluto
  2. **Nome do caminho relativo**: depende do **diretório de trabalho** (**diretório corrente**), que pode ser designado pelo usuário
     * Paths são relativos ao diretório de trabalho
     * Exemplo: diretório de trabalho `/usr/ast` então o arquivo `/usr/ast/mailbox` pode ser referenciado apenas pelo nome `mailbox`
* Cada processo tem seu próprio diretório de trabalho, portanto se ele modifica e depois se encerra, não afeta outros processos o
* **Entradas especiais**
  * `.` - diretório atual
  * `..` - pai do diretório atual

## 5.2.4 - Operações sobre diretórios

1. `Create`: cria um diretório vazio, contendo apenas `.` e `..` (automaticamente pelo UNIX ou `mkdir`)
2. `Delete`: deleta um diretório, apenas vazios podem ser deletados (diretórios vazios contém apenas `.` e `..`)
3. `Opendir`: diretórios podem ser lidos (lista todos arquivos nele, por exemplo), mas antes precisa abrir
4. `Closedir`: após leitura, o diretório deve ser fechado para economizar espaço na tabela interna
5. `Readdir`: retorna a próxima entrada um diretório aberto
6. `Rename`: renomeia o diretório, como um arquivo
7. `Link`: permite um arquivo aparecer em mais de um diretório
   * Especifica um arquivo existente e um path, criando link entre o arquivo e nome especificado no path. **Hard link**
     * Incrementa o contador no i-node do arquivo (contador deo número de entradas de diretório contendo o arquivo)
8. `Unlink`: remove a entrada do diretório. Se está presente apenas em um, o arquivo é removido do sistema de arquivos. Se está presente em vários, apenas o especificado é removido
   * Mesma chamada para deletar arquivos