# Stock–Commodity Pairs Trading

[Português](README.md) | [English](README.en.md)

A quantitative analysis of relationships between equities exposed to raw
materials and commodity futures. The notebook downloads prices with `yfinance`,
selects pairs by correlation, tests spread stationarity and compares four
mean-reversion backtesting rules.

> This is an academic experiment. The results are not financial advice or a
> strategy ready for live trading.

## Notebook analysis

### 1. Data collection and preparation

The universe includes companies from energy, mining, agriculture, industrials,
technology, consumer goods, healthcare, finance and utilities, together with
energy, metal, grain, soft-commodity and livestock futures.

- Source: Yahoo Finance through `yfinance`.
- Download period: from `2020-01-01`; the stored tests and backtests end on
  `2025-01-01`.
- The successful notebook output contains 1,351 sessions and 313 closing-price
  series after requesting 320 symbols.
- Preprocessing removes empty columns, fills missing observations and computes
  log returns.
- A z-score-standardized return dataset is also created.

An earlier stored execution shows all 202 downloads failing because of rate
limiting. The results therefore depend on Yahoo Finance availability; a more
robust pipeline should download in batches and cache responses.

### 2. Pair selection

Stocks and commodities are matched through mean Pearson correlation over a
63-session rolling window, approximately three months. The notebook stores raw
and standardized correlation matrices and selects the 30 pairs with the
largest absolute correlation.

The strongest pair was `GOLD / GC=F`, with mean correlation `0.678`, followed
by `PAAS / SI=F` (`0.671`) and `AEM / GC=F` (`0.668`). Most leading results are
economically plausible links between mining firms and metals or between energy
producers and Brent/WTI crude.

The notebook text proposes DBSCAN as a second selection method, but the related
cell contains only a placeholder. Consequently, the executed results come from
the correlation matrix; clustering is not implemented in this file.

### 3. Spread stationarity

For each pair, the second asset is scaled by the initial-price ratio and the
percentage spread is calculated as:

```text
spread = (stock - commodity_scaled) / (stock + commodity_scaled) × 100
```

The Augmented Dickey–Fuller test uses `p < 0.05`. Three stationary spreads were
identified among 30 valid pairs:

| Pair | ADF p-value |
| --- | ---: |
| `WPM / GC=F` | 0.0134 |
| `COP / CL=F` | 0.0213 |
| `WPM / SI=F` | 0.0410 |

Backtests use a fixed mean and standard deviation for these pairs and rolling
30- or 60-session statistics for the remaining pairs.

### 4. Backtesting strategies

Each pair starts with USD 100,000 of simulated capital. Four rules are
compared:

| Strategy | Main rule | Allocation | Mean return across 30 pairs* | Best stored return |
| --- | --- | ---: | ---: | --- |
| 1 | Enter at ±1σ, exit at the mean | 40% | 9.32% | `FANG / CL=F`: 49.66% |
| 2 | Enter at ±1σ, exit at the opposite threshold | 40% | 3.44% | `FANG / CL=F`: 43.04% |
| 3 | Enter at ±2σ, exit at the mean | 50% | 8.77% | `CVE / CL=F`: 51.81% |
| 4 | Progressive allocation with a trend filter | 0–90% | 13.12% | `COP / CL=F`: 55.17% |

\* Calculated from the outputs stored in the notebook, without weighting by
trade count.

The highest fourth-strategy result for `COP / CL=F` comes from a single trade
and is not evidence of robustness. A more representative example is
`CVE / CL=F`, which returned 46.57% over 147 trades. The notebook calculates
return, trade count, win rate, average profit/loss, profit factor and maximum
drawdown.

## Critical interpretation

The notebook demonstrates the full lifecycle of a quantitative hypothesis, but
the results are in-sample: the same period affects pair selection, ADF testing
and backtesting. Before treating the approach as tradable, the following would
be required:

- separate formation, validation and walk-forward test periods;
- estimate hedge ratios by regression and test cointegration rather than rely
  only on correlation and initial-price scaling;
- include fees, slippage, bid-ask spreads, financing, short-borrow costs and
  futures multipliers/rollover;
- control survivorship bias, multiple testing and parameter stability;
- compare return against volatility, Sharpe/Sortino and drawdown;
- complete the DBSCAN stage and automate artifact persistence.

A high win rate can coexist with a negative return when average losses are much
larger than average gains, as several notebook pairs demonstrate. This is one
of the experiment's most relevant findings.

## Run locally

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab projeto.ipynb
```

Run cells in order. Data collection requires Internet access and may be
temporarily rate-limited by Yahoo Finance. Intermediate CSV files and plots are
generated in the working directory and are not included in this version.

## Repository structure

```text
stock-commodity-pairs-trading/
├── projeto.ipynb
├── Pair_Trading_Stocks_and_Commodaties.pdf
├── requirements.txt
├── README.md
└── README.en.md
```

## Skills demonstrated

Python, financial time series, data acquisition and cleaning, rolling
correlation, ADF testing, mean reversion, dynamic capital allocation,
backtesting and critical risk analysis.

## Academic context

Developed by José Cunha, João Fonseca and Diogo Almeida for Data Analysis in
Financial Markets. The complete presentation is available in
[Pair_Trading_Stocks_and_Commodaties.pdf](Pair_Trading_Stocks_and_Commodaties.pdf).
