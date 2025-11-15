**SPX Iron Condor — Edge vs Baseline Band Model (band ≤ 1.0%)
**
This model is a 1-day SPX iron condor forecaster. For each trading day it decides whether to trade a very tight symmetric band around SPX for the next session, or stay flat. When it trades, it predicts that tomorrow’s close will stay inside:

```
[ base_close × (1 − band_pct), base_close × (1 + band_pct) ]
```

using daily SPX (^GSPC) and VIX (^VIX) data from yfinance.
Filters are based on realized volatility (average absolute daily return over a lookback window), VIX (low implied volatility), and a trend condition (close above SMA when use_trend = true). A grid search over these filters and band widths up to 1% is used to maximize “edge vs baseline”.

Baseline and edge_vs_baseline

Baseline hit rate:
The baseline is the unconditional probability that a 1-day SPX move stays inside the same band if you traded it every single day, with no filters:

```
baseline_hit_rate = P( |1-day SPX return| ≤ band_pct )
```

This is computed over the full sample of daily returns.

Model hit rate:
The model hit rate is the conditional probability that the same band finishes inside, but only on days when the filters fire a signal (signal_today = true). This measures how good the filters are at selecting calmer days for that payoff.

Edge vs baseline:
edge_vs_baseline = hit_rate − baseline_hit_rate

This is the lift in win rate versus the naive “trade every day” strategy for the same band. It is the core measure of edge the model is optimized to maximize.

Key configurations and results (2010–2024, daily SPX)

1. Higher coverage, strong edge (band_pct = 0.006, ±0.6% band)

Best params:

* band_pct = 0.006
* avgabs_window = 3
* avgabs_max = 0.005
* vix_max = 13
* sma_n = 12
* use_trend = true

Results:

* Total days: 3,990
* Signals (trades): 513
* Signal rate: ~12.9% of days
* Model hit rate: 84.41%
* Baseline hit rate: 57.27%
* Edge vs baseline: +27.14 percentage points

Interpretation: if you sold a ±0.6% 1-day iron condor every day, you’d be inside the band about 57% of the time. If you only sell it when this model signals, you’re inside about 84% of the time. The model adds roughly 27 percentage points of hit-rate edge over the unconditional baseline.

2. Lower coverage, maximal edge (band_pct = 0.004, ±0.4% band)

Best params:

* band_pct = 0.004
* avgabs_window = 10
* avgabs_max = 0.004
* vix_max = 12
* sma_n = 12
* use_trend = true

Results:

* Total days: 3,983
* Signals (trades): 229
* Signal rate: ~5.7% of days
* Model hit rate: 76.86%
* Baseline hit rate: 43.64%
* Edge vs baseline: +33.22 percentage points

Interpretation: for a tighter ±0.4% band, trading every day would only finish inside about 44% of the time. Restricting to the model’s signals lifts that to ~77%, adding over 33 percentage points of hit-rate edge, at the cost of trading less frequently.

Overall, the model is designed as a filter: it does not try to predict direction, only whether SPX is likely to remain inside a very tight 1-day range. The main performance metric is edge_vs_baseline, quantifying how much better a conditional, filtered strategy is compared to blindly trading the same 1-day band every day.
