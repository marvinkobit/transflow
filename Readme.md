# 💎 TransFlow: Tuberculosis Transmission Analysis Workflow

## 🧉 Introduction

TransFlow is a modular, flexible and user-friendly tuberculosis transmission analysis workflow based on whole genome sequencing (WGS) of *Mycobacterium tuberculosis* complex (MTBC).

The workflow filters non-MTBC samples using Kraken, then preforms quality control (QC) using both FastQC and MultiQC. After that, it uses the PANPASCO workflow to do pan-genome mapping and relative pairwise SNP distance calculation for transmission analysis. Next, it infers transmission clusters and networks using transcluster and SeqTrack, separately. Finally, it detects risk factors that significantly associate with transmission clustering.

---

## 🐍 Workflow

![Workflow](https://github.com/cvn001/transflow/blob/master/flowchart/flowchart.jpg)

---

## Input and output

### Input format

+ **Raw reads**: Currently, `TransFlow` supports paired-end WGS reads only from Illumina sequencing systems, such as Illumina Miseq platform in 301bp paired-end format (`*_1.fastq.gz`, `*_2.fastq.gz` file).
+ **Metadata**: Data that provide information about epidemiological characteristics of each sample. For example, age, gender and geographical coordinate of each patient, and lineage, drug resistance of each strain (tab-delimited `.txt` file).

**Note**: For transmission network inference, the collection dates of all samples are required. The `risk factor analysis` module solely relies on the epidemiological feature data provided by the user in the metadata file, the columns of which correspond to feature names, and the user can decide which feature would be analyzed in the config parameter. Missing data is allowed in the feature input, and these missing values ​​are ignored during the test.

### Output directories

+ `1.Quality_control`: Quality control of sequencing reads before and after the quality trimming and adapter removal step of the workflow by using FastQC.
+ `2.MTBC_identification`: The percentage of reads mapping to MTBC and the category detected by `kraken`.
+ `3.SNP_calling`: All the results regarding reads alignment and SNP calling by `GATK` are collected in this directory.
+ `4.SNP_distance`: The directory contains the statistics and visualizations of pairwise SNP distances between genomes from PANPASCO algorithm.
+ `5.Transmission_cluster`: The directory where all transmission analysis (SNP-based or transmission-based) results are placed including clustering statistics and transmission network.
+ `6.Risk_factor`: The directory contains the statistical testing result of transmission risk factors.
+ `7.Summary_report`: The directory contains a detailed and interactive summary report (html format) of all analyses, which can be seen by web browser.
+ `temp`: This is a negligible directory containing all temporary files from different modules. It can be totally deleted after the program has finished running to save storage space.

---

## ⚙️ Installation

### Dependencies

All dependencies are list on `workflow/envs/transflow.yaml` and can be automatically downloaded and installed with `conda` as below, otherwise you must have these in your PATH.

+ `snakemake` (version 6.6.0+)
+ `fastqc` (version 0.11.9+)
+ `multiqc` (version 1.11+)
+ `qualimap` (version 2.2.2d+)
+ `samtools` (version 1.12+)
+ `gatk` (version 3.8)
+ `picard` (version 2.18+)
+ `bedtools` (version 2.27+)
+ `flash` (version 1.2.11+)
+ `seqtk` (version 1.3+)
+ `fastp` (version 0.23+)
+ `trimmomatic` (version 0.36+)
+ `bwa` (version 0.7.17)
+ `tabix` (version 1.11+)
+ `pandoc` (version 2.16+)
+ `phantomjs` (version 2.1.1+)
+ `kraken` (version 1.1.1)
+ `python` (version 3.6+)
  + Packages: `numpy`, `matplotlib`, `pandas`, `biopython`
+ `R` (version 4.0+)
  + Packages: `argparse`, `ggpubr`, `ggplot2`, `devtools`, `ggally`, `lubridate`, `adegenet`, `network`, `pheatmap`, `gridextra`, `sna`, `scales`, `ggsci`, `yaml`, `kableextra`, `data.table`, `dt`, `ggsummary`, `tidyverse`, `genomicranges`, `transcluster`

**Note**: Version 3.8 of GATK is needed, as some functionalities of GATK3 were not ported to GATK4 yet.

### Installation with Conda

Conda can function as a package manager and is available [here](https://docs.conda.io/en/latest/miniconda.html).

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

If you have conda make sure the bioconda and conda-forge channels are added:

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
```

### Clone the repository

```bash
git clone git@github.com:cvn001/transflow.git
```

### Create the environment

After cloning this repository to a folder of your choice, it is recommended to create a general transflow conda environment with the accompanying `workflow/envs/transflow.yaml`. In the main folder of the cloned repository, execute the following command:

```bash
conda env create -f workflows/envs/transflow.yaml
```

This will create a conda environment containing all dependencies for Snakemake itself.

**Note**: It is advised to use [mamba](https://github.com/mamba-org/mamba) to speed up your conda installs.

### Activate the environment

```bash
conda activate transflow
```

### Install the **transcluster** in **R**

```R
devtools::install_github("JamesStimson/transcluster")
```

### Download MiniKraken database

A pre-built 8 GB Kraken database [MiniKraken DB_8GB](https://ccb.jhu.edu/software/kraken/dl/minikraken_20171019_8GB.tgz) for Kraken V1 is the suggested reference database for TransFlow. It is constructed from complete bacterial, archaeal, and viral genomes in RefSeq. According to the website, the latest version of MniKraken is **10/19/2017**.

---

## ⚙️ Set up configuration

To run the complete workflow do the following:

+ Place all gzipped FASTQ files of your samples (`ID_1.fastq.gz`, `ID_2.fastq.gz`) into `raw_data/`. Alternatively, you can specify the location of your gzipped FASTQ files in `config/configfile.yaml`.
+ Replace the example metadata file (`config/samples.txt`) with your own sample sheet containing:
  + A row for each sample
  + The following columns for each row:
    + (A) `sample` (id of each sample) \[Required\]
    + (B) `date` (collection date of each sample) \[Optional\]
    + (C) `latitude` (latitude of each sample) \[Optional\]
    + (D) `longitude` (longitude of each sample) \[Optional\]
    + (...) (Other epidemiological characters of each sample for risk factor inference) \[Optional\]
+ Modify the example configure file (`config/configfile.yaml`) to specify the appropriate:
  + `metadata_file` to `/path/to/metadata_file`
  + `kraken_db` to `/path/to/minikraken_20171019_8GB`
  + `characteristics` to collected characters, leave blank for skipping the risk factor analysis module.

---

## 🔧 Quick start

### Example

We provide an example dataset including WGS and corresponding epidemiological data which is publicly hosted at Zenodo: [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.6345888.svg)](https://doi.org/10.5281/zenodo.6345888).

Further information is provided in the database section below.

```bash
wget https://zenodo.org/record/6345888/files/example_data.zip
unzip example_data.zip
```

Run whole pipeline in just one command:

```bash
bash transflow.sh --configfile config/configfile.yaml -j 4
```

| Short Option | Long Option | Explanation |
| :------ | :------ | :---------- |
| `-k` | `--keep` | If a job fails, continue with independent jobs. |
| `-p` | `--printshellcmds` | Print out the shell commands that will be executed. |
| `-r` | `--reason` | Print the reason for rule execution (Missing output, updated input etc.) |
| `-j` | `--cores` | Number of CPU cores (threads) to use for this run. With no int, it uses all. Default is 1. |
| `--ri` | `--rerun-incomplete` | If Snakemake marked a file as incomplete after a crash, delete and produce it again. |
| `-n` | `--dryrun` | Just pretend to run the workflow. A similar option is `-S` (`--summary`). |
| `-q` | `--quiet` | Do not output certain information. If used without arguments, do not output any progress or rule information. |
|      | `--cleanup-shadow` | Cleanup old shadow directories which have not been deleted due to failures or power loss. |
|      | `--verbose` | Print detailed stack traces and detailed operations. Default is False. |
|      | `--nocolor` | Do not use a colored output. Default is False. |

## 🔧 Step by step

### 1. Data preprocessing and quality control

```bash
snakemake quality_control.snakefile --configfile config/configfile.yaml -j 4
```

### 2. Reads mapping and variant calling

```bash
snakemake variant_calling.snakefile --configfile config/configfile.yaml -j 4
```

### 3. Transmission analysis

```bash
snakemake transmission_analysis.snakefile --configfile config/configfile.yaml -j 4
```

### 4. Generating summary report

```bash
snakemake generating_report.snakemake --configfile config/configfile.yaml -j 4
```

---

## 🏗️ Parameters

These parameters are set in the configuration files. Please read them carefully before using TransFlow.

| Parameter | Description | Default | Discussion |
| :------ | :---------- | :----- | :---------- |
| `genome_file` | Reference genome file | *provided in this repository* | User can generate this using [seq-seq-pan](https://gitlab.com/rki_bioinformatics/seq-seq-pan) |
| `genomegaps_file` | File in .bed format with gaps in WGA of pan-genome | *provided in this repository* | For more details refer to [seq-seq-pan](https://gitlab.com/rki_bioinformatics/seq-seq-pan) |
| `exclude_regions_file` | File in .bed format with positions that should be excluded from distance analysis | - | It includes positions regarding to drug-resistance associated genes or repetitive regions of reference pan-genome (e.g. PPE/PE-PGRS family genes, phage sequence, insertion or mobile genetic elements). SNPs in these regions will be excluded for pairwise distance calculation |
| `metadata_file` | File with list of samples with one ID per line | - | Missing values will be omitted |
| `fastqdir` | Directory with .fastq.gz files named `ID_1.fastq.gz`, `ID_2.fastq.gz` | fastq | Only support Illumina PE seq data |
| `fastqpostfix` | Specification of fastq.gz format; e.g. for the format sample_R1.fastq.gz put `fastqpostfix: R` | - | Don't mix different postfixes |
| `glob_files` | [`true` or `false`], `true`: the workflow will load all fastq files in the input directory and parse all the sample names. `false`: the workflow will only read the fastq files of the samples in the metadata file. | false | |
| `kraken_cutoff` | Threshold of MTBC reads percentage | 80 | This value can be changed according to  |
| `MTBC_reads_only` | Filter out only MTBC reads | false | This would allow for slightly contaminated samples to still be reliably processed |
| `allele_frequency_threshold` | Allele frequency threshold for definition of high-quality SNPs | 0.75 |  |
| `mapping_quality_threshold` | Minimum Mapping Quality for reads to be used with GATK HaplotypeCaller | 10 |  |
| `depth_threshold` | Minimum coverage depth for high-quality regions | 5 |  |
| `flash_overlap` | Number of overlapping basepairs of paired reads for `FLASH` to merge reads | 10 |  |
| `trimmomatic_read_minimum_length` | Minimum length of reads to be kept after trimming | 50 |  |
| `trimmomatic_qual_slidingwindow` | Minimum quality in sliding window in trimming | 15 |  |
| `output_prefix` | Prefix for all distance files | all_samples | Using the project name is suggested for benefiting your management |
| `method` | Transmission clustering method [`SNP` or `trans`] | trans | You can try both methods by setting each separately |
| `snp_threshold` | SNP distance threshold for transmission clustering | 12 | Initially, a maximum distance of 12 SNPs between MTB isolates was introduced to rule in a possible epidemiological link between TB cases |
| `transmission_threshold` | The threshold for transmission clustering | 10 |  |
| `clock_rate` | Clock rate for MTBC samples (SNPs/genome/year) | 0.5 | Whilst the background SNP accumulation rate for MTB has been estimated at `0.5 SNPs/genome/year`, selection pressure and antibiotic resistance can influence this rate considerably. |
| `transmission_rate` | The rate at which the estimated number of intermediate transmissions must be | 2.0 |  |
| `coordinate` | Using sample's coordinate to improve transmission network reconstruction [`true` or `false`] | true |  |
| `characteristics` | Epidemiological characteristics for risk factor inference | - | All characters should be separated by spaces. Leave it blank to skip this step |
| `sample_threads` | Number of threads for each sample run | 1 |  |

---

## 📲 Troubleshoot common issues

### Common issues

For contacting the developer and issue reports please go to [Issues](https://github.com/cvn001/transflow/issues).

+ **Some samples are not in the transmission results**

There are a few steps where sequences can be removed:

During the filter step:
Samples that are included in the exclude file are removed
Samples that fail the current filtering criteria, as defined in the parameters.yaml file, are removed. You can modify the snakefile as desired, but currently these are:
Minimum sequence length of 25kb
No ambiguity in (sample collection) date
Samples may be randomly removed during subsampling; see :doc:`../guides/workflow-config-file` for more info.
During the refine step, where samples that deviate more than 4 interquartile ranges from the root-to-tip vs time are removed

+ **Java on Linux insufficient memory even though there is plenty of available memory being used for caching**

This is the `kernel.pid_max` limit. To solve this error, please refer to this [link](https://serverfault.com/questions/662992/java-on-linux-insufficient-memory-even-though-there-is-plenty-of-available-memor). To check and tuning the limit: [link](https://www.cyberciti.biz/tips/howto-linux-increase-pid-limits.html)

+ **To be added...**

### Unlocking

After the workflow was killed (Snakemake didn’t shutdown), the workflow directory will be still locked. If you are sure, that snakemake is no longer running `(ps aux | grep snake)`.

Unlock and clean up the working directory:

```bash
snakemake *.snakemake --unlock --cleanup-shadow
```

### Rerun incomplete

If Snakemake marked a file as incomplete after a crash, delete and produce it again.

```bash
snakemake *.snakemake --ri
```

## 🍾 License

The code is available under the [GNU GPLv3 license](https://choosealicense.com/licenses/gpl-3.0/). The text and data are available under the [CC-BY license](https://choosealicense.com/licenses/cc-by-4.0/).
