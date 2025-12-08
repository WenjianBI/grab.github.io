---
layout: default
title: Parameter Reference
parent: Two-Step GWAS Framework
nav_order: 4
---

**Overview:** this page provides a comprehensive reference for all parameters and options available in GWAS functions, including [GRAB.NullModel()](#grab-nullmodel-for-step-1), [SPAGRM.NullModel()](#spagrm-nullmodel-for-step-1), [GRAB.Marker()](#grab-marker-for-step-2), and [GRAB.Region()](#grab-region-for-step-2).

---

# `GRAB.NullModel()` for Step 1 {#grab-nullmodel-for-step-1}

## Required Parameters for `GRAB.NullModel()`

| Parameter | Type | Description |
|-----------|------|-------------|
| `formula` | formula | Formula with response variable(s) on left and covariates on right. Intercept added automatically. For SPAmix with `traitType="Residual"`, multiple response variables separated by `+` are supported. |
| `data` | data.frame | Data frame containing response variables and covariates. Missing values coded as `NA`. |
| `method` | character | Method to use: `"POLMM"`, `"SPACox"`, `"SPAmix"`, `"SPAGRM"`, or `"WtCoxG"`. |
| `traitType` | character | Trait type: `"ordinal"`, `"time-to-event"`, or `"Residual"`. |
| `GenoFile`¹ | character | Path to genotype file. Required for POLMM and WtCoxG. Extensions: `.bed` (PLINK) or `.bgen` (BGEN v1.2 with 8-bit). |
| `subjIDcol`² | character | Column name in `data` containing subject IDs. |
| `subjData`²  | character or numeric vector | Vector of subject IDs aligned with the rows of `data`. |
| `RefAfFile`³ | character | Path to reference allele frequency file with columns: `CHROM`, `POS`, `ID`, `REF`, `ALT`, `AF_ref`, `AN_ref`. |
| `RefPrevalence`³ | numeric | Population-level disease prevalence for weighting. Range: (0, 0.5). |

¹ `GenoFile` is required only for POLMM and WtCoxG.

² Exactly one of `subjIDcol` and `subjData` must be provided.

³ `RefAfFile` and `RefPrevalence` are WtCoxG specific.

## Optional Parameters for `GRAB.NullModel()`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `GenoFileIndex` | character vector | `NULL` | Index files for genotype file. Auto-detected if `NULL`. For PLINK: `c("prefix.bim", "prefix.fam")`. For BGEN: `c("prefix.bgen.bgi", "prefix.sample")`. |
| `SparseGRMFile` | character | `NULL` | Path to sparse GRM file. Tab-delimited with 3 columns: `ID1`, `ID2`, `Value`. |
| `control` | list | `NULL` | Method-specific control parameters (see below). |

## Control Parameters for `GRAB.NullModel()`

### POLMM Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `memoryChunk` | numeric | 2 | > 0 | Memory chunk size for computation (GB). |
| `seed` | integer | -1 | any | Random seed (-1 = no seed set). |
| `tracenrun` | integer | 30 | > 0 | Number of runs for trace calculation. |
| `maxiter` | integer | 100 | > 0 | Maximum iterations for model fitting. |
| `tolBeta` | numeric | 0.001 | > 0 | Convergence tolerance for beta estimates. |
| `tolTau` | numeric | 0.002 | > 0 | Convergence tolerance for tau estimates. |
| `tau` | numeric | 0.2 | > 0 | Initial variance component value. |
| `maxiterPCG` | integer | 100 | > 0 | Maximum iterations for preconditioned conjugate gradient (PCG). |
| `tolPCG` | numeric | 1e-6 | > 0 | Tolerance for PCG convergence. |
| `showInfo` | logical | `FALSE` | `TRUE`/`FALSE` | Print PCG iteration information for debugging. |
| `maxiterEps` | integer | 100 | > 0 | Maximum iterations for epsilon estimation. |
| `tolEps` | numeric | 1e-10 | > 0 | Tolerance for epsilon estimation. |
| `minMafVarRatio` | numeric | 0.1 | [0, 0.5] | Minimum MAF for variance ratio estimation. |
| `maxMissingVarRatio` | numeric | 0.1 | [0, 1] | Maximum missing rate for variance ratio estimation. |
| `nSNPsVarRatio` | integer | 20 | > 0 | Number of SNPs for variance ratio estimation. |
| `CVcutoff` | numeric | 0.0025 | > 0 | Coefficient of variation cutoff. |
| `grainSize` | integer | 1 | > 0 | Grain size for parallel processing. |
| `minMafGRM` | numeric | 0.01 | [0, 0.5] | Minimum MAF for GRM construction. |
| `maxMissingGRM` | numeric | 0.1 | [0, 1] | Maximum missing rate for GRM construction. |

### SPACox Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `range` | numeric vector | `c(-100, 100)` | symmetric | Range for SPA grid. Must satisfy `range[2] = -range[1]`. |
| `length.out` | integer | 10000 | > 1000 | Number of grid points for SPA approximation. |

### SPAmix Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `PC_columns` | character | **Required** | comma-separated | Column names of principal components (e.g., `"PC1,PC2,PC3"`). Must be in formula. |
| `OutlierRatio` | numeric | 1.5 | > 0 | IQR multiplier for outlier detection. Outliers defined as outside `[Q1 - r×IQR, Q3 + r×IQR]`. |

### WtCoxG Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `OutlierRatio` | numeric | 1.5 | > 0 | IQR multiplier for outlier detection. |

---

# `SPAGRM.NullModel()` for Step 1 {#spagrm-nullmodel-for-step-1}

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `ResidMatFile` | data.frame or character | Data frame or file path with columns `SubjID`, `Resid`. Tab/space-delimited if file. |
| `SparseGRMFile` | character | Path to sparse GRM file. Tab-delimited with 3 columns: `ID1`, `ID2`, `Value`. |
| `PairwiseIBDFile` | character | Path to pairwise IBD file. Tab-delimited with 5 columns: `ID1`, `ID2`, `pa`, `pb`, `pc`. |

## Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `MaxQuantile` | numeric | 0.75 | [0, 1] | Upper quantile for outlier detection. |
| `MinQuantile` | numeric | 0.25 | [0, 1] | Lower quantile for outlier detection. |
| `OutlierRatio` | numeric | 1.5 | > 0 | IQR multiplier for outlier bounds. |
| `ControlOutlier` | logical | `TRUE` | `TRUE`/`FALSE` | Adjust ratio to keep outliers < 5%. Set `FALSE` for higher accuracy. |
| `MaxNuminFam` | integer | 5 | ≥ 2 | Maximum family size for graph decomposition. |
| `MAF_interval` | numeric vector | `c(0.0001, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5)` | increasing | MAF breakpoints for Chow-Liu tree construction. |

---

# `GRAB.Marker()` for Step 2 {#grab-marker-for-step-2}

## Required Parameters for `GRAB.Marker()`

| Parameter | Type | Description |
|-----------|------|-------------|
| `objNull` | S3 object | Null model from `GRAB.NullModel()`, `SPAGRM.NullModel()`, or `SAGELD.NullModel()`. Supported classes: `POLMM_NULL_Model`, `SPACox_NULL_Model`, `SPAmix_NULL_Model`, `SPAGRM_NULL_Model`, `SAGELD_NULL_Model`, `WtCoxG_NULL_Model`. |
| `GenoFile` | character | Path to genotype file. Extensions: `.bed` (PLINK) or `.bgen` (BGEN v1.2). |
| `OutputFile` | character | Path for saving association results. |

## Optional Parameters for `GRAB.Marker()`

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `GenoFileIndex` | character vector | `NULL` | Index files for genotype file. Auto-detected if `NULL`. |
| `OutputFileIndex` | character | `NULL` | Progress tracking file path. Default: `paste0(OutputFile, ".index")`. Enables restart if interrupted. |
| `control` | list | `NULL` | Control parameters (see below). |

## Control Parameters for `GRAB.Marker()`

### Common Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `AlleleOrder` | character | `NULL` | `"ref-first"`, `"alt-first"`, `NULL` | Allele order in genotype file. Default: `"alt-first"` for BGEN, `"ref-first"` for PLINK. |
| `impute_method` | character | `"mean"` | `"mean"`, `"minor"`, `"drop"` | Imputation method for missing genotypes in C++ backend. |
| `missing_cutoff` | numeric | 0.15 | [0, 0.5] | Exclude markers with missing rate above this threshold. |
| `min_maf_marker` | numeric | 0.001 | [0, 0.1] | Exclude markers with MAF below this threshold. |
| `min_mac_marker` | numeric | 20 | [0, 100] | Exclude markers with MAC below this threshold. |
| `nMarkersEachChunk` | integer | 10000 | [1000, 100000] | Number of markers processed per chunk. |
| `SPA_Cutoff` | numeric | 2 | > 0 | Z-score cutoff for saddlepoint approximation. SPA used when z-score > cutoff. |
| `AllMarkers` | logical | `TRUE` | N/A | Analyze all markers. Automatically `FALSE` if include/exclude files provided. |
| `IDsToIncludeFile`¹ | character | `NULL` | N/A | File with marker IDs to include (one per line). |
| `RangesToIncludeFile`¹ | character | `NULL` | N/A | File with genomic ranges to include. Can combine with `IDsToIncludeFile` (union used). |
| `IDsToExcludeFile`¹ | character | `NULL` | N/A | File with marker IDs to exclude (one per line). |
| `RangesToExcludeFile`¹ | character | `NULL` | N/A | File with genomic ranges to exclude. Can combine with `IDsToExcludeFile` (union excluded). |

¹ Cannot use both include and exclude files simultaneously.

### POLMM-Specific Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ifOutGroup` | logical | `FALSE` | Output group-specific statistics (allele frequency, counts, sample size per ordinal category). Adds columns: `AltFreqInGroup.*`, `AltCountsInGroup.*`, `nSamplesInGroup.*`. |

### SPACox-Specific Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pVal_covaAdj_Cutoff` | numeric | 5e-05 | P-value cutoff for covariate adjustment. |

### SPAmix-Specific Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `dosage_option` | character | `"rounding_first"` | `"rounding_first"`, `"rounding_last"` | Dosage handling method. |

### SPAGRM-Specific Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `zeta` | numeric | 0 | SPA moment approximation parameter. |
| `tol` | numeric | 1e-5 | Numerical tolerance for SPA convergence. |

### WtCoxG-Specific Control Parameters

| Parameter | Type | Default | Range | Description |
|-----------|------|---------|-------|-------------|
| `cutoff` | numeric | 0.1 | [0, 1] | Batch effect test p-value cutoff. Variants with p-value below cutoff excluded from association testing. |

---

# `GRAB.Region()` for Step 2 {#grab-region-for-step-2}

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `objNull` | S3 object | Null model from `GRAB.NullModel()`. Currently supports `POLMM_NULL_Model`. |
| `GenoFile` | character | Path to genotype file (PLINK or BGEN). |
| `OutputFile` | character | Path for saving region-based results. |
| `GroupFile` | character | Path to region definition file with region-marker mappings and annotations. Tab-separated with 2-3 columns per region. |

## Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `GenoFileIndex` | character vector | `NULL` | Index files. Auto-detected if `NULL`. |
| `OutputFileIndex` | character | `NULL` | Progress tracking file. Default: `paste0(OutputFile, ".index")`. |
| `SparseGRMFile` | character | `NULL` | Path to sparse GRM file. |
| `MaxMAFVec` | character | `"0.01,0.001,0.0005"` | Comma-separated MAF cutoffs for variant inclusion. |
| `annoVec` | character | `"lof,lof:missense,lof:missense:synonymous"` | Comma-separated annotation groups for analysis. |
| `control` | list | `NULL` | Control parameters (see below). |

## Control Parameters

### Common Control Parameters

| Parameter | Type | Default | Range/Options | Description |
|-----------|------|---------|---------------|-------------|
| `impute_method` | character | `"minor"` | `"mean"`, `"minor"`, `"drop"` | Imputation method for missing genotypes. |
| `missing_cutoff` | numeric | 0.15 | [0, 0.5] | Exclude markers with missing rate above threshold. |
| `min_mac_region` | numeric | 5 | ≥ 0 | Minimum MAC threshold. Markers with MAC < threshold treated as ultra-rare. |
| `max_markers_region` | integer | 100 | ≥ 50 | Maximum markers allowed per region. |
| `r.corr` | numeric vector | `c(0, 0.1^2, 0.2^2, 0.3^2, 0.4^2, 0.5^2, 0.5, 1)` | [0, 1] | Rho parameters for SKAT-O test. |
| `weights.beta` | numeric vector | `c(1, 25)` | ≥ 0, length 2 | Beta distribution parameters for variant weights. |
| `min_nMarker` | integer | 3 | > 0 | Minimum markers required for region analysis. |
| `SPA_Cutoff` | numeric | 2 | > 0 | Z-score cutoff for SPA. |

### POLMM-Specific Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `showInfo` | logical | `FALSE` | Print PCG iteration information for debugging. |
| `tolPCG` | numeric | 0.001 | Tolerance for PCG in region testing. |
| `maxiterPCG` | integer | 100 | Maximum PCG iterations in region testing. |
