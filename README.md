# Model-based Clustering using mclust

Model-based clustering of the Swiss banknote dataset using Gaussian finite mixture models fitted with the [`mclust`](https://mclust-org.github.io/mclust/) R package.

The analysis works through univariate clustering on a single feature, compares covariance assumptions via deviance and BIC, and then extends to multivariate clustering across all six measurements.

## Data

The `banknote` dataset ships with `mclust`. It contains 200 Swiss 1000-franc notes — 100 genuine, 100 counterfeit — with six measurements (mm) per note:

| Variable | Description |
|---|---|
| `Length` | Length of the note |
| `Left` | Width of the left edge |
| `Right` | Width of the right edge |
| `Bottom` | Bottom margin width |
| `Top` | Top margin width |
| `Diagonal` | Diagonal length of the printed image |

The `Status` column (genuine / counterfeit) is dropped before clustering and used only as external context.

## Requirements

```r
install.packages(c("mclust", "tidyverse", "patchwork", "ggridges"))
```

Rendering the report also needs `rmarkdown` and a LaTeX distribution (e.g. `tinytex`) for PDF output.

## Repository contents

```
.
├── model-based-clustering.Rmd   # source analysis
├── model-based-clustering.pdf   # knitted output
└── README.md
```

## Reproducing the report

```r
rmarkdown::render("model-based-clustering.Rmd")
```

Or click **Knit** in RStudio.

## Analysis outline

**1. Data exploration**
A `Left` vs `Right` scatter plot shows the two true classes overlap heavily, so no single pair of edge measurements separates them cleanly. Marginal density plots across all six features, plus a scaled ridgeline plot, identify `Diagonal` as the only clearly bimodal feature — the natural candidate for univariate clustering.

**2. Univariate clustering on `Diagonal`**
Two-component mixtures are fitted under both covariance assumptions:

| Model | Constraint | Means | Variance(s) | Deviance | BIC |
|---|---|---|---|---|---|
| `E` | equal variance | 139.45, 141.52 | 0.244 | 548.27 | 569.47 |
| `V` | unequal variance | 139.50, 141.56 | 0.359, 0.150 | 537.02 | 563.51 |

The `V` model wins on both criteria, though BIC narrows the gap by penalising its extra parameter. Density plots make the difference visible: under `E` both bumps share a width, while under `V` the lower-`Diagonal` component is wider and flatter than the upper one.

The report also verifies `mclust`'s BIC by hand. Computing `-2 * loglik + log(n) * m` reproduces the reported value exactly apart from a sign flip — `mclust` stores BIC as the negative of the conventional definition, so values must be negated before comparison.

**3. Multivariate clustering**
Fitting `Mclust(df)` with no constraints searches all covariance structures and 1–9 components. The BIC-optimal model is **VVE with 3 components** (variable volume, variable shape, equal orientation), splitting the 200 notes 18 / 98 / 84 — one more cluster than the two known real-world classes.

Forcing two components with the most flexible structure (`VVV`) gives a matrix of bivariate density contours. `Diagonal`–`Top` and `Diagonal`–`Bottom` produce the cleanest separation, while pairs like `Left`–`Right` overlap almost completely. Plotting cluster assignments with points sized by classification uncertainty shows the model is confident nearly everywhere: only three points carry meaningful uncertainty, and they don't sit on the visual boundary in the `Left`/`Right` view — the remaining dimensions resolve them.

## Author

Anthony Kamau
