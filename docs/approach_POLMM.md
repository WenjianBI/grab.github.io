---
layout: default
title: POLMM / POLMM-GENE
nav_order: 2
description: "POLMM approaches: ordinal categorical trait analysis."
parent: Genome-wide association studies
has_children: false
has_toc: false
---

# POLMM

POLMM (Proportional Odds Logistic Mixed Model) implements association tests for ordinal categorical phenotypes while accounting for sample relatedness.

**Key Features:**

- Handles unbalanced phenotypic distributions
- More powerful than treating ordinal traits as binary or quantitative
- Support both dense GRM and sparse GRM (recommended) to adjust for sample relatedness
- Accurate p-values for low-frequency and rare variants
- Scales to biobank-size datasets
- Support both single-variant analysis and set-based analysis (Burden tests, SKAT, and SKAT-O)

**Citation:**
**POLMM**, Bi et al. (2021). Efficient mixed model approach for large-scale genome-wide association studies of ordinal categorical phenotypes. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2021.03.019](https://doi.org/10.1016/j.ajhg.2021.03.019)
**POLMM-GENE**, Bi et al. (2023). Scalable mixed model methods for set-based association studies on large-scale categorical data analysis and its application to exome-sequencing data in UK Biobank. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2023.03.010](https://doi.org/10.1016/j.ajhg.2023.03.010)

---

## Step 1: Fit Null Model

See `?GRAB.NullModel` and `?GRAB.POLMM` for detailed parameter instructions. A quick example is provided below.

### Option 1: using a dense GRM

If `SparseGRMFile` is not provided, dense GRM is calculated with `GenoFile`.

```r
# Step 1, Option 1, dense GRM
library(GRAB)
PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = TRUE)
PhenoData$OrdinalPheno <- factor(PhenoData$OrdinalPheno, levels = c(0, 1, 2))

GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
obj.POLMM = GRAB.NullModel(
  factor(OrdinalPheno) ~ AGE + GENDER,
  data = PhenoData, 
  subjData = PhenoData$IID, 
  method = "POLMM", 
  traitType = "ordinal",
  GenoFile = GenoFile,
  control = list(showInfo = FALSE, LOCO = FALSE, tolTau = 0.2, tolBeta = 0.1)
)
```

### Option 2: using a sparse GRM

If a sparse GRM is provided via `SparseGRMFile`, it will be used for model fitting.

```r
# Step 1, Option 2, sparse GRM
SparseGRMFile =  system.file("extdata", "SparseGRM.txt", package = "GRAB")
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
obj.POLMM = GRAB.NullModel(
  formula = OrdinalPheno ~ AGE + GENDER,
  data = PhenoData, 
  subjData = PhenoData$IID, 
  method = "POLMM", 
  traitType = "ordinal",
  GenoFile = GenoFile,
  SparseGRMFile =  SparseGRMFile,
  control = list(showInfo = FALSE, LOCO = FALSE, tolTau = 0.2, tolBeta = 0.1)
)

OutputDir = tempdir()
objPOLMMFile = file.path(OutputDir, "objPOLMMFile.RData")                                      
save(obj.POLMM, file = objPOLMMFile)                                        
```

**Notes:**

- `GenoFile` is always required for POLMM to estimate the variance ratio parameter
- Phenotype must be an ordered factor with levels from lowest to highest
- Missing phenotypes must be coded as `NA`

### Null Object Components

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

## Step 2(a): Marker-Level Analysis

Refer to `?GRAB.Marker` and `?GRAB.POLMM` for detailed parameter instructions. A quick example is provided below.

```r
# Load a precomputed null object to run step 2 without performing step 1
objPOLMMFile = system.file("extdata", "objPOLMMnull.RData", package = "GRAB") 
load(objPOLMMFile)   # read in an R object, obj.POLMM

GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuMarkerOutput.txt")

# Marker-level testing
GRAB.Marker(obj.POLMM, GenoFile = GenoFile, OutputFile = OutputFile,
            control = list(ifOutGroup = TRUE))

# Read results
data.table::fread(OutputFile)
```

### Output Columns

**Standard columns:**

- `Marker`: Variant identifier (rsID or CHR:POS:REF:ALT)
- `Info`: Variant information (CHR:POS:REF:ALT format)
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion of missing genotypes
- `Pvalue`: Association p-value
- `beta`: Effect size estimate (log-odds scale)
- `seBeta`: Standard error of beta
- `zScore`: Z-score from score test

**Additional columns (`ifOutGroup = TRUE`):**

- `AltFreqInGroup.1`, `AltFreqInGroup.2`, ...: Allele frequency in each ordinal category
- `AltCountsInGroup.1`, `AltCountsInGroup.2`, ...: Allele counts in each ordinal category
- `nSamplesInGroup.1`, `nSamplesInGroup.2`, ...: Sample size in each ordinal category

---

## Step 2(b): Set-Based Analysis

Refer to `?GRAB.Region` and `?GRAB.POLMM.Region` for detailed parameter instructions. A quick example is provided below.

POLMM-GENE extends POLMM to perform set-based association tests for rare variants in genomic regions (e.g., genes). It is particularly powerful for exome sequencing data.

**Key Features:**

- SKAT, SKAT-O, and Burden tests
- Annotation-based variant grouping
- Cauchy combination for multiple tests
- Handles ultra-rare variants (MAC < threshold)

```r
# Load a precomputed null object to run step 2 without performing step 1
objPOLMMFile = system.file("extdata", "objPOLMMnull.RData", package = "GRAB")  
load(objPOLMMFile)   # read in an R object, obj.POLMM

GenoFile = system.file("extdata", "simuPLINK_RV.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuRegionOutput.txt")
GroupFile = system.file("extdata", "simuPLINK_RV.group", package = "GRAB")
SparseGRMFile = system.file("extdata", "SparseGRM.txt", package = "GRAB")

# Region-level testing
GRAB.Region(
  objNull = obj.POLMM,
  GenoFile = GenoFile,
  GenoFileIndex = NULL,
  OutputFile = OutputFile,
  OutputFileIndex = NULL,
  GroupFile = GroupFile,
  SparseGRMFile = SparseGRMFile,
  MaxMAFVec = "0.01,0.005"
)

# Read results
data.table::fread(OutputFile)
```

### Group File Format

The group file defines regions and variant annotations (tab-separated):

```
GENE1    var     rs1001  rs1002  rs1003  rs1004
GENE1    anno    lof     missense        missense        synonymous
GENE1    weight  1.5     1.2     1.0     0.8
GENE2    var     rs2001  rs2002
GENE2    anno    lof     lof
```

**Format specifications:**

- Column 1: Region/gene identifier
- Column 2: Row type (`var`, `anno`, or `weight`)
- Columns 3+: Marker IDs, annotations, or weights
- `anno` row: Annotation categories for each variant
- `weight` row (optional): Custom weights for each variant

### Output Files

Region-level analysis generates four output files:

**1. Main results (`results_POLMM_region.txt`):**

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
