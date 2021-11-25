Using the DADA2 pipeline for sequence variant identification
============================================================

Here we provide a R Notebook template to generate a sequences variants table from FASTQ files generated by Paired-End Illumina sequencing. For maximal reproducibility, we also provide a R project file and its associated packrat packages management directory.

Note: Commands described in this documentation assume that you are using a Unix CLI.

## System requirements

The minimal requirements are listed below then further detailed in this section:

* **R >4.0.0:** https://www.r-project.org
* **R package renv:** https://cran.r-project.org/web/packages/renv/index.html
* **RStudio:** https://www.rstudio.com
* **GNU parallel:** https://www.gnu.org/software/parallel
* **illumina-utils:** https://github.com/merenlab/illumina-utils
* **cutadapt:** https://cutadapt.readthedocs.io/en/stable/# (For ITS only)
* **git:** https://git-scm.com/

#### R

The DADA2 pipeline comes as a R package. Make sure that R (3.6.0 or higher) is installed before starting.

#### R renv package

R packages are managed using renv. This ensures maximal reproducibility and portability of the analysis by using an encapsulated, version controlled, installation of R packages instead of any system-level R packages installation.

Make sure that the renv R package is installed before starting:

```
# R
> install.packages("renv")
```

#### RStudio

The `dada2-pipeline-16S.Rmd` and `dada2-pipeline-ITS.Rmd` files provided in this repository are a R Notebook. As such, it includes both code lines (chunks) and text and can be exported into html and pdf files.

The prefered way of using these files is with the RStudio Integrated Development Environment (IDE). Make sure you have RStudio installed before starting.

## Conda environment

Since some tools are required outside of the R pipeline, it is recommended to create a conda environment. Make sure you have an updated version of conda/miniconda. Details on how to install conda are provided [here](https://docs.conda.io/projects/conda/en/latest/user-guide/install/index.html).

```
# Create a conda environment named dada2
conda create --name dada2

# Activate the environment
conda activate dada2

# Install illumina-utils
pip3 install illumina-utils

# Install parallel
conda install -c conda-forge parallel

# Install cutadapt
python3 -m pip install --user --upgrade cutadapt

# Install git
conda install -c anaconda git
```

## Files

Before starting, make sure you have the five files listed below:

1. **`R1.fastq.gz`**: FASTQ file for the forward read (1)
2. **`R2.fastq.gz`**: FASTQ file for the reverse read (1)
3. **`Index.fastq.gz`**: FASTQ file for the index read (1)
4. **`barcode_to_sample_[runNN].txt`**: A text file mapping index barcodes to samples (2)
5. A DADA2-formatted reference database: see https://benjjneb.github.io/dada2/training.html. For example, SILVA version 132: `silva_nr_v132_train_set.fa.gz` and `silva_species_assignment_v132.fa.gz`. 

It is assumed that the FASTQ files were archived using gzip.

Please pay particular attention to the format of the files. The following points are critical:

(1) FASTQ files must use the Phred+33 quality score format and sequences headers (`@` lines) must fit the standard format of the CASAVA 1.8 output:

```
@EAS139:136:FC706VJ:2:2104:15343:197393 1:N:0:0
```

(2) The `barcode_to_sample.txt` file must contain two tab-delimited columns: the first for samples names and the second for samples barcodes as shown <a href="https://github.com/merenlab/illumina-utils/blob/master/examples/demultiplexing/barcode_to_sample.txt" target="_blank">here</a>. Avoid special characters.

If multiple sequencing runs have to be analyzed together, create a directory for each run and place the respective FASTQ and `barcode_to_sample_[runNN].txt` files inside. 

## Preparation

Get a copy of this repository (give it any name `my_project_dir`) and set it as your working directory:
```
$ git clone https://github.com/respiratory-immunology-lab/microbiome-dada2.git
 my_project_dir
$ cd my_project_dir
```
If not done yet, get a copy of the DADA2-formatted reference database of your choice at https://benjjneb.github.io/dada2/training.html. We recommend using the latest SILVA db for 16S (`silva_nr_v132_train_set.fa.gz` and `silva_species_assignment_v132.fa.gz`) and UNITE for ITS (`sh_general_release_dynamic_s_01.12.2017.fasta`). We recommend to store it in a directory dedicated to databases instead of keeping it inside the main project directory.

Place your FASTQ files and the `barcode_to_sample.txt` file in a directory, then place this directory within the directory named `run_data`.
The final directory structure should look like:
<pre>
my_project_dir
├── run_data
|   └── <b>run01</b>
|       ├── <b>R1.fastq.gz</b>
|       ├── <b>R2.fastq.gz</b>
|       ├── <b>Index.fastq.gz</b>
|       └── <b>barcode_to_sample_[runNN].txt</b>
├── renv
|   └── ...
├── renv.lock
├── dada2-pipeline-16S.Rmd
├── dada2-pipeline-ITS.Rmd
├── dada2-pipeline.Rproj
├── LICENSE.txt
└── README.md
</pre>

If multiple sequencing runs have to be analyzed together, place each run directory inside the directory named `run_data`.
In this case, the final directory structure should look like:
<pre>
my_project_dir
├── run_data
|   ├── <b>run01</b>
|   |   ├── <b>R1.fastq.gz</b>
|   |   ├── <b>R2.fastq.gz</b>
|   |   ├── <b>Index.fastq.gz</b>
|   |   └── <b>barcode_to_sample_run1.txt</b>
|   ├── <b>run02</b>
|   |   ├── <b>R1.fastq.gz</b>
|   |   ├── <b>R2.fastq.gz</b>
|   |   ├── <b>Index.fastq.gz</b>
|   |   └── <b>barcode_to_sample_run2.txt</b>
.   .
.   .
.   .
|   └── <b>runNN</b>
|       ├── <b>R1.fastq.gz</b>
|       ├── <b>R2.fastq.gz</b>
|       ├── <b>Index.fastq.gz</b>
|       └── <b>barcode_to_sample_runNN.txt</b>
├── renv
|   └── ...
├── renv.lock
├── dada2-pipeline.Rmd
├── dada2-pipeline-16S.Rmd
├── dada2-pipeline-ITS.Rmd
├── LICENSE.txt
└── README.md
</pre>

## Usage

1. Load the `dada2-pipeline.Rproj` R project file in RStudio.

2. Open the `dada2-pipeline-16S.Rmd` R Notebook template for 16S analysis and `dada2-pipeline-ITS.Rmd` for ITS analysis in RStudio and follow the instructions in the text and comments. At the end of the pipeline, inital files and results are archived into two separate archives. 

3. Store archives in a safe place!

## Citation

If you used this repository in a publication, please mention its url.

In addition, you may cite the tools used by this pipeline:

* **DADA2:** Callahan BJ, McMurdie PJ, Rosen MJ, Han AW, Johnson AJA, Holmes SP
(2016). "DADA2: High-resolution sample inference from Illumina amplicon
data." _Nature Methods_, *13*, 581-583. doi: 10.1038/nmeth.3869.

* **illumina-utils:** Eren AM, Vineis JH, Morrison HG, Sogin ML (2013). "A Filtering Method to Generate High Quality Short Reads Using Illumina Paired-End Technology." _PLOS ONE_, 8(6). doi: 10.1371/journal.pone.0066643.

* **cutadapt:** MARTIN, Marcel (2011). "Cutadapt removes adapter sequences from high-throughput sequencing reads." _EMBnet.journal, [S.l.]_ doi: 10.14806/ej.17.1.200.

## Rights

* Copyright (c) 2021 Respiratory Immunology lab, Monash University, Melbourne, Australia and Service de Pneumologie, Centre Hospitalier Universitaire Vaudois (CHUV), Switzerland.
* License: The R Notebook template (.Rmd) is provided under the MIT license (See LICENSE.txt for details)
* Authors: A. Rapin, C. Pattaroni, A. Butler, B.J. Marsland
