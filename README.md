# Precificação de Informação On-chain

Código do Trabalho de Conclusão de Curso **"Precificação de Informação On-chain: um modelo de IA explicável para mensurar o valor econômico de datasets em ecossistemas InfoFi"** - Especialização em Inteligência Artificial e Big Data, ICMC/USP São Carlos, 2026.

**Autora:** Carla Adriana Ledezma Molina

## Sobre o projeto

O trabalho propõe e avalia um modelo de inteligência artificial explicável (XAI) para mensurar o conteúdo informacional de datasets de mercado associados a cinco tokens DeFi (AAVE, UNI, COMP, CRV e ETH), no período de janeiro de 2021 a julho de 2026.

Principais componentes do pipeline:

- **Coleta híbrida**: preços diários OHLC do Kaggle (fonte primária) complementados via Yahoo Finance (volume, dias recentes e preenchimento de lacunas), com prioridade à fonte primária;
- **Engenharia de features**: 8 atributos técnicos interpretáveis, reduzidos a 7 após análise de multicolinearidade (VIF);
- **Modelagem**: baseline ingênuo, Ridge, Random Forest e Gradient Boosting, sob partição temporal com purga e validação cruzada temporal (`TimeSeriesSplit` com `gap`);
- **Ranking InfoFi**: mensuração do conteúdo informacional por token via Information Coefficient (IC) fora da amostra, com correção de Bonferroni;
- **Verificação de robustez**: block bootstrap, subamostragem não sobreposta e teste de permutação por deslocamento circular, para corrigir a dependência serial induzida pela sobreposição das janelas do alvo;
- **Explicabilidade**: análise SHAP (`TreeExplainer`) do modelo global e do modelo de AAVE.

## Principais resultados

**O que o estudo estabelece:**

- Nenhum modelo supera o baseline trivial (R² médio ≈ −0,06 em cinco janelas temporais; acerto direcional 49,4%), resultado consistente com a hipótese de eficiência de mercado em sua forma fraca. Sinais técnicos públicos não contêm informação explorável no horizonte de 7 dias;
- **Rankings de valor informacional baseados em p-valores nominais produzem falsos positivos.** O ranking nominal apontava dois tokens significativos ao nível de 1%, sobreviventes à correção de Bonferroni (AAVE, IC = +0,144; UNI, IC = −0,134). Sob inferência que incorpora a dependência serial do alvo, os intervalos de confiança dos cinco ICs contêm o zero e a dispersão observada não excede a distribuição nula (p = 0,35 e p = 0,31, nas duas variantes do teste de permutação). A heterogeneidade aparente era artefato inferencial;
- Consequentemente, o protocolo de validação é parte inseparável da métrica de valoração, e não um apêndice metodológico.

**O que o estudo não estabelece:**

- A ausência de rejeição é inconclusiva por insuficiência amostral (≈ 57 observações independentes por token), e não evidência de que os datasets sejam informacionalmente equivalentes;
- As estimativas pontuais dos ICs são estáveis sob subamostragem não sobreposta, mas o tamanho amostral disponível não permite resolver diferenças da ordem de 0,15.

**Por que o resultado negativo importa:** as features utilizadas são deliberadamente públicas, gratuitas e trivialmente replicáveis, a categoria de dado cujo valor marginal a teoria prevê ser nulo. O instrumento e o protocolo aqui propostos permanecem aplicáveis a datasets on-chain especializados (TVL, fluxos entre carteiras, concentração de holders, atividade de contratos inteligentes), onde o valor informacional marginal tende a residir. Essa é a extensão prioritária do trabalho.

## Como executar

1. Clone o repositório e instale as dependências:

```bash
pip install -r requirements.txt
```

2. Abra o notebook `TCC_InfoFi.ipynb` (Jupyter, VS Code ou Google Colab);
3. Execute as células em ordem (Kernel → Restart & Run All). O download dos dados do Kaggle é automático via `kagglehub` (dataset público, não requer credenciais).

**Nota de reprodutibilidade:** a constante `DATA_HOJE` está fixada em `2026-07-09` para que os resultados coincidam com os reportados na monografia. Para executar com dados atualizados, substitua pela data corrente — os valores podem variar marginalmente. Os testes de reamostragem usam `numpy.random.default_rng(42)` e 2.000 réplicas; alterar a semente desloca os p-valores na terceira casa decimal, sem afetar as conclusões.

## Fontes de dados

- [Crypto Currencies Daily Prices (Kaggle)](https://www.kaggle.com/datasets/svaningelgem/crypto-currencies-daily-prices) — preços OHLC diários (fonte primária)
- Yahoo Finance via [yfinance](https://github.com/ranaroussi/yfinance) — volume e complementação de lacunas

## Estrutura

```
├── TCC_InfoFi.ipynb      # notebook principal (pipeline completo)
├── requirements.txt      # dependências
├── figuras/              # figuras geradas (300 dpi) — criada na execução
└── README.md
```

Seções do notebook, na ordem de execução:

| Seção | Conteúdo |
|---|---|
| 1–3 | Configuração, coleta híbrida e verificação de qualidade |
| 4–6 | Análise exploratória (distribuições, volatilidade, drawdown, correlações) |
| 7 | Engenharia de features e seleção via VIF |
| 8 | Modelagem: partição com purga, validação cruzada temporal, comparação multimodelo |
| 9 | Ranking InfoFi por token e verificação de robustez |
| 10 | Explicabilidade (SHAP) e conclusões |

## Licença

Código disponibilizado para fins acadêmicos e de reprodutibilidade.
