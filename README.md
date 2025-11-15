# SPX-Iron-Condor-Edge-vs-Baseline-Model
SPX Iron Condor (Max Edge)

````markdown
## SPX Iron Condor – Edge-vs-Baseline Model (band ≤ 1%)

### What this model does
This script builds a **1-day SPX iron condor forecaster**:  
for each trading day it decides whether to trade a **very tight band** around SPX (±0.6%) for the next session, or stay flat.

When it trades, it predicts that **tomorrow’s close will stay inside**  
\[ base_close × (1 − 0.6%), base_close × (1 + 0.6%) \].

The model:
- Uses daily SPX (^GSPC) and VIX (^VIX) from yfinance.
- Filters days by:
  - **Realized 1-day volatility** (avg |return| over a short window)
  - **VIX level** (low implied vol)
  - **Trend filter** (close above SMA with `use_trend=True`)
- **Grid-searches these filters** (including band width ≤ 1%) to **maximize edge vs baseline**, not just raw hit rate.
- Outputs full diagnostics: by year, month, weekday, VIX bucket, realized vol bucket, ablations of each filter, local sensitivity, rolling 100-trade hit rates, and streaks.

Best params (for band ≤ 1%):
```json
{
  "band_pct": 0.006,
  "avgabs_window": 3,
  "avgabs_max": 0.005,
  "vix_max": 13,
  "sma_n": 12,
  "use_trend": true
}
````

### Baseline vs Edge (core idea)

**Band:** ±0.6% around yesterday’s close.

* **Baseline hit rate**
  This is the unconditional probability that SPX’s **1-day absolute return is ≤ 0.6%** over the full sample, with **no filters**.
  Interpreted trading-wise:

  > *“What fraction of days would a ±0.6% 1-day iron condor finish inside if I traded it every single day?”*

* **Model hit rate**
  This is the conditional probability that the same ±0.6% band finishes inside, **only on days when the model signals “safe to trade”** (filters pass).

* **Edge vs baseline**
  [
  \text{edge_vs_baseline} = \text{hit_rate} - \text{baseline_hit_rate}
  ]
  This is the **lift in win-rate** the model achieves over the naive “trade every day” baseline using the **same band width**.

### Results (2010–2024, daily)

**Optimized config (band_pct = 0.006 = 0.6%)**

* **Signals:** 513 trades over 3,990 trading days
* **Signal rate:** ~12.9% of days
* **Model hit rate:** **84.41%**
* **Baseline hit rate:** **57.27%**
* **Edge vs baseline:** **+27.14 percentage points**

Interpretation:

* If you sold this **±0.6% 1-day condor every day**, you’d be inside the band about **57%** of the time.
* If you only sell it **when this model signals**, you’re inside about **84%** of the time.
* That **~27 percentage point lift in hit-rate** is the model’s **edge**:
  it concentrates trades into environments where the same tight band has a **much higher probability of expiring inside**.

### Regime robustness (high level)

* **By year:** hit rate mostly in the **80–90%** range (e.g. 2017: 91.1%, 2019: 87.5%), with weaker performance in stress regimes (e.g. 2020).
* **By VIX bucket:** strongest edge in **low-VIX regimes** (VIX ≤ 12: 90.9% hit rate).
* **By realized vol bucket:**

  * Q1 (lowest realized vol): **100%** hit rate
  * Q2 (next lowest): **99.2%** hit rate
  * Edge decays as realized vol rises.

Ablation tests confirm that **all three filters (realized vol, VIX, trend)** together give the best trade-off between **hit rate and number of signals**.

> **Note:** This is a research / backtest tool, not trading advice. It optimizes **statistical edge vs a well-defined baseline** for a tight 1-day SPX band; risk, slippage, and execution are not modeled.

```
```

