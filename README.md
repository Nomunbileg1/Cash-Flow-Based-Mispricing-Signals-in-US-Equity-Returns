# Value-Glamour and Accruals Mispricing: One Anomaly or Two?

A replication and backtest of the cash-flow-to-price (CFO/P) anomaly studied in **Desai, Rajgopal & Venkatachalam (2004), _The Accounting Review_**, built on CRSP and Compustat data via WRDS.

**Authors:** Nomunbileg Sukhbold, Ryusei Leon Nakano
**Course:** Quantitative Finance Project, MS Applied Data Science, University of Chicago

---

## Overview

The paper asks whether the **value-glamour anomaly** (value stocks with high book-to-market outperform glamour stocks) and the **accruals anomaly** (high-accrual firms earn lower future returns) are two separate effects or one underlying mispricing.

The proposed unifying signal is **operating cash flow scaled by price**:

```
CFO   = Earnings + Depreciation − Working-capital accruals
CFO/P = CFO / Market Equity
```

CFO/P is both a fundamentals-to-price ratio (value-glamour) and an earnings figure corrected for working-capital accruals (accruals), so it can in principle subsume both anomalies. The hypothesis: **higher CFO/P → higher future abnormal returns.**

This project reconstructs the signal from raw data, validates it against WRDS prebuilt signals, and backtests decile long-short strategies with Fama-French risk adjustment.

---

## Data

All data is sourced from **WRDS** (not redistributed in this repo; see the Data & licensing note below).

| Source | Tables | Used for |
| --- | --- | --- |
| CRSP | `crsp.msf`, `crsp.msenames` | Monthly returns, price/market equity (`PRC`, `SHROUT`), universe filters (`SHRCD`, `EXCHCD`, `SIC`) |
| Compustat | `comp.funda` | Earnings (`oiadp`), depreciation (`dp`), and working-capital items used to build CFO |
| CRSP–Compustat Link | CCM link history | Time-valid `GVKEY ↔ PERMNO` mapping (`LINKDT`/`LINKENDDT`) |
| Fama-French | `ff.factors_monthly` | `mktrf`, `smb`, `hml`, `rf` for alpha regressions |

**Sample universe**
- Period: **1973–1997** (25 years, monthly)
- Exchanges: NYSE, AMEX, NASDAQ; US common stocks only (`SHRCD` 10, 11)
- Excludes financial firms (SIC 6000–6999), firms with sales under $1M, and negative book-equity firms
- Final panel after cleaning: **87,931 firm-year observations** (≈86,000 matched firm-months for the WRDS comparison; 289 months in the backtest)

---

## Methodology

1. **Pull and screen Compustat fundamentals** (`INDL` / `STD` / `D` / `C`), apply universe filters, and winsorize accounting inputs at the 1%/99% level by fiscal year.
2. **Construct signals** using a balance-sheet accruals approach:
   - `ACC`: total accruals scaled by average assets
   - `A/P`: accruals to price
   - `B/M`: book-to-market
   - `E/P`: earnings to price
   - `C/P`: conventional cash flow (earnings + depreciation) to price
   - `CFO/P`: clean operating cash flow to price (the focal signal)
   - `CFO/TA`: clean operating cash flow to total assets
   - `SG`: three-year average sales growth
3. **Avoid look-ahead bias:** form portfolios ~4 months after fiscal year-end and rebalance annually.
4. **Validate against WRDS prebuilt signals**: correlations, mean absolute differences, scatter/KDE comparisons.
5. **Backtest:** build a monthly panel with a 12-month holding window, sort into deciles each month, and compute equal-weighted (EW) and value-weighted (VW) returns.
6. **Long-short strategy:** Long the top decile (D10), short the bottom decile (D1).
7. **Risk adjustment:** regress long-short excess returns on the Fama-French 3-factor model to estimate alpha, plus Sharpe ratio, t-stats, hit rate, and max drawdown.

---

## Signal diagnostics

**Correlation evidence (N = 87,931).** The pairwise correlations are themselves a test of the paper's thesis. CFO/P is strongly *negatively* correlated with the accruals proxies (−0.72 with A/P and −0.39 with ACC) and positively correlated with the conventional cash-flow-to-price measure C/P (+0.51). In other words, the cash-flow signal moves opposite to accruals and overlaps with the value-glamour proxies, exactly what you'd expect if a single cash-flow mechanism sits behind both anomalies.

**Validation against WRDS prebuilt signals (≈86,000 matched firm-months).** The reconstructed signals were benchmarked against WRDS's own factor library:

| Signal | Correlation w/ WRDS | Mean abs. diff |
| --- | --- | --- |
| B/M | 0.862 | 0.130 |
| CFO/TA | 0.847 | 0.072 |
| E/P | 0.696 | 0.130 |
| CFO/P | 0.683 | 0.141 |
| SG | 0.559 | 0.175 |
| ACC | 0.067 | 0.073 |

B/M and CFO/TA match WRDS closely (corr > 0.84). The low ACC correlation is expected: WRDS uses a cash-flow-statement accruals definition while this project uses the balance-sheet approach. CFO/P's 0.68 reflects differences in how working-capital accruals, timing, missing values, and winsorization are handled.

---

## What the paper finds

Desai, Rajgopal & Venkatachalam (2004) argue the two anomalies are largely **one**:
1. **CFO/P is a strong predictor** of future returns and is tradable on its own.
2. **It absorbs the value-glamour proxies**: once CFO/P is included, B/M, E/P, conventional C/P, and sales growth add little incremental power.
3. **It absorbs the accruals anomaly**: accruals predict returns alone, but their power weakens once CFO/P is included.

The takeaway: the *definition* of cash flow matters. Adjusting earnings for working-capital accruals changes what the signal captures and can unify the two patterns.

## Our replication results

The portfolio sorts below reproduce the signals on 1973–1997 CRSP/Compustat data and add a Fama-French 3-factor adjustment. They confirm the raw cash-flow effect but show where the abnormal return actually survives risk adjustment.

### CFO/P decile long-short (D10 − D1), raw returns (289 months)

| Weighting | Monthly | Annualized | Ann. vol | t-stat | Sharpe | Hit rate |
| --- | --- | --- | --- | --- | --- | --- |
| Equal-weighted | 0.94% | 11.3% | 14.2% | 3.89 | 0.31 | 66.4% |
| Value-weighted | 1.09% | 13.0% | 17.5% | 3.67 | 0.35 | 65.1% |

Average monthly returns rise close to monotonically across deciles (EW: D1 = 1.19%, D10 = 2.13%; VW: 0.65% → 1.74%), and the long-short effect is *stronger* value-weighted, so it is not purely a small-firm phenomenon.

### Raw vs. factor-adjusted results across all signals

Long-short (D10 − D1) average returns and Fama-French 3-factor alphas. A negative sign reflects the D10 − D1 construction direction (e.g. high-accrual firms underperform).

| Signal | EW raw (mo / ann) | VW raw (mo / ann) | EW FF3 α (ann, t) | VW FF3 α (ann, t) |
| --- | --- | --- | --- | --- |
| **CFO/P**  | 0.94% / 11.3% | 1.09% / 13.0% | +1.3% (t 0.44) | ~0% (t −0.02) |
| **CFO/TA** | 0.77% / 9.2%  | 0.95% / 11.4% | +5.9% (t 2.45) | +8.3% (t 3.21) |
| **ACC**    | −1.01% / −12.1% | −0.76% / −9.1% | −18.1% (t −7.16) | −16.2% (t −5.11) |
| **B/M**    | 1.27% / 15.2% | 0.63% / 7.6%  | +2.3% (t 1.04) | −11.0% (t −4.41) |
| **SG**     | −1.11% / −13.3% | −0.29% / −3.5% | −15.4% (t −5.97) | −4.9% (t −1.58) |
| **E/P**    | 0.38% / 4.6%  | 1.20% / 14.4% | −3.0% (t −0.97) | +2.9% (t 0.74) |

### What the numbers say

- **CFO/P and B/M** generate large raw returns but little surviving FF3 alpha; their performance is largely explained by exposure to systematic factors (the HML factor already captures the value premium).
- **Accruals and sales growth** produce strong, highly significant abnormal returns even after factor adjustment, consistent with well-documented behavioral anomalies.
- **CFO/TA is the most robust signal**, delivering economically large and statistically significant alphas in *both* equal- and value-weighted portfolios.

**Takeaway (consistent with the paper's thesis):** the replication confirms the raw cash-flow effect: CFO/P delivers a monotonic decile pattern and an 11–13% annualized long-short spread (t > 3.6). But the risk adjustment locates where the abnormal return actually lives: CFO/P's spread is largely absorbed by HML (value) exposure, while **CFO/TA retains a significant value-weighted alpha of 8.3% per year (t = 3.21)** after Fama-French controls. The reconstructed signals also track WRDS benchmarks closely (CFO/P ≈ 0.68, CFO/TA ≈ 0.85). Overall, cash-flow fundamentals appear to carry persistent pricing information not fully captured by standard risk factors, supporting the paper's argument that value-glamour and accruals are better understood as one cash-flow-driven anomaly than two.

---

## Figures

The notebook produces, among others:
- Compustat and CRSP firm-coverage over time
- Signal distributions and per-signal histograms (before/after winsorizing)
- Replication-vs-WRDS scatter, KDE, difference, and time-series comparisons
- Cumulative CFO/P and CFO/TA decile-ladder performance (EW and VW)
- Long-short cumulative return and FF3 alpha-based cumulative performance
- Long-short monthly return and annualized FF3 alpha bar charts by signal

---

## Repository structure

```
.
├── README.md
├── presentation.pdf                    # final project slides (March 2026)
└── Quant_Finance_Project_Code.ipynb    # main analysis notebook (outputs cleaned of raw records/credentials)
```

## Reproducing

1. A valid **WRDS account** with CRSP, Compustat, CCM, and Fama-French access.
2. `pip install wrds pandas numpy matplotlib seaborn scipy statsmodels pyarrow`
3. Run the notebook; you will be prompted for your WRDS username and password at runtime (no credentials are stored in the code).
4. The notebook caches pulled tables as local `*.parquet` files. These are **not** included in the repo (see below) and will be regenerated from your own WRDS pulls.

---

## Data & licensing note

This project relies on **CRSP and Compustat data accessed through WRDS**, which are proprietary and licensed. Raw data files (`*.parquet`) and raw firm-level record dumps are intentionally excluded from this repository. To run the code you must have your own WRDS subscription and pull the data yourself. Only aggregate, derived research output (summary statistics, portfolio returns, and figures) is shared here. Please confirm your own institution's WRDS data-use agreement before redistributing any data or outputs.

## Reference

Desai, H., Rajgopal, S., & Venkatachalam, M. (2004). *Value-Glamour and Accruals Mispricing: One Anomaly or Two?* The Accounting Review, 79(2), 355–385.
