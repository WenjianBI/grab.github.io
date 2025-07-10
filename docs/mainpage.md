---
layout: default
title: Home
nav_order: 1
description: "Main page for GRAB package."
permalink: /
---

## Main Features

`GRAB` is an R package for Genome-wide Robust Analysis designed for biobank data.
The main features of the package are as follows:

- Supports multiple complex traits including:
  - quantitative traits
  - binary traits
  - time-to-event traits
  - ordinal categorical traits
  - longitudinal traits
- Performs single-variant and set-based association tests
- Accounts for sample relatedness using Genetic Relationship Matrix **(GRM)**
- Calibrates p-values using normal distribution approximation and Saddlepoint approximation **(SPA)**, which:
  - are computationally efficient for large datasets (e.g., UK Biobank)
  - can handle unbalanced phenotypic distributions (e.g., case-control imbalance in binary traits)
  - are robust for both common and rare variants

For set-based association tests, the `GRAB` package:

- performs Burden test, SKAT, and SKAT-O
- allows tests on multiple minor allele frequency cutoffs and functional annotations
- allows specification of weights for single variants in set-based tests
- performs conditional analysis to identify associations independent of nearby GWAS signals

## Supported Approaches

### POLMM / POLMM-GENE

- Supports ordinal categorical traits
- Single-variant / set-based tests
- Can account for sample relatedness
- Reference
  - Wenjian Bi, Wei Zhou, Rounak Dey, Bhramar Mukherjee, Joshua N. Sampson, and Seunggeun Lee. **Efficient mixed model approach for large-scale genome-wide association studies of ordinal categorical phenotypes.** *The American Journal of Human Genetics* 108, no. 5 (2021): 825-839.
  - Wenjian Bi, Wei Zhou, Peipei Zhang, Yaoyao Sun, Weihua Yue, and Seunggeun Lee. **Scalable mixed model approaches for set-based association studies on large-scale categorical data analysis and its application to 450k exome sequencing data in UK Biobank.** *The American Journal of Human Genetics* 110, no. 5 (2023): 762-773.

### SPACox

- Supports (but not limited to) time-to-event traits
- Supports model residuals (whose sum is zero) after fitting a null model to any type of trait
- Single-variant tests
- Cannot account for sample relatedness
- Reference
  - Wenjian Bi, Lars G. Fritsche, Bhramar Mukherjee, Sehee Kim, and Seunggeun Lee (2020). **A fast and accurate method for genome-wide time-to-event data analysis and its application to UK Biobank.** *The American Journal of Human Genetics* 107, no. 2: 222-233.

### SPAmix

- Supports (but not limited to) time-to-event traits
- Supports model residuals (whose sum is zero) after fitting a null model to any type of trait
- Can support admixed populations or multiple populations
- Single-variant tests
- Cannot account for sample relatedness
- Reference
  - Yuzhuo Ma, He Xu, Ying Li, Hyesung Kim, Lin-lin Xu, Lin Miao, Peng Xu, Fengbiao Mao, Xu-jie Zhou, Wei Zhou, Seunggeun Lee, Ji-Feng Zhang, Peipei Zhang, Wenjian Bi (2025). **A scalable, accurate, and universal analysis framework using individual-specific allele frequency for large-scale genetic association studies in an admixed population**. *Genome Biology* in press

### SPAGRM

- Supports (but not limited to) time-to-event traits
- Supports model residuals (whose sum is zero) after fitting a null model to any type of trait
- Single-variant tests
- Can account for sample relatedness
- Reference
  - He Xu, Yuzhuo Ma, Lin-lin Xu, Yin Li, Yufei Liu, Ying Li, Xu-jie Zhou, Wei Zhou, Seunggeun Lee, Peipei Zhang, Weihua Yue and Wenjian Bi (2025). **SPA(GRM): effectively controlling for sample relatedness in large-scale genome-wide association studies of longitudinal traits**. *Nature Communications* 16(1): 1413.

### WtCoxG

- Supports time-to-event traits
- Single-variant tests
- Uses external allele frequencies to improve statistical power
- Can account for sample relatedness
- Reference
  - Ying Li, Yuzhuo Ma, He Xu, Yaoyao Sun, Min Zhu, Weihua Yue, Wei Zhou and Wenjian Bi (2025). **Applying weighted Cox regression to boost powers for genome-wide association studies of time-to-event phenotypes**. *Nature Computational Science* in press.

## License

`GRAB` is distributed under a GPL license.

## Contact

If you have any questions about the `GRAB` package, please contact [wenjianb@pku.edu.cn](mailto:wenjianb@pku.edu.cn)
