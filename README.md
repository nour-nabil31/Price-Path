# 📈 PricePath

A dynamic Monte Carlo stock price simulator using Geometric Brownian Motion (GBM). Feed it any ticker and it simulates 10,000 possible price paths for the next year, extracts probabilistic statistics, and renders a 3-panel dashboard — all in one function call.

---

## What It Does

Most price forecasting tools give you one number. PricePath gives you a distribution.

It simulates 10,000 possible futures for a stock based on its historical drift and volatility, then answers the questions that actually matter for investment decisions:

| Output | What it answers |
|---|---|
| **Expected & median price** | Where does the stock most likely end up? |
| **5th / 95th percentile** | What's the realistic bear and bull case? |
| **95% Value at Risk** | How much can I lose in a bad year? |
| **P(gain)** | What's the probability the stock ends above today's price? |
| **Annualized Sharpe** | Is the expected return worth the risk? |

---

## Example Output

### Dashboard — 3 panels

**Panel 1 — Simulated price paths with percentile bands**
10,000 paths collapsed into shaded bands. The dark band covers the 25th–75th percentile (likely range). The light band covers the 5th–95th percentile (full realistic range). The median path and current price are overlaid for reference.

**Panel 2 — Final price distribution**
Histogram of all 10,000 simulated year-end prices. Current price marked as a vertical line — everything to the right is a gain, left is a loss. P(gain) annotated directly on the chart.

**Panel 3 — Historical + forecast bridge**
Last 252 trading days of actual price history seamlessly connected to the simulated forecast median. Shows the transition from known history into probabilistic future.

### Summary table

| Metric | Value | Context |
|---|---|---|
| Current Price (S₀) | 81.48 | |
| Expected Price | 89.21 | +9.5% vs today |
| Median Price | 87.63 | +7.5% vs today |
| Bull Case — 95th Percentile | 134.72 | +65.4% vs today |
| Bear Case — 5th Percentile | 54.19 | −33.5% vs today |
| 95% Value at Risk | 27.29 (33.5%) | max expected loss (95%) |
| Probability of Gain | 63.4% | P(end price > today) |
| Annualized Sharpe | 0.312 | risk-adjusted return |

---

## Installation

```bash
pip install numpy pandas yfinance matplotlib scipy
```

---

## Usage

```python
from PricePath import simulate_price

# Minimal call — all defaults
simulate_price("AAPL")

# Egyptian stock with correct risk-free rate
simulate_price("ABUK.CA", risk_free_rate=0.27)

# 6-month forecast
simulate_price("KO", t_intervals=126)

# Custom time period + more trials
simulate_price("MSFT", start="2018-01-01", iterations=20000)

# Reproducible run (same paths every time)
simulate_price("TSLA", seed=42)

# Table only — no plot
simulate_price("GOOGL", plot=False)
```

### Parameters

| Parameter | Default | Description |
|---|---|---|
| `ticker` | **required** | Any yfinance ticker — e.g. `"AAPL"`, `"ABUK.CA"`, `"KO"` |
| `start` | `"2016-01-01"` | Historical data start date for parameter estimation |
| `t_intervals` | `252` | Trading days to forecast — 252 = 1 year, 126 = 6 months |
| `iterations` | `10000` | Number of Monte Carlo simulations |
| `risk_free_rate` | `0.27` | Annualized risk-free rate for Sharpe calculation (default = Egyptian T-bill) |
| `seed` | `None` | Random seed for reproducibility — set any integer for consistent output |
| `plot` | `True` | Show the 3-panel dashboard |

> **Note on risk-free rate:** Default is 0.27 (Egyptian T-bill rate). For US stocks use `risk_free_rate=0.045`. For other markets, use the local government bond yield.

---

## The Math — Geometric Brownian Motion

PricePath uses GBM, the same model underlying Black-Scholes options pricing.

Each simulated day's price is:

```
S(t) = S(t-1) × exp(drift + σ × Z)
```

Where:
- `drift = μ − 0.5σ²` — the Itô-corrected drift term. The `0.5σ²` subtraction is the Itô correction, which ensures the simulated paths grow at the correct geometric (compounding) rate rather than an inflated arithmetic rate
- `σ` — daily standard deviation of log returns, estimated from historical data
- `Z ~ N(0,1)` — a random standard normal draw, different for every day and every path

Running this 10,000 times gives 10,000 legitimate price scenarios, each consistent with the stock's historical behavior.

---

## Honest Limitations

GBM assumes:
- Returns are independently and identically distributed (no momentum, no mean-reversion)
- Volatility is constant over the forecast horizon
- The historical drift and volatility are good estimates of future behavior

In practice, markets have fat tails (crashes happen more often than GBM predicts), volatility clusters, and regime changes. **PricePath is a probabilistic scenario engine, not a prediction.** Use the output as a structured framework for thinking about risk and return — not as a guarantee.

For short horizons (< 30 days), the random noise component dominates the drift signal. The model is most reliable at 3–12 month horizons.

---

## Part of a Larger Toolkit

PricePath pairs with two companion projects:

| Project | Purpose | How they connect |
|---|---|---|
| **Portfolio Optimizer** | Monte Carlo efficient frontier across multiple stocks | Use P(gain) from PricePath to screen stocks before optimizing weights |
| **VolumeSignal** | Volume-based conviction analysis (OBV, CMF, VWAP) | Use as a confirmation layer — high P(gain) + high conviction score = high-confidence signal |

---

## Tech Stack

`Python` · `pandas` · `NumPy` · `yfinance` · `matplotlib` · `scipy`

---

## About

**Nour Nabil** — Finance undergraduate (GPA 3.63) with experience in equity research and FP&A modeling. Building Python tools to bring quantitative rigor to financial analysis and go beyond what Excel alone can do.

[LinkedIn](https://linkedin.com/in/your-profile) · [Email](mailto:your-email)
