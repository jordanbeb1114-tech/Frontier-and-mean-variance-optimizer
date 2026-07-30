## Design decisions

Notes on the choices behind this, mostly so I don't forget my own reasoning later.

**Log returns instead of simple returns.** Log returns are time-additive — the return over N days is just the sum of daily log returns, which makes annualizing (`mean * 252`) mathematically clean. Simple returns don't compound that way, so you'd have to geometrically link them instead.

**Ledoit-Wolf shrinkage instead of the raw sample covariance matrix.** With 8 assets and ~2000 daily observations this isn't a severe case, but sample covariance still overfits — it treats noise in the historical correlations as if it were signal, and the optimizer will happily lever into whatever spurious diversification benefit it finds. Ledoit-Wolf shrinks the sample covariance toward a structured target (scaled identity), which trades a small amount of bias for a meaningful reduction in estimation error. It's a standard fix in this literature, not something I invented.

**SLSQP for the optimization.** The problem is a quadratic objective (variance) or a nonlinear one (negative Sharpe) with linear equality/inequality constraints — sum of weights equals 1, weights between 0 and 1. SLSQP handles constrained nonlinear problems like this well and it's what `scipy.optimize.minimize` supports out of the box for this constraint type.

**Long-only, fully invested.** No short selling, no leverage. This is the simplest, most realistic constraint set for a retail-style allocation — real robo-advisors mostly operate under the same restriction. Relaxing it (allowing shorts) would let the optimizer chase Sharpe ratio much more aggressively, which is a different, riskier problem.

**Why the Min Variance portfolio had a negative Sharpe ratio.** This tripped me up at first — it looks like a bug but isn't. The Min Variance portfolio is optimized purely to minimize volatility, with no regard for return. If its resulting expected return happens to fall below the risk-free rate, the Sharpe ratio goes negative. That's a real, expected outcome when your minimum-risk assets (bonds, in this universe) have lower expected returns than the prevailing T-bill rate — it says "you're better off in cash than in this specific low-risk portfolio," which is a legitimate conclusion, not an error.

**Dirichlet distribution for the Monte Carlo weights.** `np.random.dirichlet(np.ones(n))` generates random weight vectors that are automatically non-negative and sum to 1 — exactly the constraint set the optimizer uses. Sampling uniformly and normalizing afterward would bias the distribution toward more even/less concentrated portfolios, so Dirichlet is the correct way to get an unbiased sample of the feasible region.

**Biggest limitation: expected returns are just trailing historical means.** This is the standard critique of mean-variance optimization (Michael Jorion's "estimation error" work, and the whole reason Black-Litterman exists) — historical average returns are a noisy, backward-looking estimate, and MVO is disproportionately sensitive to errors in that specific input compared to errors in the covariance matrix. The frontier and the optimal weights would shift meaningfully with a different return model. This implementation is intentionally the textbook baseline, not a claim that historical means are the best forecast.
