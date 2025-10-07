# Capstone Project — Initial Report & Exploratory Data Analysis (EDA)

## Executive Summary

This project explores whether we can **predict the next-day direction of a single stock in an idiosyncratic (hedged) way** and translate small predictive edges into **economically meaningful PnL** via a market/sector hedge and thresholded trading. I conduct a thorough **EDA**, engineer features across **stock, market, sector, volatility, and macro proxies**, build a **baseline model**, and include a **results section** that will be extended in the final capstone.


## Research Question

**Note:** The wording of the problem statement has been modified slighly from previously submitted version.

Can we build an interpretable machine-learning model to predict next-day stock behavior - focusing on idiosyncratic (excess) return - using historical market, technical, sector, and macro data to support risk-aware trading?


## Data & Sources

* **Single security**: target stock (default: `AAPL`) — daily close/returns.
* **Market**: S\&P 500 (`^GSPC`) for broad beta.
* **Sector ETFs**: SPDRs (`XLK`, `XLF`, `XLY`, …). The model automatically selects the **most correlated** sector to hedge.
* **Volatility**: **VIX** (`^VIX`, level and day-over-day change).
* **Macro/rates proxies**: `HYG`, `LQD` (credit spread proxy), `UUP` (USD), `IEF`/`TLT` (UST duration), `^TNX` (10Y yield).
* **Provider**: Yahoo Finance via `yfinance`. (Other macro series may be added later per the problem statement.)&#x20;


## Methods

### Target construction (economic alignment)

Rather than raw direction, I predict **next-day excess return** of the stock after **index + sector hedging**:

$$
\text{Excess}_{t+1} = r^{\text{stock}}_{t+1} - \beta^{(mkt)}_{t}\, r^{\text{mkt}}_{t+1} - \beta^{(sect)}_{t}\, r^{\text{sector}}_{t+1},
$$

with **rolling betas** (60D, expanding fallback). The **classification label** is `1` if $\text{Excess}_{t+1} > 0$, else `0`. This focuses the model on **idiosyncratic movement**, consistent with the trading goal.


### Features (built at $t$ to predict $t+1$)

* **Stock & market**: returns (t, t−1), **HAR-style realized vol** (abs-return means over 5/22D).
* **Technical**: **RSI(14)**, **Bollinger %B(20,2)**, **MA spread (MA10−MA30)**.
* **Sectors**: previous-day sector returns (for all SPDRs available), plus **auto-selected sector** used in the hedge.
* **Vol/Rates/Macro**: VIX (level and Δ), credit spread proxy (HYG−LQD), USD (UUP), UST duration (IEF/TLT), Δ10Y yield (`^TNX`).


### EDA

* **Cleaning**: alignment across tickers, rolling windows, shift-based targets; drop NA post-feature/target build.
* **Descriptives**: summary stats, **histograms** (returns, VIX, excess target), **22-day vol trend**.
* **Correlation heatmap** across features and target.
* **PCA (2D)** on standardized features for structure exploration (variance directions, outliers).
  *(EDA emphasizes cleaning, feature engineering, and visualizations as requested.)*&#x20;


### Baseline model & evaluation

* **Model**: Logistic Regression (`class_weight='balanced'`), standardized features.
* **Split**: time-ordered **train / validation / test**.
* **Threshold tuning**: choose **probability threshold** (and optional **no-trade band**) on the **validation slice** to **maximize Sharpe** of the hedged strategy (economic objective, not accuracy).
* **Backtest**: daily **hedged strategy** on **TEST** using chosen threshold/band; benchmarks = **Buy\&Hold stock** and **Buy\&Hold index**.



## Results

* **EDA** shows sensible distributions; correlations highlight **systematic factors** (market/sector) but also stock-specific variation captured by RSI/MA spread/volatility features.
* **PCA** suggests PC1 is largely a **market/sector** factor; idiosyncratic signals are subtler (as expected).
* **Classification**: modest ranking skill (AUC slightly > 0.5 on excess-direction), which is common for daily horizons; nevertheless, **Sharpe-optimized thresholding** yields **economically meaningful performance** in the hedged backtest on the held-out test span.
* **Hedged strategy vs benchmarks**: on the latest run (see notebook output), the hedged strategy achieved **higher risk-adjusted return (Sharpe) than buy-and-hold** while controlling drawdown, consistent with the goal of monetizing **idiosyncratic** edges through a beta-neutral framework.



## Footnotes

Generative AI was used to help format this README.md file with some of the wording.


*End of README*
