# 5.4 - Segurança

## 5.4.1 - O ambiente de segurança

### Ameaças

* **Objetivos e ameaças**
  * Confidencialidade dos dados: exposição dos dados
  * Integridade dos dados: falsificação de dados
  * Disponibilidade do sistema: negação de serviço
* **01 - Confidencialidade de dados**: o dono de algum dado pode especificar quem pode acessá-lo e o sistema deve garantir isso
* **02 - Integridade dos dados**: usuários sem autorização não podem modificar dados sem a permissão do dono
* **03 - Disponibilidade do sistema**: ninguém deve ser capaz de inutilizar o sistema, negando então o serviço
* **Privacidade**: proteger usuário de mau uso de informações sobre eles

### Intrusos

* **Dois tipos**
  * **Passivo**: ler arquivos que não está autorizado
  * **Ativo**: fazer alterações não autorizadas
* **Categorias**
  * **Intromissão casual por usuários não técnicos**: pessoas lendo emails e arquivos de outra pessoa quando disponível publicamente ou computadores desbloqueados
  * **Espionagem por pessoal interno**: operadores técnicos que se desafiam a quebrar a segurança de um sistema local
  * **Determinação em obter dinheiro**: programadores que extorquem seus clientes, como bancos de maneira direta (ameaça de apagar os dados) ou indireta (manter alguns centavos por transação truncada ao invés de arredondar)
  * **Espionagem comercial ou militar**: grampeamento ou monitoramento de um sistema

### Programas maliciosos

* **Malware**
* **Vírus**: pedaço de código capaz de infiltrar em outros programas e se espalhar
  * Ataque **DOS (Denial of service)**: torna o computador inutilizável consumindo muitos recursos, como tempo de CPU e lotando o armazenamento
  * Ataque **DDOS (Distributed Denial of Service)**: vírus que aguardam após infectar um computador, até uma determinada data ou quantidade de outras infecções para ser capaz de sobrecarregar um servidor e consequentemente seu serviço
* Malware tornam computadores escravos, que reportam a um mestre via internet
  * Capaz de auxiliar a enviar spam roubando endereços de email do escravo
  * **Key logger** no escravo para registrar senhas e dados de cartão de crédito digitados no teclado
* **Verme (worm)**: programa independente de outro programa (ao contrário do vírus)
  * Consegue fazer que o sistema executa seu arquivo
* **Cavalo de tróia**: aparenta ter uma função válida (que convence o usuário a executá-lo) mas executa como adicional outra funções (maliciosas) ou permite a execução de um vírus ou verme
* **Bomba lógica**: programador insere secretamente um pedaço de código malicioso no sistema de uma empresa
  * Diariamente coloca uma senha para evitar a ativação, ao ser demitido o código dispara causando algum tipo de dano (criptografar o banco da empresa, etc), forçando ela a recontratar o programador e cedar à chantagem para não perder informações
* **Spyware**: obtido através de sites
  * Pode ser um **cookie** (informação de identificação de usuário que o site tem)
  * Exemplo são propagandas direcionadas que armazenam as informações de navegação do usuário e são críticas à privacidade
* **Plug-in** de browsers: podem dizer oferecer uma coisa ao usuário, mas conter código malicioso

### Perda de dados acidental

* **Causas comuns**:
  * Ações aleatórias (incêndio, enchentes, roedores mastigando fitas)
  * Erros de hardware ou software
  * Erros humanos (entrada incorreta de dados, comandos de leitura e escrita usados equivocadamente)

## 5.4.2 - Ataques de segurança genéricos

* **Encontrar falhas de segurança**: contratar times de especialistas (**equipe de invasão**) para tentar quebrar o sistema
* **Ataques comuns**
  * Ler uma página de memória após requisitar: podem conter informações do uso anterior se não foram apagadas antes de alocadas
  * Tentar chamadas ilegais ou com parâmetros ilegais
  * Tentar logar e interromper o processo antes de concluir: muitas vezes o login dá certo
  * Modificar estruturas mantidas no espaço de usuário
  * Criar um programa falso que finge ser o de login e consegue os dados de um usuário
  * Tentar fazer algo que o manual diz para não fazer
  * **Back door**: permitir que um usuário com determinado login não passe por verificações de segurança
  * Problemas com pessoal: suborno de admins

## 5.4.3 - Princípios de design voltados à segurança

* O design deve ser público: assim evitar assumir que o intruso não conhece o funcionamento do sistema
* O padrão deve ser nenhum acesso: erros em que o acesso legítimo é recusado são reportados, enquanto que acesso não autorizado permitido não é reportado
* Checar autoridade atual: se checar permissões apenas durante a abertura de um arquivo, um usuário pode ficar com ele aberto por semanas mesmo que o dono já tenha mudado a proteção
* Dar sempre o mínimo privilégio possível
* Mecanismo de proteção devem ser simples, uniformes e construídos nas camadas mais baixas do sistemas: é impossível adicionar segurança a um sistema inseguro
* O esquema de proteção deve ser aceitável e estimular os usuários a fazerem, pois se julgarem ser trabalhoso, não o farão

## 5.4.4 - Autenticação de usuário

### Senhas

* UNIX pede o username e a senha (imediatamente criptografada) e compara com a senha criptografada no arquivo de senhas
* Exemplo de senha: 7 caracteres levaria 2000 anos para ser quebrada considerando um conjunto de 95 caracteres imprimíveis (95^7 ou seja 7 \* 10^13 a 1000 tentativas por segundo)
* **Salt**: adicionar um número aleatório de `n` bits à senha
  * O número aleatório é salvo sem criptografia no arquivo de senha, mas a senha é concatenada com ele e aí criptografada
  * Aumenta o tamanho por `2^n` (UNIX usa `n == 12`)
* **One time password**: usuário recebe lista de senhas e a cada login usa uma diferente
* Nenhuma técnica protege bem um usuário que tenha senha igual ao username, para isso é necessário que o sistema ofereça conselhos
* Senhas nunca devem ser armazenadas em um banco de dados sem criptografia
* O computador não deve mostrar os caracteres de uma senha digitada
* Variação do sistema de senhas são o de perguntas de segurança (informações pessoais do usuário que apenas ele saiba)
* **Desafio-resposta**: no login o computador passa uma informação e o usuário precisa responder com base nisso
  * Um exemplo é o usuário souber que precisa inserir o quadrado de um dado número mostrado

### Identificação física

* Cartão de identificação em conjunto da senha
* Digital ou reconhecimento de voz (usuário primeiro informa quem ele é para o computador não gastar tempo comparando a digital com todo o banco de dados)
* Tanenbaum diz na página 535: "Direct visual recognition is not yet feasible but may be one day."
  * Hoje temos dispositivos que fazem isso nos bolsos :)
* Análise de assinatura e movimentos da caneta
* Análise do comprimento dos dedos
* O principal ponto é que o método de autenticação deve ser psicologicamente aceitável para os usuários

### Contramedidas

* Ações para tornar mais difícil o uso não autorizado (restrições de login, horários, etc)
* Aumentar tempo de restrição de login após várias tentativas sem sucesso e se continuar errando, restringir por algum tempo e notificar os admins
* Registrar logins feitos para que o usuário possa detectar logins que ele não fez
* Armadilhas para pegar invasores: login com uma senha fácil que notifica os admins imediatamente