# Time-Series Momentum with Trend Filter

A research notebook that backtests a diversified, trend-following ETF strategy against SPY buy-and-hold over 20 years of daily data.

## Contents

- Strategy logic and rationale
- Data and assumptions
- How to run the notebook
- Metrics and how to read them
- Known limitations
- Disclaimer

---

## 1. Strategy Overview

The strategy holds a diversified set of ETFs only when each one shows a positive medium-term trend. An ETF is held when both of these are true at month-end:

1. **12-month return is positive** (`lookback = 252` trading days).
2. **Price is above its 200-day simple moving average** (`ma_len = 200`).

Each active ETF receives an equal share of capital (1/N, where N is the number of ETFs in the universe). Any slice whose filters are off sits in cash. Positions are set at the month-end signal and applied starting the next trading day, so the backtest does not use information it could not have had at the time.

### Default universe

| Ticker | Exposure |
|--------|----------|
| SPY | US large-cap equities |
| QQQ | US large-cap technology |
| IWM | US small-cap equities |
| EFA | Developed international equities ex-US |
| TLT | Long-term US Treasury bonds |
| GLD | Gold |

### Rationale

The approach builds on time-series momentum, the observation that assets with positive recent returns tend to continue rising for a period. Proposed explanations include slow information diffusion and forced trading by some investors. The trend filter adds a second confirmation so the strategy avoids assets in downtrends. The effect is documented across many markets, but it is a modest premium with long periods of underperformance, not a guaranteed edge.

---

## 2. Requirements

- Python 3.9+
- Jupyter Notebook or JupyterLab (or Google Colab)
- Internet access, since data is downloaded from Yahoo Finance via `yfinance`
- Packages (installed by Cell 1): `yfinance`, `numpy`, `pandas`, `matplotlib`

---

## 3. Notebook Structure

| Cell | Purpose |
|------|---------|
| 1 | Install and import libraries |
| 2 | Download 20 years of adjusted daily closes |
| 3 | Build month-end signals, daily weights, and net returns |
| 4 | Performance metrics vs SPY |
| 5 | In-sample vs out-of-sample comparison |
| 6 | Parameter robustness grid and cost sensitivity |
| 7 | Equity curve and drawdown charts |
| 8 | Calendar-year returns vs SPY |
| 9 | Current allocation |

### How to run

1. Run cells in order from top to bottom. Cell 3 depends on Cell 2, and later cells depend on earlier outputs.
2. To change the universe, edit the `tickers` list in Cell 2. Keep SPY in the list, since it is the benchmark used in Cells 3 to 8.
3. To change the assumptions, adjust the function arguments in `build_weights` (`lookback`, `ma_len`, `cost_bps`).
4. Re-run the notebook whenever you want current data. The 20-year window is measured back from today's date, so results change over time.

---

## 4. Methodology Details

- **Data:** Adjusted closing prices (`auto_adjust=True`), which include dividends and splits.
- **Signal timing:** Signals are evaluated on the last trading day of each month and shifted forward one day before being applied, so each day's return uses only information available the previous day.
- **Returns:** Gross daily return is the sum of each asset's return times its weight. Net return subtracts turnover (the sum of absolute weight changes) times the cost in basis points.
- **Costs:** A flat 5 bps per unit of turnover by default. This is an approximation and excludes taxes, bid-ask spreads beyond the flat assumption, and financing.
- **Benchmark:** SPY buy-and-hold with no costs, using the same date range.
- **Missing data:** Rows with any missing ETF price are dropped, so the common history starts when all six ETFs have data. The 200-day and 252-day windows also need warm-up time, so early signals are unavailable.

---

## 5. Metrics

| Metric | Meaning |
|--------|---------|
| CAGR | Compound annual growth rate |
| Volatility | Annualized standard deviation of daily returns |
| Sharpe (rf=0) | Mean return divided by volatility, annualized, assuming a zero risk-free rate |
| Max drawdown | Largest peak-to-trough decline |
| Calmar | CAGR divided by absolute max drawdown |

Sharpe here ignores the risk-free rate. With meaningful T-bill yields, the true excess-return Sharpe would be lower for both the strategy and SPY, and the difference could shift.

---

## 6. Interpreting Results

- **Judge drawdowns as well as returns.** A higher CAGR with a much deeper drawdown may not suit your risk tolerance.
- **Check the parameter grid (Cell 6).** If results collapse when the lookback or MA length changes, the edge may be fragile.
- **Compare in-sample and out-of-sample (Cell 5).** Large differences suggest overfitting. Remember the split is at the midpoint of the sample, not a true forward test.
- **Expect lag in bull markets.** The strategy will likely trail SPY in strong uptrends because of cash slices and costs.
- **Watch the current allocation (Cell 9).** It shows what the rules would hold today, but it is a snapshot from the most recent month-end.

---

## 7. Limitations

- **Survivorship and selection bias:** The universe was chosen by hand, using ETFs that exist today. Results may look better than a real-time selection would have.
- **Limited sample:** Twenty years includes only a few major bear markets and one long bull run.
- **Simplified costs:** Taxes, spreads during volatile periods, and ETF expense ratios are not modeled.
- **No risk-free rate or leverage:** Cash earns nothing in the model.
- **Rebalancing lag:** Monthly signals respond slowly to fast crashes.
- **Data quality:** yfinance is a free source and can contain gaps or adjustment errors. Verify important results against another data source.

---

## 8. Disclaimer

This notebook is for educational and research purposes only. It is not investment advice, and past backtested performance does not predict future results. Before trading with real money, validate the strategy with paper trading and make sure the drawdowns match what you can tolerate. Consider speaking with a licensed financial advisor, especially for tax-sensitive decisions.
