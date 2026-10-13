# Causal Impact of California's Proposition 99 (Synthetic Control)

Estimating the effect of California's 1989 cigarette tax on per-capita cigarette sales, using observational state-year panel data and a counterfactual ("what would have happened without the tax?") built with the **synthetic control method**.

## Question

California raised its cigarette tax in 1989, and sales fell afterwards, but sales were already trending down before the tax. How much of the decline is due to the tax, and how much would have happened anyway?

## Key results

| Item | Value |
|---|---|
| Method | Synthetic Control (6 donor states, fitted on 1970-1988 only) |
| Pre-period fit (RMSE) | 1.66 packs |
| **ATT (1989-2000)** | **-19.32 packs per person per year** |
| Effect in 1989 / 2000 | -8.3 / -26.4 packs |
| Placebo test | California ranks 3rd of 39 (p ≈ 0.077) |

The effect is large and grows over time, but the placebo test is suggestive rather than significant at the 5% level, and other California-specific factors cannot be ruled out. See `note.md` for the full interpretation.

## Data

`data.csv`: balanced panel of **39 US states x 31 years (1970-2000)**.

| Column | Description |
|---|---|
| `state`, `year` | Panel identifiers |
| `cigsale` | Outcome: per-capita cigarette sales (packs) |
| `retprice`, `lnincome`, `beer`, `age15to24` | Predictors (the last three have missing years; the synthetic control uses the outcome only) |

- **Treated unit:** California. **Controls:** the other 38 states.
- **Pre-period:** 1970-1988. **Post-period:** 1989-2000.

## Repository contents

| File | Purpose |
|---|---|
| `californiya.ipynb` | Analysis notebook (data checks, EDA, method selection, synthetic control, ATT, placebo test, interpretation) |
| `note.md` | Documentation: chosen method, assumption checks, final ATT estimate, interpretation |
| `data.csv` | Dataset |(https://github.com/Matt2371/Proposition99)
| `README.md` | This file |

## Notebook outline

1. **Data:** structure, completeness and missing-value checks
2. **EDA:** California vs. other states, gap plot (parallel-trends check), individual state trajectories, rough difference-in-differences (-27.3 packs, descriptive only)
3. **Method selection:** Difference-in-Differences vs. Synthetic Control, with justification
4. **Synthetic control:** feasibility check, constrained weight optimization, pre-period RMSE
5. **Results:** real vs. synthetic California, ATT, effect over time
6. **Placebo test:** same method applied to each untreated state
7. **Final result and honest interpretation:** what can and cannot be concluded, threats to the estimate

## How to run

Requirements: Python 3 with `pandas`, `numpy`, `scipy`, `matplotlib`, and Jupyter.

```bash
pip install pandas numpy scipy matplotlib jupyter
jupyter notebook californiya.ipynb
```

Keep `data.csv` in the same folder as the notebook and run the cells from top to bottom. The synthetic control is implemented directly with `scipy.optimize` (SLSQP), with no random components, so results are reproducible. The placebo test fits 38 additional synthetic controls and may take a short while.

## Limitations

- The synthetic control is built from the outcome only; adding the predictors would be a useful robustness check.
- The estimate assumes no other California-specific change around 1989; spillovers (cross-border purchases) and unobserved shocks are possible threats.
- The in-sample pre-period fit can partly reflect overfitting (38 donors, 19 pre-period years).
- The placebo p-value (≈ 0.08) is not significant at the 5% level.