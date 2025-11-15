````markdown
### SPX Iron Condor – Edge-vs-Baseline Band Model (band ≤ 1%)

**Description**  
This model forecasts whether the next day’s SPX close will stay inside a very tight **±0.6% band** around today’s close, and only trades when conditions look favorable.  
It uses daily ^GSPC and ^VIX data and filters days by:
- Short-term realized volatility (rolling avg |return|)
- VIX ceiling
- Trend filter (close above SMA when `use_trend=True`)

A grid search over these filters and band widths (≤ 1%) **maximizes edge vs baseline**, not just raw hit rate.

In simple terms: it asks  
> “On which days is a very tight 1-day SPX iron condor much safer than usual?”  
and only trades on those days.

**Baseline definition (what we’re beating)**  
- **Band:** ±0.6% around yesterday’s close.  
- **Baseline hit rate:** fraction of *all* days where `|1-day SPX return| ≤ 0.6%` (i.e., if you sold this condor **every single day**).  
- **Model hit rate:** fraction of days the band holds **only on days when the model says ‘trade’**.  
- **Edge vs baseline:**  
  \[
  \text{edge\_vs\_baseline} = \text{model\_hit\_rate} - \text{baseline\_hit\_rate}
  \]  
  This is the **lift in win rate** the model provides versus blindly trading the same band every day.

**Best configuration (band_pct = 0.006 = ±0.6%)**  
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

**Sample performance (2013–2024, ^GSPC, band_pct=0.006)**

* Signals (trades): **513** out of **3,990** days (~12.9% of days)
* **Model hit rate:** **84.41%**
* **Baseline hit rate (same band, trade every day):** **57.27%**
* **Edge vs baseline:** **+27.14 percentage points**

Interpretation:
If you sold this ±0.6% 1-day SPX iron condor **every day**, you’d be inside the band about **57%** of the time.
If you only trade on days flagged by the model, you’re inside about **84%** of the time – a **~27 point improvement in win rate** using the **same payoff structure**.
