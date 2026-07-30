Looking Ahead: Ways to Build on This Project
There are a lot of interesting directions we could take this project! Here are a few thoughts on how to extend it:

Enhancing Return Estimates: Instead of just using historical mean returns (which MVO can be pretty sensitive to), we could explore something like Black-Litterman blending to get more robust expected return inputs.
Backtesting the Strategy: It would be really insightful to run a rolling re-optimization backtest. This would help us see how the optimized portfolios would have performed in real-world scenarios, comparing realized performance against what our in-sample analysis suggested.
Adding Real-World Constraints: For a more practical application, we could incorporate transaction costs or turnover constraints, especially if we're trying to optimize against an existing portfolio of holdings.
Interactive Interface: Imagine wrapping this whole analysis in a Streamlit or Dash application! That would allow for an interactive experience where users could adjust a risk-tolerance slider and see the portfolio changes in real-time.

Efficient Frontier & Mean-Variance Optimizer
Pulls historical prices for a basket of ETFs, estimates expected returns and a shrunk covariance matrix, and solves for the Max Sharpe and Minimum Variance portfolios. Traces the efficient frontier and plots it against a Monte Carlo cloud of random portfolios, with allocation breakdowns and a correlation heatmap.
What it does
	•	Downloads adjusted close prices via yfinance with retry logic for flaky batch pulls
	•	Cleans the price data (drops tickers with too much missing data, forward-fills small gaps)
	•	Computes annualized log returns and a Ledoit-Wolf shrunk covariance matrix
	•	Runs a 15,000-portfolio Monte Carlo simulation for the reference cloud
	•	Solves for Max Sharpe and Global Minimum Variance portfolios with SLSQP (long-only, fully invested)
	•	Traces the analytical efficient frontier by sweeping target returns
	•	Plots the frontier with the Capital Allocation Line, allocation pies, and a correlation heatmap
	•	Exports a performance summary CSV and the cleaned price data
Universe
SPY, QQQ, TLT, GLD, VNQ, EFA, IEF, HYG — 8 years lookback by default. Change TICKERS and LOOKBACK_YEARS in the config cell to use a different set.
Running it
Built for Google Colab, runs top to bottom with no manual setup beyond the two installs the first cell handles (yfinance, fredapi).
Risk-free rate is pulled live from FRED (3-month T-bill, series DTB3). To enable that, add a Colab secret named FRED_API_KEY — get a free key at https://fred.stlouisfed.org/docs/api/api_key.html. If it’s not set, the notebook falls back to a fixed 2%.
Output files
	•	efficient_frontier.png
	•	allocation_and_correlation.png
	•	performance_summary.csv
	•	prices_cache.csv
Notes / caveats
This is standard Markowitz mean-variance optimization — it’s sensitive to the input return estimates, which here are just trailing historical means. That’s a well-known weak point of MVO in general, not something specific to this implementation.
Ideas for later: swap in Black-Litterman for the return estimates, add a rolling backtest to see realized vs. in-sample performance, add turnover/transaction-cost constraints, wrap it in a small Streamlit app.
