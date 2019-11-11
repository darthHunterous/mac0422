# 4.2 - Swapping

* Dois approaches para gerenciamento de memória
  * **Swapping**: trazer todo o processo do disco, rodar por algum tempo e colocá-lo de volta no disco
  * **Memória virtual**: programas podem executar mesmo estando parcialmente na memória principal
* Memória com variação de partições dinâmicas flexibilizam a utilização de memória mas complicam a alocação e desalocação de memória e manutenção
* **Compactação de memória**: combinar todos os buracos de memória em um só, movendo todos processos para baixo
  * Requer muito tempo de CPU (da ordem de 0.5s)
* O segmento de dados do processo pode crescer (alocação dinâmica de memória do heap)
  * Crescimento para dentro de um buraco adjacente, senão deve ser movido para um buraco maior na memória ou fazer swap em processos para criar tal buraco
  * Se não pode crescer na memória e a área de swap está cheia, o processo deve aguardar ou ser morto
* Se é esperado que o processo cresça enquanto executa, é bom alocar memória extra para reduzir o overhead (sobrecarga) de mover ou fazer swap de processos
  * Porém ao fazer swap, é um desperdício fazer da memória extra também
  * **Disposição**: espaço para crescimento ENTRE a stack de um processo e seus segmentos de dados
    * A stack cresce para baixo (variávies normais e endereços de retorn) e o segmento de dados cresce para cima (heap para variáveis alocadas dinamicamente)
    * Se a memória entre os segmentos acabar, o processo deve ser movido para um nov buraco com espaço suficiente, sofrer swap ou ser morto

## 4.2.1 - Gerenciamento de memória com Bitmap

* Gerenciamento de memória dinâmica: bitmap ou free lists (listas de regiões livres)
* **Bitmap**: memória dividida em unidades de alocação
  * Problema de design: tamanho da unidade de alocação
    * Unidade de 4 bytes: `32 * n` bits usam um `n` bits no mapa, bitmap ocupa `1/33` da memória
    * Com uma unidade maior, o bitmap é menor mas gasta-se mais espaço