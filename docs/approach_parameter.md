---
layout: default
title: Parameter Reference
parent: Two-Step GWAS Framework
nav_order: 4
---

**Overview:** this page provides a comprehensive reference for all parameters and options available in GWAS functions, including [GRAB.NullModel()](#grab-nullmodel-for-step-1), [SPAGRM.NullModel()](#spagrm-nullmodel-for-step-1), [GRAB.Marker()](#grab-marker-for-step-2), and [GRAB.Region()](#grab-region-for-step-2).

---

# `GRAB.NullModel` {#grab-nullmodel-for-step-1}

## Required Parameters

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

## Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `GenoFileIndex` | character vector | `NULL` | Index files for genotype file. Auto-detected if `NULL`. For PLINK: `c("prefix.bim", "prefix.fam")`. For BGEN: `c("prefix.bgen.bgi", "prefix.sample")`. |
| `SparseGRMFile` | character | `NULL` | Path to sparse GRM file. Tab-delimited with 3 columns: `ID1`, `ID2`, `Value`. |
| `control` | list | `NULL` | Control parameters. See below; optional unless marked as required. |

## Control Parameters (optional unless marked as required)

### POLMM Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `memoryChunk` | numeric | 2 | Memory chunk size for computation (GB); must be > 0. |
| `seed` | integer | -1 | Random seed (-1 = no seed set). |
| `tracenrun` | integer | 30 | Number of runs for trace calculation; must be > 0. |
| `maxiter` | integer | 100 | Maximum iterations for model fitting; must be > 0. |
| `tolBeta` | numeric | 0.001 | Convergence tolerance for beta estimates; must be > 0. |
| `tolTau` | numeric | 0.002 | Convergence tolerance for tau estimates; must be > 0. |
| `tau` | numeric | 0.2 | Initial variance component value; must be > 0. |
| `maxiterPCG` | integer | 100 | Maximum iterations for preconditioned conjugate gradient (PCG); must be > 0. |
| `tolPCG` | numeric | 1e-6 | Tolerance for PCG convergence; must be > 0. |
| `showInfo` | logical | `FALSE` | Print PCG iteration information for debugging. |
| `maxiterEps` | integer | 100 | Maximum iterations for epsilon estimation; must be > 0. |
| `tolEps` | numeric | 1e-10 | Tolerance for epsilon estimation; must be > 0. |
| `minMafVarRatio` | numeric | 0.1 | Minimum MAF for variance ratio estimation; range [0, 0.5]. |
| `maxMissingVarRatio` | numeric | 0.1 | Maximum missing rate for variance ratio estimation; range [0, 1]. |
| `nSNPsVarRatio` | integer | 20 | Number of SNPs for variance ratio estimation; must be > 0. |
| `CVcutoff` | numeric | 0.0025 | Coefficient of variation cutoff; must be > 0. |
| `grainSize` | integer | 1 | Grain size for parallel processing; must be > 0. |
| `minMafGRM` | numeric | 0.01 | Minimum MAF for GRM construction; range [0, 0.5]. |
| `maxMissingGRM` | numeric | 0.1 | Maximum missing rate for GRM construction; range [0, 1]. |

### SPACox Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `range` | numeric vector | `c(-100, 100)` | Range for SPA grid; must be symmetric (i.e., `range[2] = -range[1]`). |
| `length.out` | integer | 10000 | Number of grid points for SPA approximation; must be > 1000. |

### SPAmix Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `PC_columns` | character | **Required** | Comma-separated column names of principal components (e.g., `"PC1,PC2,PC3"`). Must be in formula. |
| `OutlierRatio` | numeric | 1.5 | IQR multiplier for outlier detection; must be > 0. Outliers defined as outside `[Q1 - r×IQR, Q3 + r×IQR]`. |

### WtCoxG Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `OutlierRatio` | numeric | 1.5 | IQR multiplier for outlier detection; must be > 0. |

---

# `SPAGRM.NullModel` {#spagrm-nullmodel-for-step-1}

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `ResidMatFile` | data.frame or character | Data frame or file path with columns `SubjID`, `Resid`. Tab/space-delimited if file. |
| `SparseGRMFile` | character | Path to sparse GRM file. Tab-delimited with 3 columns: `ID1`, `ID2`, `Value`. |
| `PairwiseIBDFile` | character | Path to pairwise IBD file. Tab-delimited with 5 columns: `ID1`, `ID2`, `pa`, `pb`, `pc`. |

## Control Parameters (optional)

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `MaxQuantile` | numeric | 0.75 | Upper quantile for outlier detection; range [0, 1]. |
| `MinQuantile` | numeric | 0.25 | Lower quantile for outlier detection; range [0, 1]. |
| `OutlierRatio` | numeric | 1.5 | IQR multiplier for outlier bounds; must be > 0. |
| `ControlOutlier` | logical | `TRUE` | Adjust ratio to keep outliers < 5%. Set `FALSE` for higher accuracy. |
| `MaxNuminFam` | integer | 5 | Maximum family size for graph decomposition; must be ≥ 2. |
| `MAF_interval` | numeric vector | `c(0.0001, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5)` | MAF breakpoints for Chow-Liu tree construction; must be in increasing order. |

---

# `GRAB.Marker` {#grab-marker-for-step-2}

## Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `objNull` | S3 object | Null model from `GRAB.NullModel()`, `SPAGRM.NullModel()`, or `SAGELD.NullModel()`. Supported classes: `POLMM_NULL_Model`, `SPACox_NULL_Model`, `SPAmix_NULL_Model`, `SPAGRM_NULL_Model`, `SAGELD_NULL_Model`, `WtCoxG_NULL_Model`. |
| `GenoFile` | character | Path to genotype file. Extensions: `.bed` (PLINK) or `.bgen` (BGEN v1.2). |
| `OutputFile` | character | Path for saving association results. |

## Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `GenoFileIndex` | character vector | `NULL` | Index files for genotype file. Auto-detected if `NULL`. |
| `OutputFileIndex` | character | `NULL` | Progress tracking file path. Default: `paste0(OutputFile, ".index")`. Enables restart if interrupted. |
| `control` | list | `NULL` | Control parameters (see below). |

## Control Parameters (optional)

### Common Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `AlleleOrder` | character | `NULL` | Allele order in genotype file: `"ref-first"`, `"alt-first"`, or `NULL`. Default: `"alt-first"` for BGEN, `"ref-first"` for PLINK. |
| `impute_method` | character | `"mean"` | Imputation method for missing genotypes: `"mean"`, `"minor"`, or `"drop"`. |
| `missing_cutoff` | numeric | 0.15 | Exclude markers with missing rate above this threshold; range [0, 0.5]. |
| `min_maf_marker` | numeric | 0.001 | Exclude markers with MAF below this threshold; range [0, 0.1]. |
| `min_mac_marker` | numeric | 20 | Exclude markers with MAC below this threshold; range [0, 100]. |
| `nMarkersEachChunk` | integer | 10000 | Number of markers processed per chunk; range [1000, 100000]. |
| `SPA_Cutoff` | numeric | 2 | Z-score cutoff for saddlepoint approximation; must be > 0. SPA used when z-score > cutoff. |
| `AllMarkers` | logical | `TRUE` | Analyze all markers. Automatically `FALSE` if include/exclude files provided. |
| `IDsToIncludeFile`¹ | character | `NULL` | File with marker IDs to include (one per line). |
| `RangesToIncludeFile`¹ | character | `NULL` | File with genomic ranges to include. Can combine with `IDsToIncludeFile` (union used). |
| `IDsToExcludeFile`¹ | character | `NULL` | File with marker IDs to exclude (one per line). |
| `RangesToExcludeFile`¹ | character | `NULL` | File with genomic ranges to exclude. Can combine with `IDsToExcludeFile` (union excluded). |

¹ Cannot use both include and exclude files simultaneously.

### POLMM Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `ifOutGroup` | logical | `FALSE` | Output group-specific statistics (allele frequency, counts, sample size per ordinal category). Adds columns: `AltFreqInGroup.*`, `AltCountsInGroup.*`, `nSamplesInGroup.*`. |

### SPACox Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `pVal_covaAdj_Cutoff` | numeric | 5e-05 | P-value cutoff for covariate adjustment. |

### SPAmix Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `dosage_option` | character | `"rounding_first"` | Dosage handling method: `"rounding_first"` or `"rounding_last"`. |

### SPAGRM Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `zeta` | numeric | 0 | SPA moment approximation parameter. |
| `tol` | numeric | 1e-5 | Numerical tolerance for SPA convergence. |

### WtCoxG Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `cutoff` | numeric | 0.1 | Batch effect test p-value cutoff; range [0, 1]. Variants with p-value below cutoff excluded from association testing. |

---

# `GRAB.Region` {#grab-region-for-step-2}

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

## Control Parameters (optional)

### Common Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `impute_method` | character | `"minor"` | Imputation method for missing genotypes: `"mean"`, `"minor"`, or `"drop"`. |
| `missing_cutoff` | numeric | 0.15 | Exclude markers with missing rate above threshold; range [0, 0.5]. |
| `min_mac_region` | numeric | 5 | Minimum MAC threshold (≥ 0). Markers with MAC < threshold treated as ultra-rare. |
| `max_markers_region` | integer | 100 | Maximum markers allowed per region; must be ≥ 50. |
| `r.corr` | numeric vector | `c(0, 0.1^2, 0.2^2, 0.3^2, 0.4^2, 0.5^2, 0.5, 1)` | Rho parameters for SKAT-O test; each value in [0, 1]. |
| `weights.beta` | numeric vector | `c(1, 25)` | Beta distribution parameters for variant weights; length 2, each ≥ 0. |
| `min_nMarker` | integer | 3 | Minimum markers required for region analysis; must be > 0. |
| `SPA_Cutoff` | numeric | 2 | Z-score cutoff for SPA; must be > 0. |

### POLMM Control Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `showInfo` | logical | `FALSE` | Print PCG iteration information for debugging. |
| `tolPCG` | numeric | 0.001 | Tolerance for PCG in region testing. |
| `maxiterPCG` | integer | 100 | Maximum PCG iterations in region testing. |
