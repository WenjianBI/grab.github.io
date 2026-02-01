---
layout: home
title: Home
nav_order: 1
description: "GRAB, an R package of GWAS methods designed for biobank data"
permalink: /
---

# Overview

The **GRAB** (**G**enome-wide **R**obust **A**nalysis methods designed for **B**iobank data) R package is primarily designed to perform GWAS accounting for sample relatedness and population structure. It supports multiple trait types with [unified two-step framework](docs/approach.md). Additionally, the package can be used to:

- [Simulate genotype](docs/simulation_genotype.md) and [phenotype](docs/simulation_phenotype.md) data
- [Calculate sparse GRM](docs/GRM.md)
- [Read genotype data](docs/read_genotype.md) from PLINK or BGEN files

# Installation

## [![CRAN Status](https://www.r-pkg.org/badges/version/GRAB)](https://CRAN.R-project.org/package=GRAB) CRAN

![Linux](https://img.shields.io/badge/Linux-000?logo=linux&logoColor=white)
![macOS](https://img.shields.io/badge/macOS-000?logo=apple&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?logo=windows&logoColor=white)

Install GRAB from CRAN in your R console:

```r
install.packages("GRAB", dependencies = TRUE)
```

## [![Conda-Forge](https://img.shields.io/conda/vn/conda-forge/r-grab.svg)](https://anaconda.org/conda-forge/r-grab) Conda

![Linux](https://img.shields.io/badge/Linux-000?logo=linux&logoColor=white)
![macOS](https://img.shields.io/badge/macOS-000?logo=apple&logoColor=white)

Install GRAB in a new Conda environment named `grab_env` from the `conda-forge` channel:

```sh
conda create -n grab_env -c conda-forge r-grab r-skat r-dbplyr r-tidyr
```

## [![Docker Image Version](https://img.shields.io/docker/v/geneticanalysisinbiobanks/grab?sort=semver&label=Docker%20latest)](https://hub.docker.com/r/geneticanalysisinbiobanks/grab) Docker

![Linux](https://img.shields.io/badge/Linux-000?logo=linux&logoColor=white)

Pull the latest GRAB Docker image from Docker Hub:

```sh
docker pull geneticanalysisinbiobanks/grab:latest
```

## License

`GRAB` is distributed under a GPL license.

## Contact

If you have any questions about GRAB, please contact miaolin&#64;pku.edu.cn.
