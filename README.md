# BESS Co-Optimizer — Nord Pool SE3

Three-market battery energy storage system co-optimizer trained on one year of real Swedish grid data.

**+1,251.7% profit over naive dispatch. All 12 months profitable.**

[![Live Demo](https://img.shields.io/badge/Live%20Demo-Streamlit-red)](https://bess-optimizer.streamlit.app)
[![Python](https://img.shields.io/badge/Python-3.13-blue)](https://python.org)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## Results

| Metric | Value |
|---|---|
| Backtest period | 338 days (June 2025 – May 2026) |
| vs Naive dispatch | **+1,251.7%** |
| vs Day-Ahead only | **+483.4%** |
| FCR-N revenue share | **83.8%** |
| DA forecast MedAPE | 8.93% |
| FCR-D forecast MAPE | 3.49% |
| Revenue capture rate | 64.8% |
| 15-minute resolution uplift | +1.70% |
| Asset range modeled | 0.5 – 10 MW |
| Data source | 100% real (ENTSO-E + SVK Mimer + Nord Pool) |

Naive dispatch loses money in 7 of 12 months. This system is profitable every month.

---

## Live Demo

[**bess-optimizer.streamlit.app**](https://bess-optimizer.streamlit.app)

Adjust battery size (0.5–10 MW), DOD limits, and market selection. See dispatch schedule, revenue breakdown by market, and cycle count in real time.

---

## Architecture

### Data Layer

| Source | Data | Period |
|---|---|---|
| elprisetjustnu.se API | Nord Pool SE3 day-ahead prices | June 2025 – May 2026 |
| ENTSO-E REST API | Wind generation, solar, load, load forecast, generation mix | 8,349 hours |
| SVK Mimer | FCR-D and FCR-N real capacity prices | 8,685 hours |

SVK Mimer FCR price means: FCR-D = 5.86 EUR/MW/h (64.5 SEK/MW/h), FCR-N = 25.28 EUR/MW/h (278 SEK/MW/h).

### Forecasting Layer

Three LightGBM models with adaptive conformal prediction intervals:

| Model | Target | Performance |
|---|---|---|
| DA price forecaster | Hourly SE3 price | MedAPE 8.93% |
| Load forecast error predictor | Load deviation from SVK forecast | MAE 487.8 MW |
| FCR-D price forecaster | Hourly FCR-D capacity price | MAPE 3.49% |

Conformal intervals: ±0.087 SEK/kWh (stable), ±0.153 SEK/kWh (volatile). Coverage 82.2%.

Top features by importance: price lag 1h, **load forecast error** (novel finding), wind variability, hydro dispatch, price lag 24h.

### Optimization Layer

Linear Programming co-optimizer (Pyomo + HiGHS). LP outperforms MILP for Swedish FCR markets — because simultaneous FCR-N + FCR-D commitment is allowed, continuous relaxation better approximates real bidding behavior.

**Decision variables per time step:** charge[t], discharge[t], soc[t], fcrd_mw[t], fcrn_mw[t]

**Key constraints:**
- SOC dynamics, round-trip efficiency 90%
- DOD 10–90% (commercially realistic LiFePO4)
- FCR-D: SOC ≥ 50% of committed MW
- FCR-N: symmetry constraint
- SOC preservation: ≥ 30% at midnight

**Objective:** Maximize DA arbitrage revenue + FCR-D capacity payments + FCR-N capacity payments − degradation cost

---

## Revenue Breakdown (338-day backtest)

| Stream | Total (SEK) | Share |
|---|---|---|
| FCR-N capacity payments | 1,040 SEK | **83.8%** |
| Day-ahead arbitrage | 176 SEK | 14.2% |
| FCR-D capacity payments | 25 SEK | 2.1% |
| **Total** | **1,331 SEK** | 100% |

---

## Nine Hypotheses Tested

| Hypothesis | Result | Why |
|---|---|---|
| FCR-D stacking | **+65.0% ✅** | Real additional revenue stream |
| FCR-N baseline | **+4.4% ✅** | Guaranteed continuous income |
| Degradation cost modeling | **+2.0% ✅** | Removes unprofitable trades, saves 140 cycles/year |
| SOC preservation (30% midnight reserve) | **+0.75% ✅** | Captures next-morning spike value |
| CVaR risk adjustment | **+0.23% ✅** | Small but consistent downside protection |
| Conservative dispatch | -4.96% ❌ | Stable market doesn't reward caution |
| 48h multi-day optimization | -9.17% ❌ | Over-reserves capacity that never materializes |
| Regime-adaptive parameters | -3.45% ❌ | Constraints cost more than they save |
| Price rank forecasting | -1.61% ❌ | Level forecasting captures more value |

**Meta-finding:** In stable Nordic markets, the standard LP co-optimizer is near-optimal. Sophisticated extensions add value only during crisis-period regimes.

---

## Original Research Contributions

1. **MAPE ≠ capture rate** — 85.2% rank accuracy explains 64.8% capture rate. Price ordering matters more than price level accuracy.
2. **Load forecast error = top-6 price feature** — When SVK's forecast is wrong, prices deviate. Novel finding for SE3.
3. **LP outperforms MILP for Swedish FCR markets** — Simultaneous commitment rules make continuous relaxation more accurate than binary dispatch.
4. **FCR-N dominance** — 83.8% of revenue from FCR-N on real 2025-2026 data.
5. **SOC preservation quantified** — Full depletion costs 14.75 SEK in missed morning spike revenue per 68 days.

---

## Independent Validation — Flower Technologies

Cross-referenced with Flower's published market intelligence (February – May 2026):

| Finding | This backtest | Flower validation |
|---|---|---|
| FCR-N dominates revenue | 83.8% share | "FCR revenues now exceed mFRR for first time since March 2025 reform" |
| mFRR saturating | Not modeled (correct call) | "mFRR now at bottom of revenue stack" |
| Multi-market required | DA + FCR-D + FCR-N | "Profitability depends on multi-market optimization" |

Both reflect the same structural market shift — independently, from different data sources.

---

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.13 |
| Forecasting | LightGBM + adaptive conformal prediction |
| Optimization | Pyomo + HiGHS (LP/MILP) |
| Data | ENTSO-E REST API + SVK Mimer CSV + elprisetjustnu.se API |
| Dashboard | Streamlit (deployed on Streamlit Cloud) |

---

## Known Gaps

| Gap | Status |
|---|---|
| Real asset validation | Requires physical battery data — planned as Hitachi Energy thesis |
| aFRR market | Requires ≥1 MW minimum bid; PICASSO rollout 2027+ |
| Intraday re-optimization | Partial (+1.70%) — production server needed |
| Calendar degradation | Not modeled |

---

## Author

**Racem Kamel** — Renewable Energy Engineering, MedTech Tunisia  
Exchange semester: Mälardalen University (MDU), Västerås, Sweden — Autumn 2026  
[GitHub](https://github.com/Racem1000) · [LinkedIn](https://linkedin.com/in/racem-kamel)

---

*Data: ENTSO-E, SVK Mimer, Nord Pool SE3. No simulated or synthetic data.*
