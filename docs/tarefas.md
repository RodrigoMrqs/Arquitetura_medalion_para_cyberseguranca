# Divisão de Tarefas — 2º Bimestre

Pipeline Medallion para Cibersegurança | Camadas Ouro, ML-Ready e PySpark

---

## Resumo por Integrante

| Integrante | Responsabilidade Principal |
|------------|---------------------------|
| **Rodrigo** | EDA orientada a hipóteses + Data Lineage atualizado |
| **Lucas** | Camada Ouro (encoding, scaling, outliers) + Documentação |
| **Bernardo** | Modelagem ML (Decision Trees) + Refatoração PySpark + README |

---

## Rodrigo — EDA Orientada a Hipóteses

### Objetivo
Conduzir a análise exploratória usando os dados da camada Prata como base, formulando hipóteses testáveis e produzindo visualizações com conclusões orientadas a decisão.

### Tarefas

**H1 — Hipótese sobre severidade por tipo de ataque**
- Hipótese implementada: *"APT gera o maior prejuízo financeiro típico por incidente, enquanto ransomware é o vetor que mais produz perdas extremas, dada a combinação de alto volume e alta frequência de severidade."*
- Resultado: **Confirmada** — APT lidera em mediana ($32.3M); ransomware lidera em outliers absolutos (32 casos acima de $122M, 34% de todos os extremos)
- Gráfico 1: Mediana de `total_loss_usd` por `attack_vector` + razão outlier/total (barplot duplo)
- Gráfico 2: Contagem de outliers IQR por vetor + taxa interna de outliers por vetor

**H2 — Hipótese sobre impacto no mercado por setor**
- Hipótese implementada: *"Empresas do setor financeiro levam mais dias para o preço das ações se recuperar após incidentes de cibersegurança, comparado aos outros setores."*
- Resultado: **Parcialmente confirmada** — Financials aparece em 5º lugar (116 dias); Information Technology (~140 dias) e Consumer Discretionary (~130 dias) lideram
- Gráfico 3: Distribuição de `days_to_price_recovery` por setor (barplot com intervalo de confiança)
- Gráfico 4: Matriz de correlação entre `abnormal_return_1d`, `abnormal_return_30d`, `car_0_to_30`, `car_0_to_90`, `pre_incident_volatility_30d`, `post_incident_volatility_30d`, `days_to_price_recovery`

**H3 — Hipótese sobre evolução temporal e severidade**
- Hipótese implementada: *"A proporção de incidentes de alta severidade (_is_high_impact = 1) cresceu entre 2021 e 2025, mostrando que os ataques estão ficando mais graves."*
- Resultado: **Parcialmente confirmada** — volume cresceu 75% (131→229 incidentes), mas a taxa de alto impacto *caiu* de 50,4% para 41,0% (−2,1 p.p./ano)
- Gráfico 5: Frequência de incidentes por ano, separada por `_is_high_impact` (barplot empilhado + linha de proporção)
- Gráfico 6: Análise por recorte — volume e proporção de alto impacto por `attack_vector` (barplot duplo)

### Requisitos técnicos
- Cada gráfico deve ter título, eixos rotulados e uma célula markdown com **interpretação textual + conclusão orientada a decisão**
- Obrigatório cobrir: distribuição de variáveis-chave, análise de outliers, matriz de correlação, ao menos 1 análise por recorte
- Mínimo: 3 hipóteses, 6 gráficos (pode adicionar mais)

**Entregável adicional:**
- Atualizar o arquivo de **data lineage** para refletir o fluxo completo até a camada Ouro (incluindo as etapas que Lucas implementará)
- Formato sugerido: diagrama textual (Mermaid ou tabela Markdown) dentro do notebook ou em arquivo separado

---

## Lucas — Camada Ouro

### Objetivo
Transformar os dados da camada Prata em um dataset ML-ready (camada Ouro), aplicando encoding, scaling, tratamento de nulos e outliers com o padrão fit/transform.

### Tarefas

**1. Split treino/teste antes de qualquer transformação**
- Separar os datasets em treino (80%) e teste (20%) com `stratify` na label alvo (`_is_high_impact` ou `_is_severe_loss`)
- Garantir que o `fit` ocorra apenas no treino e o `transform` seja aplicado em ambos

**2. Encoding (mínimo 2 técnicas distintas)**
- Técnica 1 — **Label Encoding** ou **Ordinal Encoding**: para colunas com ordem natural (ex: `severity_level` se houver — baixo/médio/alto)
- Técnica 2 — **One-Hot Encoding**: para colunas nominais sem ordem (ex: `attack_vector`, `sector`, `country`)
- Justificar a escolha de cada técnica por coluna

**3. Scaling (mínimo 1 estratégia)**
- Aplicar **StandardScaler** ou **RobustScaler** nas variáveis numéricas contínuas relevantes
- Sugestão: usar `RobustScaler` dado que dados financeiros tendem a ter outliers — justificar a escolha
- Variáveis candidatas: `downtime_hours`, `records_compromised`, `total_loss_usd`, `ransom_paid_usd`, `price_recovery_days`

**4. Tratamento de missing values (mínimo 2 estratégias distintas)**
- Estratégia 1 — **Mediana**: para variáveis numéricas com distribuição assimétrica (ex: `total_loss_usd`, `ransom_paid_usd`)
- Estratégia 2 — **Moda ou categoria 'desconhecido'**: para variáveis categóricas com nulos residuais
- Documentar quais colunas receberam cada estratégia e por quê

**5. Identificação e tratamento de outliers (mínimo 2 colunas)**
- Coluna 1 — ex: `total_loss_usd`: identificar com IQR, tratar com **winsorização** (clamp no percentil 1–99)
- Coluna 2 — ex: `records_compromised`: identificar com Z-score, tratar com **log-transform** ou winsorização
- Para cada coluna: gráfico antes/depois + justificativa da abordagem

**6. Revisão de data leakage**
- Verificar o `checklist_anti_leakage.csv` da camada Prata
- Confirmar remoção das colunas ALTO risco já removidas; documentar decisão para colunas MEDIO risco
- Garantir que labels (`_is_high_impact`, `_is_severe_loss`, `_is_significant_drop`) não sejam usadas como features

**7. Salvar e documentar**
- Salvar dataset Ouro em: `data/gold/gold_incidents.parquet`, `gold_market.parquet`, `gold_financials.parquet`
- Criar tabela de transformações da camada Ouro em `data/gold/documentacao_transformacoes_ouro.csv` com colunas: `etapa`, `coluna`, `tecnica`, `justificativa`
- Atualizar `checklist_anti_leakage.csv` refletindo o estado pós-Ouro

### Entregáveis
- Código da camada Ouro no notebook (seção dedicada)
- `data/gold/` com 3 arquivos Parquet
- `data/gold/documentacao_transformacoes_ouro.csv`
- `checklist_anti_leakage.csv` atualizado
- Relatório de qualidade atualizado cobrindo Prata + Ouro

---

## Bernardo — Modelagem ML + Refatoração PySpark + README

### Objetivo
Treinar e avaliar modelos de Árvore de Decisão sobre os dados da camada Ouro, comparar com resultados da camada Prata, e refatorar etapas do pipeline usando PySpark.

### Tarefas — Modelagem ML

**1. Preparação**
- Usar os datasets da camada Ouro produzidos por Lucas
- Definir a variável target (sugestão: `_is_high_impact`) e as features
- Confirmar que o split treino/teste está balanceado (verificar distribuição da classe)

**2. Treinar 2 modelos de Árvore de Decisão com configurações distintas**

| Modelo | Configuração sugerida |
|--------|-----------------------|
| DT_v1 | `max_depth=5`, `criterion='gini'` |
| DT_v2 | `max_depth=10`, `criterion='entropy'` |

- Comparar resultados lado a lado

**3. Avaliação (mínimo 3 métricas)**
- Acurácia, Precisão, Recall e F1-score para ambos os modelos
- Gerar **matriz de confusão** para o melhor modelo (heatmap via Seaborn)
- Gerar **visualização da árvore** do melhor modelo (`plot_tree` ou `export_graphviz`)

**4. Comparação Prata vs Ouro (obrigatório)**
- Treinar o mesmo modelo (mesma config) com os dados da camada Prata (sem encoding/scaling)
- Comparar as métricas Prata × Ouro em uma tabela
- Redigir célula markdown com conclusão: *o pré-processamento melhorou (ou não) a performance e por quê*

### Tarefas — Refatoração PySpark

**Objetivo:** Refatorar ≥2 etapas do pipeline substituindo Pandas por PySpark.

**Etapa 1 — Ingestão e qualidade (refatoração da camada Bronze)**
- Ler os CSVs da pasta `data/raw/` com `spark.read.csv`
- Aplicar ao menos 1 **join** entre `incidents_master` e `financial_impact` por `incident_id`
- Aplicar ao menos 1 **groupBy com agregação**: ex: contagem de incidentes por `attack_vector` e média de `total_loss_usd`
- Escrever resultado em Parquet: `data/gold/spark_output.parquet`

**Etapa 2 — Feature engineering com window function**
- Ler o Parquet da etapa 1
- Aplicar ao menos 1 **função de janela**: ex: `rank()` de incidentes por `total_loss_usd` dentro de cada `sector`, ou média móvel de perdas por trimestre
- Escrever resultado final em Parquet ou Delta

**Comparação de performance**
- Cronometrar a mesma operação (ex: groupBy + aggregation) em Pandas e em PySpark
- Registrar os tempos e discutir os resultados em célula markdown

### Tarefas — README

Atualizar o `README.md` com:
- Pré-requisitos (Python, Java para PySpark, versões)
- Como instalar dependências (`pip install -r requirements.txt`)
- Como executar o notebook do início ao fim (ordem das seções)
- Descrição de cada camada (Bronze, Prata, Ouro, ML-Ready)
- Descrição dos entregáveis e onde encontrá-los

### Entregáveis
- Seção de modelagem ML no notebook com tabela comparativa Prata × Ouro
- Seção PySpark no notebook com join, groupBy, window function e comparação de tempo
- `data/gold/spark_output.parquet` ou equivalente
- `README.md` atualizado

---

## Checklist Final — Requisitos do Projeto

### EDA (Rodrigo)
- [x] ≥3 hipóteses formuladas e testadas — H1 (APT/ransomware), H2 (setor financeiro), H3 (evolução temporal)
- [x] ≥6 gráficos com interpretação textual — 2 gráficos por hipótese + 4 gráficos EDA geral
- [x] Cobre: distribuição, outliers, correlação, análise por recorte
- [x] Data lineage atualizado até camada Ouro — célula markdown completa no notebook

### Camada Ouro (Lucas)
- [x] ≥2 técnicas de encoding distintas — OrdinalEncoder (`attribution_confidence`) + OneHotEncoder (nominais)
- [x] ≥1 estratégia de scaling — RobustScaler em 7 colunas numéricas contínuas
- [x] ≥2 estratégias de tratamento de missing values — mediana (numéricas) + constante `'nao_disponivel'` (categóricas)
- [x] Outliers tratados em ≥2 colunas (com justificativa) — winsorização IQR (`company_revenue_usd`) + log1p (`total_loss_usd`)
- [x] Colunas de data leakage removidas ou justificadas — `data_compromised_records` e `downtime_hours` removidas
- [x] Pipeline no padrão fit/transform (sem vazamento treino→teste) — ColumnTransformer fittado só no treino
- [x] Dataset salvo em Parquet em `data/gold/` — `gold_dataset.parquet`, `gold_incidents.parquet`, `gold_market.parquet`, `gold_financials.parquet`
- [x] Tabela de transformações documentada — `data/gold/documentacao_transformacoes_ouro.csv`
- [x] Checklist anti-leakage atualizado — `data/gold/checklist_anti_leakage_ouro.csv`

### Modelagem ML (Bernardo)
- [x] ≥2 modelos de Árvore de Decisão com configs distintas — DT_v1 (`max_depth=5`, gini) e DT_v2 (`max_depth=10`, entropy)
- [x] Split treino/teste com representatividade de classes — 80/20 com `stratify=_is_high_impact`
- [x] ≥3 métricas avaliadas (acurácia, precisão, recall, F1)
- [x] Matriz de confusão do melhor modelo — gerada no notebook (célula 111)
- [x] Visualização da árvore resultante — gerada no notebook (célula 112)
- [x] Comparação explícita Prata × Ouro com conclusão — tabela Delta + célula markdown

### PySpark (Bernardo)
- [x] ≥2 etapas refatoradas com PySpark — ingestão/join e window functions
- [x] Leitura em Parquet (ou Delta)
- [x] ≥1 operação de join — `incidents` + `financials` por `incident_id`
- [x] ≥1 groupBy com agregação — contagem e média por `attack_vector_primary`
- [x] ≥1 função de janela — rank por `total_loss_usd` dentro de cada setor
- [x] Escrita em Parquet (ou Delta) — `spark_output.parquet` e `spark_ranked.parquet`
- [x] Comparação de tempo Pandas vs PySpark discutida — célula markdown com tempos cronometrados

### Entregáveis Gerais
- [x] notebook.ipynb com todas as etapas executáveis
- [ ] Relatório de qualidade atualizado (Prata + Ouro) — **executar célula 105 do notebook para gerar `relatorio_qualidade_ouro.csv`**
- [x] README com instruções completas