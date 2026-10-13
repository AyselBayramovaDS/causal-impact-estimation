# Estimating the Causal Impact of California's Proposition 99 on Cigarette Sales

This note documents the analysis carried out in `californiya.ipynb`: the chosen method, the assumption checks, the final ATT estimate, and its interpretation. All numbers below are taken from the notebook outputs.

---

## 1. Objective

California raised its cigarette tax in 1989 (Proposition 99), and per-capita cigarette sales fell afterwards. The difficulty is that sales were already trending downward before the tax. The question is therefore:

> How much of California's decline is due to the tax, and how much would have happened anyway?

Answering this requires a **counterfactual**: what California's sales would have been without the tax. That scenario does not exist in reality, so it has to be estimated from the untreated states, and the estimate must be tested before it is trusted.

## 2. Data

The dataset is a balanced state-year panel: **39 states x 31 years (1970-2000) = 1,209 rows, 7 columns**.

| Item | Description |
|---|---|
| Outcome | `cigsale`, per-capita cigarette sales (packs) |
| Predictors | `retprice`, `lnincome`, `beer`, `age15to24` |
| Treated unit | California |
| Control (donor) pool | The other 38 states |
| Pre-period | 1970-1988 (19 years) |
| Post-period | 1989-2000 (12 years) |

**Data checks performed**

- 39 unique states, years 1970-2000 with no gaps, every state has 31 observations, no duplicate rows.
- `cigsale` and `retprice` have no missing values.
- `lnincome` (195), `beer` (663) and `age15to24` (390) have missing values. Each count is divisible by 39, which indicates that whole years are missing for all states rather than random gaps.

**Decision on missing values:** they were not dropped, because dropping them would remove a large part of the pre-period. The synthetic control was built from the outcome (`cigsale`) alone, so the missing predictor values do not affect the estimate. Adding the predictors is left as a possible robustness extension.

## 3. Exploratory Analysis

Four checks were made before choosing a method.

**3.1 California vs. the average of the other 38 states.** After 1989 California's sales fall sharply, but the other states also decline, only more weakly. A general downward trend therefore exists independently of the tax, and the whole decline cannot be attributed to the policy. Before 1989 the two curves are not parallel: in the mid-1970s other states rise while California stays flat, and after 1980 California declines faster.

**3.2 Gap plot (California minus the average of other states).** This was used to check the parallel-trends assumption visually.
- 1970-1982: the gap falls from about +3 to about -21, so California diverges systematically; parallel trends is violated here.
- 1982-1987: the gap is roughly stable around -20.
- After 1989: the gap widens further, from about -27 to about -50.

**3.3 California among individual states.** Before 1989 California lies inside the dense cluster of other states (about 100-140 packs), so states with similar trajectories exist. Some states reach 200-300 packs and pull the simple average upward, which makes the plain average a weak comparison for California. After 1989 California separates downward and ends near the lowest trajectories.

**3.4 Rough difference-in-differences.** Average pre/post change: California -55.9 packs, other states -28.5 packs, difference **-27.3 packs**. This is only a descriptive estimate. Because the gap was already changing substantially before the tax, the parallel-trends assumption is in doubt and -27.3 should not be read as the amount caused by the tax.

## 4. Method Choice and Justification

The reasoning rests on one criterion: an estimation method can be trusted for the post-tax period only if it reproduces California well in the pre-tax period, where the tax-free outcome is known.

| | Difference-in-Differences | Synthetic Control |
|---|---|---|
| Counterfactual | Simple average of other states | Weighted combination of other states |
| Key assumption | Parallel trends before 1989 | Combination tracks California closely before 1989 |
| In this data | Visual check shows the gap between California and the average changes before 1989, so the assumption appears violated | California lies within the range of the other states, so a weighted combination appears feasible |

The problem structure (1 treated unit, 38 controls, 19 pre-period years) also suits synthetic control. **Chosen method: Synthetic Control.** The fit of this method in the pre-period was still to be verified at the time of the choice, and it is checked in Section 6.

## 5. Synthetic Control Implementation

**Feasibility check.** For every year of 1970-1988, California's value lies between the minimum and maximum of the donor states (result `1.0`). This is a useful necessary check but does not prove that non-negative weights can reproduce California closely.

**Weights.** Weights over the 38 donor states were found by minimizing the squared difference between California and the weighted combination over 1970-1988 only, subject to: weights non-negative (0 to 1) and summing to 1 (SLSQP optimizer).

- *First attempt:* the optimizer returned equal weights of 0.026 (1/38) for every state. This means it stopped at its starting point, and the result would have been identical to the simple average already rejected.
- *Fix:* the objective function was rescaled (data unchanged), with `maxiter=1000` and `ftol=1e-10`.

**Resulting weights (6 states with weight above 0.01):**

| State | Weight |
|---|---|
| Utah | 0.404 |
| Montana | 0.204 |
| Nevada | 0.202 |
| Connecticut | 0.106 |
| New Hampshire | 0.048 |
| Colorado | 0.036 |

The remaining states receive (near) zero weight. The weighted combination is "Synthetic California", the estimated tax-free scenario.

## 6. Assumption Validation

| Check | Result |
|---|---|
| California inside donor min-max range (pre-period) | 1.0 (every year) |
| Pre-period RMSE | **1.66 packs** |
| Sum of weights | 1.0 |
| Optimizer converged | True |

In the plot of real vs. synthetic California, the two lines almost overlap before 1989. The small pre-period RMSE supports the credibility of the counterfactual. This is an in-sample fit (19 years, 38 donors), so it can partly reflect overfitting; the placebo test (Section 8) is the complementary check.

## 7. Results

After 1989 real California falls well below Synthetic California.

| Quantity | Value |
|---|---|
| **ATT** (average of real minus synthetic, 1989-2000) | **-19.32 packs per person per year** |
| Effect in 1989 | -8.3 packs |
| Effect in 2000 | -26.4 packs |

The effect-over-time plot shows the gap close to zero before 1989, then falling steadily after 1989 (about -8 in 1989 to about -26 in 2000), so the effect grows gradually and flattens towards the end.

For comparison, the rough DiD estimate was -27.3 packs, larger than the synthetic control estimate of -19.32 packs. Since the rough DiD relied on a parallel-trends assumption that the pre-period data did not support, the synthetic control estimate is treated as the main result.

## 8. Placebo Test

To check whether a gap as large as California's could appear by chance, the same synthetic control method was applied to each of the 38 untreated states as if it had been treated in 1989 (no data was changed). For every state the ratio of post-1989 RMSPE to pre-1989 RMSPE was computed.

- California's ratio is **12.3**, ranking **3rd among 39 states**, behind Missouri (21.6) and Virginia (19.0).
- Placebo p-value = 3/39 ≈ **0.077**.
- In the placebo plot (states with a pre-period fit comparable to California's), the placebo gaps stay close to zero before 1989 and then spread widely after 1989 (roughly -29 to +25 packs), while California falls to about -26, near the lower edge of this range but not alone.

California's gap is among the largest but not unique, so the effect is suggestive but **not statistically significant at the 5% level** in this specification.

## 9. Interpretation

**What we can say**
- After 1989, California's sales fell by an average of 19.32 packs per person per year more than its synthetic counterfactual (estimated ATT).
- The gap grew over time, from about -8 packs in 1989 to about -26 packs in 2000.
- The synthetic control tracked California closely over the 19-year pre-period (RMSE = 1.66), which supports the credibility of the counterfactual.

**What we cannot say**
- That the tax alone caused this gap. The estimate relies on the assumption that no other California-specific change occurred around 1989.
- That the effect is statistically proven; the placebo p-value (≈ 0.08) is above the conventional 5% level.

**Threats to the estimate**
1. Other California-specific interventions around 1989 (anti-smoking campaigns, local bans) could be wrongly attributed to the tax.
2. Spillovers: cross-border purchases by Californians in neighboring donor states could contaminate the control data.
3. Unobserved shocks over time cannot be detected by this method.
4. The synthetic control was built from the outcome only, omitting economic and demographic predictors.
5. A tight in-sample fit with 38 donors and 19 pre-period years can partly reflect overfitting.
6. Two untreated states (Missouri and Virginia) show larger post/pre ratios than California, which points to baseline statistical noise.

## 10. Conclusion

A synthetic control built from six donor states reproduces California's pre-1989 sales closely (RMSE = 1.66 packs). After 1989 real sales fall below this counterfactual by 19.32 packs per person per year on average, with the gap widening from about -8 to about -26 packs. A certain portion of California's decline can therefore be explained by the effect of the tax. However, because other confounding factors are possible and the placebo test is suggestive rather than significant (p ≈ 0.08), the decline cannot be attributed solely to the tax.

## 11. Limitations and Possible Extensions

- Add the predictors (`retprice`, `lnincome`, `beer`, `age15to24`) to the matching as a robustness check.
- Estimates are based on a single outcome-only specification; other methods (e.g., EconML meta-learners) were not applied.

## 12. Reproducibility

`californiya.ipynb` reads `data.csv` from the same folder and uses `pandas`, `numpy`, `scipy` and `matplotlib`. Cells should be run top to bottom. The method is implemented directly (SLSQP optimization), with no random components.