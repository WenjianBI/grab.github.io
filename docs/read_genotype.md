---
layout: default
title: Read Genotype Data
nav_order: 4
---

# Read Genotypes

The `GRAB` package gives function `GRAB.ReadGeno` to read in genotype data from `PLINK` and `BGEN` files.

## Quick Start-up Guide

The below gives an example to read in genotype data of SNPs in `IDsToIncludeFile` from PLINK/BGEN files to R

```r
IDsToIncludeFile = system.file("extdata", "simuGENO.IDsToInclude", package = "GRAB")

## PLINK files
PLINKFile = system.file("extdata", "simuPLINK.bed", package = "GRAB")
GenoList = GRAB.ReadGeno(PLINKFile, 
                         control = list(IDsToIncludeFile = IDsToIncludeFile)) 
GenoMat = GenoList$GenoMat
markerInfo = GenoList$markerInfo
head(GenoMat[,1:6])
head(markerInfo)

## BGEN files (Note the different REF/ALT order for BGEN and PLINK formats)
BGENFile = system.file("extdata", "simuBGEN.bgen", package = "GRAB")
GenoList = GRAB.ReadGeno(BGENFile, 
                         control = list(IDsToIncludeFile = IDsToIncludeFile))
GenoMat = GenoList$GenoMat
markerInfo = GenoList$markerInfo
head(GenoMat[,1:6])
head(markerInfo)
```

## Usage

```r
GRAB.ReadGeno(
  GenoFile,
  GenoFileIndex = NULL,
  SampleIDs = NULL,
  control = NULL,
  sparse = FALSE
)
```

## Arguments

- `GenoFile` a character of genotype file. See Details section for more details.
- `GenoFileIndex` additional index file(s) corresponding to GenoFile. See Details section for more details.
- `SampleIDs` a character vector of sample IDs to extract. The default is NULL, that is, all samples in GenoFile will be extracted.
- `control` a list of parameters to decide which markers to extract. See Details section for more details.
- `sparse` a logical value (default: FALSE) to indicate if the output of genotype matrix is sparse.

## Details

### Details about GenoFile and GenoFileIndex

Currently, we support two formats of genotype input including PLINK and BGEN. Other formats such as VCF will be added later. Users do not need to specify the genotype format, GRAB package will check the extension of the file name for that purpose. If GenoFileIndex is not specified, GRAB package assumes the prefix is the same as GenoFile.

- PLINK format (Check link for more details about this format)

  - GenoFile: "prefix.bed". The full file name (including the extension ".bed") of the PLINK binary bed file.

  - GenoFileIndex: c("prefix.bim", "prefix.fam"). If not specified, GRAB package assumes that bim and fam files have the same prefix as the bed file.

- BGEN format (Check link for more details about this format. Currently, only version 1.2 with 8 bits suppression is supported)

  - GenoFile: "prefix.bgen". The full file name (including the extension ".bgen") of the BGEN binary bgen file.

  - GenoFileIndex: "prefix.bgen.bgi" or c("prefix.bgen.bgi", "prefix.sample"). If not specified, GRAB package assumes that bgi and sample files have the same prefix as the bgen file. If only one element is given for GenoFileIndex, then it should be a bgi file. Check link for more details about bgi file.

  - If the bgen file does not include sample identifiers, then sample file is required, whose detailed description can ben seen in link. If you are not sure if sample identifiers are in BGEN file, please refer to checkIfSampleIDsExist.

- VCF format will be supported later. GenoFile: "prefix.vcf"; GenoFileIndex: "prefix.vcf.tbi"

### Details about argument control

Argument control is used to include and exclude markers for function GRAB.ReadGeno. The function supports two include files of (IDsToIncludeFile, RangesToIncludeFile) and two exclude files of (IDsToExcludeFile, RangesToExcludeFile), but does not support both include and exclude files at the same time.

- IDsToIncludeFile: a file of marker IDs to include, one column (no header). Check system.file("extdata", "IDsToInclude.txt", package = "GRAB") for an example.

- IDsToExcludeFile: a file of marker IDs to exclude, one column (no header).

- RangesToIncludeFile: a file of ranges to include, three columns (no headers): chromosome, start position, end position. Check system.file("extdata", "RangesToInclude.txt", package = "GRAB") for an example.

- RangesToExcludeFile: a file of ranges to exclude, three columns (no headers): chromosome, start position, end position.

- AlleleOrder: a character, "ref-first" or "alt-first", to determine whether the REF/major allele should appear first or second. Default is "alt-first" for PLINK and "ref-first" for BGEN. If the ALT allele frequencies of most markers are > 0.5, you should consider resetting this option. NOTE, if you use plink2 to convert PLINK file to BGEN file, then 'ref-first' modifier is to reset the order.

- AllMarkers: a logical value (default: FALSE) to indicate if all markers are extracted. It might take too much memory to put genotype of all markers in R. This parameter is to remind users.

- ImputeMethod: a character, "none" (default), "bestguess", or "mean". By default, missing genotype is NA. Suppose alternative allele frequency is p, then missing genotype is imputed as 2p (ImputeMethod = "mean") or round(2p) (ImputeMethod = "bestguess").

## Value

The function returns an R list including a genotype matrix and an information matrix.

- GenoMat: Genotype matrix, each row is for one sample and each column is for one marker.

- markerInfo: Information matrix including 5 columns of CHROM, POS, ID, REF, and ALT.
