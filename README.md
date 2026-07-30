# Efficient Frontier & Mean-Variance Optimizer

Pulls historical prices for a basket of ETFs, estimates expected returns and a shrunk covariance matrix, and solves for the Max Sharpe and Minimum Variance portfolios. Traces the efficient frontier and plots it against a Monte Carlo cloud of random portfolios, with allocation breakdowns and a correlation heatmap.

## What it does

- Downloads adjusted close prices via `yfinance` with retry logic for flaky batch pulls
- Cleans the price data (drops tickers with too much missing data, forward-fills small gaps)
- Computes annualized log returns and a Ledoit-Wolf shrunk covariance matrix
- Runs a 15,000-portfolio Monte Carlo simulation for the reference cloud
- Solves for Max Sharpe and Global Minimum Variance portfolios with SLSQP (long-only, fully invested)
- Traces the analytical efficient frontier by sweeping target returns
- Plots the frontier with the Capital Allocation Line, allocation pies, and a correlation heatmap
- Exports a performance summary CSV and the cleaned price data

## Universe

SPY, QQQ, TLT, GLD, VNQ, EFA, IEF, HYG — 8 years lookback by default. Change `TICKERS` and `LOOKBACK_YEARS` in the config cell to use a different set.

## Running it

Built for Google Colab, runs top to bottom with no manual setup beyond the two installs the first cell handles (`yfinance`, `fredapi`).

Risk-free rate is pulled live from FRED (3-month T-bill, series `DTB3`). To enable that, add a Colab secret named `FRED_API_KEY` — get a free key at https://fred.stlouisfed.org/docs/api/api_key.html. If it's not set, the notebook falls back to a fixed 2%.

## Output files

- `efficient_frontier.png`
- `allocation_and_correlation.png`
- `performance_summary.csv`
- `prices_cache.csv`

## Notes / caveats

This is standard Markowitz mean-variance optimization — it's sensitive to the input return estimates, which here are just trailing historical means. That's a well-known weak point of MVO in general, not something specific to this implementation.

Ideas for later: swap in Black-Litterman for the return estimates, add a rolling backtest to see realized vs. in-sample performance, add turnover/transaction-cost constraints, wrap it in a small Streamlit app.
