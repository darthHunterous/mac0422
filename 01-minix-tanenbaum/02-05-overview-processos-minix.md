# 2.5 - Overview de processos no Minix 3

- **UNIX**: kernel monolítico
- **MINIX**: coleção de processos que se comunicam entre si por meio de passagem de mensagens
  - Mais modular e flexível
  - Fácil de trocar todo o sistema de arquivos por outro sem precisar recompilar o Kernel

## 2.5.1 - Estrutura interna do MINIX 3

### 4 camadas (da mais profunda para  a menos)

- **01 - Kernel**: kernel, tarefa de relógio, tarefa de sistema
- **02 - Drivers de dispositivo**: driver de disco, de TTY, de ethernet, etc
- **03 - Processos de servidor**: gerenciador de processos, sistema de arquivos, servidor de informações, servidor de rede, etc
- **04 - Processos de usuário**: init e vários processos de usuários

### Kernel

- Apenas processos nessa camada podem usar instruções privilegiadas (de **modo kernel**)
  - Suporte para acesso de portas entrada/saída e interrupções
- Escalona processos
- Gerencia a transição de estado dos processos
- Gerencia mensagens entre processos
- **Tarefa de relógio**: driver de dispositivo de entrada/saída que interage com o hardware que gera sinais de temporização
- **Tarefa de sistema**: implementa uma das principais funções da camada 01: prover **chamadas de kernel** para os drivers e servidores acima
- As tarefas de relógio e de sistema são compiladas no espaço de endereçamento do Kernel, mas escalonado como processos separados
- **Partes do kernel escritas em Assembly**: gerenciamento de interrupções, gerenciamento da troca de contexto dos processos e manipulação do hardware MMU

### Camadas acima do Kernel

- 3 camadas podem ser consideradas com o uma pois o kernel as trata da mesma maneira

- Limitadas a instruções de **modo usuário**

- Não podem acessar portas de entrada/saída diretamente e nem memória fora do segmento alocado para elas

- Diferença entre camadas 02 a 04: privilégios especiais

- **Camada 02 - Drivers de Dispositivo**: mais privilégios

  - Podem requisitar à *tarefa de sistema* que leiam ou escrevem na entrada/saída por eles
  - Driver necessário para cada tipo de dispositivo
  - Podem fazer outras chamadas do kernel: requisitar que um dado lido seja copiado para o espaço de endereços de um outro processo

- **Camada 03 - Servidores**: alguns privilégios

  - Servidores: processos que provêm serviços úteis aos processos de usuário
  - Dois servidores essenciais:
    - **Gerenciador de processos**: gerenciam todas chamadas de sistemas que iniciam ou param a execução de processos, bem como lidar com sinais (podem alterar o estado de execução de um processo)
      - Também responsável por gerenciar memória (chamada `brk`)
    - **Sistema de arquivos**: executa todas chamadas de sistema relacionadas a arquivos (`read`, `mount`, `chdir`)
  - Servidores específicos para MINIX 3:
    - **Servidor de informação**: debug e status de drivers e servidores
      - Útil para um sistema didático como MINIX e não para sistemas comerciais
    - **Servidor de reencarnação**: inicia/reinicia drivers não carregados na memória ao mesmo tempo que o kernel
      - Detecta falhas de drivers, matando-os e inicia uma nova cópia do driver
      - Torna o sistema tolerante a falhas
      - Ausente na maior parte dos SOs
  - Servidor opcional para sistemas em rede:
    - **Servidor de rede**: não podem fazer entrada/saída diretamente mas podem requisitar aos drivers
  - Servidores podem comunicar com o kernel via *tarefa de sistema* 

  

### Diferença chamada do kernel e chamada de sistema POSIX

- **Chamada do kernel**: funções de baixo nível fornecidas pela *tarefa de sistema* para permitir o trabalho dos servidores e dos drivers
- Exemplo: ler uma porta de entrada/saída do hardware
- **Chamadas POSIX**: chamadas de alto nível como `read`, `fork` e `unlink`, disponíveis para programas de usuário na camada 04
- Chamadas de kernel podem ser consideradas um subconjunto especial de chamadas de sistema

### Funções do SO e equivalente nas camadas

- **Gerenciar recursos**: drivers na camada 02, com ajuda da camada do kernel para acesso privilegiado das portas de entrada/saída e sistema de interrupções
- **Implementar chamadas de sistema**: interpretação pelo gerenciador de processos e pelos servidores do sistema de arquivos na camada 03

### Mais sobre camadas 02 e 03

- O sistema não precisa ser recompilado para adicionar novos servidores
- Drivers são tipicamente iniciados com o sistema mas podem ser iniciados depois
- Drivers e servidores são armazenados como executáveis comuns
- **Service**: programa de usuário que fornece interface com o servidor de reencarnação
- Drivers e servidores diferem dos processos de usuário pois nunca terminam enquanto o sistema está ativo
- **Processos de sistema**: drivers e servidores nas camadas 02 e 03
  - Parte do SO, não pertecem a um usuário (ativados antes do primeiro usuário logar)
  - Processos de sistema tem prioridade de execução maior que processos de usuário
    - Normalmente drivers têm prioridade maior que servidores, mas pode ser atribuída caso a caso no MINIX, para que um driver que provenha serviço a um dispositivo lento tenha menor prioridade que um servidor que precisa responder rapidamente

### Camada 04 - Processos de usuário

- Vários processos de usuário só operam enquanto o usuário está logado
- Exemplos de processos de usuários que iniciam com o sistema e sempre executam:
  - **Init** 
  - **Daemons**: processo de segundo plano executado periodicamente ou que aguarda um certo evento
    - Servidor iniciado independentemente e que executa como processo de usuário
    - Pode ser configurada maior prioridade a um daemon que outros processos de usuário

### "Task" e "device driver"

- Nas versões antigas do MINIX, os drivers de dispositivos eram compilados no kernel, dando acesso às estruturas de dados do kernel e acesso às portas de entrada/saída diretamente. Referidos como **tasks** para diferenciar dos processos no espaço do usuário
- No MINIX 3, os drivers foram implementados completamente no espaço do usuário
- No livro, **task** se refere apenas às tarefas de sistema e de relógio, implementadas no kernel, tendo sido alterado para **driver de dispositivo** ao se referir a drivers no espaço de usuário, porém o código fonte pode não ter sido atualizado dessa forma, aparecendo a palavra **task** onde deveria ser **driver de dispositivo**

## 2.5.2 - Gerenciamento de processos em MINIX 3

