# CAP-423 - Ciência de Dados Geoespaciais

## Projeto

Este projeto tem como objetivo criar uma base de dados com informações extraídas das imagens de satélite disponíveis no INPE através de técnicas de IA para possibilitar a recuperação por conteúdo.

Nesta primeira etapa, quatro classes (ou "alvos") foram escolhidas para se extrair das imagens: cicatriz de queimadas, pivôs, área de agricultura e estradas. Este repositório guarda dados referentes especificamente à etapa de reconhecimento de vias. 

## Sub-projeto

A seguir algumas informações básicas a respeito do problema e estratégias utilizadas.

### Características do problema

As imagens serão do sensor [MSI](https://sentiwiki.copernicus.eu/web/s2-mission#S2Mission-MSIInstrumentS2-Mission-MSI-Instrumenttrue) do satélite [Sentinel-2](https://sentiwiki.copernicus.eu/web/s2-mission]), portanto tem-se à disposição bandas tanto no espectro visível (RGB) quanto fora (NIR, SWIR, etc.), com imagens variando entre 10m a 20 de resolução espacial e tempo de revisita de 5 dias (em média).

A classe abordada aqui é "estrada". Esta classe pode ser sub-dividida em duas: estradas asfaltadas e não-asfaltadas. Referente a estradas asfaltadas, a base de dados OpenStreetMap é uma fonte relativamente confiável, portanto para a extração desta categoria basta um cruzamento de dados. A classe estradas não-asfaltadas apresenta um desafio maior, uma vez que podem aparecer em lugares remotos e repentinamente, o que torna a base OpenStreetMap (entre outras) não confiável.

Deve-se notar que qualquer análise deve ser reprodutível, uma vez que se pretende implementar em um dataflow maior.

### Estratégias utilizada

Foram testadas 3 estratégias, utilizando técnicas tanto clássicas quanto de IA.

#### Extração de contornos e análise de linearidade

A estratégia geral aqui é extrair as feições de estradas o melhor possível para posterior utilização do algoritmo de Hough, "limpando" os contornos não lineares. 

- Pré-processamento:
  - conversão de imagens RGB para escala de cinza.
  - cálculo do índice NDVI (Normalized Difference Vegetation Index) para ajudar a distinguir vegetação.
- Detecção de Bordas:
  - utilização do método de detecção de bordas Canny com ajustes nos parâmetros para diferentes níveis de separação (nenhuma, moderada e alta).
  - análise visual para selecionar os melhores parâmetros.
- Transformação de Hough:
  - configuração de parâmetros para identificar feições lineares, como estradas, após a aplicação dos detectores de bordas.
- Análise de Índices Específicos:
  - uso do índice BSI (Bare Soil Index) para realçar áreas não vegetadas, possivelmente relacionadas a estradas ou clareiras.

O primeiro problema encontrado é a qualidade da máscara de contornos. A maior parte das estradas são delineadas, entretanto nem todas e não só estradas, o que por sí só mina o resultado da transformação de Hough. 
