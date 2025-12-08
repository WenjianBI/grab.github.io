---
layout: default
title: Residual-Based Methods
parent: Two-Step GWAS Framework
nav_order: 3
---


**Overview:** SPACox, SPAmix, and SPAGRM are **residual‑based methods** for genome‑wide association studies that use residuals from fitted null models together with genotype data to test associations for a wide range of complex traits. They share a common framework: **SPACox** is the baseline method for homogeneous populations of unrelated individuals; **SPAmix** extends SPACox to model **population structure** (e.g., admixed or multi‑population cohorts); **SPAGRM** extends SPACox to account for **sample relatedness**.

**Features of the Methods:**

| Method            | Population Structure | Sample Relatedness | Modeling Approach                  |
|-------------------|----------------------|--------------------|------------------------------------|
| [SPACox](#spacox) | Not                  | Not                | Residuals random                   |
| [SPAmix](#spamix) | Modeled              | Not                | Genotypes random                   |
| [SPAGRM](#spagrm) | Not                  | Modeled            | Genotypes random                   |

All three methods implement the saddlepoint approximation (SPA), making them robust and accurate for common, low‑frequency, and rare variants, including cases where phenotype or residual distributions are highly unbalanced. To apply these three methods, the residuals must satisfy the following conditions:

$$
\sum_{i=1}^n X_{ij} R_i = 0 \quad \text{for each } j, \quad \text{and} \quad \sum_{i=1}^n R_i = 0
$$

where $R_i$ is the residual for subject $i$, and $X_{ij}$ is the covariate $j$ for subject $i$.

---

# SPACox

**SPACox** uses an empirical cumulant generating function (CGF) to perform SPA-based single-variant association tests, enabling analysis with residuals from any null model.

> **Citations:**
>
> Bi *et al.* (2020). Fast and accurate method for genome-wide time-to-event data analysis and its application to UK Biobank. *American Journal of Human Genetics*. [doi:10.1016/j.ajhg.2020.06.003](https://doi.org/10.1016/j.ajhg.2020.06.003)

## Step 1: Model Fitting and Preprocessing

In `GRAB.NullModel`, specify `traitType = "Residual"` for residual-based methods. A quick example is provided below. Refer to `?GRAB.NullModel` and `?GRAB.SPACox` for detailed parameter instructions.

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

SPAmix performs retrospective single-variant association tests using genotypes and residuals from null models of any complex trait in large-scale biobanks. It extends SPACox to support complex population structures, such as admixed ancestry and multiple populations, but does not account for sample relatedness.

> **Citation:**
>
> Ma *et al.* (2025). Sparse estimation of high-dimensional genetic correlation and its application to global biobank meta-analysis. *Genome Biology*. [doi:10.1186/s13059-025-03827-9](https://doi.org/10.1186/s13059-025-03827-9)

## Step 1: Model Fitting and Preprocessing

A quick example is shown below. See `?GRAB.NullModel` and `?GRAB.SPAmix` for full parameter details. In `GRAB.NullModel`:

- Set `traitType = "Residual"`.
- Provide `control$PC_columns` as a comma-separated list of SNP-derived PC column names (e.g., `"PC1,PC2"`) — this is required.
- To analyze multiple residuals in one run, place them on the left side of the formula separated by `+` (e.g., `res1 + res2 ~ covariates`); each residual is tested independently, while common preprocessing steps are executed once to save time.

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
  formula = res_cox + res_lm ~ AGE + GENDER + PC1 + PC2,
  data = PhenoData,
  subjIDcol = "IID",
  method = "SPAmix",
  traitType = "Residual",
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

# SPAGRM

SPAGRM is a scalable and accurate framework for retrospective association tests. It treats genetic loci as random vectors and uses a precise approximation of their joint distribution. This enables SPAGRM to handle any type of complex trait, including longitudinal and unbalanced phenotypes. SPAGRM extends SPACox to support sample relatedness.

> **Note**:
>
> Detailed documentation for SPAGRM is available at the [SPAGRM online tutorial](https://hexupku.github.io/SPAGRM.github.io/).
>
> **Citation:**
>
> Xu *et al.* (2025). Scalable and accurate variance component analysis with large sample relatedness. *Nature Communications*. [doi:10.1038/s41467-025-56669-1](https://doi.org/10.1038/s41467-025-56669-1)

## Step 1: Preprocessing

A quick example is provided below. Refer to `?SPAGRM.NullModel` and `?GRAB.SPAGRM` for detailed parameter instructions.

```r
# Load data
ResidMatFile <- system.file("extdata", "ResidMat.txt", package = "GRAB")
SparseGRMFile <- system.file("extdata", "SparseGRM.txt", package = "GRAB")
PairwiseIBDFile <- system.file("extdata", "PairwiseIBD.txt", package = "GRAB")
GenoFile <- system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile <- file.path(tempdir(), "resultSPAGRM.txt")

# Pre-calculate genotype distributions
obj.SPAGRM <- SPAGRM.NullModel(
  ResidMatFile = ResidMatFile,
  SparseGRMFile = SparseGRMFile,
  PairwiseIBDFile = PairwiseIBDFile,
  control = list(ControlOutlier = FALSE)
)
```

### `ResidMatFile` Format

Whitespace-delimited file with two columns:

```
SubjID  Resid
ID001  -0.234
ID002   0.512
ID003  -0.089
ID004   0.157
```

Format specifications:

- Header row required
- `SubjID` must match those in GRM and IBD files
- `Resid` computed from external null models (e.g., `lmer()`, `coxph()`, `glm()`) should have mean ≈ 0

### `PairwiseIBDFile` Format

A pairwise IBD (identical by decent) file must be whitespace-delimited with five columns in the following order:

```
ID1   ID2   pa      pb      pc
f1_5  f1_1  0.0000  0.9296  0.07038
f1_5  f1_2  0.0755  0.8916  0.03285
f1_6  f1_1  0.0000  0.9466  0.05338
```

Format specifications:

- ID1: subject 1 identifier
- ID2: subject 2 identifier
- pa: probability that the pair share both alleles (IBD = 2) at a locus.
- pb: probability that the pair share one allele (IBD = 1) at a locus.
- pc: probability that the pair share no alleles (IBD = 0) at a locus.

See [getPairwiseIBD](https://hexupku.github.io/SPAGRM.github.io/docs/Step%200b%20Calculate%20a%20IBD%20file.html) for details on generating a pairwise IBD file.

## Step 2: Association Testing

Refer to `?GRAB.Marker` and `?GRAB.SPAGRM` for detailed parameter instructions.

```r
# Perform association tests
GRAB.Marker(obj.SPAGRM, GenoFile, OutputFile)

# Read results
head(data.table::fread(OutputFile))
```

**Output Columns:**

- `Marker`: Variant identifier
- `Info`: CHR:POS:REF:ALT
- `AltFreq`: Alternative allele frequency
- `AltCounts`: Alternative allele count
- `MissingRate`: Proportion missing
- `zScore`: Test statistic
- `Pvalue`: Association p-value
- `hwepval`: Hardy-Weinberg equilibrium p-value
