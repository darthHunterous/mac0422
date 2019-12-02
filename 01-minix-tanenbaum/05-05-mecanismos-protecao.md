# 5.5 - Mecanismos de proteção

* Distinção entre **política** (dados e de quem devem ser protegidos) e **mecanismo** (como o sistema impõe a política)
* **Monitor de referência**: programa que impõe a proteção, checando a legalidade de toda tentativa de acesso a um recurso protegido e decidindo de acordo com suas tabela de políticas

## 5.5.1 - Domínios de proteção

* **Domínio**: conjunto de pares (objeto, direitos)
  * Especifica um objeto e um subconjunto de operações que pode ser efetuado nele
* **Direito**: permissão para executar uma operação
* Cada processo roda em um domínio de proteção e podem trocar durante execução
* UNIX: domínio definido pela combinação (UID, GID)
  * Possível listar todos objetos que podem ser acessados e o tipo de acesso
  * Cada processo tem duas partes: parte do usuário e do kernel
    * Ao fazer syscall, troca de usuário para kernel e se torna capaz de acessar outros objetos (recursos protegidos)
* SETUID e SEGID também alteram o domínio quando um processo dá `exec` em um arquivo com esses bits (SETUID e SETGID), tendo novos UID e GID efetivos
* O SO pode armazenar quais objetos pertencem a qual domínio através de uma matriz de proteção, com fileiras referentes aos domínios e objetos como colunas (para permitir troca de domínio, os domínios podem ser incluídos como objetos, permitindo uma operação `enter`). Cada entrada na matriz lista as operações permitidas

## 5.5.2 - Listas de controle de acesso

* A matriz de proteção é esparsa, portanto um desperdício de disco (a maioria dos domínios não tem acesso à maioria dos objetos)]
* **Duas técnicas**: armazenar a matriz por fileiras ou colunas, e apenas o elementos não vazios
  * **Access Control List (ACL)**: associa a cada objeto uma lista ordenada contendo os domínios que podem acessá-lo
* Na literatura sobre segurança, usuários são chamados de **sujeitos** ou **principais** que contrastam com os **objetos**, como arquivos
* **Função**: usuários podem ter permissões diferentes dependendo do grupo atual de acesso deles
* **Grupos** de usuários
  * Exemplo: `UID1, GID1: rights1; UID2, GID2: rights2;`
* Exemplo de mesmo usuário e diferente função
  * `tana, sysadm: RW` para `password` e `tana, pigfan: RW` para `pigeon_data`
* **Curingas (wildcards)**: `tana, *: RW`
  * Nesse caso não importa o grupo
  * Exemplo para bloquear usuário: `virgil, *: (none); *, *: RW`
* Para alterar o acesso só precisa alterar a ACL
  * Mas a alteração de um arquivo aberto se mantém enquanto ele estiver aberto

## 5.5.3 - Capacidades

* Matriz de permissões por fileiras
  * **Lista de capacitação (C-list)**: a cada processo se associa uma lista de objetos que podem ser acessados bem como as permissões (objetos + permissões = domínio)
    * Cada item se chama **capacidade** e consiste de um identificador do objeto (em UNIX o i-node) e um bitmap para as permissões
    * Listas de capacitação também são objetos e podem ser apontadas por outras listas para compartilhar subdomínios
* **Três métodos de proteção de lista de capacitação quanto a falsificação por usuários**
  * **Arquitetura rotulada**: bit extra em cada palavra da memória para indicar se contém uma capacidade
    * Só pode ser modificado por programas no modo kernel (SO)
  * **Manter C-list dentro do SO**: capacidades referidas pela posição na C-list
    * Similar a usar descritores de arquivo em UNIX
  * **Manter C-list no espaço de usuário**: gerenciar capacidades criptograficamente para evitar falsificação dos usuários
    * Para sistemas distribuídos
    * Quando um cliente requisita ao um servidor de arquivos para criar um objeto, o servidor gera um número aleatórios (campo Check) que é armazenado no i-node do servidor e nunca transmitido para o usuário ou a rede
    * O servidor gera uma capacidade para o usuário no formato: `Server, Object, Rights, f(Objects, Rights, Check)`
      * Identificador do servidor, número i-node do objeto, bitmap de permissões e output da função de criptografia que toma os parâmetros acima
    * Ao tentar acessar o objeto, o usuário envia essa capacidade, cujo campo criptografado é comparado com os valores que o servidor calcula (ele sabe o Check)
    * Não é possível falsificar a capacidade pois o usuário nunca saberá o Check
* **Direitos genéricos** para capacidades
  * Copiar capaciade: nova capacidade para mesmo objeto
  * Copiar objeto: duplicar objeto com nova capacidade
  * Remover capacidade: não afeta o objeto e retira entrada da C-list
  * Destruir objeto: remove objeto e capacidade
* Na versão gerenciada pelo kernel é difícil revogar acesso a um objeto: dificuldade para encontrar todas capacidades para um objeto pois podem estar armazenadas em várias C-lists
  * Solução: capacidades apontar para um objeto indireto que aponta para o real. Quebrando essa conexão, o acesso está revogado
* Na versão distribuída, para revogar acesso é só alterar o campo Check
* Revogação seletiva não é possível nos esquemas: é um problema com sistemas de capacidades
* **Princípio do privilégio mínimo**: executar código com a menor quantidade possível de direitos de acesso, diretriz para sistemas seguros

### Complementaridade ACL e Capacidades

* Abrir um arquivo apontado por uma dada capacidade não requer checagem com Capacidades mas com ACL faz-se necessário uma busca
* Sem suporte a grupo, dar acesso de um arquivo a todos requer enumerar todos usuários da ACL
* Capacidades permitem encapsulamento fácil de código mas ACL não
* ACL permite revogação seletiva de permissões mas capacidades não
* Problemas se um objeto for removido e sua capacidade não ou vicer versa, não ocorre com ACL

## 5.5.4 - Canais secretos

* **Problema do confinamento**: arquitetura com cliente, servidor e colaborador
  * Projetar um sistema em que é impossível o servidor vazar para o colaborador informações legítimas recebidas do cliente
  * Com matriz de proteção pode-se evitar que o servidor possa escrever em um arquivo que o colaborador tem acesso e o mecanismo de comunicação interprocessos pode também garantir que o servidor não comunica com o colaborador
  * Problema: **canais secretos**
    * Canais de comunicação mais sustis como o servidor enviar um bit 1 para o colaborador fazendo computação pesada por um tempo determinado e enviando zero através dele dormindo por um tempo determinado. O colaborador pode obter essas informações monitorando o tempo de resposta
    * O canal secreto possui muito ruído mas a informação pode ser confiávle através de um código de correção de erros (hamming code ou outro)
    * Modulação do uso de CPU não é o único canal secreto: pode-se alterar a taxa de paginação ou travamento de arquivos específicos
    * Travar arquivos específicos pode escapar de problemas de timing: se a informação é passada pelo arquivo `S`, ao alterar ele, o servidor altera o estado de outro arquivo `F1` para indicar que um bit está sendo enviado para o colaborador, que lê `S` e aí altera outro arquivo `F2` para indicar que leu, e o processo se repete
      * A banda pode ser aumentada usando mais arquivo, transmitindo um byte usando oito arquivos de sinais, `S0` a `S7`
  * Difícil solução para canais secretos, pois causar page faults aleatoriamente ou degradar a performance de sistema propositalmente não é desejável