---
layout: default
title: Tool Developer 
nav_order: 10
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: false
---

# For analysis tool developer

`GRAB` package provides a generic framework of GWAS on a large-scale biobank data. The below gives a brief idea about how to incorporate a new tool into `GRAB` package.

Suppose that the method is `ABCDE`, then the following functions are required.

- checkControl.NullModel.ABCDE(control, optionGRM)
- fitNullModel.ABCDE(response, designMat, subjData, control, optionGRM, genoType, markerInfo)
- checkControl.Marker.ABCDE(control, optionGRM)
- checkControl.Region.ABCDE(control, optionGRM)

## Function `checkControl.NullModel.ABCDE`

### Input

- control: a list of parameters  
- optionGRM:

### Output

- control: a list

## Function `fitNullModel.ABCDE`

The function `fitNullModel.ABCDE` is a lower function to fit null model based on appraoch `ABCDE`. In main function `GRAB.NullModel`, the input is first converted to the following data, which is then passed to function `fitNullModel.ABCDE`.

### Input

- response: output of function `model.response`
- designMat: output of function `model.matrix` after removing column of `(Intercept)` if any.
- subjData: subject ID
- control: output of function `checkControl.NullModel`.
- optionGRM: `"SparseGRM"` or `"DenseGRM"`
- genoType: `"PLINK"` or `"BGEN"`
- markerInfo: a data frame to record marker infomation used in GRM construction.

### Output

- control: a list

## Function `checkControl.Marker.ABCDE`

## Function `checkControl.Region.ABCDE`
