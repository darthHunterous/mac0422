# 5 - Sistema de Arquivos

* Processo tem espaço limitado para armazenar informação na memória (restrito ao tamanho do espaço de endereçamento virtual)
* Quando o processo termina a informação é perdida, informação precisa ser retida (mesmo se ocorrer falha no computador)
* Múltiplos processos podem precisar acessar a mesma informação ao mesmo tempo
  * Impossível se a informação estiver no espaço de endereços de um único processo, portanto ela precisa estar disponível independente de processos
* **Requisitos de armazenamento de informação a longo prazo**:
  1. Possível armazenar uma quantidade muito grande de informação
  2. Informação sobreviver ao fim do processo que a usa
  3. Vários processos acessarem a informação concomitantemente
* **Arquivos**: unidades de armazenamento de informação em discos e outras mídias externas
  * Precisam ser **persistentes** (não afetados por criação e término de processos), só desaparecendo quando seu dono explicitamente remover
* **Sistema de arquivos**: indica como o SO deve gerir, estruturar, nomear, acessar, usar, proteger e implementar arquivos