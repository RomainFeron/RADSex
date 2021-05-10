[![GitHub release (latest by date)](https://img.shields.io/github/v/release/SexGenomicsToolkit/RADSex?color=lightorange)](https://github.com/SexGenomicsToolkit/RADSex/releases)
[![Conda (channel only)](https://img.shields.io/conda/vn/bioconda/radsex?color=lightorange)](https://bioconda.github.io/recipes/radsex/README.html)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4707041.svg)](https://doi.org/10.5281/zenodo.4707041)

# radsex

## Overview

The `radsex` software is part of RADSex, a computational workflow for the analysis of sex-determination using RAD-Sequencing data. This workflow contains the software `radsex` and the R package `sgtr`; a Snakemake implementation of the workflow is available [here](https://github.com/SexGenomicsToolkit/RADSex-workflow).

The first step of the RADSex workflow is to use `radsex` to generate a summary file for a set demultiplexed RAD reads and use this file to:

- infer the type of sex-determination system
- identify sex-biased markers
- align markers to a genome and identify genomic regions differentiated between sexes
- compute marker depth statistics

Results from `radsex` can be visualized with the `sgtr` R package available [here](https://github.com/SexGenomicsToolkit/sgtr).

Although RADSex was developed specifically to study sex-determination, it was designed to be flexible and can be used to compare two groups for any binary trait.

The RADSex computational workflow was developed in the [LPGP](https://www6.rennes.inra.fr/lpgp/) lab from INRA, Rennes, France for the PhyloSex project, which investigates sex determining factors in a wide range of fish species.

### Citing RADSex

If you use RADSex in your work, please cite the [manuscript describing RADSex](https://onlinelibrary.wiley.com/doi/abs/10.1111/1755-0998.13360):

> Feron, R., Pan, Q., Wen, M., Imarazene, B., Jouanno, E., Anderson, J., Herpin, A., *et al.* (2021), RADSex: A computational workflow to study sex determination using restriction site‐associated DNA sequencing data. Mol Ecol Resour. https://doi.org/10.1111/1755-0998.13360


To properly cite RADSex, you should also cite the software and its version using the DOI provided in the badges above:

> Romain Feron. (2021, April 21). SexGenomicsToolkit/radsex: 1.2.0 (Version 1.2.0). Zenodo. http://doi.org/10.5281/zenodo.4707041

## Documentation

This README file includes a basic installation guide and a quick start section. The full documentation for RADSex, including a complete example walkthrough, is available [here](https://sexgenomicstoolkit.github.io/html/radsex/introduction.html).

Internal documentation generated with Doxygen is provided [here](https://sexgenomicstoolkit.github.io/radsex).

## Installation

### Requirements

RADSex was tested on Linux (ubuntu >= 18.04, Arch) and OSX. To install RADSex, you will need:

- A C++11 compliant compiler (GCC >= 6.1.0, Clang >= 6.0)
- The zlib library (usually installed on linux and osx by default)

### Install the latest official release

- Download the [latest release](https://github.com/SexGenomicsToolkit/radsex/releases)
- Unzip the archive
- Navigate to the `radsex` directory
- Run `make`

The compiled `radsex` binary will be located in **radsex/bin/**.

### Install the latest stable development version

```bash
git clone https://github.com/SexGenomicsToolkit/radsex.git
cd radsex
make
```

The compiled `radsex` binary will be located in **radsex/bin/**.

### Install radsex with Conda

All versions of radsex are available in [Bioconda](https://bioconda.github.io/recipes/radsex/README.html?#recipe-Recipe%20&#x27;radsex&#x27;). To install the latest radsex release with Conda, run the following command:

```bash
conda install -c bioconda radsex
```

## Quick start

### Preparing the data

Before running the workflow, you should prepare the following elements:

- A **set of demultiplexed reads**. The current version of RADSex does not implement demultiplexing;
  raw sequencing reads can be demultiplexed using [Stacks](http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php)
  or [pyRAD](http://nbviewer.jupyter.org/gist/dereneaton/af9548ea0e94bff99aa0/pyRAD_v.3.0.ipynb#The-seven-steps-described).

- A **group info file** (popmap): a tabulated file with individual IDs in the first column and sex (or group) in the second column. Individual IDs in the popmap must be the same as the names of the demultiplexed reads files (*e.g.* 'individual_1' for the reads file 'individual_1.fq.gz')

- To align the markers to a genome: the **genome** sequence in a FASTA file.
  Note that when visualizing `map` results with `sgtr`, linkage groups / chromosomes are automatically inferred from scaffold names in the genome if their name starts with *LG*, *CHR*, or *NC* (case unsensitive). If chromosomes are named differently in the genome, you can use a tabulated file with contig ID in the first column and corresponding chromosome name in the second column (see the doc for details).

### Computing the marker depths table

The first step of RADSex is to create a table of marker depths for the entire dataset using the `process` command:

```bash
radsex process --input-dir ./samples --output-file markers_table.tsv --threads 16 --min-depth 1
```

In this example, demultiplexed reads are located in **./samples** and the markers table generated by `process` will be saved to **markers_table.tsv**. The parameter `--threads` specifies the number of threads to use, and `--min-depth` specifies the minimum depth to consider a marker present in an individual: markers which are not present with depth higher than this value in at least one individual will not be retained in the markers table.
It is advised to keep the minimum depth to the default value of 1 for this step, as it can be adjusted for each analysis later.


### Computing the distribution of markers between sexes

The `distrib` command computes the distribution of markers between males and females from a marker depths table:

```bash
radsex distrib --markers-table markers_table.tsv --output-file distribution.tsv --popmap popmap.tsv --min-depth 5 --groups M,F
```

In this example, `--markers-table` is the table generated with `process` and the distribution of markers between males and females will be saved to **distribution.tsv**. The sex of each individual in the population is given by **popmap.tsv**. Groups of individuals to compare (as defined in the popmap) are specified manually with the parameter `--groups`. The minimum depth to consider a marker present in an individual is set to 5, meaning that markers with depth lower than 5 in an individual will not be considered present in this individual.

The resulting distribution can be visualized with the `radsex_distrib()` function of [sgtr](https://github.com/SexGenomicsToolkit/sgtr), which generates a tile plot of marker counts with number of males on the x-axis and number of females on the y-axis.

### Extracting markers significantly associated with sex

Markers significantly associated with sex are obtained with the `signif` command:

```bash
radsex signif --markers-table markers_table.tsv --output-file markers.tsv --popmap popmap.tsv --min-depth 5 --groups M,F [ --output-fasta ]
```

In this example, `--markers-table` is the table generated with `process` and markers significantly associated with sex are saved to **markers.tsv**. The sex of each individual in the population is given by **popmap.tsv**. Groups of individuals to compare (as defined in popmap) are specified manually with the parameter `--groups`. The minimum depth to consider a marker present in an individual is set to 5, meaning that markers with depth lower than 5 in an individual will not be considered present in this individual.

By default, the `signif` function generates an output file in the same format as the markers depth table. Markers can also be exported to a fasta file using the parameter `--output-fasta`.

The markers table generated by `signif` can be visualized with the `radsex_markers_depth()` function of [sgtr](https://github.com/SexGenomicsToolkit/sgtr), which generates a heatmap showing the depth of each marker in each individual.


### Aligning markers to a genome

Markers can be aligned to a genome using the `map` command:

```bash
radsex map --markers-file markers_table.tsv --output-file alignment_results.tsv --popmap popmap.tsv --genome-file genome.fasta --min-quality 20 --min-frequency 0.1 --min-depth 5 --groups M,F
```

In this example, `--markers-file` is the markers depth table generated with `process` and the path to the reference genome file is given by `--genome-file`; results will are saved to **alignment_results.tsv**. The sex of each individual in the population is given by **popmap.tsv** and the minimum depth to consider a marker present in an individual is set to 5, meaning that markers with depth lower than 5 in an individual will not be considered present in this individual. Groups of individuals to compare (as defined in the popmap) are specified manually with the parameter `--groups`

The parameter `--min-quality` specifies the minimum mapping quality (as defined in [BWA](http://bio-bwa.sourceforge.net/bwa.shtml)) to consider a marker properly aligned and is set to 20 in this example. The parameter `--min-frequency` specifies the minimum frequency of a marker in the population to retain this marker and is set to 0.1 here, meaning that only sequences present in at least 10% of individuals of the population are aligned to the genome.

Alignment results from `map` can be visualized with the `radsex_map_circos()` function of [sgtr](https://github.com/SexGenomicsToolkit/sgtr), which generates a circular plot showing bias and association with sex for each marker aligned to the genome.

Alignment results for a specific contig can be visualized with the `radsex_map_region()` function to show the same metrics for a single contig.


## LICENSE

Copyright (C) 2018-2020 Romain Feron

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation,
either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see https://www.gnu.org/licenses/
