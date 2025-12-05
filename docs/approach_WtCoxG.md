---
layout: default
title: WtCoxG
nav_order: 3
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
parent: Genome-wide association studies
has_children: false
has_toc: false
---

# WtCoxG

WtCoxG is a Cox-based association test method for time-to-event traits that addresses case ascertainment bias (cases are enriched compared to general population) commonly present in biobank data. It uses weighted Cox regression with external allele frequencies to boost statistical power.

**Key Features:**

- Corrects for case ascertainment bias in biobank studies
- Leverages external MAF from public resources (e.g., gnomAD)
- Incorporates sample relatedness via sparse GRM
- Performs batch effect QC between study cohort and reference population
- Saddlepoint approximation (SPA) provides accurate p-values, especially for rare variants and extreme case-control ratios

**Citation:**
Li *et al.* (2025). High-powered, robust, and versatile survival analysis via weighted Cox regression. *Nature Computational Science*. [doi:10.1038/s43588-025-00864-z](https://doi.org/10.1038/s43588-025-00864-z)

---

## Step 1: Model Fitting and Preprocessing

Refer to `?GRAB.NullModel` and `?GRAB.WtCoxG` for detailed parameter instructions. A quick example is provided below.

```r
# Load files
PhenoFile <- system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData <- data.table::fread(PhenoFile, header = TRUE)
SparseGRMFile <- system.file("extdata", "SparseGRM.txt", package = "GRAB")
GenoFile <- system.file("extdata", "simuPLINK.bed", package = "GRAB")
RefAfFile <- system.file("extdata", "simuRefAf.txt", package = "GRAB")
OutputFile <- file.path(tempdir(), "resultWtCoxG.txt")

# Fit null model and test batch effects
obj.WtCoxG <- GRAB.NullModel(
  survival::Surv(SurvTime, SurvEvent) ~ AGE + GENDER,
  data = PhenoData,
  subjIDcol = "IID",
  method = "WtCoxG",
  traitType = "time-to-event",
  GenoFile = GenoFile,
  SparseGRMFile = SparseGRMFile,
  RefAfFile = RefAfFile,
  RefPrevalence = 0.1
)
```

**WtCoxG specific mandatory parameters:**

- `RefAfFile`: Reference allele frequency file (see format below)
- `RefPrevalence`: Population disease prevalence (0 < p < 0.5)
- `obj.WtCoxG` contains the data structure for step 2

### `RefAfFile` Format

The reference allele frequency file must be whitespace-delimited and include the following columns:

- **CHROM**: Chromosome
- **POS**: Position
- **ID**: Variant identifier
- **REF**: Reference allele
- **ALT**: Alternative allele
- **AF_ref**: Allele frequency in reference population
- **AN_ref**: Allele number in reference population

**Example:**

```
CHROM   POS      ID          REF  ALT  AF_ref   AN_ref
1       10177    rs367896724  A    C   0.4258   251390
1       10235    rs540538026  T    A   0.0009   251306
1       10352    rs555500075  T    A   0.4104   251480
```

### `obj.WtCoxG` Components

- `N`: Sample size
- `subjData`: Subject IDs
- `mresid`: Martingale residuals from weighted Cox model
- `Cova`: Design matrix
- `yVec`: Event indicator
- `weight`: Observation weights
- `RefPrevalence`: Reference population prevalence
- `outLierList`: Outlier information for SPA
  - `posOutlier`: Outlier subject indices (0-based)
  - `posNonOutlier`: Non-outlier subject indices (0-based)
  - `resid`, `resid2`: Residuals and squared residuals
  - `residOutlier`, `residNonOutlier`: Stratified residuals
- `mergeGenoInfo`: Data frame with marker-level QC results
  - Allele frequencies (study vs. reference)
  - Batch effect p-values
  - Estimated parameters (TPR, $\sigma^2$, weights)
  - Variance ratios

---

## Step 2: Association Testing

Refer to `?GRAB.Marker` and `?GRAB.WtCoxG` for detailed parameter instructions. A quick example is provided below.

```r
# Marker-level testing
GRAB.Marker(obj.WtCoxG, GenoFile, OutputFile)

# View results
head(data.table::fread(OutputFile))
```

### Output Columns

- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency in study
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion of missing genotypes
- `WtCoxG.ext`: P-value using external MAF
- `WtCoxG.noext`: P-value without external MAF
- `AF_ref`: Reference allele frequency
- `AN_ref`: Reference allele number
- `pvalue_bat`: Batch effect test p-value
- `TPR`: True positive rate estimate
- `sigma2`: Variance parameter estimate
- `w.ext`: Optimal external weight
- `var.ratio.w0`: Variance ratio for internal analysis
- `var.ratio.int`: Variance ratio adjustment
- `var.ratio.ext`: Variance ratio with external MAF

**Key columns for interpretation:**

- Use `WtCoxG.ext` as primary p-value (more powerful)
- Use `WtCoxG.noext` if `pvalue_bat < cutoff` (batch effect detected)
- Check `pvalue_bat` for quality control
