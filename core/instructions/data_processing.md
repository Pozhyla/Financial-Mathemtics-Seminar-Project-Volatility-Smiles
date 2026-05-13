# Data Processing Instructions: OMXS30 Options

These rules ensure consistent handling of the Nasdaq OMX dataset for the Volatility Smile project.

## File Format
- **Filename**: `OMXS30_DownloadDate_YYYY-MM-DD_ExpiryDate_YYYY-MM-DD.csv`
- **Separator**: `;`
- **First Line**: `sep=;` (must be skipped via `skiprows=1` in `pd.read_csv`).

## Available Datasets (2026-05-12)
| Dataset | File | Expiry | Use |
|---------|------|--------|-----|
| Short Maturity | `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-06-12.csv` | 2026-06-12 | Deliverable #3 |
| Long Maturity | `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-07-17.csv` | 2026-07-17 | Deliverable #4 |

## Column Mapping & Cleaning
- **Strike**: Format is `"X,XXX.XX"`. Must remove quotes and replace `,` with nothing to convert to float.
- **Expiry (`Exp.`)**: Date format `YYYY-MM-DD`.
- **Market Price (`C0`)**: Computed via the priority cascade below (see §Market Price Calculation).
- **Numeric Columns**: `Bid`, `Ask`, `Last`, `High`, `Low`, `Open Interest`, `Volume` — coerce to numeric with `errors='coerce'`.

## Market Price Calculation (CRITICAL — Rule 1)
**This step is mandatory and must execute before any IV computation.**

Compute `C0` (market option price) using the following **priority cascade**:

| Priority | Condition | C0 Value |
|:---:|-----------|----------|
| **1** | Both `Bid` and `Ask` exist and are > 0 | `C0 = (Bid + Ask) / 2` (Mid-Price) |
| **2** | `Bid` or `Ask` missing, but `Last` exists and > 0 | `C0 = Last` |
| **3** | `Bid`, `Ask`, AND `Last` are all missing or ≤ 0 | **Drop the row entirely** |

Rationale: The mid-price is the best estimate of fair value. `Last` is a fallback for strikes where no current quote exists but a recent trade was recorded. Rows with no pricing data at all are illiquid and must be discarded — feeding `C0 = 0.00` or `NaN` to the Newton-Raphson solver causes fatal failure.

```python
# ── Market Price Calculation (Rule 1) ──────────────────────────────
# Priority 1: Mid-price from Bid/Ask
has_bid_ask = df['Bid'].notna() & (df['Bid'] > 0) & df['Ask'].notna() & (df['Ask'] > 0)
df['C0'] = np.where(has_bid_ask, (df['Bid'] + df['Ask']) / 2.0, np.nan)

# Priority 2: Fall back to Last if Mid is not available
has_last = df['Last'].notna() & (df['Last'] > 0)
df['C0'] = np.where(df['C0'].isna() & has_last, df['Last'], df['C0'])

# Priority 3: Drop rows where C0 is still NaN (no pricing data)
df = df[df['C0'].notna()].copy()
```

## Parameters (Locked for 2026-05-12)
| Parameter | Symbol | Value | Source |
|-----------|--------|-------|--------|
| Evaluation Date | — | `2026-05-12` | Assignment specification |
| Spot Price | `S0` | `3046.71` | OMXS30 index level |
| Risk-Free Rate | `r` | `0.05` | 5% annualized, continuously compounded |
| Trading Days/Year | — | `252` | Standard convention |

## Time-to-Maturity Calculation (CRITICAL)
`T` must be expressed in **trading-day years**, not calendar days.

**Algorithm:**
1. Enumerate every date from `Evaluation Date + 1` through `Expiry Date` (inclusive).
2. Exclude Saturdays and Sundays (weekday index 5 and 6).
3. Count the remaining days → `trading_days`.
4. `T = trading_days / 252`

```python
import numpy as np

eval_date = np.datetime64('2026-05-12')
exp_date  = np.datetime64('2026-06-12')
trading_days = np.busday_count(eval_date, exp_date)
T = trading_days / 252
```

> **⚠ Do NOT use `days / 365` — this project uses the 252 trading-day convention.**

## Variable Mapping Summary
| CSV Column | BSM Variable | Description |
|------------|-------------|-------------|
| `Strike` | `K` | Strike price |
| `Bid`, `Ask` (→ Mid) or `Last` | `C0` | Market option price (priority cascade) |
| `Exp.` | → `T` | Expiration date → time to maturity |
