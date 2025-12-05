---
layout: default
title: Genome-wide association studies
nav_order: 4
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: true
has_toc: true
---

# GWAS Framework

All GWAS methods in GRAB are implemented using the unified two-step analysis framework.

## Two-Step Analysis Framework

### Step 1: Model Fitting and Preprocessing

The first step prepares all necessary components before conducting association tests on each marker or region. This step is performed once per phenotype for all markers or regions and includes:

- Fitting a null model with covariates only
- Completing other tasks needed only once for all markers or regions

**Basic Syntax:**

```r
obj.null <- GRAB.NullModel(
  formula,                   # Phenotype ~ Covariates (without intercept)
  data = phenoData,          # Data frame containing variables in the formula
  subjIDcol = "IID",         # Subject ID column name
  method = "METHOD",         # method name ("POLMM", "SPACox", "SPAmix", or "WtCoxG")
  traitType = "TYPE",        # trait type ("ordinal", "time-to-event", or "Residual")
  SparseGRMFile = "GRM.txt", # Path to a sparse GRM file (optional)
  ...                        # Additional method-specific parameters.
)
```

**Notes:**

- `obj.null` contains the data structure for step 2.
- Refer to `?GRAB.NullModel` for detailed parameter instructions.

### `SparseGRMFile` Format

A sparse GRM file must be whitespace-delimited with three columns in the following order:

```
ID1      ID2     Value
f1_1     f1_2    0.1550
f1_1     f1_3    0.2272
f1_2     f1_3    0.1192
```

**Format specifications:**

- **Column 1:** Subject ID 1
- **Column 2:** Subject ID 2
- **Column 3:** Genetic correlation between the two subjects
 
See [getSparseGRM](GRM.md) for details on generating a sparse GRM.

### Step 2: Association Testing

The second step uses `obj.null` and genotype data to perform association tests for each marker or region.

#### Marker-Level Analysis

- Single-variant association tests
- Outputs p-values
- Outputs related statistics and marker info

**Basic Syntax:**

```r
GRAB.Marker(
  objNull = obj.null,        # Null model object from Step 1
  GenoFile = "geno.bed",     # Path to PLINK or BGEN genotype file
  OutputFile = "result.txt", # Output file path
  control = list()           # List of additional parameters (optional)
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
  objNull = obj.null,          # Null model object from Step 1
  GenoFile = "geno.bed",       # Path to PLINK or BGEN genotype file
  OutputFile = "result.txt",   # Main result file
  GroupFile = "group.txt"      # File of gene/region definitions
)
```

**Notes:**

- The function returns `NULL` invisibly.
- Results are saved to four files, including `OutputFile` and related result files.
- Refer to `?GRAB.Region` for detailed parameter instructions.

### `GroupFile` Format

The group file defines regions and variant annotations (tab-separated):

```
GENE1    var     rs1001  rs1002    rs1003    rs1004
GENE1    anno    lof     missense  missense  synonymous
GENE1    weight  1.5     1.2       1.0       0.8
GENE2    var     rs2001  rs2002
GENE2    anno    lof     lof
```

**Format specifications:**

- Column 1: Region/gene identifier
- Column 2: Row type (`var`, `anno`, or `weight`)
- Columns 3+: Marker IDs, annotations, or weights
- `anno` row: Annotation categories for each variant
- `weight` row (optional): Custom weights for each variant

## Supported Methods

GRAB supports the following statistical methods designed for different scenarios:

| Method                                   | Trait Type                  | Analysis Level      | Sample Structure | Population Structure | Key Feature                              |
|-------------------------------------------|-----------------------------|---------------------|------------------|---------------------|------------------------------------------|
| [POLMM](approach_POLMM.md)                | Ordinal categorical         | Marker, Region      | Related          | Homogeneous         | Proportional odds logistic mixed model   |
| [SPACox](approach_Residual.md)            | Any (primarily time-to-event)| Marker             | Unrelated        | Homogeneous         | Empirical distribution of the score statistic |
| [SPAmix](approach_Residual.md)            | Any                         | Marker              | Unrelated        | Admixed             | Individual-specific allele frequencies   |
| [SPAGRM](https://hexupku.github.io/SPAGRM.github.io/) | Any                  | Marker              | Related          | Homogeneous         | Joint distribution of genotypes          |
| [WtCoxG](approach_WtCoxG.md)              | Time-to-event               | Marker              | Related          | Homogeneous         | Reference population allele frequencies  |
