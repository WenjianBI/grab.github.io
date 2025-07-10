---
layout: default
title: Genetic Relationship Matrix (GRM)
nav_order: 6
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: true
has_toc: true
---

# Genetic Relation Matrix (GRM)

Both dense GRM and sparse GRM are supported in `GRAB` package to characterize family relatedness, which can avoid high type one error rates.

| Which GRM   | Pros.    | Cons       | Required arguments  |
|:-----------:|:----------:|:--------:|:-------------------:|
| Dense GRM   | More powerful | Slow  | `GenoFile`          |
| Sparse GRM  | Fast  | Less powerful | `SparseGRMFile`     |

NOTE: Based on simulation and real data analysis results, for binary and ordinal categorical data analysis, analyses using dense and sparse GRM perform similarly in terms of both type one error rates and powers.

## Prior to using GRM

LD prune, maximal MAF cutoff
