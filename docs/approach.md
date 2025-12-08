---
layout: default
title: Two-Step GWAS Framework
nav_order: 2
has_children: true
---


All GWAS methods in **GRAB** are implemented using the **unified two-step analysis framework**.

## Step 1: Model Fitting and Preprocessing

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

- `obj.null` contains the data structure for step two.
- Refer to `?GRAB.NullModel` for detailed parameter instructions.

### `SparseGRMFile` Format

A sparse GRM file must be whitespace-delimited with three columns in the following order:

```
ID1      ID2     Value
f1_1     f1_2    0.1550
f1_1     f1_3    0.2272
f1_2     f1_3    0.1192
```

Format specifications:

- Column 1: Subject ID 1
- Column 2: Subject ID 2
- Column 3: Genetic correlation between the two subjects

See [getSparseGRM](GRM.md) for details on generating a sparse GRM.

---

## Step 2: Marker-Level Analysis

The second step uses `obj.null` from step one and genotype data to perform association tests for each marker or region. All methods implement the saddlepoint approximation (SPA) to compute p‑values accurately for common, low‑frequency, and rare variants, including when phenotype distributions are highly biased. This step:

- Perform single-variant association tests
- Outputs p-values and related statistics

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

---

## Step 2: Region-Level Analysis

- Variant-set association tests
- Outputs p-values of SKAT, Burden, and SKAT-O tests
- Outputs p-values of single variants and related statistics

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

Format specifications:

- Column 1: Region/gene identifier
- Column 2: Row type (`var`, `anno`, or `weight`)
- Columns 3+: Marker IDs, annotations, or weights
- `anno` row: Annotation categories for each variant
- `weight` row (optional): Custom weights for each variant

---

## Supported Trait Types and Scenarios

GRAB supports the following methods[^1] for different trait types and study designs:

| Trait Type                            | Unrelated & Homogeneous | Related                                  | Population Structure                     |
|---------------------------------------|:-----------------------:|:----------------------------------------:|:----------------------------------------:|
| Binary or quantitative                 | SAIGE                  | SAIGE                                   | [SPAmix]({{ site.baseurl }}{% link docs/approach_Residual.md %}#spamix)    |
| [Ordinal categorical]({{ site.baseurl }}{% link docs/approach_ordinal.md %})        | [POLMM]({{ site.baseurl }}{% link docs/approach_ordinal.md %}) | [POLMM]({{ site.baseurl }}{% link docs/approach_ordinal.md %})[^2] | [SPAmix]({{ site.baseurl }}{% link docs/approach_Residual.md %}#spamix)    |
| [Time-to-event]({{ site.baseurl }}{% link docs/approach_survival.md %}) | [SPACox]({{ site.baseurl }}{% link docs/approach_survival.md %}#spacox) | [WtCoxG]({{ site.baseurl }}{% link docs/approach_survival.md %}#wtcoxg) | [SPAmix]({{ site.baseurl }}{% link docs/approach_survival.md %}#spamix)    |
| [The others]({{ site.baseurl }}{% link docs/approach_Residual.md %})[^3] | [SPACox]({{ site.baseurl }}{% link docs/approach_Residual.md %}#spacox) | [SPAGRM]({{ site.baseurl }}{% link docs/approach_Residual.md %}#spagrm) | [SPAmix]({{ site.baseurl }}{% link docs/approach_Residual.md %}#spamix)    |

[^1]: Methods designed for more complex scenarios also work for simpler ones; when a simpler method suffices, we recommend using it. For binary or quantitative traits in a homogeneous population, consider [SAIGE](https://saigegit.github.io/SAIGE-doc) (not included in GRAB), which is specialized for the scenarios.

[^2]: POLMM supports both marker-lever [(Bi et al, 2021)](https://doi.org/doi:10.1016/j.ajhg.2021.03.019) and region-level [(Bi et al, 2023)](https://doi.org/10.1016/j.ajhg.2023.03.010) analyses.

[^3]: Residual‑based methods provide a generalized framework for GWAS of any trait type [(Bi et al, 2020)](https://doi.org/10.1016/j.ajhg.2020.06.003), and can account for sample relatedness [(Xu et al, 2025)](https://doi.org/10.1038/s41467-025-56669-1) or population structure [(Ma et al, 2025)](https://doi.org/10.1186/s13059-025-03827-9).
