# Precificação de Informação On-chain

Código do Trabalho de Conclusão de Curso **"Precificação de Informação Onchain: um modelo de IA explicável para mensurar o valor econômico de datasets em ecossistemas InfoFi"**

**Autora:** Adriana Ledezma Molina

---

## Contribuições

Este repositório acompanha três contribuições do trabalho:

**1. Um instrumento.** Um ranking de valor informacional baseado no *Information Coefficient* (IC) fora da amostra. Diferentemente de métricas de erro absoluto (R², RMSE), o IC permanece computável e informativo em regimes de baixa razão sinal-ruído, precisamente o regime em que se encontram os mercados eficientes.

**2. Um protocolo.** Demonstra-se que a validação por reamostragem em blocos estacionários não é uma etapa acessória de verificação, mas parte constitutiva da métrica: o mesmo dataset produz rankings distintos conforme o tratamento dado à dependência serial do alvo.

**3. Evidência empírica de um falso positivo.** Documenta-se, com dados reais e código público, um caso em que o ranking nominal atribuiria prêmio de preço ao AAVE com significância formal após correção de multiplicidade, e que não sobrevive à inferência corrigida.

> Em mercados de informação, o protocolo de validação é inseparável da métrica de valor. Um instrumento incapaz de recusar preço a um sinal espúrio não é um instrumento de valoração: é um mecanismo de transferência de risco do vendedor para o comprador.

---

## Sobre o projeto

O trabalho propõe e avalia um modelo de inteligência artificial explicável (XAI) para mensurar o conteúdo informacional de datasets de mercado associados a cinco tokens DeFi (AAVE, UNI, COMP, CRV e ETH), no período de janeiro de 2021 a julho de 2026.

Principais componentes do pipeline:

- **Coleta híbrida**: preços diários OHLC do Kaggle (fonte primária) complementados via Yahoo Finance (volume, dias recentes e preenchimento de lacunas), com prioridade à fonte primária;
- **Engenharia de features**: 8 atributos técnicos interpretáveis, reduzidos a 7 após análise de multicolinearidade (VIF);
- **Modelagem**: baseline ingênuo, Ridge, Random Forest e Gradient Boosting, sob partição temporal com purga e validação cruzada temporal (TimeSeriesSplit com gap);
- **Ranking InfoFi**: mensuração do conteúdo informacional por token via Information Coefficient (IC) fora da amostra, com correção de Bonferroni;
- **Verificação da inferência**: reamostragem em blocos estacionários e teste de permutação, para tratar a dependência serial induzida pela sobreposição das janelas do alvo;
- **Explicabilidade**: análise SHAP (TreeExplainer) do modelo global e do modelo de AAVE.

---

## Principais resultados

**H1 — previsibilidade: refutada.** Nenhum modelo supera o baseline trivial em média (R² ≈ −0,06; acerto direcional de 49,4%), resultado consistente com a hipótese de eficiência de mercado. O modelo mais flexível (Gradient Boosting) obteve o pior desempenho, o que é o comportamento esperado quando não há sinal a capturar.

**H2 — heterogeneidade entre ativos: inconclusiva.** Este é o resultado central e exige atenção à distinção entre as duas etapas da análise:

| Etapa | AAVE | UNI | Conclusão |
|---|---|---|---|
| Ranking **nominal** | IC = +0,144 (p < 0,01) | IC = −0,134 (p < 0,01) | heterogeneidade aparente |
| Inferência **corrigida** | IC = +0,153, IC 95% cruza zero | IC 95% cruza zero | não distinguível do acaso (p = 0,349) |

Os p-valores nominais supõem observações independentes. Como o alvo é o retorno acumulado de 7 dias calculado sobre dados diários, dois dias consecutivos compartilham 6 dos 7 dias do horizonte: as observações se sobrepõem. O tamanho efetivo da amostra cai de 396 para aproximadamente **57 observações independentes por token**.

A correção de Bonferroni não resolve esse problema, porque opera sobre os mesmos p-valores otimistas, ela corrige a multiplicidade de testes, não a dependência serial.

**Ausência de rejeição não é prova de igualdade.** Com 57 observações efetivas, o teste tem pouco poder para diferenças de IC dessa magnitude. O resultado delimita o alcance desta amostra, não a validade do instrumento.

**SHAP.** O modelo de AAVE aprendeu relação positiva entre volatilidade de 30 dias e retorno futuro — oposta à do modelo global. A análise SHAP audita o que o modelo aprendeu; não valida se aquilo é uma propriedade do mercado. Como nenhum modelo passou no teste de significância, este achado é apresentado como hipótese descritiva.

---

## Aplicando a outro domínio

O método não é específico de criptomoedas. Ele responde a uma pergunta que aparece em qualquer área: **este conjunto de dados ajuda mesmo a tomar uma decisão melhor, ou o que parece ser sinal é sorte?**

Para aplicar em outro domínio, você precisa de quatro coisas:

1. **Dados organizados no tempo.** Uma linha por dia, por mês, por atendimento, desde que exista uma ordem cronológica clara.

2. **Algo concreto a prever, que valha dinheiro.** Aqui foi o retorno futuro de um token. Em energia, seria a demanda da próxima semana. Em crédito, a inadimplência. Em saúde, a reinternação do paciente.

3. **Treinar sempre no passado e testar no futuro** -- nunca o contrário -- descartando os dias da fronteira entre os dois. Se o modelo enxergar qualquer pedaço do futuro, o resultado fica bom por engano.

4. **Saber quanto os seus dados se repetem.** Este é o ponto que a maioria dos trabalhos ignora, e é o que muda o resultado aqui. Se você prevê "os próximos 7 dias" usando dados diários, dois dias seguidos compartilham 6 dos 7 dias. As linhas não são independentes: parecem 400 observações, mas valem como 57. Todo teste estatístico comum supõe independência, então devolve um resultado otimista demais.

Cumpridas as quatro condições, o procedimento é sempre o mesmo: calcular o IC fora da amostra, construir o intervalo de confiança por reamostragem em blocos, testar por permutação se os grupos realmente diferem, e corrigir para o número de testes feitos.

**O que muda de um domínio para outro é só o que você quer prever e o quanto os seus dados se sobrepõem.** No código, isso significa trocar duas coisas: a definição do alvo (`future_return`) e o parâmetro do horizonte (`N_FUTURO`). O resto roda igual.

> A lição transferível: um ranking construído sobre p-valores que supõem independência vai parecer informativo mesmo quando não é. Se o seu método não consegue dizer "aqui não há sinal", ele não está medindo nada, está só confirmando o que você já esperava encontrar.

---

## Como executar

1. Clone o repositório e instale as dependências:

```bash
pip install -r requirements.txt
```

2. Abra o notebook `TCC_InfoFi.ipynb` (Jupyter, VS Code ou Google Colab);

3. Execute as células em ordem (Kernel → Restart & Run All). O download dos dados do Kaggle é automático via `kagglehub` (dataset público, não requer credenciais).

**Nota de reprodutibilidade:** a constante `DATA_HOJE` está fixada em `2026-07-09` para que os resultados coincidam com os reportados na monografia. Para executar com dados atualizados, substitua pela data corrente — os valores podem variar marginalmente.

---

## Acessibilidade das figuras

As figuras usam a paleta Okabe-Ito, segura para os principais tipos de daltonismo, e aplicam **codificação redundante**: nenhuma informação é transmitida apenas por cor. Séries são diferenciadas por cor *e* estilo de linha; barras positivas e negativas por cor *e* hachura; mapas de calor usam `cividis`, monotônico em luminância.

Isso é necessário porque nenhuma paleta categórica de cinco ou mais cores mantém todas as luminâncias separadas — a luminância é unidimensional. Trocar as cores, isoladamente, não resolve o problema de legibilidade em impressão monocromática.

Para verificar, converta qualquer figura para escala de cinzas e pergunte: *é possível dizer qual série é qual sem ver a cor?*

---

## Fontes de dados

- [Crypto Currencies Daily Prices (Kaggle)](https://www.kaggle.com/datasets/svaningelgem/crypto-currencies-daily-prices) — preços OHLC diários (fonte primária)
- Yahoo Finance via [yfinance](https://github.com/ranaroussi/yfinance) — volume e complementação de lacunas

---

## Estrutura

```
├── TCC_InfoFi.ipynb      # notebook principal (pipeline completo)
├── requirements.txt      # dependências
├── figuras/              # figuras geradas (300 dpi) — criada na execução
└── README.md
```

---

## Como citar

```bibtex
@mastersthesis{ledezma2026infofi,
  author  = {Ledezma Molina, Carla Adriana},
  title   = {Precificação de Informação Onchain: um modelo de IA explicável
             para mensurar o valor econômico de datasets em ecossistemas InfoFi},
  school  = {Instituto de Ciências Matemáticas e de Computação,
             Universidade de São Paulo},
  address = {São Carlos},
  year    = {2026},
  type    = {Monografia (Especialização em Inteligência Artificial e Big Data)}
}
```

---

## Licença

Código disponibilizado para fins acadêmicos e de reprodutibilidade.
