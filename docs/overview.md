---
layout: default
title: Overview
nav_order: 2
description: "Overview of GRAB package."
has_children: false
has_toc: false
---

# Overview

The `GRAB` package is primarily designed to conduct **genome-wide association studies (GWAS)** for both single-variant and set-based analyses. Additionally, the package can be used:

- to simulate genotype/phenotype data
- to calculate sparse GRM
- to read genotype data from PLINK/BGEN files

## Genome-wide Association Studies

All approaches share the same analysis framework, which includes the following two steps:

- Step 1: Fit a null model using traits, covariates, and GRM (if applied).
- Step 2: Conduct single-variant or set-based tests to identify markers or marker-sets (e.g., genes) significantly associated with the trait of interest.

The `GRAB` package supports multiple trait types including:

- Ordinal categorical traits (POLMM / POLMM-GENE)
- Time-to-event traits (SPACox)
- Model residuals after fitting a null model

The `GRAB` package includes `SPACox`, `SPAmix`, and `SPAGRM` methods that support model residuals as input. For any type of trait, users can select an appropriate model to fit the trait against confounding factors and then calculate model residuals. If the sum of the residuals is zero, they can serve as input for a GWAS.

The `GRAB` package supports the following genotype file formats for association studies:

- `PLINK` binary files (.bed, .bim, .fam)
- `BGEN` (.bgen, .bgi, .sample)

## Preparation Before Using the GRAB Package

The `GRAB` package supports using genotype data to adjust for sample relatedness via genetic relationship matrix (GRM). PLINK binary files with high-quality genotyped variants are required for this purpose. In UK Biobank real data analysis, we used the following cutoffs in PLINK to select ~340K SNPs for White British subjects:

```sh
--maf 0.05
--indep-pairwise 500 50 0.2
```

If the sample size in the analysis is greater than 100,000, we recommend using sparse GRM (instead of dense GRM) to adjust for sample relatedness. The function `getSparseGRM()` internally uses `GCTA` software (we tested v1.93) to create a `SparseGRMFile` that will be passed to functions as needed. As required by the `GCTA` software, the function only supports Linux and PLINK files. Two steps are required:

- Step 1: Run `getTempFilesFullGRM()` to save temporary files to tempDir.

- Step 2: Run `getSparseGRM()` to combine the temporary files to create a `SparseGRMFile`.

Users can customize parameters including `(minMafGRM, maxMissingGRM, nPartsGRM)`, but these parameters should be consistent for functions `getTempFilesFullGRM()` and `getSparseGRM()`. Otherwise, the temporary files cannot be accurately located.

## Other Functions in the `GRAB` Package

The `GRAB` package provides additional functions to facilitate user workflows. More details can be found in the corresponding sections.

### Data Simulation

The `GRAB` package can be used to simulate genotype/phenotype data.

### Genetic Relationship Matrix (GRM)

The `GRAB` package supports sparse GRM to adjust for family relatedness. Functions `getTempFilesFullGRM()` and `getSparseGRM()` can be used to generate a sparse GRM using PLINK files.

### Reading Genotype Data

The `GRAB` package can also be used to read genotype data into R from `PLINK` and `BGEN` files.
