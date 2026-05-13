# Mathematical Conventions

This file defines the notation, mathematical standards, and numerical safety rules for this seminar project.

## ⚠️ Source of Truth — Professor's Functions (MANDATORY)
The three core BSM functions (`bsm_call_value`, `bsm_vega`, `bsm_call_imp_vol`) were **provided by the professor** in [`core/src/main.ipynb`](file:///Users/andrejpozilenkov/Desktop/University/Financial%20Mathematics/Seminar%20Project/core/src/main.ipynb).

**Rules:**
1. These functions MUST be used **exactly as defined** in `main.ipynb`. Do NOT rewrite, refactor, or alter the internal logic.
2. The function **signatures** (`bsm_call_value(S0, K, T, r, sigma)`, `bsm_vega(S0, K, T, r, sigma)`, `bsm_call_imp_vol(S0, K, T, r, C0, sigma_est, it=100)`) must remain identical.
3. Additional safety wrappers (Rule 2: convergence safety, Rule 3: seed fallback) must be built **around** the professor's functions — never modify the originals.
4. When copying into the final notebook, preserve the professor's docstrings and comments.

## Standard Notation
- **S₀** (`S0`): Underlying index spot price at evaluation date.
- **K**: Strike price.
- **r**: Risk-free interest rate (continuously compounded).
- **σ** (`sigma`): Volatility.
- **T**: Time to maturity (in trading-day year fractions: `trading_days / 252`).
- **C₀** (`C0`): Observed market price of the call option.

## Core Functions (Three Required)

### 1. BSM Call Valuation
```
bsm_call_value(S0, K, T, r, sigma)
```
- Formula: $C = S_0 N(d_1) - K e^{-rT} N(d_2)$
- $d_1 = \frac{\ln(S_0 / K) + (r + \frac{1}{2}\sigma^2) T}{\sigma \sqrt{T}}$
- $d_2 = d_1 - \sigma \sqrt{T}$
- Uses: `math.log`, `math.sqrt`, `math.exp`, `scipy.stats.norm.cdf`

### 2. BSM Vega (Derivative of BSM w.r.t. σ)
```
bsm_vega(S0, K, T, r, sigma)
```
- Formula: $\nu = S_0 \phi(d_1) \sqrt{T}$
- $\phi(\cdot)$ = standard normal PDF → `scipy.stats.norm.pdf`
- Uses: `math.log`, `math.sqrt`, `scipy.stats.norm.pdf`

### 3. Implied Volatility Solver (Newton-Raphson)
```
bsm_call_imp_vol(S0, K, T, r, C0, sigma_est=0.20, it=100)
```
- Iteration: $\sigma_{n+1} = \sigma_n - \frac{C^{BS}(\sigma_n) - C_0}{\nu(\sigma_n)}$
- Convergence check after each iteration.

## Numerical Standards
- **Libraries**: Use `math` (not `numpy`) for scalar BSM computations + `scipy.stats` for distributions.
- **Precision**: Standard float64.
- When implementing a formula, always include a comment with the LaTeX representation.
- Example: `# $C = S_0 N(d_1) - K e^{-rT} N(d_2)$`

## Convergence Safety (CRITICAL — Rule 2)
The Newton-Raphson solver MUST be wrapped in a `try-except` block:

```python
try:
    for i in range(it):
        vega = bsm_vega(S0, K, T, r, sigma_est)
        if abs(vega) < 1e-12:
            raise ZeroDivisionError("Vega near zero")
        sigma_est -= (bsm_call_value(S0, K, T, r, sigma_est) - C0) / vega
except (ZeroDivisionError, ValueError) as e:
    logger.warning(f"IV solver failed for K={K}: {e}")
    return None  # or np.nan
```

**Why:** When Vega approaches zero (deep ITM/OTM), division by zero occurs. The solver must catch this, log a warning for the specific strike, and continue to the next strike.

## Seed Value Strategy (CRITICAL — Rule 3)
Initialize Newton-Raphson with `sigma_est = 0.20`.

If convergence fails (returns `None` / `NaN`), implement a **fallback cascade**:
1. Retry with `sigma_est = 0.05`
2. Retry with `sigma_est = 0.50`
3. If all three fail → permanently discard that strike point with a logged warning.

```python
SEED_CASCADE = [0.20, 0.05, 0.50]

def solve_iv_with_fallback(S0, K, T, r, C0):
    for seed in SEED_CASCADE:
        result = bsm_call_imp_vol(S0, K, T, r, C0, sigma_est=seed)
        if result is not None and np.isfinite(result) and result > 0:
            return result
    logger.warning(f"All seeds failed for K={K}. Discarding strike.")
    return np.nan
```
