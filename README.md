# Precificação de Informação On-chain

Código do Trabalho de Conclusão de Curso **"Precificação de Informação Onchain: um modelo de IA explicável para mensurar o valor econômico de datasets em ecossistemas InfoFi"** — Especialização em Inteligência Artificial e Big Data, ICMC/USP São Carlos, 2026.

**Autora:** Adriana Ledezma Molina

## Sobre o projeto

O trabalho propõe e avalia um modelo de inteligência artificial explicável (XAI) para mensurar o conteúdo informacional de datasets de mercado associados a cinco tokens DeFi (AAVE, UNI, COMP, CRV e ETH), no período de janeiro de 2021 a julho de 2026.

Principais componentes do pipeline:

- **Coleta híbrida**: preços diários OHLC do Kaggle (fonte primária) complementados via Yahoo Finance (volume, dias recentes e preenchimento de lacunas), com prioridade à fonte primária;
- **Engenharia de features**: 8 atributos técnicos interpretáveis, reduzidos a 7 após análise de multicolinearidade (VIF);
- **Modelagem**: baseline ingênuo, Ridge, Random Forest e Gradient Boosting, sob partição temporal com purga e validação cruzada temporal (TimeSeriesSplit com gap);
- **Ranking InfoFi**: mensuração do conteúdo informacional por token via Information Coefficient (IC) fora da amostra, com correção de Bonferroni;
- **Explicabilidade**: análise SHAP (TreeExplainer) do modelo global e do modelo de AAVE.

## Principais resultados

- Nenhum modelo supera o baseline trivial em média (R² ≈ −0,06; acerto direcional 49,4%) — resultado consistente com a hipótese de eficiência de mercado;
- O conteúdo informacional é heterogêneo entre tokens: AAVE apresenta IC positivo e significativo (+0,144; p < 0,01) e UNI apresenta sinal invertido (−0,134; p < 0,01);
- O modelo de AAVE aprendeu relação positiva entre volatilidade de 30 dias e retorno futuro — oposta à do modelo global — ilustrando que uma mesma feature carrega valor informacional distinto por ativo.

## Como executar

1. Clone o repositório e instale as dependências:

```bash
pip install -r requirements.txt
```

2. Abra o notebook `TCC_InfoFi.ipynb` (Jupyter, VS Code ou Google Colab);

3. Execute as células em ordem (Kernel → Restart & Run All). O download dos dados do Kaggle é automático via `kagglehub` (dataset público, não requer credenciais).

**Nota de reprodutibilidade:** a constante `DATA_HOJE` está fixada em `2026-07-09` para que os resultados coincidam com os reportados na monografia. Para executar com dados atualizados, substitua pela data corrente — os valores podem variar marginalmente.

## Fontes de dados

- [Crypto Currencies Daily Prices (Kaggle)](https://www.kaggle.com/datasets/svaningelgem/crypto-currencies-daily-prices) — preços OHLC diários (fonte primária)
- Yahoo Finance via [yfinance](https://github.com/ranaroussi/yfinance) — volume e complementação de lacunas

## Estrutura

```
├── TCC_InfoFi.ipynb      # notebook principal (pipeline completo)
├── requirements.txt      # dependências
├── dados/                # figuras geradas (300 dpi) — criada na execução
└── README.md
```

## Licença

Código disponibilizado para fins acadêmicos e de reprodutibilidade.
