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
- Hipótese sugerida: *"Ataques do tipo ransomware e supply_chain concentram maior impacto financeiro (total_loss_usd) em comparação com outros vetores."*
- Gráfico 1: Distribuição de `total_loss_usd` por `attack_vector` (boxplot ou violin plot)
- Gráfico 2: Análise de outliers de `total_loss_usd` (boxplot + IQR visualizado) com identificação de casos extremos por setor

**H2 — Hipótese sobre impacto no mercado por setor**
- Hipótese sugerida: *"Empresas do setor financeiro sofrem quedas de preço de ação mais prolongadas (mais dias para recuperação) após incidentes graves."*
- Gráfico 3: Distribuição de `price_recovery_days` por setor (barplot com erro)
- Gráfico 4: Matriz de correlação entre variáveis numéricas do `silver_market` (heatmap com `abnormal_return_1d`, `abnormal_return_30d`, `car_30d`, `volatility_change`, `price_recovery_days`)

**H3 — Hipótese sobre evolução temporal e severidade**
- Hipótese sugerida: *"Incidentes classificados como alta severidade (_is_high_impact = 1) aumentaram em frequência ao longo dos anos (2021–2025)."*
- Gráfico 5: Frequência de incidentes por trimestre, separada por `_is_high_impact` (lineplot ou barplot empilhado)
- Gráfico 6: Análise por recorte — frequência de ataque × severidade (heatmap com `attack_vector` × `_is_high_impact` ou `severity_level`)

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
- [ ] ≥3 hipóteses formuladas e testadas
- [ ] ≥6 gráficos com interpretação textual
- [ ] Cobre: distribuição, outliers, correlação, análise por recorte
- [ ] Data lineage atualizado até camada Ouro

### Camada Ouro (Lucas)
- [ ] ≥2 técnicas de encoding distintas
- [ ] ≥1 estratégia de scaling
- [ ] ≥2 estratégias de tratamento de missing values
- [ ] Outliers tratados em ≥2 colunas (com justificativa)
- [ ] Colunas de data leakage removidas ou justificadas
- [ ] Pipeline no padrão fit/transform (sem vazamento treino→teste)
- [ ] Dataset salvo em Parquet em `data/gold/`
- [ ] Tabela de transformações documentada
- [ ] Checklist anti-leakage atualizado

### Modelagem ML (Bernardo)
- [ ] ≥2 modelos de Árvore de Decisão com configs distintas
- [ ] Split treino/teste com representatividade de classes
- [ ] ≥3 métricas avaliadas (acurácia, precisão, recall, F1)
- [ ] Matriz de confusão do melhor modelo
- [ ] Visualização da árvore resultante
- [ ] Comparação explícita Prata × Ouro com conclusão

### PySpark (Bernardo)
- [ ] ≥2 etapas refatoradas com PySpark
- [ ] Leitura em Parquet (ou Delta)
- [ ] ≥1 operação de join
- [ ] ≥1 groupBy com agregação
- [ ] ≥1 função de janela
- [ ] Escrita em Parquet (ou Delta)
- [ ] Comparação de tempo Pandas vs PySpark discutida

### Entregáveis Gerais
- [ ] notebook.ipynb com todas as etapas executáveis
- [ ] Relatório de qualidade atualizado (Prata + Ouro)
- [ ] README com instruções completas
