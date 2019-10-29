# 04 - Gerenciamento de memória

* **Lei de Parkinson**: programas e seus dados expandem para preencher toda a memória disponível a eles
* **Memória ideal**: infinita em capacidade, infinitamente veloz e não volátil (não perde conteúdo ao ser desenergizada). Também barata
* **Hierarquia de memória**
  * Pequena quantidade, rápida, cara memória cache volátil
  * Memória RAM com preço e quantidades medianas
  * Armazenamento em disco lento, barato e não volátil
* **Gerenciador de memória**
  * **Funções**
    * Registra as partes em uso
    * Alocar memória para processos
    * Swap entre a memória principal e o disco
  * No kernel na maior parte dos SOs (não no MINIX 3)