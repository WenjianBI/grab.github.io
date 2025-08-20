---
layout: default
title: Installation
nav_order: 3
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: false
has_toc: false
---

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
install.packages("GRAB")
```

## Install via Conda

[![Conda-Forge](https://img.shields.io/conda/vn/conda-forge/r-grab.svg)](https://anaconda.org/conda-forge/r-grab)
[![Anaconda-Server Badge](https://anaconda.org/conda-forge/r-grab/badges/downloads.svg)](https://anaconda.org/conda-forge/r-grab)

Install GRAB in a new Conda environment named `grab_env` from the `conda-forge` channel:

```sh
conda create -n grab_env -c conda-forge r-grab
```

## Install from source code

[![GitHub main](https://img.shields.io/badge/GitHub-main-black?logo=github)](https://github.com/GeneticAnalysisinBiobanks/GRAB)
[![License: GPL v2+](https://img.shields.io/badge/License-GPL%20v2%2B-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.html)

First, create an environment for GRAB using Conda:

```sh
conda create --name grab_env --channel conda-forge \
  zlib r-bh r-rcpp r-rcpparmadillo r-rcppparallel r-data.table r-dplyr r-lme4 r-mvtnorm \
  r-ordinal r-survival r-rsqlite r-skat r-remotes r-dbplyr r-igraph r-optparse r-r.utils
```

Then, activate the environment and install GRAB:

```sh
conda activate grab_env
R -e "remotes::install_github('GeneticAnalysisinBiobanks/GRAB', upgrade='never')"
```

To verify that GRAB was installed successfully, check its version with:

```sh
R -e "packageVersion('GRAB')"
```

## Install GRAB with Docker

Build a Docker image for GRAB named `grab_img` using the following command:

```sh
docker build -t grab_img - <<EOF
FROM condaforge/miniforge3
RUN conda install r-grab
EOF
```

Then, verify that GRAB can be loaded properly in a container:

```sh
docker run grab_img R -e "library(GRAB); message('GRAB loaded successfully')"
```

For instructions on using GRAB with Docker on the UK Biobank Research Analysis Platform (RAP), please see the section [UK Biobank RAP](UKBB_RAP.md).
