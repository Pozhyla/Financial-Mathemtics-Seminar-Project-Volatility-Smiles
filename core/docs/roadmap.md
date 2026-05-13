# Project Roadmap: Volatility Smiles Analysis

This roadmap outlines the path to completing the Financial Derivatives seminar project on Implied Volatility (IV) and Volatility Smiles.

## Phase 1: Foundation ✅
- [x] Set up Python virtual environment and Jupyter kernel.
- [x] Initialize agentic workspace structure (`core/instructions/`, `core/tasks/`, `core/docs/`).
- [x] Upload project description and reference PDFs.
- [x] Upload both option chain datasets (short & long maturity).
- [x] Lock global constants: `S0 = 3046.71`, `r = 0.05`, `252` trading days/year.

## Phase 2: Environment Initialization ✅
- [x] Define data processing pipeline rules (`data_processing.md`).
- [x] Define mathematical architecture and function signatures (`math.md`).
- [x] Define strict execution constraints — Rules 1, 2, 3 (`general.md`).
- [x] Define full task checklist with execution steps (`current.md`).
- [x] Audit and refactor all control documents for specification alignment.

## Phase 3: Short-Maturity Analysis (Next — Awaiting Trigger)
- [ ] Parse `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-06-12.csv`.
- [ ] Apply illiquidity filter (Rule 1).
- [ ] Compute T via trading-day count (T = busday_count / 252).
- [ ] Implement BSM functions with convergence safety (Rule 2) and seed fallback (Rule 3).
- [ ] Generate **Deliverable #3**: Short-Maturity IV Smile Plot (T < 30 days).

## Phase 4: Long-Maturity Analysis
- [ ] Parse `OMXS30_DownloadDate_2026-05-12_ExpiryDate_2026-07-17.csv`.
- [ ] Apply same pipeline (filter → T → IV solver).
- [ ] Generate **Deliverable #4**: Long-Maturity IV Smile Plot.
- [ ] Compare short- vs. long-maturity smile shapes.

## Phase 5: Final Reporting
- [ ] Consolidate all IV computations into clean `main.ipynb`.
- [ ] Output summary data tables for both maturities.
- [ ] Address Key Analysis Concepts (skew intuition, model limitations).
- [ ] Finalize Deliverables #1 (Notebook) and #2 (Data Table).
