---
layout: home
title: Home
nav_order: 1
description: "GRAB, an R package of methods designed for biobank data"
permalink: /
---

# Overview

The **GRAB** (**G**enome-wide **R**obust **A**nalysis methods designed for **B**iobank data) package is primarily designed to perform genome-wide association studies (GWAS) for both single-variant and set-based analyses, accounting for sample relatedness and population structure. It supports multiple trait types ([GWAS framework]({% link docs/approach.md %})), including:

- Ordinal categorical traits
- Time-to-event traits
- Any trait: using residuals of a fitted null model

Additionally, the package can be used to:

- [Simulate genotype]({% link docs/simulation_genotype.md %}) and [phenotype]({% link docs/simulation_phenotype.md %}) data
- [Calculate sparse GRM]({% link docs/GRM.md %})
- [Read genotype data]({% link docs/read_genotype.md %}) from PLINK or BGEN files

# Installation

![Linux](https://img.shields.io/badge/Linux-000?logo=linux&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D6?logo=windows&logoColor=white)
![macOS](https://img.shields.io/badge/macOS-000?logo=apple&logoColor=white)

GRAB is an R package, with part of its code written in C++ for improved performance. GRAB can be installed on Linux, Windows, or macOS via CRAN, Conda, or from source code.

## Install via CRAN

[![CRAN Status](https://www.r-pkg.org/badges/version/GRAB)](https://CRAN.R-project.org/package=GRAB)
[![CRAN Downloads](https://cranlogs.r-pkg.org/badges/grand-total/GRAB)](https://CRAN.R-project.org/package=GRAB)

Install GRAB from CRAN in your R console:

```r
install.packages("GRAB", dependencies = TRUE)
```

## Install via Conda

[![Conda-Forge](https://img.shields.io/conda/vn/conda-forge/r-grab.svg)](https://anaconda.org/conda-forge/r-grab)
[![Anaconda-Server Badge](https://anaconda.org/conda-forge/r-grab/badges/downloads.svg)](https://anaconda.org/conda-forge/r-grab)

Install GRAB in a new Conda environment named `grab_env` from the `conda-forge` channel:

```sh
conda create -n grab_env -c conda-forge r-grab r-skat r-dbplyr r-tidyr
```

## Build a Docker image

Build a Docker image for GRAB named `grab_img` using the following command:

```sh
docker build -t grab_img - <<EOF
FROM condaforge/miniforge3
RUN conda install r-grab r-skat r-dbplyr r-tidyr
EOF
```

## Pull a Prebuilt Docker Image

A prebuilt Docker image for GRAB is available and can be pulled directly from Docker Hub with the following command:

```sh
docker pull geneticanalysisinbiobanks/grab:latest
```

## License

`GRAB` is distributed under a GPL license.

## Contact

If you have any questions about GRAB, please contact [wenjianb@pku.edu.cn](mailto:wenjianb@pku.edu.cn).
