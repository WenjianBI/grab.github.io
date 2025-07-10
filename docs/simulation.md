---
layout: default
title: Data Simulation 
nav_order: 5
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: true
has_toc: true
---

# Data Simulation Using the GRAB Package

The example data (including 50 families with 10 members and 500 unrelated subjects) in the directory `system.file("extdata", package = "GRAB")` were simulated using functions in the `GRAB` package.

## Genotype Simulation

This section demonstrates how to:

- simulate a genotype matrix for unrelated and related subjects
- create PLINK binary files using the genotype matrix
- create BGEN files using PLINK binary files

## Phenotype Simulation

This section demonstrates how to:

- simulate binary, quantitative, time-to-event, and ordinal categorical phenotypes
- simulate phenotypes with random effects following a multivariate normal distribution
- simulate phenotypes using real genotype data (given PLINK binary files)
