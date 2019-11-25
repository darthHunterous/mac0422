# 5.1 - Arquivos

## 5.1.1 - Nomes de arquivos

* Arquivos: abstrações para armazenar e recuperar informação do disco
* Ao criar um arquivo, o processo dá um nome. O arquivo continua a exisitir quando o processo termina
* Regras para nomenclatura variam entre SOs
* UNIX distingue entre letras maiúsculas e minúsculas, MS-DOS não
  * Sistemas Windows mais recentes suportam **NTFS** (sistema de arquivos nativo) com regras diferentes, como suporte a unicode para nomes de arquivos)
* **Extensão de arquivo**: nomes de arquivos em duas partes, separadas por ponto
  * A extensão indica algo sobre o arquivo
  * MS-DOS: nome de 1 a 8 caracteres com extensão opcional de 1 a 3
  * UNIX: tamanho da extensão a critério do usuário e arquivos podem ter mais de uma extensão
    * No UNIX extensões são convenções e servem como lembretes ao usuário ao invés de darem informações úteis ao trabalho do SO
  * Windows: próprio SO lida com as extensões para determinar qual usuário ou processo aquela extensão pertence, iniciando uma arquivo com o programa associado àquela extensão
* **Exemplos de extensões**
  * `.bak`: backup
  * `.c`: fonte em C
  * `.gif`: imagem Graphical Interchange Format
  * `.iso`: imagem de um CD
  * `.o`: arquivo objeto (saída do compilador, não linkado)

## 5.1.2 - Estrutura do arquivo

* **Três formas de estruturação**
  1. **Sequência de bytes**
     * Sem estrutura
     * SO não faz ideia do conteúdo, só vê bytes
     * Significado vem de programas a nível de usuário
     * Usado por UNIX e Windows 98
     * Flexibilidade
  2. **Sequência de registros**
     * Cada registro (mesmo tamanho) tem estrutura interna
     * Ler retorna um registro e escrever sobreescreve ou anexa um registro
  3. **Árvore**
     * Árvore de registros, de tamanhos variáveis
     * Árvore ordenada pelo campo `chave` para ser capaz de obter registros com tal chave
     * Gerenciado pelo SO
     * Usado em mainframes e processamento de dados comerciais

## 5.1.3 - Tipos de arquivos

* UNIX e Windows: arquivos normais e diretórios
* UNIX: arquivos especiais de caracter e de bloco
* Windows XP: arquivos de **metadados**

### Tipos

* **Arquivos normais**: informação do usuário
  * ASCII ou binário
  * ASCII: linhas de texto
    * Terminadas em carriage return ou line feed (ambos no WIndows)
    * Podem ser mostrados no estado que se encontram
    * Fácil de conectar saídas com pipe
  * Binário: estrutura interna conhecida apenas aos programas que os usam
* **Diretórios**: arquivos do sistema para manter a estrutura do sistema de arquivos
* **Arquivos especiais de caracter**: relacionados a I/O, modelam dispositivos de saída
* **Arquivos especiais de bloco**: modelam discos

## 5.1.4 - Acesso a arquivo

* **Acesso sequencial**: ler bytes na ordem, do começo, sem poder pular e ler fora de ordem
  * É possível voltar ao começo do arquivo para ler novamente
  * Conveniente com armazenamento em fita magnética
* **Acesso aleatório**: bytes podem ser lidos em qualquer ordem
  * Métodos para especificar de onde ler:
    1. Operação `read` dá a posição de onde deve-se começar a ler
    2. Operação `seek` para determinar posição atual

## 5.1.5 - Atributos de arquivos

* **Atributos** ou **metadados**: informações associadas ao arquivo, como data, nome e tamanho

### Exemplos de atributos

* **Relacionados à proteção**:
  * Proteção: quem pode acessar e de que forma
  * Senha: para acessar o arquivo
  * Criador: ID de quem criou
  * Dono: dono atual
* **Flags**: controlam alguma propriedade específica
  * Somente leitura: 0 para r/w, 1 para r apenas
  * Oculto: 0 normal, 1 oculto
  * Sistema: 0 normal, 1 arquivo do sistema
  * Arquivamento: 0 se foi feito backup, 1 para necessidade de ainda fazer
  * ASCII/binário: 0 para ASCII e 1 para binário
  * Acesso aleatório: 0 para acesso sequencial, 1 para acesso aleatório
  * Temporário: 0 para normal e 1 para deletar quando o processo termina
  * Travamento: 0 para destravado e diferente de zero para travado
* **Atributos gerais**
  * Tamanho do registro: número de bytes
  * Posição da chave: offset da chave no registro
  * Tamanho da chave: quantidade de bytes no campo da chave
  * Data de criação
  * Data do último acesso
  * Data da última alteração
  * Tamanho atual
  * Tamanho máximo

## 5.1.6 - Operações sobre arquivos

1. `Create`: arquivo criado sem dados
2. `Delete`: deletado para liberar espaço
3. `Open`: processo precisar abrir arquivo antes de usar
   * Pega os atributos e a lista de endereços no disco
4. `Close`: fechar arquivo para liberar espaço gasto no armazenamento dos atributos e endereços no disco do arquivo
5. `Read`: bytes lidos vêm da posição atual. Precisa-se especificar quantos dados são necessários e prover um buffer para colocá-los
6. `Write`: dados escritos no arquivo na posição atual
   * Se a posição atual está no fim, o tamanho do arquivo aumenta
   * Se a posição atual está no meio, os dados são sobrescritos
7. `Append`: forma restrita do write. Adiciona dados apenas ao final do arquivo
8. `Seek`: arquivos de acesso aleatório, especifica de onde pegar os dados, reposicionando o ponteiro de arquivo para um lugar específico
9. `Get attributes`: processo ler os atributos do arquivo para efetuar trabalho (exemplo do make, olhando horário de modificações)
10. `Configurar atributos`: alguns atributos podem ser modificados após criação do arquivo, como proteção e algumas flags
11. `Rename`: alterar nome de um arquivo existente
12. `Lock`: previne acesso simultâneo ao mesmo arquivo por processos diferentes

