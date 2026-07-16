# Metagenomic_repo
Bu repo çevrimiçi bir metagenomik rehberi.

# **SHOTGUN METAGENOMICS GUIDELINE**

## **Purpose Of This Repository**

This repository is a coding guideline for starters in shotgun metagenomics data analysis. It includes codes of installing tools, creating environments and running analyses. If you're new to metagenomic analyses, you can see the [Metagenomic guideline](https://docs.google.com/document/d/1kU-FlgxgB0zssmJtbSXxJsxBNo2GGQfwhAVUqOSdKl4/edit?tab=t.0) for more information.

> Note: This repository is for Ubuntu/Debian systems only. If you're using other oprating systems such as Mac or Windows, you can either install WSL for Windows or use Mac terminal instead.

## Installation & Environment Setup

This pipeline requires specific bioinformatics tools. To ensure all tools run smoothly without library conflicts, using **Conda** or **Mamba** is suggested.

> Mamba works exactly like Conda, yet resolves package dependencies in parallel, completing your installations faster. If Conda is not installed in your computer yet, Mamba is recommended. If you have Conda, you can use the same script with writing "conda" isntead of "mamba".


### Step 1: Initialize your Environment Manager

If you do not have Mamba installed yet, you can install it with running the code below in your computer's terminal.

```bash
# 1. Download the Miniforge installer script for Linux (x86_64)
curl -L -O "[https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh)"

# 2. Run the installer script
Miniforge3-Linux-x86_64.sh

# Note: Press ENTER to read the license, and type 'yes' to accept it. 
# When asked if you wish to initialize Miniforge3, type 'yes'.
```
To understand how these codes work, you can see the details.

<details>

1. **`curl -L -O "..."`**
   * **`curl`** is a command-line tool used to transfer data from a server.
   * **`-L`** tells curl to follow any redirects if the webpage has moved.
   * **`-O`** (capital letter O) saves the downloaded file on your computer using its original online name (`Miniforge3-Linux-x86_64.sh`).

3. **`bash Miniforge3-Linux-x86_64.sh`** * This executes the downloaded shell script (`.sh`) using the Bash interpreter to start the interactive installation wizard on your Ubuntu system.

4. **`source ~/.bashrc`** * This reloads your terminal's configuration file (`.bashrc`). It registers the newly installed `mamba` command so you can use it immediately without needing to close and reopen your terminal window.

</details>

### Step 2: Configure Bioinformatics Channels (Required Once)

Before downloading any pipeline tools, channels for bioinformatics is needed. This help Mamba & Miniforge to find suitable libraries for the specific tools.

You can install them by running the codes below:

```bash
# Add Conda-Forge (hosts general open-source packages)
mamba config --add channels conda-forge

# Add Bioconda (hosts specialized bioinformatics software)
mamba config --add channels bioconda

# Set channel priority to strict (ensures stable, fast dependency resolution)
mamba config --set channel_priority strict
```
To understand what these channel configuration commands do, you can see details.

<details>
  
Here is a simple breakdown of what these configuration commands do under the hood:

1. **`mamba config --add channels conda-forge`**
   * This commands Mamba to add **Conda-Forge** to its search list. Conda-Forge is a massive community repository that contains thousands of general open-source software libraries, compilers, and dependencies. Many biology tools require these underlying libraries to run.

2. **`mamba config --add channels bioconda`**
   * This adds **Bioconda** to the search list. Bioconda is a specialized repository maintained by scientists, containing tens of thousands of pre-compiled bioinformatics software packages (including FastQC, Minimap2, and MetaPhlAn4). Without this, Mamba cannot find any biology-specific tools.

3. **`mamba config --add channel_priority strict`**
   * This is a crucial setting that tells Mamba to prioritize the channels in the exact order they are listed. This prevents version conflicts, makes installations incredibly stable, and speeds up download times.

</details>

### Step 3: Creating The Environments and Installing Tools
During the analysis, different tools may need different versions of programming languages, or some tools may need to run subsequently. To arrange this better, creating working environments are needed. It is recommended to create environments by their purpose (for example: binning, dehosting, pre-processing ect.), yet it can be named by the user's preferences.

#### Pre processing

For running the analysis properly, clean and complete data is needed. To achieve this; quality control, trimming and dehosting is necessary. As a frst step, run this code to create the environment and install the tools:

```bash
mamba create -n pre_processing_env fastqc trimmomatic bowtie2 samtools
```
This code commands mamba to create the environment and install the tools needed. Note that the "pre_processing_env" name can be changed according to your style.

Here's what the tools do,respectively:

1. FastQC: Generates visual quality control reports from your raw FASTQ files, helping you identify low-quality bases, adapter contamination, and sequence duplication levels.
2. Trimmomatic: Used to trim off low-quality bases and remove adapter sequences.
3. Bowtie2: A fast memory-efficient tool used to align your sequencing reads against a host reference genome (e.g., human or host animal) so host DNA can be separated from microbial DNA.
4. Samtools: A foundational tool used to process, filter, and sort the alignment files (SAM/BAM formats) generated by Bowtie2, allowing to extract only the unmapped (microbial) reads.  

#### Profiling

Shotgun metagenomics analysis can be done in two ways: Assembly free, which depend on the donwloaded databases and assembly-based,which is a *de novo* gene asembly by binning the reads by their similarities.

##### Assembly-Free (Read-Based) Profiling

We split this analysis into two distinct paths depending on the tools you wish to use.

###### **Path A: Kraken2 & Bracken Environment (`kraken_env`)**

This environment utilizes a high-speed, k-mer-based approach to classify reads and estimate exact species abundances.

> **Hardware Warning:** Running Kraken2 requires downloading a reference database (e.g., Standard, PlusPF, or MiniKraken). Standard databases can easily require 40GB to 100GB+ of RAM to load.If you are on a starter system, you may need to use the downsized *MiniKraken* database.

To create the environment and install the tools:

```bash
mamba create -n kraken_env kraken2 bracken

```
What these tools do:
1. Kraken2: Rapidly assigns taxonomic labels to short DNA reads by matching k-mers (short sequence fragments) against a local genomic database.
2. Bracken (Bayesian Reestimation of Abundance with Cracken): A companion program that mathematically processes Kraken2's output to estimate highly accurate, probabilistic relative abundances at the species or strain level.

###### **Path B: MetaPhlAn4 & HUMAnN3 Environment (`biobakery_env`)**

If you want highly precise taxonomic profiling alongside deep functional and metabolic pathway analysis without massive RAM requirements, this environment is recommended.

To create the environment and install the tools, run the code below:

```bash
mamba create -n biobakery_env metaphlan humann3

```
What these tools do:
1. MetaPhlAn4: Profiles the taxonomic composition of your sample using unique clade-specific marker genes. This method produces very few false positives and runs perfectly on standard computing setups.
2. HUMAnN3: Tool for characterizing the functional potential of the microbial community. It maps reads to gene families and compiles them into active metabolic pathways, telling what they are capable of doing.

















































