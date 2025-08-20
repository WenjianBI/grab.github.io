---
layout: default
title: POLMM / POLMM-GENE
nav_order: 2
description: "POLMM approaches: ordinal categorical trait analysis."
parent: Genome-wide association studies
has_children: false
has_toc: false
---

# POLMM Approaches

`POLMM` and `POLMM-GENE` are accurate and efficient approaches for associating ordinal categorical traits with single variants and variant sets (e.g., genes), respectively.

## Main Features

`POLMM` and `POLMM-GENE` are:

- designed for ordinal categorical trait analysis
- accurate for unbalanced phenotypic distributions (e.g., sample size proportions across three levels of 30:1:1)
- scalable for large-scale biobank data analysis (e.g., UK Biobank)
- support both dense GRM and sparse GRM (recommended) to adjust for family relatedness
- support both single-variant analysis and set-based analysis (Burden tests, SKAT, and SKAT-O)

## Important Notes Prior to Analysis

- For the function `GRAB.NullModel`, the left side of the `formula` argument should be a factor when fitting a null model in step 1. If the `factor` function is used to convert phenotype to a factor, we highly recommend specifying the `levels` argument explicitly.

- We recommend using sparse GRM to adjust for family relatedness due to its high computational efficiency. Generally, we did not observe power loss compared to using dense GRM.

## Quick Start-up Guide

The following example demonstrates the usage of POLMM approaches.

### First, Read Data and Convert Phenotype to a Factor

```r
library(GRAB)
PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = TRUE)
PhenoData$OrdinalPheno <- factor(PhenoData$OrdinalPheno, levels = c(0, 1, 2))
```

### Step 1(a): If dense GRM is used in model fitting, `GenoFile` is required

```r
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

### Step 1(b): If sparse GRM is used in model fitting, `SparseGRMFile` is required

```r
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

### Step 2(a): Single-variant tests using POLMM

```r
# Load a precomputed example object to perform step 2 without repeating step 1
objPOLMMFile = system.file("extdata", "objPOLMMnull.RData", package = "GRAB") 
load(objPOLMMFile)

GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuMarkerOutput.txt")
GRAB.Marker(obj.POLMM, GenoFile = GenoFile, OutputFile = OutputFile)

data.table::fread(OutputFile)
```

### Step 2(b): Set-based tests using POLMM-GENE

```r
objPOLMMFile = system.file("extdata", "objPOLMMnull.RData", package = "GRAB")  
load(objPOLMMFile)   # read in an R object of "obj.POLMM"

GenoFile = system.file("extdata", "simuPLINK_RV.bed", package = "GRAB")
OutputDir = tempdir()
OutputFile = file.path(OutputDir, "simuRegionOutput.txt")
GroupFile = system.file("extdata", "simuPLINK_RV.group", package = "GRAB")
SparseGRMFile = system.file("extdata", "SparseGRM.txt", package = "GRAB")

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

data.table::fread(OutputFile)
```

## Citation

- POLMM: Bi, Wenjian, Wei Zhou, Rounak Dey, Bhramar Mukherjee, Joshua N. Sampson, and Seunggeun Lee. **Efficient mixed model approach for large-scale genome-wide association studies of ordinal categorical phenotypes.** *The American Journal of Human Genetics* 108, no. 5 (2021): 825-839.

- POLMM-GENE: Bi, Wenjian, Wei Zhou, Peipei Zhang, Yaoyao Sun, Weihua Yue, and Seunggeun Lee. **Scalable mixed model approaches for set-based association studies on large-scale categorical data analysis and its application to 450k exome sequencing data in UK Biobank.** To be submitted.
