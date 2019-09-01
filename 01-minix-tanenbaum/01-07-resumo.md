# 1.7 - Resumo

* SOs: gerenciam recursos e são máquinas estendidas
  * Recursos: gerenciar eficientemente cada parte do sistema
  * Máquina estendida: fornecer uma máquina virtual mais conveniente de ser usada que a máquina real
* O coração do SO são as chamadas de sistema
  * Seis grupos em MINIX
    * Criação e término de processos
    * Gerenciamento de sinais
    * Ler e escrever arquivos
    * Gerenciamento de diretórios
    * Proteção de informação
    * Gerenciamento do tempo
* Tipos de Estrutura de um SO
  * Sistema monolítico
  * Hierarquias de camadas
  * Sistema de máquina virtual
  * Exokernel
  * Modelo cliente-servidor



### PROBLEMAS

1. As funções principais de um SO são gerenciar recursos do computador de maneira eficiente e prover uma máquina estendida que abstraia a utilização do hardware bruto, tornando mais conveniente o uso
2. No modo Kernel, todas instruções são permitidas. No modo usuário, há restrições no conjunto de instruções bem como em operações de entrada e saída. Tal distinção é importante pois operações no modo kernel costumam causar sérios problemas ao computador quando falham, portanto, restringindo as operações de usuário podemos garantir que os processos de usuário não causarão falhas que impedirão o funcionamento do computador
3. O conceito de multiprogramação consiste em minimizar o tempo ocioso da CPU, portanto enquanto o SO espera uma operação de entrada acabar, outra tarefa pode ir realizando seu trabalho na CPU
4. Spooling se resume a fazer um buffer de certos dados para a CPU não ficar esperando um dispositivo lento de entrada ou saída terminar seu trabalho para poder dar continuidade a outras tarefas que necessitam dela. Hoje em dia, spooling é muito utilizado para tarefas de impressão, onde gerenciar e calcular a saída é extremamente rápido, porém a impressão leva muito mais tempo
5. Computadores sem acesso direto de memória têm um impacto negativo sobre a multiprogramação, pois a CPU não fica livre para executar outros processos enquanto se executa procedimentos de entrada e saída
6. Computadores de segunda geração não possuíam compatilhamento de tempo pois não havia hardware para proteger os SO de programas de usuários maliciosos
7. Alternativas *a*, *c* e *d* apenas em modo Kernel
   * Configurar o relógio só pode ser feito em modo kernel, pois senão um processo poderia configurar o relógio para algum tempo atrás para se manter em execução no processador. As outras duas são óbvias, pois deve-se reservar ao modo kernel instruções que tenham grande impacto sobre a operação e gerenciamento do computador
8. Um SO de mainframe não precisa se preocupar tanto com a interação com um usuário quanto um SO de computador pessoal
9. Um SO com código proprietário poderia ter melhor qualidade pois se torna mais fácil de estabelecer padrões de desenvolvimento, bem como planejamento do software em si. Enquanto que um SO de código aberto seria mais maleável às necessidades reais do usuário, quanto mais eficiente em correção de bugs e desenvolvimento de melhorias e novas versões
10. Como o usuário pertence ao mesmo grupo do dono do arquivo e a execução é permitida para usuário do mesmo grupo, o arquivo executa sem problemas
11. O super usuário é essencial para gerenciamento de um sistema grande usado por muitos outros usuários comuns. Caso algum dos usuários faça algo que consuma muito recurso, por exemplo. Neste caso, o super usuário é capaz de desfazer essa ação, restaurando o sistema ao normal
12. Seria possível ficar com apenas uma das formas. Sem o path relativo, seria apenas uma pequena inconveniência ter que usar o path absoluto inteiro para se referir a qualquer arquivo. Ficar sem o path absoluto seria uma inconveniência maior, mas ainda assim seria possível se referir a qualquer arquivo por meio de `../` para acessar pastas acima
13. A tabela de processos se faz necessária para armazenar o estado de um processo que foi suspenso, para posterior execução. Em um sistema que lida com apenas um processo por vez ela não é necessária, pois tal processo jamais será suspenso
14. Arquivos especiais de bloco referem-se a blocos numerados que podem ser lidos ou escritos independetemente dos outros. Isso não é possível com arquivos especiais de caracteres, que na verdade se tratam de um fluxo contínuo de caracteres
15. Nada, pois o arquivo não é removido do sistema de arquivos por ainda conter uma referência a ele (a do usuário 02), checado através da contagem de referências no i-node correspondente
16. Pipes não são essenciais, apenas adicionam praticidade. Se não existem o primeiro programa poderia escrever sua saída em um arquivo, que posteriormente seria lido pelo segundo programa
17. O processamento de comandos de uma câmera é similar à shell de um SO de computador, pois fica aguardando um input, processando ele após o recebimento
18. Windows não possui a chamada `fork`, por isso possui a chamada `spawn` que cria o novo processo e já inicia um programa específico nele (combinação de `fork` e `exec`)
19. A chamada `chroot` precisa ser protegida pois um usuário comum poderia explorar várias brechas com isso, como por exemplo tornar sua home a raiz e criar um arquivo `etc/passwd` nela, fazendo o SO pensar que aí se encontra a senha do superusuário de fato, dando então privilégios a esse usuário comum
20. As chamadas `getpid`, `getpgrp`, `getgid` e `getuid` apenas pegam um valor da tabela de processos e retornam, portanto são as chamadas mais prováveis de serem as mais rápidas
21. 500 mil chamadas de sistema
22. Não, pois `unlink` funciona para remoção
23. MINIX roda o programa `update` para garantir que os blocos do cache de escrita sejam sincronizados com o disco com frequência através da chamada `sync` (a cada 30 segundos), uma vez que quando um programa escreve em um arquivo, ele não vai imediatamente para o disco
24. Não faz sentido ignorar um `SIGALRM` pois ele é resultado de um temporizador (chamada de sistema `alarm`) que foi configurado por alguma razão inicialmente
25. Sim, o modelo cliente-servidor pode ser utilizado em um sistema de apenas um computador
26. Para uma máquina ser virtualizável, ela precisa ser capaz de interromper e reportar sempre que um programa de usuário tentar executar uma instrução do modo kernel. Os Pentiums iniciais não eram capazes disso.
27. [Testar todas as chamadas de sistema de MINIX 3]
28. [Será feito algo similar no primeiro EP da disciplina]