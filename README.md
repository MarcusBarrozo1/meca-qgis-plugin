*Read this in [English](README_en.md).*
# MECA: Modelo Estimativo de Consumo na Agricultura 💧🛰️

Uma implementação Python/QGIS baseada no algoritmo SAFER para estimativa de evapotranspiração e quantificação do consumo hídrico no semiárido.

## ⚠️ Aviso de Arquitetura:
Este repositório contém a Prova de Conceito (PoC) original. O código atual atinge o objetivo biofísico, mas possui alto acoplamento (interface/motor) e dívida técnica. Não é recomendado para uso em produção no estado atual. O foco deste repositório é demonstrar a modelagem matemática (SAFER) aplicada ao QGIS. Veja a seção "Roadmap de Refatoração" abaixo para a arquitetura alvo.

## 📌 Visão Executiva: O Problema de Negócio
A gestão eficiente de recursos hídricos em regiões semiáridas, especialmente durante a quadra chuvosa e períodos de seca, exige monitoramento rigoroso. O objetivo central deste projeto não é a classificação agronômica de culturas, mas sim a **quantificação volumétrica da demanda hídrica (em m³/dia e mm/mês)** com base na extensão real (em hectares) da área plantada pertencente a **usuários de água bruta**. 

O modelo visa fornecer dados concretos para auxiliar na fiscalização, outorga e gestão do balanço hídrico de bacias hidrográficas, filtrando o processamento estritamente para os polígonos mapeados de cada usuário.

---

## 🚧 Status Atual: Versão Inicial (v0.1.0-alpha)
Este repositório contém o código do Produto Mínimo Viável (MVP) do algoritmo, desenvolvido ao longo de 7 meses de pesquisa e estruturado como um plugin para o software QGIS.

Na arquitetura atual (legado):
- O processamento matemático de Sensoriamento Remoto está acoplado à interface gráfica (`PyQt`).
- A ingestão de dados exige o download prévio de cenas de satélite e seleção manual das bandas ópticas pelo usuário.
- O *output* é uma estimativa de consumo baseada em um recorte temporal único (diário), gerando matrizes (`.tif`) intermediárias de Albedo, NDVI e Radiação Líquida.

---

## 🔬 Fundamentação Científica e Metodológica
A base do MECA é o algoritmo **SAFER** (*Simple Algorithm for Evapotranspiration Retrieving*), originalmente estruturado em linguagem `R`. O processo de cálculo deste repositório envolve o balanço de energia à superfície, estruturado nas seguintes etapas principais:

1. **Correção e Calibração Radiométrica:** Obtenção do Albedo no topo da atmosfera e da superfície a partir de bandas ópticas.
2. **Balanço de Ondas Curtas e Longas:** Cálculo da Radiação Solar Global, Radiação Líquida (Rn) e Emissividade Atmosférica/Superficial.
3. **Temperatura de Superfície (TS):** Estimativa térmica da superfície do polígono.
4. **Evapotranspiração (ET):** Obtenção do coeficiente de evaporação e da Evapotranspiração Real diária, cruzando dados de satélite com variáveis climáticas locais (Tmax e Tmin).

---

## 🚀 Roadmap de Refatoração e Ciência de Dados (Próximos Passos)
Para transformar este MVP em um pipeline de **Geographic Data Science** escalável, os seguintes marcos arquiteturais e científicos estão definidos para as próximas versões:

### 1. Desacoplamento Arquitetural e Performance
- Extrair a lógica matemática do algoritmo SAFER para módulos de serviço independentes (puramente Python).
- Substituir o pesado I/O de disco (escrita de `.tif` intermediários) por processamento estritamente em memória RAM utilizando operações vetorizadas de matrizes via `NumPy`.

### 2. Cloud-Native Geospatial (Fim dos Downloads Manuais)
- Eliminar o download de cenas orbitais inteiras. Implementar o consumo de dados via APIs (padrão STAC/COGs) ou plataformas como Google Earth Engine.
- O pipeline deverá requisitar dinamicamente apenas a matriz de pixels correspondente ao polígono do usuário de água bruta.

### 3. Agregação Temporal Automatizada (O Desafio Mensal)
- Evoluir a saída diária para um **consumo mensal real**. 
- Criar rotinas automatizadas para processar múltiplas cenas dentro de um mês (ex: 3 a 4 passagens do satélite) e calcular a média ponderada temporal do volume consumido (m³).

### 4. Fusão de Dados Multimodais (Solução para Cobertura de Nuvens)
- **O Desafio Físico:** A quadra chuvosa no Ceará gera severos *gaps* de dados ópticos orbitais devido à alta nebulosidade.
- **A Solução Mapeada:** Implementar metodologias de imputação de dados e fusão multimodal, integrando as imagens de satélite com dados de **RPA (Drones com sensores multiespectrais)**. Isso garantirá a continuidade da série temporal mensal e servirá como camada de validação física das equações.

### 5. Validação Científica e CI/CD
- Construção de testes unitários (`pytest`) para injetar matrizes de controle no código Python e comparar os resultados (pixel a pixel) com os *outputs* da metodologia original estabelecida em `R`, garantindo rigor e reprodutibilidade científica.

### 6. Inteligência Fiscal e Governança Territorial (CAR vs. Cultivo Real)
- **O Desafio:** O Cadastro Ambiental Rural (CAR) é um limite jurídico declaratório. Executar o modelo SAFER sobre a extensão total do CAR superestima o consumo, pois contabiliza áreas de reserva legal e solo exposto.
- **A Solução Mapeada:** Implementar um motor automatizado de **Cruzamento Espacial Relacional**:
  1. **A Máscara Física:** Utilizar modelos de visão computacional e *Machine Learning* para classificação de fronteiras agrícolas (*Crop Boundary Classification*), isolando exclusivamente a área plantada ativa.
  2. **Interseção Espacial (*Spatial Join*):** Cruzar a máscara física de cultivo com a malha fundiária jurídica do CAR para atribuição de posse.
  3. **Validação Relacional:** Relacionar o titular do polígono validado com o Banco de Dados de Usuários de Água Bruta.
- **Impacto de Negócio:** Essa arquitetura eleva o MECA de um estimador de evapotranspiração para uma ferramenta de **auditoria hídrica automatizada**, capaz de identificar instantaneamente o uso regular, a captação clandestina (sem registro) e a sobreutilização (área plantada superior à outorga).

### 7. Features Futuras (Opcionais)
- Desenvolvimento de módulo auxiliar para categorização espectral do tipo de cultura agrícola, visando refinar coeficientes de consumo específicos caso demandado pela gestão hídrica.
