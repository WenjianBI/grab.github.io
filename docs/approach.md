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

The following example demonstrates how to use POLMM and POLMM-GENE to analyze ordinal categorical traits.

```r
library(GRAB)
library(dplyr)

PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)
PhenoData = PhenoData %>% mutate(OrdinalPheno = factor(OrdinalPheno, 
                                                       levels = c(0, 1, 2)))

# Step 1: fit a null model
SparseGRMFile = system.file("SparseGRM", "SparseGRM.txt", package = "GRAB")
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
obj.POLMM = GRAB.NullModel(formula = OrdinalPheno ~ AGE + GENDER,
                           data = PhenoData, 
                           subjData = PhenoData$IID, 
                           method = "POLMM", 
                           traitType = "ordinal",
                           GenoFile = GenoFile,
                           SparseGRMFile =  SparseGRMFile,
                           control = list(showInfo = FALSE, 
                                          LOCO = FALSE, 
                                          tolTau = 0.2, 
                                          tolBeta = 0.1))

# Step 2(a): conduct a marker-level association study
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuMarkerOutput.txt")
GRAB.Marker(obj.POLMM, GenoFile = GenoFile,
            OutputFile = OutputFile)

results = data.table::fread(OutputFile)
hist(results$Pvalue)

# Step 2(b): conduct a set-based association study
GenoFile = system.file("extdata", "simuPLINK_RV.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuRegionOutput.txt")
GroupFile = system.file("extdata", "simuPLINK_RV.group", package = "GRAB")
SparseGRMFile = system.file("SparseGRM", "SparseGRM.txt", package = "GRAB")

GRAB.Region(objNull = obj.POLMM,
            GenoFile = GenoFile,
            GenoFileIndex = NULL,
            OutputFile = OutputFile,
            OutputFileIndex = NULL,
            GroupFile = GroupFile,
            SparseGRMFile = SparseGRMFile,
            MaxMAFVec = "0.01,0.005")

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
