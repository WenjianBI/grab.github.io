---
layout: default
title: Ordinal Categorical
parent: Two-Step GWAS Framework
nav_order: 1
---


**POLMM** (Proportional Odds Logistic Mixed Model) implements association tests for **ordinal categorical phenotypes** while accounting for sample relatedness.

**Key Features:**

- Handles unbalanced phenotypic distributions
- More powerful than treating ordinal traits as binary or quantitative
- Support both dense GRM and sparse GRM to adjust for sample relatedness
- Accurate p-values for low-frequency and rare variants
- Scales to biobank-size datasets
- Support both single-variant analysis and set-based analysis

> **Citation:**
>
> Bi *et al.* (2021). Efficient mixed model approach for large-scale genome-wide association studies of ordinal categorical phenotypes. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2021.03.019](https://doi.org/10.1016/j.ajhg.2021.03.019)

---

**POLMM-GENE** extends POLMM to perform set-based association tests for rare variants in genomic regions (e.g., genes). It is particularly powerful for exome sequencing data.

**Key Features:**

- SKAT, SKAT-O, and Burden tests
- Annotation-based variant grouping
- Cauchy combination for multiple tests
- Handles ultra-rare variants (MAC < threshold)

> **Citation:**
>
> Bi *et al.* (2023). Scalable mixed model methods for set-based association studies on large-scale categorical data analysis and its application to exome-sequencing data in UK Biobank. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2023.03.010](https://doi.org/10.1016/j.ajhg.2023.03.010)

---

## Step 1: Model Fitting and Preprocessing

This step is shared by both marker- and region-level analyses and returns the `obj.POLMM` object required for step two. See `?GRAB.NullModel` and `?GRAB.POLMM` for detailed parameter instructions. A quick example is provided below.

```r
# Load data
GenoFile <- system.file("extdata", "simuPLINK.bed", package = "GRAB")
SparseGRMFile <- system.file("extdata", "SparseGRM.txt", package = "GRAB")
OutputFile <- file.path(tempdir(), "resultPOLMMmarker.txt")

PhenoFile <- system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData <- data.table::fread(PhenoFile, header = TRUE)
PhenoData$OrdinalPheno <- factor(PhenoData$OrdinalPheno, levels = c(0, 1, 2))

# Step 1
obj.POLMM <- GRAB.NullModel(
 OrdinalPheno ~ AGE + GENDER,
 data = PhenoData,
 subjIDcol = "IID",
 method = "POLMM",
 traitType = "ordinal",
 GenoFile = GenoFile,
 SparseGRMFile = SparseGRMFile
)                                
```

**Notes:**

- `GenoFile` is always required for POLMM to estimate the variance ratio parameter
- If `SparseGRMFile` is not provided, a dense GRM is calculated from `GenoFile`; if given, the sparse GRM is used for model fitting
- Phenotype must be coded as an ordered factor
- Missing phenotypes must be coded as `NA`
- `obj.POLMM` contains the data structure for step 2

### `obj.POLMM` Components

The POLMM null model object contains:

- `N`: Sample size
- `subjData`: Subject IDs
- `M`: Number of ordinal categories
- `tau`: Variance component estimate
- `beta`: Covariate effect estimates
- `eps`: Threshold parameters
- `bVec`: Random effect estimates
- `eta`: Linear predictor
- `yVec`: Phenotype matrix (1 col)
- `Cova`: Design matrix
- `muMat`: Fitted probabilities
- `YMat`: Category indicator matrix

---

## Step 2 of Marker-Level Analysis

Refer to `?GRAB.Marker` and `?GRAB.POLMM` for detailed parameter instructions. A quick example is provided below.

```r
# Step 2
GRAB.Marker(obj.POLMM, GenoFile, OutputFile,
            control = list(ifOutGroup = TRUE))

# View results
head(data.table::fread(OutputFile))
```

**Output Columns:**

Standard columns:

- `Marker`: Variant identifier (rsID or CHR:POS:REF:ALT)
- `Info`: Variant information (CHR:POS:REF:ALT format)
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion of missing genotypes
- `Pvalue`: Association p-value
- `beta`: Effect size estimate (log-odds scale)
- `seBeta`: Standard error of beta
- `zScore`: Z-score from score test

Additional columns (`ifOutGroup = TRUE`):

- `AltFreqInGroup.1`, `AltFreqInGroup.2`, ...: Allele frequency in each ordinal category
- `AltCountsInGroup.1`, `AltCountsInGroup.2`, ...: Allele counts in each ordinal category
- `nSamplesInGroup.1`, `nSamplesInGroup.2`, ...: Sample size in each ordinal category

---

## Step 2 of Region-Level Analysis

Refer to `?GRAB.Region` and `?GRAB.POLMM.Region` for detailed parameter instructions. A quick example is provided below. The `obj.POLMM` object generated in [Step 1: Model Fitting and Preprocessing](#step-1-model-fitting-and-preprocessing) is needed.

```r
# Load data
GenoFileRV <- system.file("extdata", "simuPLINK_RV.bed", package = "GRAB")
SparseGRMFile <- system.file("extdata", "SparseGRM.txt", package = "GRAB")
GroupFile <- system.file("extdata", "simuPLINK_RV.group", package = "GRAB")
OutputFile <- file.path(tempdir(), "resultPOLMMregion.txt")

# Step 2
GRAB.Region(obj.POLMM, GenoFileRV, OutputFile,
  GroupFile = GroupFile,
  SparseGRMFile = SparseGRMFile,
  MaxMAFVec = "0.01,0.005"
)

# View results
head(data.table::fread(OutputFile))
head(data.table::fread(paste0(OutputFile, ".markerInfo")))
```

### Output Files

Region-level analysis generates four output files:

**1. Main results (`OutputFile`):**

- `Region`: Gene/region identifier
- `nMarkers`: Number of rare variants included (MAC ≥ threshold)
- `nMarkersURV`: Number of ultra-rare variants (MAC < threshold)
- `Anno.Type`: Annotation category
- `MaxMAF.Cutoff`: MAF cutoff used
- `pval.SKATO`: SKAT-O p-value
- `pval.SKAT`: SKAT p-value
- `pval.Burden`: Burden test p-value

**2. Marker info (`.markerInfo`):**
Detailed statistics for rare variants included in region tests:

- `Region`, `ID`, `Info`, `Anno`
- `AltFreq`, `MAC`, `MAF`, `MissingRate`
- `StatVec`: Score statistic
- `altBetaVec`: Effect size estimate
- `seBetaVec`: Standard error
- `pval0Vec`: Unadjusted p-value
- `pval1Vec`: SPA-adjusted p-value

**3. Other marker info (`.otherMarkerInfo`):**
Information for excluded markers (ultra-rare or failed QC)

**4. Burden summaries (`.infoBurdenNoWeight`):**
Summary statistics for burden tests without variant weights
