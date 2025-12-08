---
layout: default
title: Time-to-Event
parent: Two-Step GWAS Framework
nav_order: 2
---


**Overview:** **SPACox** implements an empirical saddlepoint approximation (SPA) for GWAS of **time‑to‑event traits**, providing accurate p‑values for low‑frequency and rare variants. **SPAmix** extends SPACox to handle **population structure**, while **WtCoxG** further accounts for **sample relatedness** and **case ascertainment** and leverages **external allele frequencies** to increase statistical power.

**Features of the Methods:**

| Method | Population structure | Sample relatedness | Case ascertainment |
|--------|----------------------|--------------------|--------------------|
| [SPACox](#spacox) | Not | Not | Not |
| [SPAmix](#spamix) | Modeled | Not | Not |
| [WtCoxG](#wtcoxg) | Not | Modeled | Modeled |

---

# SPACox

SPACox is the baseline method for analyzing unrelated subjects in a homogeneous population.

> **Citations:**
>
> Bi *et al.* (2020). Fast and accurate method for genome-wide time-to-event data analysis and its application to UK Biobank. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2020.06.003](https://doi.org/10.1016/j.ajhg.2020.06.003)

## Step 1: Model Fitting and Preprocessing

A quick example is provided below. Refer to `?GRAB.NullModel` and `?GRAB.SPACox` for detailed parameter instructions.

```r
# Load data
PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)

# Step 1, time-to-event trait, SPACox
obj.SPACox = GRAB.NullModel(
  survival::Surv(SurvTime, SurvEvent) ~ AGE + GENDER, 
  data = PhenoData, 
  subjIDcol = "IID", 
  method = "SPACox", 
  traitType = "time-to-event"
)
```

## Step 2: Association Testing

Refer to `?GRAB.Marker` and `?GRAB.SPACox` for detailed parameter instructions.

```r
# Step 2, SPACox
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile = file.path(tempdir(), "Results_SPACox.txt")

# Marker-level testing
GRAB.Marker(obj.SPACox, GenoFile = GenoFile, OutputFile = OutputFile)

# Read results
head(data.table::fread(OutputFile))
```

**Output Columns:**

- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion missing
- `Pvalue`: Association p-value
- `zScore`: Test statistic

---

# SPAmix

SPAmix extends SPACox to support complex population structures, including admixed ancestry and multiple populations. It does not account for sample relatedness.

> **Citations:**
>
> Ma *et al.* (2025). Sparse estimation of high-dimensional genetic correlation and its application to global biobank meta-analysis. *Genome Biology*. [doi:10.1186/s13059-025-03827-9](https://doi.org/10.1186/s13059-025-03827-9)

## Step 1: Model Fitting and Preprocessing

 `PC_columns` in the `control` list is a comma-separated column names of SNP-derived principal components (e.g., `"PC1,PC2"`) and is required by SPAmix. A quick example is provided below. Refer to `?GRAB.NullModel` and `?GRAB.SPAmix` for detailed parameter instructions.

```r
# Load data
PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)

# Step 1, time-to-event trait, SPAmix
obj.SPAmix = GRAB.NullModel(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData, 
  subjIDcol = "IID", 
  method = "SPAmix", 
  traitType = "time-to-event", 
  control = list(PC_columns = "PC1,PC2")
)
```

### Step 2: Association Testing

Refer to `?GRAB.Marker` and `?GRAB.SPAmix` for detailed parameter instructions.

```r
# Step 2, SPAmix
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile = file.path(tempdir(), "Results_SPAmix.txt")

# Marker-level testing
GRAB.Marker(obj.SPAmix, GenoFile = GenoFile, OutputFile = OutputFile)

# Read results
head(data.table::fread(OutputFile))
```

**Output Columns:**

- `Pheno`: Phenotype identifier (pheno_1, pheno_2, ...)
- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion missing
- `Pvalue`: Association p-value
- `zScore`: Test statistic

---

# WtCoxG

**WtCoxG** is a Cox-based association test for time‑to‑event traits that accounts for sample relatedness and corrects for case ascertainment.

**Key Features:**

- Corrects for case ascertainment bias in biobank studies
- Leverages external MAF from public resources (e.g., gnomAD)
- Incorporates sample relatedness via sparse GRM
- Performs batch effect QC between study cohort and reference population
- Saddlepoint approximation (SPA) provides accurate p-values, especially for rare variants and extreme case-control ratios

> **Citation:**
>
> Li *et al.* (2025). High-powered, robust, and versatile survival analysis via weighted Cox regression. *Nature Computational Science*. [doi:10.1038/s43588-025-00864-z](https://doi.org/10.1038/s43588-025-00864-z)

---

## Step 1: Model Fitting and Preprocessing

A quick example is provided below. Refer to `?GRAB.NullModel` and `?GRAB.WtCoxG` for detailed parameter instructions.

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

### WtCoxG specific mandatory parameters

- `RefAfFile`: Reference allele frequency file (see format below)
- `RefPrevalence`: Population disease prevalence (0 < p < 0.5)
- `obj.WtCoxG` contains the data structure for step 2

### `RefAfFile` Format

The reference allele frequency file is whitespace-delimited with the following columns:

```
CHROM   POS      ID          REF  ALT  AF_ref   AN_ref
1       10177    rs367896724  A    C   0.4258   251390
1       10235    rs540538026  T    A   0.0009   251306
1       10352    rs555500075  T    A   0.4104   251480
```

Format specifications:

- CHROM: Chromosome
- POS: Position
- ID: Variant identifier
- REF: Reference allele
- ALT: Alternative allele
- AF_ref: Allele frequency in reference population
- AN_ref: Allele number in reference population

## Step 2: Association Testing

Refer to `?GRAB.Marker` and `?GRAB.WtCoxG` for detailed parameter instructions.

```r
# Marker-level testing
GRAB.Marker(obj.WtCoxG, GenoFile, OutputFile)

# View results
head(data.table::fread(OutputFile))
```

**Output Columns:**

- `WtCoxG.ext`: P-value using external MAF
- `WtCoxG.noext`: P-value without external MAF
- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency in study
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion of missing genotypes
- `AF_ref`: Reference allele frequency
- `AN_ref`: Reference allele number
- `pvalue_bat`: Batch effect test p-value
- `TPR`: True positive rate estimate
- `sigma2`: Variance parameter estimate
- `w.ext`: Optimal external weight
- `var.ratio.w0`: Variance ratio for internal analysis
- `var.ratio.int`: Variance ratio adjustment
- `var.ratio.ext`: Variance ratio with external MAF
