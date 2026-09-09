# Pairs Trading entre Ações e Commodities

[Português](README.md) | [English](README.en.md)

Análise quantitativa de relações entre ações de setores ligados a matérias-primas
e futuros de commodities. O notebook recolhe preços com `yfinance`, seleciona
pares por correlação, testa a estacionariedade do spread e compara quatro regras
de backtesting baseadas em reversão à média.

> Projeto académico e experimental. Os resultados não constituem aconselhamento
> financeiro nem uma estratégia pronta para negociação com capital real.

## Análise do notebook

### 1. Recolha e preparação dos dados

O universo inclui empresas de energia, mineração, agronegócio, indústria,
tecnologia, consumo, saúde, finanças e utilities, juntamente com futuros de
energia, metais, cereais, soft commodities e pecuária.

- Fonte: Yahoo Finance através de `yfinance`.
- Período de recolha: desde `2020-01-01`; os testes e backtests guardados terminam
  em `2025-01-01`.
- O output bem-sucedido do notebook contém 1 351 sessões e 313 séries de fecho,
  depois de solicitar 320 símbolos.
- O pré-processamento elimina colunas vazias, preenche valores em falta e calcula
  retornos logarítmicos.
- É também criada uma versão padronizada dos retornos com z-score.

Uma execução anterior guardada no mesmo notebook mostra os 202 downloads a
falharem por rate limiting. Isto evidencia que os resultados dependem da
disponibilidade do Yahoo Finance e que a recolha deve ser feita em lotes ou com
cache para maior robustez.

### 2. Seleção dos pares

A seleção cruza ações com commodities através da média da correlação de Pearson
calculada numa janela móvel de 63 sessões, aproximadamente três meses. São
guardadas matrizes para dados tratados e normalizados e selecionados os 30 pares
com maior correlação absoluta.

O par mais correlacionado foi `GOLD / GC=F`, com correlação média de `0,678`,
seguido por `PAAS / SI=F` (`0,671`) e `AEM / GC=F` (`0,668`). Os resultados
mostram sobretudo relações economicamente plausíveis entre empresas mineiras e
metais, e entre produtoras de energia e petróleo Brent/WTI.

O texto do notebook propõe DBSCAN como segunda abordagem de seleção, mas a célula
correspondente contém apenas um placeholder. Assim, os resultados executados e
documentados são os da matriz de correlação; o clustering ainda não está
implementado neste ficheiro.

### 3. Estacionariedade do spread

Para cada par, o segundo ativo é reescalado pela razão entre os preços iniciais
e o spread percentual é calculado por:

```text
spread = (stock - commodity_scaled) / (stock + commodity_scaled) × 100
```

O Augmented Dickey–Fuller usa `p < 0,05` como critério. Entre 30 pares válidos,
foram encontrados três spreads estacionários:

| Par | p-value ADF |
| --- | ---: |
| `WPM / GC=F` | 0,0134 |
| `COP / CL=F` | 0,0213 |
| `WPM / SI=F` | 0,0410 |

Para estes pares, o backtest utiliza média e desvio padrão fixos. Para os
restantes, utiliza estatísticas móveis de 30 ou 60 sessões.

### 4. Estratégias de backtesting

Cada par começa com capital simulado de 100 000 dólares. O notebook compara:

| Estratégia | Regra principal | Alocação | Retorno médio dos 30 pares* | Melhor retorno guardado |
| --- | --- | ---: | ---: | --- |
| 1 | Entrada a ±1σ, saída na média | 40% | 9,32% | `FANG / CL=F`: 49,66% |
| 2 | Entrada a ±1σ, saída no limite oposto | 40% | 3,44% | `FANG / CL=F`: 43,04% |
| 3 | Entrada a ±2σ, saída na média | 50% | 8,77% | `CVE / CL=F`: 51,81% |
| 4 | Alocação progressiva com filtro de tendência | 0–90% | 13,12% | `COP / CL=F`: 55,17% |

\* Média calculada a partir dos outputs armazenados no notebook, sem ponderação
pelo número de operações.

O maior valor da quarta estratégia em `COP / CL=F` resulta de apenas uma
operação e não é evidência de robustez. Um exemplo mais representativo nessa
configuração é `CVE / CL=F`, com 46,57% em 147 operações. O notebook calcula
retorno, número de operações, taxa de acerto, lucro/perda médios, profit factor e
drawdown máximo.

## Leitura crítica dos resultados

O notebook demonstra bem o ciclo completo de uma hipótese quantitativa, mas os
resultados são in-sample: o mesmo período influencia a seleção dos pares, o
teste ADF e o backtest. Antes de interpretar a estratégia como negociável seria
necessário:

- separar períodos de formação, validação e teste walk-forward;
- estimar hedge ratio por regressão e testar cointegração, em vez de depender
  apenas de correlação e escala inicial;
- incluir custos, slippage, bid-ask spread, financiamento, short borrow e
  multiplicadores/rollover de futuros;
- controlar survivorship bias, múltiplos testes e estabilidade dos parâmetros;
- comparar retorno com volatilidade, Sharpe/Sortino e drawdown;
- corrigir o placeholder de DBSCAN e automatizar a persistência dos artefactos.

Uma taxa de acerto elevada pode coexistir com retorno negativo quando as perdas
médias são muito superiores aos ganhos, como acontece em alguns pares do
notebook. Esta é uma das conclusões mais relevantes da experiência.

## Executar

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab projeto.ipynb
```

Execute as células pela ordem apresentada. A recolha requer acesso à Internet e
pode ser limitada temporariamente pelo Yahoo Finance. Os CSV e gráficos
intermédios são gerados no diretório atual e não estão incluídos nesta versão.

## Estrutura

```text
stock-commodity-pairs-trading/
├── projeto.ipynb
├── Pair_Trading_Stocks_and_Commodaties.pdf
├── requirements.txt
├── README.md
└── README.en.md
```

## Competências demonstradas

Python, séries temporais financeiras, recolha e limpeza de dados, correlação
móvel, testes ADF, mean reversion, gestão dinâmica de capital, backtesting e
análise crítica de risco.

## Contexto académico

Trabalho desenvolvido no âmbito da unidade curricular de Análise de Dados para
Mercados Financeiros.
