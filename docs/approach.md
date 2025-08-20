---
layout: default
title: Genome-wide association studies
nav_order: 4
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: true
has_toc: true
---

# Genome-wide Association Studies

The **GRAB** package provides a generic framework to analyze a wide variety of phenotypes.

## Quick Start-up Examples

Here is a quick tutorial for GWAS of a time-to-event trait using SPAmix.

### Step 1: fit a null model

```r
library(GRAB)
PhenoFile <- system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData <- data.table::fread(PhenoFile, header = TRUE)

obj.SPAmix <- GRAB.NullModel(
  survival::Surv(SurvTime, SurvEvent) ~ AGE + GENDER + PC1 + PC2,
  data = PhenoData,
  subjData = IID,
  method = "SPAmix",
  traitType = "time-to-event",
  control = list(PC_columns = "PC1,PC2")
)
```

### Step 2: conduct score test

```r
GenoFile <- system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputFile <- file.path(tempdir(), "Results_SPAmix.txt")

GRAB.Marker(
  objNull = obj.SPAmix,
  GenoFile = GenoFile,
  OutputFile = OutputFile,
  control = list(outputColumns = "zScore")
)
data.table::fread(OutputFile)
```

## Step 1: Choose `traitType` and `method`

Arguments `method` and `traitType` specify the type of phenotype data and the analysis approach. Currently, `GRAB.NullModel()` supports the following combinations:

| method                | traitType        | Related subjects | Other features                                                         |
|:----------------------|:-----------------|:-----------------|:-----------------------------------------------------------------------|
| `POLMM`, `POLMM-GENE` | `ordinal`        | Yes              | POLMM-GENE is a variant-set-based test                                 |
| `SPACox`, `SPAmix`    | `time-to-event`  | No               | SPAmix is designed for admixed population using individual-specific AF |
| `WtCoxG`              | `time-to-event`  | Yes              | WtCoxG boosts power using reference population AF                      |

## Step 2: Choose Dense GRM or Sparse GRM

Both dense GRM and sparse GRM are supported in the `GRAB` package to adjust for family relatedness, which can prevent inflated type I error rates.

| GRM Type   | Advantages     | Disadvantages   | Required arguments |
|:----------:|:--------------:|:---------------:|:------------------:|
| Dense GRM  | More powerful  | Slow            | `GenoFile`         |
| Sparse GRM | Fast           | Less powerful   | `SparseGRMFile`    |

**NOTE:** Extensive simulation results suggest that for binary and ordinal categorical data analysis, dense and sparse GRM perform similarly in terms of both type I error rates and statistical power.

## Note About the `control` Argument

The `control` argument specifies a list of parameters for controlling the fitting and association testing process.
