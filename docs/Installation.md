---
layout: default
title: Installation
nav_order: 3
description: "Just the Docs is a responsive Jekyll theme with built-in search that is easily customizable and hosted on GitHub Pages."
has_children: false
has_toc: false
---

# Installation

GRAB is [an R package hosted on Github](https://github.com/GeneticAnalysisinBiobanks/GRAB). GRAB contains C++ code, and along with its dependency packages, they need to be compiled before they can be installed, which requires an appropriate compilation environment.

## Install GRAB on Linux, Windows, or macOS using Conda

Regardless of the operating system, we recommend the following two-step installation. 

The first step is to create an environment for GRAB using Conda.

```
conda create --name grab_env --channel conda-forge \
  r-bh r-data.table r-dbplyr r-dplyr r-igraph r-lme4 r-optparse r-ordinal \
  r-rcpparmadillo r-rcppparallel r-remotes r-rsqlite r-skat r-survival r-tidyr zlib
```

The second step is to activate the environment and install GRAB.

```
conda activate grab_env
R -e "remotes::install_github('GeneticAnalysisinBiobanks/GRAB', upgrade='never')"
R -e "library(GRAB)"  # to verify that GRAB can be loaded properly
```

## Install GRAB on Windows using Rtools

If you are using the Windows and already have R and Rtools installed, it's also convenient to install GRAB in the R console.

```
install.packages('remotes')
library(remotes)  
install_github('GeneticAnalysisinBiobanks/GRAB')
library(GRAB)  # to verify that GRAB can be loaded properly
```

## Install GRAB in Docker

The following is the content of a Dockerfile. Write it into a file named ```./Dockerfile``` and execute ```docker build -t grab_img .``` to create a Docker image. Then execute ```docker run grab_img R -e "library(GRAB)"``` to verify that GRAB can be loaded properly in a container.

```
FROM condaforge/miniforge3
RUN conda install r-bh r-data.table r-dbplyr r-dplyr r-igraph r-lme4 r-optparse r-ordinal \
  r-rcpparmadillo r-rcppparallel r-remotes r-rsqlite r-skat r-survival r-tidyr zlib
RUN R -e "remotes::install_github('GeneticAnalysisinBiobanks/GRAB', upgrade='never')"
```

For the use of GRAB in docker on UK Biobank Research Analysis Platform (RAP), please check the section [UK Biobank RAP](UKBB_RAP.md).


## Snapshot of R session info after loading GRAB

When the packages that GRAB depends on are upgraded, there may be compatibility issues. We record the output of ```sessionInfo()``` from a test build in May 2025 for potential debugging needs.

```
> sessionInfo()
R version 4.4.3 (2025-02-28)
Platform: x86_64-conda-linux-gnu
Running under: CentOS Linux 7 (Core)

Matrix products: default
BLAS/LAPACK: /gpu01ssd/miaolin/app/miniforge3/envs/grab_env/lib/libopenblasp-r0.3.29.so;  LAPACK version 3.12.0

locale:
 [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C
 [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8
 [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8
 [7] LC_PAPER=en_US.UTF-8       LC_NAME=C
 [9] LC_ADDRESS=C               LC_TELEPHONE=C
[11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C

time zone: Asia/Shanghai
tzcode source: system (glibc)

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
 [1] GRAB_0.1.2          Rcpp_1.0.14         igraph_2.0.3
 [4] dbplyr_2.5.0        dplyr_1.1.4         tidyr_1.3.1
 [7] SKAT_2.2.5          RSpectra_0.16-2     SPAtest_3.1.2
[10] Matrix_1.7-3        ordinal_2023.12-4.1 survival_3.8-3

loaded via a namespace (and not attached):
 [1] vctrs_0.6.5         nlme_3.1-168        cli_3.6.5
 [4] rlang_1.1.6         DBI_1.2.3           purrr_1.0.4
 [7] generics_0.1.4      RcppParallel_5.1.9  glue_1.8.0
[10] grid_4.4.3          tibble_3.2.1        MASS_7.3-64
[13] lifecycle_1.0.4     numDeriv_2016.8-1.1 compiler_4.4.3
[16] pkgconfig_2.0.3     ucminf_1.2.2        lattice_0.22-7
[19] R6_2.6.1            tidyselect_1.2.1    pillar_1.10.2
[22] splines_4.4.3       magrittr_2.0.3
```