# Score de Telefone — Santander | Emdia

### Machine Learning para Ranking e Priorização de Contatos

Projeto acadêmico desenvolvido em equipe durante a MarTech da ESPM, em 2026, a partir de um case Santander | Emdia.

A solução utiliza Machine Learning para analisar o histórico de contatos e priorizar os telefones com maior probabilidade de gerar um contato produtivo, transformando dados brutos em um ranking dos cinco melhores números por cliente.

## Sobre minha participação

Participei do desenvolvimento deste projeto de forma colaborativa junto à equipe, contribuindo nas atividades realizadas pelo grupo ao longo das sprints e na construção das entregas do projeto.

> Este repositório é um fork do projeto desenvolvido coletivamente pela equipe e é mantido como registro da minha participação e para fins de portfólio acadêmico.

---

## 1. Problema de Negócio

A base de clientes possui vários telefones associados a cada `id_pessoa`.

Na operação de discagem, ligar para todos os números sem prioridade aumenta custo, tempo improdutivo e volume de tentativas em telefones com baixa chance de sucesso.

O objetivo do projeto é criar um pipeline que:

- consolide e trate os dados históricos de acionamento;
- crie features relevantes para prever sucesso de contato;
- treine modelos de classificação e ranking;
- gere score de conversão por telefone;
- entregue os Top 5 telefones por cliente;
- exporte o resultado em um formato padronizado acordado com a Emdia.

---

## 2. Solução Proposta

A solução foi estruturada como um pipeline de dados e Machine Learning:

```text
base_emdia/
  -> leitura e consolidação
  -> tratamento de nulos e inconsistências
  -> feature engineering
  -> treino e avaliação de modelos
  -> scoring diário
  -> ranking Top 5 por cliente
  -> exportação padrão Emdia
  -> versionamento do modelo treinado
```

O projeto utiliza principalmente:

- Python 3
- Pandas e NumPy
- PyArrow / Parquet
- scikit-learn
- XGBoost
- Optuna
- Joblib
- Streamlit e Plotly

---

## 3. Estrutura do Projeto

```text
.
├── santander/
│   ├── base_emdia/                         # arquivos brutos em Parquet
│   ├── docs/adr/                           # decisões técnicas do projeto
│   ├── models/                             # modelos versionados
│   ├── querys/                             # arquivos auxiliares de consultas
│   ├── leitura.py                          # leitura, consolidação e limpeza inicial
│   ├── eda_analysis.py                     # análise exploratória
│   ├── eda_report.md                       # relatório da EDA
│   ├── feature_engineering.py              # criação de features e target
│   ├── baseline_model.py                   # baseline com Regressão Logística
│   ├── hyperparameter_tuning.py            # tuning com Optuna e XGBoost
│   ├── evaluation_metrics.py               # métricas de ranking
│   ├── feature_importance.py               # análise de importância das features
│   ├── pipeline_diario.py                  # pipeline end-to-end de scoring
│   ├── export_output.py                    # exportação no schema Emdia
│   ├── model_registry.py                   # versionamento de modelos
│   ├── version_existing_model.py           # registra modelo já treinado
│   ├── dashboard_pandas.py                 # dashboard Streamlit
│   └── requirements.txt                    # dependências
├── README.md
└── martech-emdia-2026.code-workspace
```

---

## 4. Principais Etapas Executadas

### 4.1. Escolha do Ambiente

Foi definido o uso de Python 3 para garantir compatibilidade com bibliotecas modernas de ciência de dados, f-strings, encoding UTF-8 e arquivos Parquet.

A decisão está documentada em:

```text
santander/docs/adr/0001-escolha-ambiente-python.md
```

### 4.2. Leitura e Consolidação dos Dados

O script `leitura.py` carrega os arquivos da pasta `base_emdia/`, consolida os dados e aplica tratamentos iniciais.

O resultado principal desta etapa é:

```text
santander/dataset_consolidado.parquet
```

### 4.3. Análise Exploratória

A análise exploratória identificou pontos importantes relacionados à qualidade dos dados:

- idades negativas ou impossíveis;
- telefones com volume extremo de acionamentos;
- telefones com muitas tentativas e nenhum CPC;
- necessidade de padronização e tratamento de valores nulos.

Os achados estão documentados em:

```text
santander/eda_analysis.py
santander/eda_report.md
santander/docs/adr/0002-tratamento-outliers.md
```

### 4.4. Feature Engineering

O script `feature_engineering.py` transforma a base consolidada em uma base preparada para modelagem.

Entre as principais features criadas ou organizadas estão:

- `recencia`: aproximação do tempo desde o último acionamento;
- `frequencia`: volume de acionamentos;
- `tipo_linha`: classificação entre móvel ou fixo;
- `operadora_prefix`: proxy de operadora pelo prefixo;
- `ddd`;
- `total_telefones_cliente`;
- features históricas de alô, CPC, produtivo, promessa, quedas, caixa postal e cliente desliga.

Também foi criada a variável alvo:

```text
target_label = 1 quando cpc_futuro == 1 OU promessa_futuro == 1
```

Essa definição faz o modelo priorizar telefones com maior chance de contato produtivo e conversão.

Output da etapa:

```text
santander/dataset_features.parquet
```

Documentação:

```text
santander/docs/adr/0003-feature-engineering.md
santander/docs/adr/0006-definicao-target.md
```

### 4.5. Modelo Baseline

Foi treinado um modelo baseline utilizando Regressão Logística balanceada.

A escolha ocorreu por ser um modelo:

- simples;
- rápido;
- interpretável;
- útil como ponto de comparação para modelos mais complexos.

Resultado documentado:

```text
AUC-ROC baseline: 0.8131
```

Artefato:

```text
santander/baseline_logistic_model.joblib
```

Documentação:

```text
santander/docs/adr/0004-modelo-baseline.md
```

### 4.6. Métricas de Referência

Como o objetivo final do projeto é a geração de um ranking, foram utilizadas métricas voltadas à capacidade preditiva e à qualidade da priorização:

- AUC-ROC global;
- Hit Rate @ 1;
- Hit Rate @ 3;
- Hit Rate @ 5.

O Hit Rate @ K mede se pelo menos um telefone produtivo aparece entre os K primeiros telefones do cliente.

Script:

```text
santander/evaluation_metrics.py
```

Documentação:

```text
santander/docs/adr/0005-metricas-referencia.md
```

### 4.7. Modelo XGBoost com Otimização

Após o baseline, foi treinado um modelo XGBoost com hiperparâmetros otimizados via Optuna.

O pipeline de modelagem inclui:

- imputação de valores nulos;
- padronização de variáveis numéricas;
- OneHotEncoding de variáveis categóricas;
- classificador XGBoost.

Artefato principal:

```text
santander/tuned_xgboost_model.joblib
```

Arquivos auxiliares:

```text
santander/best_hyperparams.json
santander/docs/optuna_optimization_history.html
santander/docs/optuna_param_importances.html
```

O modelo final apresentou AUC-ROC de aproximadamente:

```text
0.8751
```

### 4.8. Pipeline Diário de Scoring

O `pipeline_diario.py` executa o fluxo end-to-end:

1. lê os dados brutos;
2. trata valores nulos;
3. gera features;
4. carrega o modelo treinado;
5. gera a probabilidade de conversão;
6. gera a classe prevista;
7. ordena os telefones por cliente;
8. filtra os Top 5;
9. chama a exportação no padrão Emdia.

Output intermediário:

```text
santander/dataset_scored_diario_top5.parquet
```

### 4.9. Exportação no Padrão Emdia

O script `export_output.py` gera o output final validado.

Schema acordado:

| Coluna | Tipo | Descrição |
| --- | --- | --- |
| `id_pessoa` | string | Identificador do cliente |
| `telefone` | string | Telefone com DDD |
| `rank_telefone` | int8 | Posição no ranking |
| `score_conversao` | float32 | Probabilidade prevista |
| `flag_conversao` | int8 | Classe prevista |
| `sexo` | category | Sexo do cliente |
| `uf` | category | UF |
| `tipo_linha` | category | Móvel ou Fixo |
| `idade` | Int16 | Idade nullable |
| `dt_processamento` | string | Data YYYY-MM-DD |

Regras validadas:

- máximo de 5 telefones por cliente;
- `rank_telefone` entre 1 e 5;
- ausência de duplicatas de `(id_pessoa, telefone)`;
- `score_conversao` sempre entre 0 e 1;
- colunas na ordem acordada.

Outputs finais:

```text
santander/output_scored_20260531.parquet
santander/output_scored_20260531.csv
```

Resultado validado:

```text
684.388 linhas
0 duplicatas de (id_pessoa, telefone)
rank máximo = 5
score dentro do intervalo [0, 1]
```

Documentação:

```text
santander/docs/adr/0007-schema-exportacao-output.md
```

### 4.10. Versionamento do Modelo

Foi criado um registry local de modelos em `model_registry.py`.

Cada versão salva:

- modelo `.joblib`;
- metadata `.json`;
- log consolidado em `model_versions.csv`;
- alias legado, como `tuned_xgboost_model.joblib`, para manter compatibilidade com o pipeline.

Formato:

```text
santander/models/<model_name>/<model_name>_YYYYMMDD_HHMMSS.joblib
```

Versão registrada:

```text
tuned_xgboost_20260531_204055
```

Métricas registradas:

| Métrica | Valor |
| --- | ---: |
| AUC-ROC | 0.875087 |
| Accuracy | 0.840134 |
| Precision | 0.016330 |
| Recall | 0.775362 |
| F1 | 0.031986 |

Arquivos:

```text
santander/model_registry.py
santander/version_existing_model.py
santander/model_versions.csv
santander/models/tuned_xgboost/tuned_xgboost_20260531_204055.joblib
santander/models/tuned_xgboost/tuned_xgboost_20260531_204055.json
```

Documentação:

```text
santander/docs/adr/0008-versionamento-modelo.md
```

---

## 5. Como Executar

Clone este repositório:

```powershell
git clone https://github.com/maduortega/martech-emdia-2026.git
```

Entre na pasta do projeto:

```powershell
cd martech-emdia-2026/santander
```

Instale as dependências:

```powershell
pip install -r requirements.txt
```

Execute a consolidação e leitura:

```powershell
python leitura.py
```

Execute a engenharia de features:

```powershell
python feature_engineering.py
```

Treine o baseline:

```powershell
python baseline_model.py
```

Treine o modelo XGBoost com Optuna:

```powershell
python hyperparameter_tuning.py
```

Execute o pipeline diário completo:

```powershell
python pipeline_diario.py
```

Execute apenas a exportação formatada:

```powershell
python export_output.py
```

Registre uma versão de um modelo já treinado:

```powershell
python version_existing_model.py
```

Execute o dashboard:

```powershell
streamlit run dashboard_pandas.py
```

---

## 6. Entregáveis

### Datasets

```text
dataset_consolidado.parquet
dataset_features.parquet
dataset_scored_diario.parquet
dataset_scored_diario_top5.parquet
```

### Outputs para Emdia

```text
output_scored_20260531.parquet
output_scored_20260531.csv
```

### Modelos

```text
baseline_logistic_model.joblib
tuned_xgboost_model.joblib
tuned_xgboost_reduced_model.joblib
models/tuned_xgboost/tuned_xgboost_20260531_204055.joblib
```

### Governança

```text
model_versions.csv
docs/adr/
```

### Dashboard

```text
dashboard_pandas.py
```

---

## 7. Status das Sprints

### Sprint 1 — Concluída

- leitura e consolidação da base;
- análise exploratória;
- tratamento de problemas de qualidade;
- feature engineering;
- definição de features e dataset de treinamento.

### Sprint 2 — Concluída

- definição da target `target_label`;
- baseline com Regressão Logística;
- métricas de ranking;
- XGBoost com tuning via Optuna;
- pipeline diário de scoring;
- exportação formatada no padrão Emdia;
- versionamento do modelo treinado.

### Sprint 3 — Próximos Passos

- automatizar a execução do pipeline por data de processamento;
- criar testes automáticos para schema, ranking e exportação;
- adicionar monitoramento de drift e distribuição de scores;
- evoluir o dashboard para visão executiva e operacional;
- documentar instalação e execução para usuários não técnicos;
- revisar performance do modelo por UF, tipo de linha e grupos de clientes;
- preparar apresentação final com problema, metodologia, resultados e próximos passos.

---

## 8. Resumo Executivo

O projeto transforma uma base bruta de histórico de telefones em uma solução completa de ranking para discagem.

A solução cria variáveis de comportamento, treina modelos preditivos, calcula scores por telefone, seleciona os Top 5 por cliente e exporta o resultado em um schema validado para a Emdia.

O projeto possui pipeline end-to-end, output padronizado, modelo versionado e documentação das principais decisões técnicas.

O resultado final busca melhorar a eficiência operacional da discagem ao reduzir tentativas em números de baixo potencial e priorizar telefones com maior probabilidade de contato produtivo.

---

## 9. Créditos

Projeto desenvolvido de forma colaborativa durante a MarTech da ESPM — 2026.

**Case:** Santander | Emdia

**Participação:** Maria Eduarda Ortega

**Repositório original:**  
https://github.com/bevolpi/martech-emdia-2026

Este repositório é um fork mantido por Maria Eduarda Ortega como registro de sua participação no projeto e para composição de portfólio acadêmico.