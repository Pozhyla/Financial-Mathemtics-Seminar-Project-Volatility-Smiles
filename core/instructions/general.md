# General Instructions

These are the project-wide standards for the **Financial Mathematics Seminar Project**.

## Global Constants (Locked)
| Constant | Value |
|----------|-------|
| Evaluation Date (Today) | `2026-05-12` |
| Underlying Index | OMXS30 |
| Spot Price (`S0`) | `3046.71` |
| Risk-Free Rate (`r`) | `0.05` |
| Trading Days / Year | `252` |
| Target Short Maturity Expiry | `2026-06-12` |
| Target Long Maturity Expiry | `2026-07-17` |

## Coding Standards
- **Language**: Python 3.14+
- **Style**: Follow PEP 8 guidelines.
- **Logging**: Use `logging` module for all important events and errors. No `print()` for diagnostics.
- **Comments**: Use verbose comments to explain all important events and errors. Do not over-explain obvious code.
- **Documentation**: Use Google-style docstrings for all functions and classes.
- **Error Handling**: Use explicit exceptions; avoid bare `except:`. Always catch specific exception types (`ZeroDivisionError`, `ValueError`).

## Required Libraries
| Library | Use |
|---------|-----|
| `math` | `log`, `sqrt`, `exp` for scalar BSM computations |
| `scipy.stats` | `norm.cdf`, `norm.pdf` for normal distribution |
| `pandas` | DataFrame operations, CSV parsing |
| `numpy` | `busday_count` for trading days, `np.nan`, `np.isfinite` |
| `matplotlib.pyplot` | Visualization |
| `logging` | Structured logging |
| `datetime` | Date handling |

## Strict Execution Constraints
Three critical rules must be enforced in every implementation:

1. **Rule 1 — Market Price & Illiquidity Filter**: Compute `C0` via priority cascade: (1) Mid-Price `(Bid+Ask)/2` if both exist and > 0, (2) `Last` as fallback, (3) drop row if all missing/zero. See `data_processing.md`.
2. **Rule 2 — Convergence Safety**: Wrap Newton-Raphson in `try-except`. Catch `ZeroDivisionError` and `ValueError`. See `math.md`.
3. **Rule 3 — Seed Fallback Strategy**: Try seeds `[0.20, 0.05, 0.50]` in sequence. Discard strike only after all three fail. See `math.md`.

## AI Interaction Guidelines
- **Professor's Functions**: The three BSM functions in `core/src/main.ipynb` are the **source of truth**. Use them exactly as provided — do NOT rewrite. Safety wrappers go around them. See `math.md` §Source of Truth.
- **Verification**: Always verify mathematical formulas against `core/instructions/math.md` before implementation.
- **Environment**: Always use the virtual environment located at `.venv`.
- **Context**: Check `core/docs/roadmap.md` before suggesting major structural changes.
- **Constants**: Never hardcode constants inline — always reference the Global Constants table above.
