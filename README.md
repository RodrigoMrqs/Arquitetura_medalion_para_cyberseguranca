# Projeto: Pipeline Medalion — Bronze, Prata, Ouro e ML-Ready

## Descrição do Projeto
Este repositório implementa um pipeline de cibersegurança em camadas: Bronze (ingestão), Prata (limpeza), Ouro (feature engineering) e ML-Ready (modelagem). O objetivo é transformar dados de incidentes e impactos financeiros em um conjunto robusto para análise e predição de alto impacto.

## Pré-requisitos
- Python 3.9+
- Java 11+
- Apache Spark 3.x
- Dependências Python disponíveis em `requirements.txt`

## Instalação
```bash
pip install -r requirements.txt
```

## Como executar
1. Abra o notebook `notebook.ipynb`.
2. Execute a seção de **Bronze** para ingestão dos dados brutos.
3. Execute a seção de **Prata** para limpeza e validação.
4. Execute a seção de **Ouro** para feature engineering e preparação ML-Ready.
5. Execute a seção de **ML-Ready** para treino e avaliação de Decision Trees.
6. Execute a seção de **PySpark** para refatoração, join, agregações e window function.

## Arquitetura das camadas
| Camada | Descrição | Formato de Saída |
|--------|-----------|------------------|
| Bronze | Dados brutos ingeridos e convertidos para Parquet | Parquet |
| Prata | Dados limpos, validados e sem leakage | Parquet |
| Ouro | Dados com feature engineering e pré-processamento | Parquet |
| ML-Ready | Dataset pronto para treino de modelos | Parquet |

## Entregáveis
| Entregável | Caminho |
|------------|---------|
| Notebook principal | `notebook.ipynb` |
| Saída PySpark join | `data/gold/spark_output.parquet` |
| Saída PySpark ranked | `data/gold/spark_ranked.parquet` |
| Matriz de confusão | `docs/confusion_matrix.png` |
| Visualização da árvore | `docs/decision_tree.png` |
| Checklist anti-leakage | `data/silver/checklist_anti_leakage.csv` |
| Documentação de transformações | `data/silver/documentacao_transformacoes.csv` |

## Estrutura de pastas
```text
Arquitetura_medalion_para_cyberseguranca/
├── data/
│   ├── bronze/
│   ├── raw/
│   ├── silver/
│   └── gold/
├── docs/
├── notebook.ipynb
├── README.md
└── requirements.txt
```

## Observações importantes
- Todas as implementações de código para a modelagem e PySpark são feitas dentro de `notebook.ipynb`.
- As figuras são salvas em `docs/`.
- O modelo de comparação Prata × Ouro valida a importância do pré-processamento para a performance de produção.
