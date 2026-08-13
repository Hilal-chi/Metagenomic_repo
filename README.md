# Metagenomic_repo
Bu repo çevrimiçi bir metagenomik rehberi. Henüz düzenleme aşamasında.

# **SHOTGUN METAGENOMICS GUIDELINE**

## **Purpose Of This Repository**

This repository is a coding guideline for starters in shotgun metagenomics data analysis. It includes codes of installing tools, creating environments and running analyses. If you're new to metagenomic analyses, you can see the [Metagenomic guideline](https://docs.google.com/document/d/1kU-FlgxgB0zssmJtbSXxJsxBNo2GGQfwhAVUqOSdKl4/edit?tab=t.0) for more information.

> Note: This repository is for Ubuntu/Debian systems only. If you're using other oprating systems such as Mac or Windows, you can either install WSL for Windows or use Mac terminal instead.

## Installation & Environment Setup

This pipeline requires specific bioinformatics tools. To ensure all tools run smoothly without library conflicts, using **Conda** or **Mamba** is suggested.

> Mamba works exactly like Conda, yet Mamba resolves package dependencies in parallel, completing your installations faster. If Conda is not installed in your computer yet, Mamba is recommended. If you have Conda, you can use the same script with writing "conda" isntead of "mamba".


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

#### Trimming The Samples

After examining the quality reports, it can be seen that some samples are below the threshold quality (usually Q25 or Q28), so trimming is recommended to exlude low-quality reads. Since samples can be either single ended or paired ended; the codes below has options for both, including single and multiple samples for each.


```bash

# Make sure your preprocessing environment is active before running these commands:
conda activate pre_processing_env
```

> When is trimming necessary?
> 
> Check your FastQC reports. If your sequencing data shows high-quality Phred scores (safely above Q28) and no adapter contamination, minimal trimming -or no trimming at all- may be treated as optional depending on your downstream analysis requirements. Hence, a literature review is also recommended to determinate the parameters.

Below is the complete command block for trimming samples, titled with read types and sample number.

> Note: This trimming codes are written for trimmomatic only.

##### 1. Defining Variables & Paths

Define your project directories and adapter paths first to keep your scripts clean and reusable. Defining each type of output in a different folder is recommended to follow the steps and files are recommended.

```bash
# Define project directories
WORKING_DIR="/path/to/your/project"
RAW_DATA_DIR="${WORKING_DIR}/data/raw_reads"
TRIM_OUT_DIR="${WORKING_DIR}/results/trimmed_reads"

# Define file suffixes
FORWARD_SUFFIX="_R1.fastq.gz"
REVERSE_SUFFIX="_R2.fastq.gz"
SE_SUFFIX=".fastq.gz"

# Define path to the adapter FASTA file (bundled with Trimmomatic/Conda)
ADAPTERS_PE="${CONDA_PREFIX}/share/trimmomatic/adapters/TruSeq3-PE.fa"
ADAPTERS_SE="${CONDA_PREFIX}/share/trimmomatic/adapters/TruSeq3-SE.fa"

# Create output folder
mkdir -p "${TRIM_OUT_DIR}"

```
##### 2. Running Trimmomatic on a Single Sample

###### Option A: Paired-End Mode (PE)

For paired-end sequencing, Trimmomatic produces four output files: two for reads where both mates survived trimming (paired), and two for reads where one mate failed trimming (unpaired).

```bash
SAMPLE_NAME="sample_01"

trimmomatic PE -threads 4 \
  "${RAW_DATA_DIR}/${SAMPLE_NAME}${FORWARD_SUFFIX}" \
  "${RAW_DATA_DIR}/${SAMPLE_NAME}${REVERSE_SUFFIX}" \
  "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_paired.fq.gz" \
  "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_unpaired.fq.gz" \
  "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_paired.fq.gz" \
  "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_unpaired.fq.gz" \
  ILLUMINACLIP:"${ADAPTERS_PE}":2:30:10 \
  LEADING:3 \
  TRAILING:3 \
  SLIDINGWINDOW:4:15 \
  MINLEN:36

```

###### Option B: Single-End Mode (SE)

For single-end data, Trimmomatic takes one input file and generates a single trimmed output file:

```bash
SAMPLE_NAME="sample_01"

trimmomatic SE -threads 4 \
  "${RAW_DATA_DIR}/${SAMPLE_NAME}${SE_SUFFIX}" \
  "${TRIM_OUT_DIR}/${SAMPLE_NAME}_trimmed.fq.gz" \
  ILLUMINACLIP:"${ADAPTERS_SE}":2:30:10 \
  LEADING:3 \
  TRAILING:3 \
  SLIDINGWINDOW:4:15 \
  MINLEN:36

```

---

##### 3. Automated Processing for All Samples Using a Loop

###### Option A: Paired-End Loop (PE)

To process an entire paired-end dataset sequentially using consistent quality parameters:

```bash
cd "${RAW_DATA_DIR}"

for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do

    # Extract base sample identifier
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")
    REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"

    echo "=========================================="
    echo "Trimming Paired-End reads for: ${SAMPLE_NAME}"
    echo "=========================================="

    trimmomatic PE -threads 4 \
      "${FORWARD_FILE}" \
      "${REVERSE_FILE}" \
      "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_paired.fq.gz" \
      "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_unpaired.fq.gz" \
      "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_paired.fq.gz" \
      "${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_unpaired.fq.gz" \
      ILLUMINACLIP:"${ADAPTERS_PE}":2:30:10 \
      LEADING:3 \
      TRAILING:3 \
      SLIDINGWINDOW:4:15 \
      MINLEN:36

done

```

###### Option B: Single-End Loop (SE)

To process an entire single-end dataset sequentially:

```bash
cd "${RAW_DATA_DIR}"

for SE_FILE in *"${SE_SUFFIX}"; do

    # Extract base sample identifier
    SAMPLE_NAME=$(basename "${SE_FILE}" "${SE_SUFFIX}")

    echo "=========================================="
    echo "Trimming Single-End read for: ${SAMPLE_NAME}"
    echo "=========================================="

    trimmomatic SE -threads 4 \
      "${SE_FILE}" \
      "${TRIM_OUT_DIR}/${SAMPLE_NAME}_trimmed.fq.gz" \
      ILLUMINACLIP:"${ADAPTERS_SE}":2:30:10 \
      LEADING:3 \
      TRAILING:3 \
      SLIDINGWINDOW:4:15 \
      MINLEN:36

done

```
To understand how these codes work, you can see the details below.

<Details>
  
### 1. Variables & Path Setup Line-by-Line

* **`ADAPTERS_PE="${CONDA_PREFIX}/share/trimmomatic/adapters/TruSeq3-PE.fa"`**:
* `${CONDA_PREFIX}` is a built-in variable that points to your currently active Conda/Mamba environment folder.
* This line automatically locates the reference FASTA file containing Illumina adapter sequences that was installed alongside Trimmomatic.


* **`mkdir -p "${TRIM_OUT_DIR}"`**: Creates the output directory specified in `${TRIM_OUT_DIR}` if it does not already exist, suppressing errors if it does. mkdir stands for "make direction" while -p is for supressing the error message.


### 2. Paired-End Loop Breakdown

* **`cd "${RAW_DATA_DIR}"`**: Navigates your terminal session into the directory where your raw FASTQ files reside.
* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**:
* Finds every file in the directory matching the forward suffix (e.g., `_R1.fastq.gz`).
* Repeats the command for each sample, storing the current file path in the variable `${FORWARD_FILE}` during each cycle.


* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**:
* Uses `basename` to strip away folder paths and the trailing forward suffix.
* Converts a full filename like `sample01_R1.fastq.gz` into just the clean identifier `sample01`.


* **`REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"`**: Reconstructs the corresponding reverse read filename (e.g., `sample01_R2.fastq.gz`) by combining the clean sample name with the reverse suffix variable.
* **`echo "=========================================="`**: Prints visual separation lines and status updates in the terminal window to check which sample is executed.
* **`trimmomatic PE -threads 4`**:
* **`trimmomatic`**: Calls the main software executable.
* **`PE`**: Sets the execution mode to **Paired-End** processing.
* **`-threads 4`**: Restricts CPU allocation to 4 processing cores. This can be changed according to your work station's properties.


* **`"${FORWARD_FILE}" "${REVERSE_FILE}"`**: Passes the input raw forward and reverse FASTQ files into the program.
* **`"${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_paired.fq.gz"`**: Output file #1. Contains forward reads whose reverse pairs also survived trimming. (Used for downstream analysis).
* **`"${TRIM_OUT_DIR}/${SAMPLE_NAME}_R1_unpaired.fq.gz"`**: Output file #2. Contains forward reads that passed trimming, but whose matching reverse reads failed.
* **`"${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_paired.fq.gz"`**: Output file #3. Contains reverse reads whose forward pairs also survived trimming. (Used for downstream analysis).
* **`"${TRIM_OUT_DIR}/${SAMPLE_NAME}_R2_unpaired.fq.gz"`**: Output file #4. Contains reverse reads that passed trimming, but whose matching forward reads failed.
* **`ILLUMINACLIP:"${ADAPTERS_PE}":2:30:10`**:
* Tells Trimmomatic to search for and clip Illumina adapter sequences using the supplied FASTA file.
* **`2`**: Allows a maximum of 2 seed mismatches.
* **`30`**: Specifies the score threshold required to clip adapters in paired reads.
* **`10`**: Specifies the score threshold required to clip adapters in single reads.
  
>**Note**: These values are for example only. Please check the numbers before running the analysis and change if necessary.

* **`LEADING:15`**: Scans the start (5' end) of each read and removes bases if their Phred quality score is below 15.
* **`TRAILING:15`**: Scans the end (3' end) of each read and removes bases if their Phred quality score is below 15.
* **`SLIDINGWINDOW:4:15`**:
* Moves a sliding window of **4 bases** across the read.
* Cuts the read off permanently at the point where the average quality score inside that 4-base window falls below 15.
* **`MINLEN:36`**: Evaluates the total length of the read after all trimming steps are applied and discards any read shorter than 36 base pairs.
* **`done`**: Marks the end of the loop block and moves to the next sample file in the folder.

>**Note**: These values are also for example only. Please check the numbers before running the analysis and change if necessary.

### 3. Single-End Loop Differences Breakdown

* **`for SE_FILE in *"${SE_SUFFIX}"; do`**: Looks for files matching the single-end extension (e.g., `.fastq.gz`) and assigns each to `${SE_FILE}`.
* **`trimmomatic SE -threads 4`**: Sets the execution mode to **Single-End** processing.
* **`"${SE_FILE}"`**: Provides the single raw input FASTQ file.
* **`"${TRIM_OUT_DIR}/${SAMPLE_NAME}_trimmed.fq.gz"`**: Produces a single output file containing all surviving trimmed reads (unpaired files are not created in SE mode).

</Details>

#### Dehosting The Samples

##### Assembly Based Profiling

### 1. Defining Variables & Paths

Define your project directories, input read directories, and suffix patterns to keep your scripts structured, reproducible, and easy to adjust. Each sample assembled by metaSPAdes will generate a set of sub-directories, so storing outputs in a dedicated assembly folder is strongly recommended.

```bash
# Define project directories
WORKING_DIR="/path/to/your/project"
INPUT_DIR="${WORKING_DIR}/results/host_removed_reads"   # or /results/trimmed_reads
ASSEMBLY_OUT_DIR="${WORKING_DIR}/results/assemblies"

# Define file suffixes (matching your upstream QC/host removal outputs)
FORWARD_SUFFIX="_R1_paired.fq.gz"
REVERSE_SUFFIX="_R2_paired.fq.gz"

# Create base assembly output directory
mkdir -p "${ASSEMBLY_OUT_DIR}"

```

---

### 2. metaSPAdes Assembly Code

```bash
cd "${INPUT_DIR}"

for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do

    # Extract base sample identifier
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")
    REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"

    # Define sample-specific output subfolder
    SAMPLE_OUT_DIR="${ASSEMBLY_OUT_DIR}/${SAMPLE_NAME}"

    echo "=========================================="
    echo "Running metaSPAdes Assembly for: ${SAMPLE_NAME}"
    echo "=========================================="

    metaspades.py \
      -1 "${FORWARD_FILE}" \
      -2 "${REVERSE_FILE}" \
      -o "${SAMPLE_OUT_DIR}" \
      -t 8 \
      -m 64

done

```
To understand how these codes work, see the details below.

<Details>

#### **Variables & Path Setup**

* **`INPUT_DIR="${WORKING_DIR}/results/host_removed_reads"`**: Points to the directory holding quality-trimmed and human/host-filtered paired FASTQ files.
* **`ASSEMBLY_OUT_DIR="${WORKING_DIR}/results/assemblies"`**: The destination directory where metaSPAdes will create individual subfolders for each processed sample.
* **`mkdir -p "${ASSEMBLY_OUT_DIR}"`**: Creates the main assembly output directory if it does not already exist while suppressing warnings if it does.

#### **Paired-End Assembly Loop Breakdown**

* **`cd "${INPUT_DIR}"`**: Moves the active session into the input read directory.
* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**: Iterates over every forward read file matching the specified suffix pattern.
* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**: Strips the path and suffix from the filename to isolate the unique sample identifier (e.g., `sample01`).
* **`REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"`**: Reconstructs the corresponding reverse read filename.
* **`SAMPLE_OUT_DIR="${ASSEMBLY_OUT_DIR}/${SAMPLE_NAME}"`**: Generates a dedicated subfolder for each sample (e.g., `results/assemblies/sample01`), preventing metaSPAdes output files from overwriting each other.
* **`metaspades.py`**: Invokes the metaSPAdes wrapper script designed specifically for metagenomic assembly datasets with uneven species coverage.
* **`-1 "${FORWARD_FILE}"`**: Passes the paired forward reads (`R1`).
* **`-2 "${REVERSE_FILE}"`**: Passes the paired reverse reads (`R2`).
* **`-o "${SAMPLE_OUT_DIR}"`**: Specifies the output directory for all resulting assembly files and logs for the current sample.
* **`-t 8`**: Allocates 8 CPU threads to the process. Adjust this number according to your server hardware resources.
* **`-m 64`**: Sets the memory limit to 64 Gigabytes (GB). MetaSPAdes is memory-intensive; ensure this value is scaled to your available system RAM.
* **`done`**: Concludes the loop and proceeds to assemble the next sample in line.

</Details> 
