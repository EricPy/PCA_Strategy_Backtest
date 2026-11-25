# PCA Portfolios and Statistical Arbitrage Under a Trading-Desk Mandate

Can Principal Component Analysis help build better long-only stock portfolios than simply holding the stocks equally? Two PCA-based portfolios are built, backtested day by day from 2017 to 2026, and then tested against luck, trading costs and different market conditions.

Group take-home assessment for QF627 Programming and Computational Finance, Master of Science in Quantitative Finance, Singapore Management University (SMU). Each member builds and submits their own notebook and strategy functions.

## Key results

Both books and two benchmarks, with a 10 basis point (0.10%) cost on every trade. The research window is 2017-01-04 to 2024-12-31 (2,011 trading days). The holdout is 2025-01-02 to 2026-11-06 (464 days), data the design never used.

|                           | Research Sharpe | Research return a year | Research worst drop | Holdout Sharpe | Holdout return a year | Holdout worst drop |
| ------------------------- | --------------- | ---------------------- | ------------------- | -------------- | --------------------- | ------------------ |
| Max-variance, covariance  | 1.36            | 27.7%                  | -28.4%              | 0.67           | 11.9%                 | -19.8%             |
| Max-variance, correlation | 1.18            | 20.3%                  | -23.1%              | 0.77           | 13.4%                 | -23.0%             |
| Stat-arb, variance rule   | 1.10            | 18.5%                  | -25.0%              | 1.61           | 26.4%                 | -16.6%             |
| Stat-arb, noise rule      | 1.16            | 19.7%                  | -24.7%              | 1.51           | 24.5%                 | -17.2%             |
| Equal weight (benchmark)  | 1.30            | 22.3%                  | -23.2%              | 1.41           | 22.1%                 | -19.1%             |
| SPY (benchmark)           | 0.85            | 12.7%                  | -25.7%              | 0.71           | 10.7%                 | -18.6%             |

- **Nothing clearly beats holding the 12 stocks equally.** In the research window, only the covariance max-variance book edges ahead (1.36 vs 1.30), and the gap is small.
- **The stat-arb signal adds almost nothing before costs.** Sharpe ratios with no costs are 1.26 and 1.32, against 1.30 for equal weight. Shifting the signal in time at random (a placebo test) gives similar results (p-values 0.57 and 0.29), so the timing may not matter.
- **Stat-arb is costly to run.** It trades about ten times as much as the max-variance book. Its returns reach zero at a cost of about 80 bp per trade, against 770 to 1,020 bp for the max-variance books.
- **The holdout favours stat-arb, but it is short.** The holdout is about 1.8 years, which is too short to separate skill from luck. The order of the versions within each family flipped between the research window and the holdout.
- **All four versions pass a standard luck test** (t-statistic 3.0 to 3.8, deflated Sharpe ratio above 0.99). But equal weight passes too (t = 3.56). This shows the 12 stocks did well, not that PCA added value.

Plain-language terms:

- **PCA (principal component analysis):** a way to split the stocks' daily moves into independent directions, ordered by how much of the movement each explains. The first direction (PC1) is usually "the whole market moving together".
- **Max-variance book:** puts money where PC1 puts it, in the stocks that move the most.
- **Stat-arb (statistical arbitrage):** removes the market-wide directions from each stock's return and bets that the leftover will snap back to normal. The desk allows no short selling, so the book starts from equal weight, doubles the stocks that look oversold and drops the ones that look overbought.
- **Sharpe ratio:** return per unit of risk. Higher is better.
- **Basis point (bp):** 0.01%.
- **Placebo test:** rerunning the strategy with a scrambled signal to see whether the real signal does better than chance.
- **Deflated Sharpe ratio:** a Sharpe ratio adjusted for the number of versions tried, since trying many versions makes a lucky winner more likely.
- **Holdout:** later data kept aside and not used to design the books.

## The trading desk and its rules (R1 to R12)

The assignment pretends to be a **trading desk** at an investment firm. The desk gives each book USD 1,000,000 in cash and strict limits, and a **desk engine** replays history day by day to score it. Throughout the notebook, "R1" to "R12" refer to the numbered rules of the course reference (Problem Set 8), which the take-home applies to later dates. In plain words:

**The mandate (the five limits every book must meet every day, R5)**

1. Only the 13 allowed tickers: 12 US stocks (NVDA, GOOGL, AMZN, AAPL, MSFT, JPM, JNJ, CAT, XOM, WMT, META, KO) and SPY, a fund that tracks the S&P 500.
2. Long only: no short selling.
3. No borrowing: weights add up to at most 100%, and any rest is cash, which earns nothing.
4. At most 25% in any one stock.
5. Risk limit: expected volatility (how much the portfolio typically swings) of at most 20% a year, measured on the last 63 days. If a book is above the limit, all its weights are scaled down and the rest is held in cash. This notebook aims for 19.5% to leave a safety margin.

**The rules, one by one**

| Rule      | In plain words                                               | Used in this notebook?                     |
| --------- | ------------------------------------------------------------ | ------------------------------------------ |
| R1        | The data: a fixed price file with daily closes (adjusted for splits and dividends) for the 13 tickers. Returns are simple daily percentage changes. | Yes                                        |
| R2        | The calendar. A *decision day* is a day when a book sets tomorrow's weights using only prices up to that day's close. It looks back 252 return days. Research window 2017-01-04 to 2024-12-31, holdout from 2025-01-02. | Yes                                        |
| R3        | How PCA is done, so everyone gets the same numbers: use the covariance matrix, sort components from largest to smallest, and flip each component's sign so its elements add up to a positive number. | Yes                                        |
| R4        | The "noise" cut-off: which PCA components carry real information and which are just noise. Used here to choose how many factors the stat-arb book keeps. | Partly (noise rule only)                   |
| R5        | The mandate above.                                           | Yes                                        |
| R6        | The desk engine. Decide at the close, then earn the next day's return minus 10 bp of the value traded. A book that breaks a limit sits in cash for a day. It also defines the scorecard: Sharpe ratio, return per year (CAGR), volatility, worst drop (maximum drawdown), turnover (how much is traded per day), cost drag (yearly cost of trading) and "days scaled" (days the risk limit forced the book into cash). | Yes                                        |
| R7        | The six reference books: equal weight, long-only PC1, two minimum-variance books, risk parity and SPY. | Equal weight, SPY and PC1 only             |
| R8 and R9 | Diagnostics and reference books for the second book, built from another method (trend, momentum, mean reversion, a classifier). | No, this notebook covers the PCA book only |
| R10       | Three checks for luck: a t-statistic, the deflated Sharpe ratio, and the placebo test (500 random shifts, seed 627). | Yes                                        |
| R11       | Rules for the submitted strategy functions: each must be self-contained, read only the prices it is given, name no dates, give the same answer every time, and return in under 30 seconds. | Yes                                        |
| R12       | Extending the price file with fresh data: chain new daily returns onto the last frozen price, after checking they match the old file to within 0.000001. | Yes                                        |

## What the notebook does

1. **Loads and checks the data** (a price file frozen by the course, plus a file of later prices).
2. **Builds four candidate books** from two families, each with two versions: max-variance (covariance or correlation matrix) and stat-arb (variance or noise rule for the number of factors). Equal weight and SPY are the benchmarks.
3. **Backtests day by day** from 2017 to 2026 through the desk engine, with zero limit breaches.
4. **Compares** the research window and the holdout.
5. **Tests for luck**: t-statistic, deflated Sharpe ratio and the placebo test.
6. **Stress tests**: results in five market regimes (including the 2020 crash and the 2022 selloff), cost sensitivity and break-even cost, checks for look-ahead, and timing and limit checks.
7. **Critiques its own books** in writing, then puts forward one book per family.
8. **Provides self-contained strategy functions** for each version, checked to match the notebook's weights.

## Repository contents

| File                        | Description                                                  |
| --------------------------- | ------------------------------------------------------------ |
| `strategies_backtest.ipynb` | The full analysis and written critique                       |
| `prices.csv`                | The course's frozen price file, 2016-01-04 to 2026-10-29     |
| `extended_prices.csv`       | Fresh daily prices from 2026-09-14 to 2026-11-16, used to extend the frozen file |
| `requirements.txt`          | Python packages needed                                       |

## How to run

1. Install Python 3, then run `pip install -r requirements.txt`.
2. Keep the notebook, `prices.csv` and `extended_prices.csv` in the same folder.
3. Open the notebook with `jupyter notebook` and choose **Kernel > Restart & Run All**.

No internet is needed. The frozen price file ends on 2026-10-29, so the notebook adds the returns from `extended_prices.csv` (30 October to 16 November) on top of it. The two files overlap for 33 days, and the notebook prints how far their returns differ on those days (at most 0.0000001). Everything after that uses the combined file.

## Data

Daily closing prices for 12 US large caps and SPY, adjusted for splits and dividends, from Yahoo Finance. `prices.csv` is the frozen file provided by the course. `extended_prices.csv` was downloaded from Yahoo Finance for the days after it.

## Limitations

- **Survivorship bias:** the 12 stocks were chosen in 2026, so they are names that already did well. This flatters every book.
- **Short holdout:** 464 days is too short to separate skill from luck.
- **Holdout seen before the final choice:** the two books put forward were chosen by research-window results, but the holdout table had already been seen, so it is a second look, not a clean test.
- **Small differences between versions:** the gaps between the four versions are smaller than the measurement error of a Sharpe ratio over eight years (about 0.46).
- **Simplified costs:** a flat 10 bp per trade, no charge for rebalancing drifted weights, and trades at the same close the signal is read from.
- **No interest on cash.**

## Reference

Avellaneda, M. and Lee, J.-H. (2010). Statistical Arbitrage in the U.S. Equities Market. *Quantitative Finance*, 10(7), 761-782.

## Authors

- Eric Jansen
