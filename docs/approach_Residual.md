---
layout: default
title: SPACox / SPAmix / SPAGRM
nav_order: 3
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
parent: Genome-wide association studies
has_children: false
has_toc: false
---

# Residual-Based Methods

## Overview

GRAB provides three residual-based methods for genome-wide association studies that offer flexibility to analyze various complex traits. These methods perform association tests using genotypes and residuals from null models, enabling analysis of traits for which standard GWAS methods may not be suitable. All three methods are computationally efficient and use saddlepoint approximation (SPA) for accurate p-values. To apply these three methods, the residuals must satisfy the following conditions:

$$
\sum_{i=1}^n X_{ij} R_i = 0 \quad \text{for each } j, \quad \text{and} \quad \sum_{i=1}^n R_i = 0
$$

where $R_i$ is the residual for subject $i$, and $X_{ij}$ is the value of covariate $j$ for subject $i$.

**Features of the Methods:**

| Method     | Population Structure | Sample Relatedness | Supported `traitType`       | Modeling Approach                  |
|------------|---------------------|--------------------|-----------------------------|------------------------------------|
| **SPACox** | Not                  | Not                 | `Residuals`, `time-to-event`| Residuals random     |
| **SPAmix** | Modeled                 | Not                 | `Residuals`, `time-to-event`| Genotypes random   |
| **SPAGRM** | Not                  | Modeled                | `Residuals`                 | Genotypes random   |

> **Note:**
Documentation for SPAGRM is available at the [SPAGRM online tutorial](https://hexupku.github.io/SPAGRM.github.io/).

---

**Citation:**
**SPACox**, Bi et al. (2020). Fast and accurate method for genome-wide time-to-event data analysis and its application to UK Biobank. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2020.06.003](https://doi.org/10.1016/j.ajhg.2020.06.003)

**SPAmix**, Ma et al. (2025). Sparse estimation of high-dimensional genetic correlation and its application to global biobank meta-analysis. *Genome Biology*. [doi:10.1186/s13059-025-03827-9](https://doi.org/10.1186/s13059-025-03827-9)

**SPAGRM**, Xu et al. (2025). Scalable and accurate variance component analysis with large sample relatedness. *Nature Communications*. [doi:10.1038/s41467-025-56669-1](https://doi.org/10.1038/s41467-025-56669-1)

## Step 1: Fit Null Model (SPACox and SPAmix)

Refer to `?GRAB.NullModel`, `?GRAB.SPACox`, and `?GRAB.SPAmix` for detailed parameter instructions. A quick example is provided below.

### Option 1: Direct analysis of a time-to-event trait

If the left side of the `formula` is an `Surv` object, specify `traitType = "time-to-event"`.

```r
library(GRAB)
library(survival)

PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)

# Step 1, Option 1, SPACox
obj.SPACox = GRAB.NullModel(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER, 
  data = PhenoData, 
  subjIDcol = "IID", 
  method = "SPACox", 
  traitType = "time-to-event"
)

# Step 1, Option 1, SPAmix
obj.SPAmix = GRAB.NullModel(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData, 
  subjIDcol = "IID", 
  method = "SPAmix", 
  traitType = "time-to-event", 
  control = list(PC_columns = "PC1,PC2")
)
```

**SPAmix-specific mandatory control parameter:**  
`PC_columns`: Comma-separated column names of SNP-derived principal components (e.g., `"PC1,PC2"`).

### Option 2: Analysis from residuals

If the left side of the `formula` are residuals, specify `traitType = "Residuals"`.

#### SPACox, residuals of one trait

```r
# Step 1, Option 2, SPACox
# Fit null model and get residuals
residuals = coxph(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER, 
  data = PhenoData
)$residuals

# Calculate parameters needed for step 2
obj.SPACox = GRAB.NullModel(
  residuals ~ AGE + GENDER, 
  data = PhenoData, 
  subjIDcol = "IID", 
  method = "SPACox", 
  traitType = "Residual"
)
```

##### Null Object Components

- `N`: Sample size
- `mresid`: Martingale residuals
- `cumul`: Empirical CGF grid (t, K0, K1, K2)
- `tX`: Transpose of design matrix
- `yVec`: Event indicator
- `X.invXX`: Projection matrix for variance

#### SPAmix, multiple traits at once

```r
# Step 1, Option 2, SPAmix
# Fit one null model and get its residuals
res_cox <- coxph(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER + PC1 + PC2,
  data = PhenoData
)$residuals

# Fit another null model and get its residuals
res_lm <- lm(
  QuantPheno ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData
)$residuals

# Calculate parameters needed for step 2
obj.SPAmix <- GRAB.NullModel(
  res_cox + res_lm ~ AGE + GENDER + PC1 + PC2,
  data = PhenoData,
  subjIDcol = "IID",
  method = "SPAmix",
  traitType = "Residual",
  control = list(PC_columns = "PC1,PC2")
)
```

##### Null Object Components

- `N`: Sample size
- `resid`: Residual matrix (n × k phenotypes)
- `yVec`: Response variable (event indicator)
- `PCs`: Selected principal components
- `nPheno`: Number of phenotypes
- `outLierList`: List with per-phenotype outlier info
  - `posOutlier`, `posNonOutlier`: Subject indices
  - `residOutlier`, `residNonOutlier`: Stratified residuals

---

## Step 2: Association Testing

Refer to `?GRAB.Marker`, `?GRAB.SPACox`, and `?GRAB.SPAmix` for detailed parameter instructions. A quick example is provided below.

### SPACox

```r
# Step 2, SPACox
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile = file.path(tempdir(), "Results_SPACox.txt")

# Marker-level testing
GRAB.Marker(obj.SPACox, GenoFile = GenoFile, OutputFile = OutputFile)

# Read results
head(data.table::fread(OutputFile))
```

**Output Columns**

- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion missing
- `Pvalue`: Association p-value
- `zScore`: Test statistic

### SPAmix

```r
# Step 2, SPAmix
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile = file.path(tempdir(), "Results_SPAmix.txt")

# Marker-level testing
GRAB.Marker(obj.SPAmix, GenoFile = GenoFile, OutputFile = OutputFile)

# Read results
head(data.table::fread(OutputFile))
```

**Output Columns**

- `Pheno`: Phenotype identifier (pheno_1, pheno_2, ...)
- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion missing
- `Pvalue`: Association p-value
- `zScore`: Test statistic
