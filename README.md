# Projeto: Pipeline Medallion — Bronze, Prata, Ouro e ML-Ready

## Descrição do Projeto
Este repositório implementa um pipeline de engenharia de dados para cibersegurança seguindo a Arquitetura Medallion: Bronze (ingestão), Prata (limpeza), Ouro (feature engineering) e ML-Ready (modelagem). O objetivo é transformar dados de incidentes e impactos financeiros em um conjunto robusto para análise e predição de alto impacto.

## Pré-requisitos
- Python 3.9+
- Java 17 ou superior (testado com Java 24)
- PySpark 4.x (instalado via `requirements.txt`)
- Dependências Python listadas em `requirements.txt`

> **Atenção (Windows):** Defina a variável `JAVA_HOME` apontando para o diretório do JDK (ex: `C:\Program Files\Java\jdk-24`). Essa configuração já está aplicada na célula de inicialização do Spark no notebook. **Não use `-Djava.security.manager=allow` com Java 24+**: o SecurityManager foi removido nessa versão e o flag causa falha imediata na inicialização da JVM.

## Instalação

```bash
pip install -r requirements.txt
```

## Como executar

1. Abra o arquivo `notebook.ipynb` no VS Code, JupyterLab ou Jupyter Notebook.
2. Execute **todas as células em ordem**, de cima para baixo.
3. O notebook está organizado nas seguintes seções:

| # | Seção | O que faz |
|---|-------|-----------|
| 1 | **Bronze** | Lê os CSVs brutos, converte para Parquet e adiciona metadados |
| 2 | **Silver** | Limpeza, padronização, imputação de nulos e criação de labels ML |
| 3 | **EDA** | Análise exploratória e validação de 3 hipóteses de negócio |
| 4 | **Ouro** | Feature engineering, encoding, scaling e salvamento ML-ready |
| 5 | **Modelagem ML** | Treino e avaliação de Decision Trees; comparação Prata × Ouro |
| 6 | **PySpark** | Join, aggregation e window functions; comparação de tempo Pandas × Spark |
| 7 | **Data Lineage** | Registro completo do fluxo de dados do pipeline |

## Arquitetura das camadas

| Camada | Descrição | Formato de Saída |
|--------|-----------|------------------|
| Bronze | Dados brutos convertidos para Parquet com metadados de ingestão | Parquet |
| Prata | Dados limpos, validados, sem leakage e com labels ML criadas | Parquet |
| Ouro | Features encoded e scaled, prontas para treinamento (fit/transform) | Parquet |
| ML-Ready | Dataset unificado com coluna `_split` para treino e teste | Parquet |

## Entregáveis

| Entregável | Caminho |
|------------|---------|
| Notebook principal | `notebook.ipynb` |
| Dataset ML-Ready | `data/gold/gold_dataset.parquet` |
| Relatório de qualidade — Ouro | `data/gold/relatorio_qualidade_ouro.csv` |
| Documentação de transformações — Ouro | `data/gold/documentacao_transformacoes_ouro.csv` |
| Checklist anti-leakage — Ouro | `data/gold/checklist_anti_leakage_ouro.csv` |
| Saída PySpark join | `data/gold/spark_output.parquet` |
| Saída PySpark window | `data/gold/spark_ranked.parquet` |
| Checklist anti-leakage — Silver | `data/silver/checklist_anti_leakage.csv` |
| Documentação de transformações — Silver | `data/silver/documentacao_transformacoes.csv` |

## Estrutura de pastas

```text
Arquitetura_medalion_para_cyberseguranca/
├── data/
│   ├── raw/          ← CSVs originais (não modificar)
│   ├── bronze/       ← Parquet + metadados de ingestão
│   ├── silver/       ← Parquet limpo + labels ML
│   └── gold/         ← Parquet ML-ready + relatórios Ouro
├── docs/             ← Figuras geradas (matriz de confusão, árvore)
├── notebook.ipynb    ← Único arquivo de código
├── requirements.txt
├── README.md
└── CLAUDE.md
```

## Observações importantes

- Todo o código está em `notebook.ipynb`. Execute as células em ordem — cada seção depende das anteriores.
- A camada Ouro aplica o padrão `fit/transform` corretamente: o `fit` ocorre exclusivamente nos dados de treino.
- As colunas `data_compromised_records` e `downtime_hours` foram **removidas** da camada Ouro pois são a fonte direta do target `_is_high_impact` (leakage).
- O PySpark salva os resultados via `.toPandas().to_parquet()` para evitar dependência do `winutils.exe` no Windows.
