# 01 - Introdução
* Dois tipos de programas
    * Sistema: gerenciam operação do computador
    * Aplicações: fazem o trabalho que o usuário deseja
* **Sistema Operacional**
    * Gerenciam os recursos do computador e provêm uma base para que programas de aplicação sejam escritos
    * Protegem programadores da complexidade do hardware sendo uma camada de software acima do hardware bruto apresentado uma **máquina virtual**
* **Hierarquia do Sistema do Computador**
    * **Programas de aplicações**
        * Browsers e programas instaláveis em geral
    * **Programas do sistema**
        * Compiladores, editores e interpretadores de comando
        * Sistema operacional
            * Parte do software que executa em **modo kernel** ou **modo supervisor**
            * Protegido via hardware de ser alterado pelo usuário
    * **Hardware**
        * Linguagem de máquina
            * **ISA (Instruction Set Architecture - Arquitetura do Conjunto de Instruções)**: hardware junto com as instruções assembly
            * I/O controlado por carregar valores específicos em **registradores de dispositivo** especiais
        * Microarquitetura
            * Agrupamento de dispositivos físicos em unidades mais funcionais: registradores internos da CPU e caminho de dados com a ULA
            * A operação do caminho de dados pode ser controlada via software (**microprograma**) ou diretamente nos circuitos do hardware, e tem como objetivo executar um conjunto de instruções
        * Dispositivos físicos (circuitos integrados, cabos, etc)