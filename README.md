# Causal Inference — Credit Card Default

This project estimates the **Average Causal Effect (ACE)** of Payment Protection Plan (PPP) enrollment on credit card default (binary outcome `Y`). The pipeline combines causal structure learning, data-driven variable selection, and two semiparametric estimators — G-computation and IPW — under four competing adjustment sets, with uncertainty via non-parametric bootstrap.

## Project Overview

Whether PPP enrollment causally increases or decreases default risk cannot be answered by a naive comparison of enrollees vs. non-enrollees, since enrollment is confounded by a priori risk. Four adjustment strategies are compared as a sensitivity analysis: convergence across methods supports robustness of the identified effect.

| Set | Source |
|---|---|
| `L_Full` | All 24 available covariates (naive upper bound) |
| `L_LASSO` | Post-Double-Selection LASSO (union of variables predictive of `Y` and `PPP`) |
| `L_PC` | Markov Boundary of `Y` from the Rank PC algorithm (nonparanormal transformation, robust to financial outliers) |
| `L_IAMB` | Markov Boundary of `Y` estimated by Fast-IAMB (G² test on discretized variables) |

Each adjustment set is evaluated under two estimators — Outcome Regression (G-computation) and Inverse Probability Weighting — whose agreement serves as a robustness check against model misspecification. Statistical significance is assessed via a non-parametric bootstrap (`B = 500`) with 95% percentile confidence intervals.

## Results

The naive observational difference (Δ̂ = 0.132) overstates the effect due to selection bias. After adjustment, the causal effect of PPP enrollment on default probability is **positive and statistically significant** across all correctly specified adjustment sets:

| Estimator | Adjustment | ACE | 95% CI | Width |
|---|---|---|---|---|
| OR | Full | 0.098 | [0.017, 0.185] | 0.168 |
| OR | LASSO | 0.105 | [0.020, 0.189] | 0.169 |
| OR | Rank PC | 0.118 | [0.051, 0.187] | 0.136 |
| OR | Fast-IAMB | 0.111 | [0.045, 0.181] | 0.136 |
| IPW | Full | 0.087 | **[−0.051, 0.216]** | 0.267 |
| IPW | LASSO | 0.084 | **[−0.028, 0.193]** | 0.221 |
| IPW | Rank PC | 0.130 | [0.062, 0.200] | 0.138 |
| IPW | Fast-IAMB | 0.111 | [0.041, 0.183] | 0.142 |

**Key finding:** under IPW, the Full (24 covariates) and LASSO (15 covariates) adjustment sets produce CIs that cross zero — a practical positivity violation, not a failure of IPW. High-dimensional adjustment predicts treatment assignment too well for some individuals, driving propensity scores toward 0/1 and inflating variance. The sparser structural sets (Rank PC: 5 variables, Fast-IAMB: 2 variables) block the relevant backdoor paths while preserving well-behaved propensity scores, yielding CIs roughly half as wide (0.136–0.142) under both estimators.

Causal interpretation: after adjusting for ~3.4 percentage points of confounding bias, PPP enrollment increases default probability by **+11 to +13 percentage points**, consistent with a moral hazard mechanism — protection from immediate repayment penalties weakens the incentive to manage liquidity carefully.

## Repository Structure

```
.
├── main.Rmd                        # Full analysis pipeline (R Markdown)
├── data/                           # Not included — see Data note below
└── export/                         # Auto-generated at runtime
    ├── plots/
    │   ├── pc.pdf                  # PC Algorithm graph (CPDAG)
    │   └── ace_bootstrap_CI_.png   # Forest plot of ACE estimates with 95% CI
    ├── dag/
    │   └── with_pay_edges.csv      # Edge list from the PC graph
    └── csv/
        ├── adjustment_sets_summary.csv  # Variables included in each adjustment set
        └── ace_bootstrap_CI.csv         # ACE point estimates and bootstrap CI
```

## Data

The dataset (`low2791upd.csv`, n = 500, 24 variables) is not included in this repository, as it was provided for coursework and is not mine to redistribute. To reproduce the analysis, place a compatible dataset in `data/` with the expected schema (see `main.Rmd` for variable names).

## Requirements

- R ≥ 4.2
- Required packages:

```r
install.packages(c(
  "pcalg", "bnlearn", "glmnet", "graph",
  "boot", "parallel", "ggplot2", "dplyr", "jsonlite"
))
BiocManager::install("Rgraphviz")  # for graph visualisation
```

## Reproducing the Analysis

To run the full pipeline, knit `main.Rmd` from the project root:

```r
rmarkdown::render("main.Rmd")
```

The bootstrap uses `clusterSetRNGStream(cl, 3009)` for reproducibility across parallel workers. All `export/` subdirectories are created automatically at runtime.

## Authors

Valerio Viscovo, Edoardo Lanzetti