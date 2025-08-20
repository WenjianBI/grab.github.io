---
layout: default
title: SPACox / SPAmix / SPAGRM
nav_order: 3
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
parent: Genome-wide association studies
has_children: false
has_toc: false
---

# SPACox / SPAmix / SPAGRM approaches

## Main features

`SPACox`, `SPAmix`, and `SPAGRM` are accurate and efficient approaches to associating complex traits (including but not limited to time-to-event traits) to single-variant.

The three methods use empirical SPA approaches and share the below features.

- Model residuals (whose sum is zero) are needed as input. Users can select appropriate statistical models depending on the type of traits in analysis.
- High computational efficiency to analyze large-scale biobank data with millions of individuals
- High accuracy to analyze common, low-frequency, and rare variants, even if the phenotypic distribution (or residual distribution) is highly unbalanced.

The three methods are different in terms of

- `SPACox` is the basic function to analyze unrelated subjects in a homogeneous population.

- `SPAmix` extends `SPACox` to analyze an admixture population or multiple populations. The method is still only valid to analyze unrelated subjects.

- `SPAGRM` extends `SPACox` to analyze a study cohort in which subjects can be genetically related to each other. The method is still only valid to analyze a homogeneous population.

## Important notes about function `GRAB.NullModel`

- If the left side of argument `formula` is model residual, please specify the argument `traitType = "Residuals"`. Otherwise, the argument `traitType` can be specified based on the type of trait in analysis.

- For method `SPAmix`, the top SNP-derived PC and related information is required in the arguments `formula` and `control`.

- For method `SPAGRM`, arguments `GenoFile`, `GenoFileIndex`, and `SparseGRMFile` are required to characterize the family relatedness.

## Quick Start-up Guide

The below gives examples to demonstrate the usage of `SPACox`, `SPAmix`, and `SPAGRM` approaches.

### Step 1. Read in data and fit a null model

```r
library(GRAB)
library(survival)

PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)

obj.SPACox = GRAB.NullModel(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER, 
  data = PhenoData, 
  subjData = IID, 
  method = "SPACox", 
  traitType = "time-to-event"
)
```

`SPACox` method can also support model residuals as input. The above codes are the same as below.

```r
obj.coxph = coxph(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER, 
  data = PhenoData
)

obj.SPACox = GRAB.NullModel(
  obj.coxph$residuals ~ AGE + GENDER, 
  data = PhenoData, 
  subjData = IID, 
  method = "SPACox", 
  traitType = "Residual"
)
```

`SPAmix` method also support both original trait or model residuals as input. For `SPAmix`, the confounding factors of SNP-derived PCs are required and should be specified in `control`.

```r
PhenoFile = system.file("extdata", "simuPHENO.txt", package = "GRAB")
PhenoData = data.table::fread(PhenoFile, header = T)

obj.SPAmix = GRAB.NullModel(
  Surv(SurvTime, SurvEvent) ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData, 
  subjData = IID, 
  method = "SPAmix", 
  traitType = "time-to-event", 
  control = list(PC_columns = "PC1,PC2")
)
```

The same results can be obtained via using model residuals

```r
obj.coxph = coxph(
  Surv(SurvTime, SurvEvent ) ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData
)

obj.SPACox = GRAB.NullModel(
  obj.coxph$residuals ~ AGE + GENDER + PC1 + PC2, 
  data = PhenoData, 
  subjData = IID, 
  method = "SPAmix", 
  traitType = "Residual", 
  control = list(PC_columns = "PC1,PC2")
)
```

### Step 2. Conduct genome-wide association studies

For different types of traits and methods, the step 2 is the same as below.

```r
GenoFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
OutputDir = tempdir()

# step 2 for SPACox method
OutputFile = file.path(OutputDir, "Results_SPACox.txt")
GRAB.Marker(obj.SPACox, GenoFile = GenoFile, OutputFile = OutputFile)

# step 2 for SPAmix method
OutputFile = file.path(OutputDir, "Results_SPAmix.txt")
GRAB.Marker(obj.SPAmix, GenoFile = GenoFile, OutputFile = OutputFile)
```

Detailed documentation about how to use SPAGRM is available at [SPAGRM online tutorial](https://hexupku.github.io/SPAGRM.github.io/).

## Citation

- SPACox: Wenjian Bi, Lars G. Fritsche, Bhramar Mukherjee, Sehee Kim, and Seunggeun Lee. **A fast and accurate method for genome-wide time-to-event data analysis and its application to UK Biobank.** *The American Journal of Human Genetics* 107, no. 2 (2020): 222-233.

- SPAmix: Yuzhuo Ma, He Xu, Ying Li, Hyesung Kim, Lin-lin Xu, Lin Miao, Peng Xu, Fengbiao Mao, Xu-jie Zhou, Wei Zhou, Seunggeun Lee, Ji-Feng Zhang, Peipei Zhang, Wenjian Bi (2025). **A scalable, accurate, and universal analysis framework using individual-specific allele frequency for large-scale genetic association studies in an admixed population**. *Genome Biology* in press

- SPAGRM: He Xu, Yuzhuo Ma, Lin-lin Xu, Yin Li, Yufei Liu, Ying Li, Xu-jie Zhou, Wei Zhou, Seunggeun Lee, Peipei Zhang, Weihua Yue and Wenjian Bi (2025). **SPA(GRM): effectively controlling for sample relatedness in large-scale genome-wide association studies of longitudinal traits**. *Nature Communications* 16(1): 1413.
