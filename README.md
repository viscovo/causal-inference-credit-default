[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/uyATYxY5)

# Causal Inference — Credit Card Default
 
This project estimates the **Average Causal Effect (ACE)** of a binary treatment `PPP` on the binary outcome `Y` (credit card default). The pipeline combines causal structure learning, data-driven variable selection, and two semiparametric estimators — G-computation and IPW — under four competing adjustment sets.
 
---
 
## 📊 Project Overview
 
The four adjustment strategies are compared as a sensitivity analysis: convergence of estimates across methods supports robustness of the identified effect.
 
| Set | Source |
|---|---|
| `L_Full` | All available covariates (benchmark) |
| `L_LASSO` | Post-Double-Selection LASSO (union of variables predictive of `Y` and `PPP`) |
| `L_PC` | Local structure of `Y` from the PC Algorithm (parents, children, spouses) |
| `L_IAMB` | Markov Blanket of `Y` estimated by Fast-IAMB |
 
Statistical significance is assessed via a non-parametric bootstrap (`B = 500`) providing 95% percentile confidence intervals for all estimator–set combinations.
 
---
 
## 📂 Repository Structure
 
```
.
├── main.Rmd                        # Full analysis pipeline (R Markdown)
├── data/
│   └── low2791upd.csv              # Input dataset (500 obs, 24 variables)
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
 
---
 
## ⚙️ Requirements
 
- R ≥ 4.2
- Required packages:
 
```r
install.packages(c(
  "pcalg", "bnlearn", "glmnet", "graph",
  "boot", "parallel", "ggplot2", "dplyr", "jsonlite"
))
BiocManager::install("Rgraphviz")  # for graph visualisation
```
 
---
 
## ▶️ Reproducing the Analysis
 
To run the full pipeline, knit `main.Rmd` from the project root:
 
```r
rmarkdown::render("main.Rmd")
```
 
The bootstrap uses `clusterSetRNGStream(cl, 3009)` for reproducibility across parallel workers. All `export/` subdirectories are created automatically at runtime.