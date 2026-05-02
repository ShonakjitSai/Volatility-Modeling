# Nifty 50 Volatility Modelling — ARCH/GARCH/EGARCH

## Overview
This project models time-varying volatility in the Nifty 50 index using
ARCH/GARCH family models. Rather than assuming constant variance (as ARIMA
does), these models treat volatility itself as a predictable, time-varying
quantity — capturing the well-known phenomenon of volatility clustering
in financial markets.

**Volatility clustering:** calm periods tend to be followed by calm periods,
and turbulent periods by turbulent periods. Standard regression models are
blind to this structure. GARCH was built specifically to capture it.

---

## Data
- **Source:** Yahoo Finance via `yfinance`
- **Ticker:** ^NSEI (Nifty 50 Index)
- **Period:** January 2015 — December 2024
- **Frequency:** Daily
- **Variable:** Log returns (%) = log(P_t / P_(t-1)) × 100

---

## Methodology

### Step 1 — Test for ARCH Effects
Before fitting any model, Engle's ARCH-LM test confirms whether
time-varying variance is present in the data.

- **H0:** Variance is constant (no ARCH effects)
- **H1:** Variance is time-varying (ARCH effects present)
- **Result:** LM statistic = 638.79, p ≈ 0 → GARCH modelling fully justified

### Step 2 — Model Fitting
Three models estimated and compared via AIC (lower = better fit):

| Model | AIC | Notes |
|---|---|---|
| GARCH(1,1) — Normal errors | 6392.78 | Baseline |
| GARCH(1,1) — Student-t errors | 6265.92 | Fat tails confirmed |
| EGARCH(1,1) — Student-t errors | 6198.98 | Leverage effect confirmed |

### Step 3 — Model Selection
EGARCH(1,1) with Student-t errors achieved the lowest AIC (6198.98),
outperforming the baseline by 194 points across two improvements:

- **Normal → Student-t (−127 AIC points):** Financial returns have fat
  tails — extreme daily moves occur far more often than a Normal
  distribution predicts. Student-t captures this via the degrees of
  freedom parameter (ν).

- **GARCH → EGARCH (−67 AIC points):** Negative shocks raise volatility
  more than positive shocks of equal size — the leverage effect. EGARCH
  captures this asymmetry via the gamma parameter.

### Step 4 — Residual Diagnostics
The winning model was validated against three diagnostic checks:

| Check | Result | Verdict |
|---|---|---|
| ACF of squared standardised residuals | All 20 lags within confidence bands | ✅ Pass |
| Residual ARCH-LM test | p = 0.1101 | ✅ Pass |
| Residual kurtosis | 2.314 (theoretical: 2.21) | ✅ Pass |

All checks passed — the model has fully captured the variance structure.

---

## Key Findings

| Finding | Value | Interpretation |
|---|---|---|
| ARCH-LM statistic | 638.79 (p ≈ 0) | Variance is time-varying — GARCH fully justified |
| Persistence (α + β) | 0.969 | Volatility shocks take ~22 trading days to halve |
| Leverage effect (γ) | −0.109 | Negative shocks raise vol ~19× more than positive shocks |
| Fat tails (ν) | 6.71 | Crashes far more probable than Normal distribution predicts |
| Long-run volatility | ~16% annualised | Nifty's calm-period baseline — the level vol reverts to |
| 10-day forecast | 13.1–13.6% annualised | Below long-run avg — currently calm, gradually drifting up |

---

## What is Annualised Volatility?
Volatility measures how much prices move around on a daily basis.
Annualised volatility scales this to a yearly number using:

    Annual vol = Daily vol × √252

where 252 is the number of trading days in a year. This allows
direct comparison across assets, time periods, and models on a
single universal scale. A value of 16% means the index is expected
to move within roughly ±16% of its starting level over a typical year.

---

## Notable Events Captured

| Period | Estimated Vol | Event |
|---|---|---|
| Late 2015 | ~35% | China currency devaluation, commodity crash |
| March 2020 | ~90% | COVID-19 crash — largest spike in sample |
| 2022 | ~30% | Russia-Ukraine war, aggressive Fed rate hikes |
| Early 2025 | ~38% | Visible on right edge of conditional vol chart |

---

## Charts
1. **Squared returns** — visual confirmation of volatility clustering
2. **GARCH(1,1) conditional volatility** — annualised vol over 2015–2025
3. **ACF of squared standardised residuals** — diagnostic check
4. **10-day ahead volatility forecast** — model output

---

## How to Run

### Install dependencies
```bash
pip install arch yfinance pandas numpy matplotlib scipy statsmodels
```

### Run the notebook
```bash
jupyter notebook notebooks/garch_analysis.ipynb
```

---

## Repository Structure
nifty-volatility-garch/
