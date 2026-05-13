# Current Tasks

## Active Task: Full IV Smile Pipeline (Deliverables #3 & #4)

### Step 1 — Data Loading & Cleaning
- [ ] Load `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-06-12.csv` (Short Maturity).
- [ ] Load `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-07-17.csv` (Long Maturity).
- [ ] Handle `;` separator, skip `sep=;` header line.
- [ ] Parse `Strike` as numeric (remove `"` quotes and `,` thousands separator).
- [ ] Coerce `Bid`, `Ask`, `Last`, `High`, `Low`, `Open Interest`, `Volume` to numeric.
- [ ] **MARKET PRICE (Rule 1)**: Compute `C0` via priority cascade: (1) `Mid = (Bid+Ask)/2` if both > 0, (2) `Last` fallback, (3) drop row if all missing/zero.

### Step 2 — Time-to-Maturity Computation
- [ ] Use `numpy.busday_count()` to count trading days between `2026-05-12` and each expiry.
- [ ] Compute `T = trading_days / 252` for each maturity.
- [ ] **Do NOT use `days / 365`** — strict 252 trading-day convention.

### Step 3 — BSM Functions & IV Solver
- [ ] Use `bsm_call_value(S0, K, T, r, sigma)` from `core/src/main.ipynb`.
- [ ] Use `bsm_vega(S0, K, T, r, sigma)` from `core/src/main.ipynb`.
- [ ] Use `bsm_call_imp_vol(S0, K, T, r, C0, sigma_est=0.20, it=100)` from `core/src/main.ipynb`.
- [ ] **CONVERGENCE SAFETY (Rule 2)**: Wrap solver in `try-except` for `ZeroDivisionError` / `ValueError`.
- [ ] **SEED FALLBACK (Rule 3)**: Cascade through `[0.20, 0.05, 0.50]` before discarding strike.

### Step 4 — IV Computation Loop
- [ ] For each valid row in short-maturity DataFrame: solve IV using `solve_iv_with_fallback()`.
- [ ] For each valid row in long-maturity DataFrame: solve IV using `solve_iv_with_fallback()`.
- [ ] Log warnings for any strikes that fail all seeds.
- [ ] Collect results into DataFrames with columns: `Strike`, `C0`, `IV`.

### Step 5 — Visualization (Deliverables #3 & #4)
- [ ] **Plot 1**: Short Maturity Volatility Smile (T < 30 days).
  - X-axis: Strike (K)
  - Y-axis: Implied Volatility
  - Scatter plot with connecting lines
  - Dynamic title indicating T value and expiry date
- [ ] **Plot 2**: Long Maturity Volatility Smile.
  - Same format as Plot 1
  - Dynamic title indicating T value and expiry date

### Step 6 — Summary Table
- [ ] Output final IV data table for both maturities.

## Global Parameters (Locked)
- `S0 = 3046.71` (OMXS30 spot on 2026-05-12)
- `r = 0.05`
- `Trading Days/Year = 252`

## Completed
- [x] Workspace initialization.
- [x] Initial data upload (both short and long maturity CSVs).
- [x] Project description uploaded.
- [x] Reference PDFs uploaded.

## Status
🔴 **AWAITING EXECUTION TRIGGER** — All constraints locked. Code generation blocked until user commands execution.
