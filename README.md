# Weekly Handle Forecast — Time Series Modeling in Excel

Three-model forecast of weekly sportsbook betting handle, built entirely with live Excel formulas: **4-week moving average**, **exponential smoothing**, and **Holt's linear trend method**, evaluated on an honest multi-step holdout.

**Tools:** Excel (all modeling) · SQL Server (weekly aggregation from ~89K bets)

**Headline: Holt's trend method cut holdout forecast error by 38% vs. baseline — and the analysis explains exactly why the winning model still fails at longer horizons.**

---

## Results

| Model | Holdout MAPE (4 weeks) |
|---|---|
| **Holt's Linear Trend** | **43.7%** |
| 4-week Moving Average | 70.4% |
| Exponential Smoothing | 91.8% |

- **Why the level-only models failed**: MA and ES have no trend term — they project the recent level flat. The series was declining ~15%/week, so both froze at ~$20–23K while actuals fell to $10K. No parameter tuning fixes a model-class mismatch.
- **Why Holt won — and where it breaks**: Holt's second smoothing equation tracks the slope. One step ahead it forecast **$12,545 vs. actual $12,573 (0.2% miss)**. But four steps out it forecast $1,523 vs. actual $10,365 — because it extrapolates a *straight line*, and churn-driven decline is *exponential decay* (each week removes a fraction of a shrinking base, so the curve flattens). Linear trend on decay = overshoot.
- **Decomposition insight**: handle per active player held at **~$195–207 the entire period**, including through the collapse. The decline is 100% player-count shrinkage — remaining players never changed behavior. This ties directly to the companion [retention analysis](../retention-cohort-analysis): the churn curves measured there *are* the driver of this series.

## The Series

Ramp-up (Jan–May 2025, player base accumulating) → plateau ~$50–60K/week (Jun–Dec 2025) → structural decline (Jan–Mar 2026, after signups stop and the base only churns).

---

## Workbook Structure

```
Weekly_Handle_Forecast.xlsx
├── README     — purpose, usage, design decisions, legend
├── Forecast   — 64 weeks of data · one-step fits (MA/ES/Holt) ·
│                gray holdout rows with frozen multi-step forecasts · APE columns
└── Dashboard  — editable α/β inputs · MAPE comparison · best-model cell ·
                 4-week forward forecast · actual-vs-fits line chart
```

**Interactive**: α (level smoothing) and β (trend smoothing) live in yellow input cells. Change α from 0.1 to 0.8 and every model column, MAPE, and the chart re-fit instantly — the smooth-vs-reactive tradeoff made visible.

## Methodology Notes

**Partial-week exclusion.** The raw extract had 66 weeks, but the first (2024-12-29) and last (2026-03-29) covered only 2–3 days — $1,240 and $1,304 against $10K+ neighbors. Left in, one poisons model initialization and the other reads as a crash right at the forecast origin. Both excluded; 64 complete weeks modeled.

**Honest holdout construction.** One-step in-sample fits flatter every model. The evaluation that matters is multi-step: at the training cutoff (2026-02-22), each model commits to 4 weeks with no updates —

- MA: frozen flat at the trailing 4-week average
- ES: frozen flat at its final smoothed level (computed from training data only)
- Holt: `level + h × trend` from cutoff values

All three use absolute references to the cutoff row — structurally incapable of peeking at holdout actuals.

**Holt's method in two formulas** (each row references the prior row; α, β are input cells):

```
Level:  L(t) = α·y(t) + (1−α)·(L(t−1) + T(t−1))
Trend:  T(t) = β·(L(t) − L(t−1)) + (1−β)·T(t−1)
Forecast h ahead:  F(t+h) = L(t) + h·T(t)
```

## Known Limitations & Next Steps

1. **Linear trend on decay-shaped data** — quantified above. Fixes: damped-trend Holt (φ parameter shrinking trend contribution per step), or the structural model — forecast active players from retention curves × stable per-player handle
2. **No seasonality** — real sportsbook handle follows the sports calendar (NFL season, March Madness); real data would demand Holt-Winters
3. **Point forecasts only** — production version needs prediction intervals
4. **Single fixed holdout** — production evaluation should use rolling-origin cross-validation

---

## Files

- `Weekly_Handle_Forecast.xlsx` — the complete workbook
- `sql/weekly_aggregation.sql` — the week-snapping aggregation query

Companion projects on the same dataset: [Player Retention & Cohort Analysis](../retention-cohort-analysis) · [RFM Segmentation](../rfm-segmentation) · [Activation Funnel](../funnel-analysis)

## About

Part of my data analytics portfolio — full portfolio on [Notion](https://literate-motion-b44.notion.site/Data-Portfolio-1e3fc4aaea2f807eb1e8d078ecba9b33).

**Ganesh Reddy Peesari** · [LinkedIn](https://linkedin.com/in/ganesh-reddy-peesari-27293527b) · peesariganeshreddy@gmail.com
