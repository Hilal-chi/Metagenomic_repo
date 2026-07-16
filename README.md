# Metagenomic_repo
Bu repo çevrimiçi bir metagenomik rehberi. Henüz düzenleme aşamasında.

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

## **Running The Analysis**

After installing the manager, tools and the environments, analysis can be started. Details are explained below.

### Pre Processing

Pre processing starts with quality control and continues with taxonomic profiling based on local (downloaded on PC) computers. To start the analysis, follow the steps below respectively.

#### Quality Control

To get a clean data, checking the reading quality and trimming (if necessary) is crucial. For quality control, FASTQC from Babraham Bioinformatics is recommended. Below is the code for paired ended, multiple samples.

```bash
mamba activate pre_processing_env

# Define your main project directory
WORKING_DIR="/path/to/your/project"

# Define where your raw FASTQ files are stored
RAW_DATA_DIR="${WORKING_DIR}/data/raw_reads"

# Define where you want to save your QC outputs
QC_OUT_DIR="${WORKING_DIR}/results/qc_reports"

# Define your file suffix (e.g., _R1_001.fastq.gz, _1.fq.gz, etc.)
FORWARD_SUFFIX="_R1.fastq.gz"
REVERSE_SUFFIX="_R2.fastq.gz"

# Create the output directory if it doesn't exist
mkdir -p "${QC_OUT_DIR}"

# Change directory to where your raw reads are
cd "${RAW_DATA_DIR}"

# Start the loop for each forward read file found
for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do
    
    # Extract the base sample name by removing the suffix
    # e.g., "sample1_R1.fastq.gz" becomes "sample1"
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")
    
    # Define the matching reverse file name
    REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"
    
    # Print a status message to the terminal to track progress
    echo "=========================================="
    echo "Processing Quality Control for: ${SAMPLE_NAME}"
    echo "=========================================="
    
    # Run FastQC on both pair-end files
    fastqc -o "${QC_OUT_DIR}" "${FORWARD_FILE}" "${REVERSE_FILE}"
    
done
```
To understand how these codes work, you can see the details.
<details>

### 1. Variables & Paths Setup
Setting up variables at the beginning of your script makes it incredibly easy to modify later if your file names or directories change.

* **`WORKING_DIR="..."`**: This is main project folder chosen by user. Using a single base folder path prevents you from wiring the direction repeatedly.
* **`RAW_DATA_DIR="${WORKING_DIR}/data/raw_reads"`**: Tells the script where data is located. The `${WORKING_DIR}` syntax dynamically inserts your main project path here.
* **`QC_OUT_DIR="${WORKING_DIR}/results/qc_reports"`**: Defines the target directory where FastQC will save your visual reports.
* **`FORWARD_SUFFIX="_R1.fastq.gz"` and `REVERSE_SUFFIX="_R2.fastq.gz"`**: These define the file name patterns. If your sequencing center names files differently (e.g., `_1.fq.gz` instead of `_R1.fastq.gz`), you need to change it in this line.
* **`mkdir -p "${QC_OUT_DIR}"`**: The "make direction" command in terminal. The `-p` flag prevents the codes causing an error if the direction already exists.

### 2. The Bash Loop Breakdown
Instead of manually typing the FastQC command for every single sample, using a `for` loop to automate the process sequentially is recommended.

* **`cd "${RAW_DATA_DIR}"`**: Moves your active terminal session directly into the folder where your raw sequence files live.
* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**: 
  * The asterisk `*` is used for commanding the terminal to find every file in this folder that ends with the suffix (e.g., `_R1.fastq.gz`) given, and process them one by one. During each turn of the loop, call the current file `FORWARD_FILE`."*
* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**:
  * **`basename`** is a Linux command that strips away folder paths and file extensions. It can be removed if full names are needed present.
  * This line takes a file like `/path/raw_reads/sample01_R1.fastq.gz`, strips off the suffix `_R1.fastq.gz`, and saves just the clean name `sample01` inside the variable `SAMPLE_NAME`.
* **`REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"`**: 
  * After the clean name is specified (e.g., `sample01`), this line automatically reconstructs the matching reverse read file name by appending the reverse suffix (resulting in `sample01_R2.fastq.gz`).
* **`echo "Processing Quality Control for: ${SAMPLE_NAME}"`**: 
  * Prints a clear progress message to your terminal. This is crucial for tracking long-running pipelines so you know exactly which sample is currently being analyzed.
* **`fastqc -o "${QC_OUT_DIR}" "${FORWARD_FILE}" "${REVERSE_FILE}"`**:
  * **`fastqc`**: The tool itself. Writing it runs the tool.
  * **`-o "${QC_OUT_DIR}"`**: Tells FastQC to save the output files inside your predefined QC directory instead of cluttering your raw data folder. -o represents "output".
  * **`"${FORWARD_FILE}" "${REVERSE_FILE}"`**: Passes both the forward and reverse paired-end files into FastQC for analysis.
* **`done`**: Tells the terminal that the commands for the current sample are finished, and instructs it to go back to the top of the loop to process the next sample.
</details>















































