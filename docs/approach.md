---
layout: default
title: Genome-wide association studies
nav_order: 4
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: true
has_toc: true
---

# GWAS Framework

GRAB provides a unified two-step framework for GWAS in large-scale biobanks. The framework supports multiple statistical methods designed for different trait types and sample and population structures.

## Two-Step Analysis Framework

### Step 1: Fit Null Model

The first step fits a null model using the function `GRAB.NullModel`. This step:

- Fits a null model with only covariates
- Estimates model parameters needed for step two
- Performs once per phenotype

**Basic Syntax:**

```r
obj.null <- GRAB.NullModel(
  formula,                # Phenotype ~ Covariates (without intercept)
  data = phenoData,       # Data frame containing variables in the formula
  subjIDcol = "IID",      # Subject ID column name
  method = "METHOD",      # method name ("POLMM", "SPACox", "SPAmix", or "WtCoxG")
  traitType = "TYPE",     # trait type ("ordinal", "time-to-event", or "Residual")
  GenoFile = "geno.bed",  # Path to PLINK or BGEN genotype file
  SparseGRMFile = "sparseGRM.txt", # Path to a sparse GRM file.
  control = list(),       # List of additional, less commonly used parameters
  ...                     # Additional method-specific parameters.
)
```

**Notes:**

- `obj.null` contains the data required for step 2.
- Refer to `?GRAB.NullModel` for detailed parameter instructions.
- See [getSparseGRM](GRM.md) for details on generating a sparse GRM.

### Step 2: Association Testing

The second step performs association tests using the object from step 1 and genotype data:

#### Marker-Level Analysis

- Single-variant association tests
- Outputs p-values
- Outputs related statistics and marker info

**Basic Syntax:**

```r
# Marker-level testing
GRAB.Marker(
  objNull = obj.null,         # Null model object from Step 1
  GenoFile = "geno.bed",      # Path to PLINK or BGEN genotype file
  OutputFile = "result.txt"   # Output file path
)
```

**Notes:**

- The function returns `NULL` invisibly.
- Results are written to `OutputFile`.
- Refer to `?GRAB.Marker` for detailed parameter instructions.

#### Region-Level Analysis

- Variant-set association tests
- Outputs p-values of SKAT, Burden, and SKAT-O tests
- Outputs p-values of single variants
- Outputs related statistics and marker info

**Basic Syntax:**

```r
GRAB.Region(
  objNull = obj.null,              # Null model object from Step 1
  GenoFile = "geno_rv.bed",        # Genotype file with rare variants
  OutputFile = "result.txt",       # Main result file
  GroupFile = "group.txt",         # File of gene/region definitions
  SparseGRMFile = "sparseGRM.txt", # Sparse GRM file
  MaxMAFVec = "0.01,0.005",        # MAF cutoffs
  annoVec = "lof,missense"         # Annotation categories
)
```

The function returns `NULL` invisibly. Results are saved to four files, including `OutputFile` and related result files.

Refer to `?GRAB.Region` for detailed parameter instructions.

## Supported Methods

GRAB supports the following statistical methods designed for different scenarios:

| Method                                   | Trait Type                  | Analysis Level      | Sample Structure | Population Structure | Key Feature                              |
|-------------------------------------------|-----------------------------|---------------------|------------------|---------------------|------------------------------------------|
| [POLMM](approach_POLMM.md)                | Ordinal categorical         | Marker, Region      | Related          | Homogeneous         | Proportional odds logistic mixed model   |
| [SPACox](approach_Residual.md)            | Any (primarily time-to-event)| Marker             | Unrelated        | Homogeneous         | Empirical distribution of the score statistic |
| [SPAmix](approach_Residual.md)            | Any                         | Marker              | Unrelated        | Admixed             | Individual-specific allele frequencies   |
| [SPAGRM](https://hexupku.github.io/SPAGRM.github.io/) | Any                  | Marker              | Related          | Homogeneous         | Joint distribution of genotypes          |
| [WtCoxG](approach_WtCoxG.md)              | Time-to-event               | Marker              | Related          | Homogeneous         | Reference population allele frequencies  |
