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

Metagenomic samples derived from host-associated environments (e.g., gut, saliva, skin, or tissue) often contain substantial amounts of host DNA. Dehosting is the process of mapping your sequencing reads against the host reference genome and extracting only the unmapped reads (the authentic microbial DNA) for downstream taxonomic and functional profiling.

Make sure your preprocessing environment is active before running the commands:

```bash
conda activate pre_processing_env

```

##### 1. Define Variables & System Paths

Define your project directories, host reference files, and thread allocations first. Using variables keeps the scripts modular and prevents directory path errors.

```bash
# Define base project directory
WORKING_DIR="/path/to/your/project"

# Define input and output directories
READS_DIR="${WORKING_DIR}/results/trimmed_reads"    # Or raw_reads if trimming was skipped
DEHOST_DIR="${WORKING_DIR}/results/dehosted_reads"

# Define path to the host reference index (.mmi for Minimap2 or .fasta)
HOST_REF="/path/to/host_reference/host_genome.mmi"

# Define file naming pattern/suffixes
FORWARD_SUFFIX="_R1_paired.fq.gz"
REVERSE_SUFFIX="_R2_paired.fq.gz"

# CPU Allocation: Adjust according to your machine (leave 2-4 cores free for system stability)
THREADS=8

# Create output folder
mkdir -p "${DEHOST_DIR}"

```

##### 2. Option A: Running Dehosting on a Single Sample (Paired-End)

To process a single sample, pipe the sequence alignment directly into Samtools to filter and sort the unmapped microbial reads without storing massive uncompressed intermediate SAM files:

```bash
SAMPLE_NAME="sample_01"

echo "=========================================="
echo "Running Dehosting for: ${SAMPLE_NAME}"
echo "=========================================="

# 1. Align reads against the host genome and extract unmapped reads
minimap2 -ax sr -t ${THREADS} "${HOST_REF}" \
  "${READS_DIR}/${SAMPLE_NAME}${FORWARD_SUFFIX}" \
  "${READS_DIR}/${SAMPLE_NAME}${REVERSE_SUFFIX}" | \
samtools view -@ ${THREADS} -b -f 12 -F 256 | \
samtools sort -@ ${THREADS} -n -o "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"

# 2. Convert the unmapped BAM back into paired FASTQ files
bedtools bamtofastq \
  -i "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam" \
  -fq "${DEHOST_DIR}/${SAMPLE_NAME}_R1.dehost.fq" \
  -fq2 "${DEHOST_DIR}/${SAMPLE_NAME}_R2.dehost.fq"

# 3. Compress the resulting FASTQ files to save storage
gzip "${DEHOST_DIR}/${SAMPLE_NAME}_R1.dehost.fq"
gzip "${DEHOST_DIR}/${SAMPLE_NAME}_R2.dehost.fq"

# 4. (Optional) Remove the temporary unmapped BAM file
# rm "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"

```

##### 3. Option B: Automated Processing for All Samples Using a Loop

To process an entire cohort of samples sequentially:

```bash
cd "${READS_DIR}"

for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do

    # Extract clean sample identifier
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")
    REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"

    echo "=========================================="
    echo "Processing Dehosting for: ${SAMPLE_NAME}"
    echo "=========================================="

    # Step A: Align and filter out host reads
    minimap2 -ax sr -t ${THREADS} "${HOST_REF}" \
      "${FORWARD_FILE}" \
      "${REVERSE_FILE}" | \
    samtools view -@ ${THREADS} -b -f 12 -F 256 | \
    samtools sort -@ ${THREADS} -n -o "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"

    # Step B: Convert unmapped BAM to paired FASTQ files
    bedtools bamtofastq \
      -i "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam" \
      -fq "${DEHOST_DIR}/${SAMPLE_NAME}_R1.dehost.fq" \
      -fq2 "${DEHOST_DIR}/${SAMPLE_NAME}_R2.dehost.fq"

    # Step C: Compress the clean microbial reads
    gzip "${DEHOST_DIR}/${SAMPLE_NAME}_R1.dehost.fq"
    gzip "${DEHOST_DIR}/${SAMPLE_NAME}_R2.dehost.fq"

    # Step D: (Optional) Clean up intermediate BAM files to save disk space
    rm "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"

done

```

To understand how these codes work, see the details below:

<details>

### 1. Variable Assignment & Path Syntax

* **`THREADS=8`**: Restricts the maximum number of CPU cores used by the pipeline.
> *Important:* If your machine has 16 or 32 cores, always leave 2–4 cores free so your operating system does not freeze or crash during intensive alignment tasks.
* **`HOST_REF="/path/to/reference.mmi"`**: Points to the host reference index file.
* *Note:* Do not leave spaces around the `=` sign in Bash variable assignments (e.g., `HOST_REF = "..."` will fail). Always wrap file paths in double quotes `""` to prevent path breaking if directory names contain spaces or special characters.

### 2. Alignment & Filtering

The first command chains three tools together using Linux pipes (`|`), passing data in memory without writing massive intermediate files to your storage drive:

* **`minimap2`**: The high-speed alignment software.
* **`-ax sr`**: `-a` generates output in SAM format; `sr` specifies that the input consists of short paired-end sequencing reads.
* **`-t ${THREADS}`**: Allocates the specified number of processing threads to the alignment engine.
* **`"${HOST_REF}"`**: The reference database of the host organism (e.g., Human, Mouse, Avian).
* **`"${FORWARD_FILE}" "${REVERSE_FILE}"`**: Passes the paired input FASTQ files.

* **`| samtools view -@ ${THREADS} -b -f 12 -F 256`**:
* **`|` (Pipe)**: Directly sends the output from `minimap2` into `samtools view` in RAM without writing an uncompressed multi-gigabyte SAM file to the disk.
* **`-@ ${THREADS}`**: Allocates processing threads for compression and filtering.
* **`-b`**: Directs Samtools to output binary-compressed BAM format instead of plain text SAM.
* **`-f 12` (Include Flag)**: Extracts read pairs where both read 1 and read 2 failed to align to the host reference.
* *Flag math:* Bit `4` (Read unmapped) + Bit `8` (Mate unmapped) = Flag `12`. This guarantees that no host fragments remain.
* **`-F 256` (Exclude Flag)**: Excludes secondary and non-primary alignments to prevent redundant mapping artifacts.

* **`| samtools sort -@ ${THREADS} -n -o "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"`**:
* **`samtools sort`**: Sorts the filtered read records.
* **`-n`**: Sorts reads **by name** rather than by genomic coordinates.
> ⚠️ *Critical:* `bedtools bamtofastq` requires name-sorted BAM files to properly pair forward (`R1`) and reverse (`R2`) mates back together!
* **`-o ...`**: Specifies the path and filename for the final sorted BAM output.

### 3. BAM to FASTQ Conversion & Compression

* **`bedtools bamtofastq`**: The conversion utility from Bedtools.
* **`-i ...unmapped.bam`**: The input unmapped BAM file containing clean microbial reads.
* **`-fq ..._R1.dehost.fq`**: Output path for the forward microbial reads.
* **`-fq2 ..._R2.dehost.fq`**: Output path for the reverse microbial reads.
* **`gzip "${DEHOST_DIR}/${SAMPLE_NAME}_R*.dehost.fq"`**:
* Metagenomic FASTQ files can quickly reach hundreds of gigabytes.
* `gzip` compresses the text files into standard `.fq.gz` archives, drastically reducing disk space requirements while remaining directly readable by downstream tools like MetaPhlAn4, HUMAnN3, or Kraken2.


* **`rm "${DEHOST_DIR}/${SAMPLE_NAME}.unmapped.bam"`**:
* Once the reads have been converted back to `.fq.gz` format, the intermediate BAM files are no longer needed. Removing them prevents storage bottlenecks on large cohorts.

</details>


#### Taxonomic Profiling with MetaPhlAn4

Now that host-derived genomic sequences have been removed, clean microbial reads are ready for taxonomic classification. MetaPhlAn4 (Metagenomic Phylogenetic Analysis) profiles the composition of microbial communities (Bacteria, Archaea, Eukaryotes, and Viruses) at the species and strain levels using unique clade-specific marker genes.

Make sure your taxonomic profiling environment is active before running these commands:
```bash
conda activate biobakery_env

```

##### 1. Define Variables & System Paths

Define your project directories, file suffixes, and core limits first to keep the execution modular and clean:

```bash
# Define base project directory
WORKING_DIR="/path/to/your/project"

# Define input and output directories
DEHOST_DIR="${WORKING_DIR}/results/dehosted_reads"
TAX_OUT_DIR="${WORKING_DIR}/results/taxonomy_metaphlan"

# Define input file suffixes (clean microbial reads)
FORWARD_SUFFIX="_R1.dehost.fq.gz"
REVERSE_SUFFIX="_R2.dehost.fq.gz"

# CPU Allocation: Restrict processor cores to maintain system stability
NPROC=6

# Create output folder
mkdir -p "${TAX_OUT_DIR}"

```

##### 2. Running MetaPhlAn4 on a Single Sample (Paired-End)

To profile a single sample, we stream both paired-end files simultaneously using Linux process substitution (`<()`) and `zcat`:

```bash
SAMPLE_NAME="sample_01"

echo "=========================================="
echo "Running MetaPhlAn4 for: ${SAMPLE_NAME}"
echo "=========================================="

metaphlan \
  --input_type fastq \
  -t rel_ab_w_read_stats \
  --nproc ${NPROC} \
  -o "${TAX_OUT_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt" \
  <(zcat "${DEHOST_DIR}/${SAMPLE_NAME}${FORWARD_SUFFIX}" \
         "${DEHOST_DIR}/${SAMPLE_NAME}${REVERSE_SUFFIX}")

```

##### 3. Automated Processing for All Samples Using a Loop

To sequentially analyze an entire batch of dehosted samples:

```bash
cd "${DEHOST_DIR}"

for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do

    # Extract base sample identifier
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")
    REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"

    echo "=========================================="
    echo "Processing taxonomy for: ${SAMPLE_NAME}"
    echo "=========================================="

    metaphlan \
      --input_type fastq \
      -t rel_ab_w_read_stats \
      --nproc ${NPROC} \
      -o "${TAX_OUT_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt" \
      <(zcat "${FORWARD_FILE}" "${REVERSE_FILE}")

done

```

To understand how these codes work, see the details below:

<details> 
  
### 1. Setup & Loop Navigation Line-by-Line

* **`cd "${DEHOST_DIR}"`**: Navigates into the folder containing your clean, host-filtered microbial FASTQ files.
* **`mkdir -p "${TAX_OUT_DIR}"`**: Creates the output directory for taxonomy profiles. The `-p` flag ensures no error is thrown if the directory already exists.
* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**: Iterates through all forward read files in the directory matching the specified suffix pattern.
* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**: Strips the path and the forward suffix from the file name, extracting only the base sample identifier (e.g., `sample_01`).
* **`echo "Processing taxonomy for: ${SAMPLE_NAME}"`**: Prints a progress notification to the terminal to track live execution during lengthy computational runs.

### 2. MetaPhlAn4 Parameters Breakdown

* **`metaphlan`**: Executes the MetaPhlAn software suite.
* **`--input_type fastq`**: Informs the program that the raw input streams are in standard FASTQ format.
* **`-t rel_ab_w_read_stats`**:
* Sets the profiling type to Relative Abundance with Read Statistics.
* Generates not only the relative percentages of detected taxa but also reports estimated read counts and coverage stats for each clade. These numerical counts are required for downstream statistical models in R (e.g., Phyloseq, MaAsLin2).
* **`--nproc ${NPROC}`**: Limits the number of CPU processing cores used during alignment to maintain system responsiveness.
* **`-o "${TAX_OUT_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt"`**: Specifies the exact destination path and file name for the final plain-text taxonomic profile table.

### 3. Streaming Reads with Process Substitution (`zcat` & `<(...)`)

* **`zcat "${FORWARD_FILE}" "${REVERSE_FILE}"`**:
* Uncompresses and concatenates the forward (`R1`) and reverse (`R2`) files sequentially on-the-fly directly into RAM without decompressing gigabytes of data onto the physical hard drive.
* **`<(...)` (Process Substitution)**:
* Creates a temporary Unix file descriptor (a stream pipe in memory) that presents the combined `zcat` output to MetaPhlAn as if it were a single, seamless input file.
* This bypasses format incompatibility errors that can arise when handling paired-end reads individually.

</details>


#### Functional Profiling with HUMAnN3

Now that the taxonomic composition of microbial community determinated, the next step is to profile its metabolic and functional potential. HUMAnN3 (HMP Unified Metabolic Analysis Network 3) characterizes the biological capabilities of a community by mapping cleaned reads to gene families and quantifying complete metabolic pathways (e.g., UniRef gene families, MetaCyc metabolic pathways).

Make sure your functional profiling environment is active before running these commands:
```bash
conda activate biobakery_env

```

##### 1. Define Variables, Databases & System Paths

HUMAnN3 relies on three core reference databases (ChocoPhlAn, UniRef, and utility mapping). Define your project directories, database paths, and processor allocations first:

```bash
# Define base project directory
WORKING_DIR="/path/to/your/project"

# Define input and output directories
DEHOST_DIR="${WORKING_DIR}/results/dehosted_reads"
TAX_DIR="${WORKING_DIR}/results/taxonomy_metaphlan"
FUNC_OUT_DIR="${WORKING_DIR}/results/functional_humann3"

# Define local paths to HUMAnN3 reference databases
DB_DIR="/path/to/humann3_databases"
CHOCOPHLAN_DB="${DB_DIR}/chocophlan"
UNIREF_DB="${DB_DIR}/uniref"
UTILITY_DB="${DB_DIR}/utility_mapping"

# Define file naming pattern/suffixes
FORWARD_SUFFIX="_R1.dehost.fq.gz"

# CPU Allocation: Adjust according to system capacity
THREADS=8

# Create output folder
mkdir -p "${FUNC_OUT_DIR}"

```

##### 2. Running HUMAnN3 on a Single Sample

To run functional profiling on a single sample:

```bash
SAMPLE_NAME="sample_01"

echo "=========================================="
echo "Running HUMAnN3 for: ${SAMPLE_NAME}"
echo "=========================================="

humann \
  --input "${DEHOST_DIR}/${SAMPLE_NAME}${FORWARD_SUFFIX}" \
  --output "${FUNC_OUT_DIR}/${SAMPLE_NAME}_functional" \
  --taxonomic-profile "${TAX_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt" \
  --nucleotide-database "${CHOCOPHLAN_DB}" \
  --protein-database "${UNIREF_DB}" \
  --utility-database "${UTILITY_DB}" \
  --threads ${THREADS}

```

##### 3. Automated Processing for All Samples Using a Loop

To sequentially run functional profiling across all dehosted samples in your dataset:

```bash
cd "${DEHOST_DIR}"

for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do

    # Extract base sample identifier
    SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")

    echo "============================================="
    echo "LAUNCHING HUMAnN3 FOR: ${SAMPLE_NAME}"
    echo "============================================="

    humann \
      --input "${FORWARD_FILE}" \
      --output "${FUNC_OUT_DIR}/${SAMPLE_NAME}_functional" \
      --taxonomic-profile "${TAX_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt" \
      --nucleotide-database "${CHOCOPHLAN_DB}" \
      --protein-database "${UNIREF_DB}" \
      --utility-database "${UTILITY_DB}" \
      --threads ${THREADS}

done

```

To understand how each parameter, custom database, and paired-end handling option works, expand the details below:

<details> 

### 1. Variables & Database Setup Line-by-Line

* **`CHOCOPHLAN_DB`**: Points to the pangenome nucleotide database (ChocoPhlAn) used for initial tiered mapping.
* **`UNIREF_DB`**: Points to the protein reference database (e.g., UniRef90 or UniRef50) used for translated homology searches of reads that fail nucleotide-level alignment.
* **`UTILITY_DB`**: Points to mapping tables that translate raw UniRef IDs into structured metabolic classifications, such as MetaCyc pathways, KEGG Orthologies (KO), or GO terms.
* **`mkdir -p "${FUNC_OUT_DIR}"`**: Creates the main functional results directory if it does not already exist.

### 2. Script Execution & Input Parsing

* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**: Iterates through all clean forward read files matching the target suffix pattern.
* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**: Extracts the pure sample identifier (e.g., `sample_01`) from the input file path.
* **`humann`**: Executes the HUMAnN3 functional quantification workflow.
* **`--input "${FORWARD_FILE}"`**:
* Provides the sequencing read input file to the profiler.
* *Paired-End Processing Note:* When providing the forward (`R1`) paired-end FASTQ file, HUMAnN3 automatically detects and incorporates matching reverse (`R2`) reads residing in the same directory under matching prefix naming schemes.

* **`--output "${FUNC_OUT_DIR}/${SAMPLE_NAME}_functional"`**:
* Creates an isolated subfolder per sample containing three primary output files:
1. `*_genefamilies.tsv`: Abundance of individual UniRef gene families.
2. `*_pathabundance.tsv`: Abundance of complete metabolic pathways.
3. `*_pathcoverage.tsv`: Coverage/confidence scores (0 to 1) for pathway completeness.

* **`--taxonomic-profile "${TAX_DIR}/${SAMPLE_NAME}_metaphlan_profile.txt"`**:
* Feeds the custom taxonomic profile generated by MetaPhlAn4 in the previous step into HUMAnN3.
* *Why this is critical:* Instead of scanning the entire global pangenome database, HUMAnN3 builds a custom, downsized pangenome restricted only to the specific microbial taxa verified to exist in that sample. This reduces processing time and computational memory overhead.

</details>


#### 📊 Table Merging & Abundance Normalization

In the previous step, HUMAnN3 generated individual output tables for each sample across isolated subdirectories. To prepare these functional matrices for cross-sample statistical analysis and visualization in R or Python, merging the individual pathway abundance tables into a single cohesive matrix and perform depth-normalization (CPM) is needed.

Make sure your functional profiling environment is active before running these commands:

```bash
conda activate biobakery_env

```

##### 1. Define Variables & Paths

Define your project directories and cohort identifiers first to keep the commands modular:

```bash
# Define base project directory and project identifier
WORKING_DIR="/path/to/your/project"
PROJECT_NAME="metagenome_project"

# Define functional analysis directories
FUNC_DIR="${WORKING_DIR}/results/functional_humann3"
FLAT_DIR="${FUNC_DIR}/flat_tables"

# Create a staging directory to aggregate individual sample tables
mkdir -p "${FLAT_DIR}"

```

##### 2. Aggregating, Merging & Normalizing Pathway Abundances

Execute the following workflow to stage table files, combine columns into a single matrix, and normalize sequencing depth across all samples:

```bash
# 1. Copy all individual sample pathway abundance tables into the staging directory
cp "${FUNC_DIR}"/*_functional/*_pathabundance.tsv "${FLAT_DIR}/"

# 2. Merge individual pathway abundance tables into a single matrix
humann_join_tables \
  --input "${FLAT_DIR}" \
  --output "${FUNC_DIR}/${PROJECT_NAME}_pathabundance_merged.tsv" \
  --file_name "pathabundance"

# 3. Normalize raw abundance counts to Copies Per Million (CPM)
humann_renorm_table \
  --input "${FUNC_DIR}/${PROJECT_NAME}_pathabundance_merged.tsv" \
  --output "${FUNC_DIR}/${PROJECT_NAME}_pathabundance_normalized_cpm.tsv" \
  --units cpm

```

To understand how file aggregation, matrix joining, and mathematical normalization work, see the details below:

<details> 

### 1. File Aggregation & Staging

* **`mkdir -p "${FLAT_DIR}"`**: Creates an intermediate staging directory (`flat_tables`). Isolating target tables in a dedicated folder ensures downstream merging tools process only relevant files.
* **`cp "${FUNC_DIR}"/*_functional/*_pathabundance.tsv "${FLAT_DIR}/"`**:
* **`cp`**: The standard Unix file copy utility.
* **`*` (Wildcard)**: Matches any character string. HUMAnN3 outputs results into isolated subdirectories (e.g., `sample_01_functional/`, `sample_02_functional/`). This command collects all `*_pathabundance.tsv` files across every sample subdirectory and places copies in the staging directory in a single operation.


### 2. Table Joining (`humann_join_tables`)

* **`humann_join_tables`**: A utility bundled within the HUMAnN3 suite designed to align tabular outputs across multiple samples. It matches identical gene family or pathway identifiers across files and aligns samples side-by-side as adjacent columns.
* **`--input "${FLAT_DIR}"`**: Specifies the directory containing the individual per-sample `.tsv` tables to be joined.
* **`--output "${FUNC_DIR}/${PROJECT_NAME}_pathabundance_merged.tsv"`**: Defines the target location and file name for the resulting merged multi-sample count matrix.
* **`--file_name "pathabundance"`**: Acts as an inclusion filter. It instructs the script to merge only files whose names include the string `pathabundance`, ignoring any unrelated temporary or intermediate files present in the folder.

---

### 3. Depth Normalization (`humann_renorm_table`)

* **`humann_renorm_table`**: A utility that rescales raw read-count abundance estimates to standardize sequencing depth variations across samples.
* **`--input ..._merged.tsv`**: Provides the unnormalized, merged raw abundance matrix as input.
* **`--output ..._normalized_cpm.tsv`**: Designates the final destination path for the normalized pathway matrix.
* **`--units cpm`**:
* Directs the software to compute **Copies Per Million (CPM)**.
* **Why Normalization is Essential:** Sequencing depth naturally fluctuates across library preparations and sequencing lanes. A sample sequenced to a depth of 40 million microbial reads will show roughly twice the raw counts for a metabolic pathway compared to a sample sequenced to 20 million reads, even if the underlying biological activity is identical. Rescaling each sample's total abundance sum to 1,000,000 (CPM) removes library-size artifacts and ensures data is ready for downstream statistical testing and visualization in R or Python.

</details>

### Assembly Based Profiling

###### 1. Defining Variables & Paths

Define your project directories and suffixes to keep your scripts structured, reproducible, and easy to adjust. Each sample assembled by metaSPAdes will generate a set of sub-directories, so storing outputs in a dedicated assembly folder is strongly recommended.

```bash
# Define project directories
WORKING_DIR="/path/to/your/project"
INPUT_DIR="${WORKING_DIR}/data/dehost"                 # Path to de-hosted, quality-filtered reads
ASSEMBLY_OUT_DIR="${WORKING_DIR}/results/assemblies"
QUAST_OUT_DIR="${WORKING_DIR}/results/quast_reports"

# Define file suffixes (matching your upstream host-removal outputs)
FORWARD_SUFFIX="_R1.dehost.fq.gz"
REVERSE_SUFFIX="_R2.dehost.fq.gz"

# Hardware resources
THREADS=16
MEM_GB=64
MIN_CONTIG_LEN=500

# Create base output directories
mkdir -p "${ASSEMBLY_OUT_DIR}" "${QUAST_OUT_DIR}"

```

---

##### 2. metaSPAdes Assembly & MetaQUAST Assessment Code

```bash
cd "${INPUT_DIR}"

# --- Step 1: De Novo Metagenome Assembly ---
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
      -t "${THREADS}" \
      -m "${MEM_GB}"

done

# --- Step 2: Assembly Quality Assessment (MetaQUAST) ---
cd "${ASSEMBLY_OUT_DIR}"

for SAMPLE_DIR in */; do

    # Strip trailing slash to extract sample name
    SAMPLE_NAME=$(basename "${SAMPLE_DIR}")
    CONTIGS_FILE="${SAMPLE_DIR}/contigs.fasta"
    SAMPLE_QUAST_DIR="${QUAST_OUT_DIR}/${SAMPLE_NAME}"

    if [ -f "${CONTIGS_FILE}" ]; then
        echo "=========================================="
        echo "Running MetaQUAST for: ${SAMPLE_NAME}"
        echo "=========================================="

        metaquast.py "${CONTIGS_FILE}" \
          -o "${SAMPLE_QUAST_DIR}" \
          -m "${MIN_CONTIG_LEN}" \
          --threads "${THREADS}" \
          --max-ref-number 0
    else
        echo "Warning: contigs.fasta not found in ${SAMPLE_DIR}, skipping..."
    fi

done

```

To understand how these codes work, see the details below:

<details>

### **Variables & Path Setup**

* **`INPUT_DIR="${WORKING_DIR}/data/dehost"`**: Points to the folder containing paired FASTQ reads that have passed quality trimming and host filtering.
* **`ASSEMBLY_OUT_DIR="${WORKING_DIR}/results/assemblies"`**: The destination directory where metaSPAdes generates isolated subfolders for each sample's assembly output.
* **`QUAST_OUT_DIR="${WORKING_DIR}/results/quast_reports"`**: The destination directory where MetaQUAST stores assembly metrics, summary tables, and interactive HTML dashboards.
* **`mkdir -p "${ASSEMBLY_OUT_DIR}" "${QUAST_OUT_DIR}"`**: Creates the assembly and QC directories if they do not exist while suppressing warnings if they do.


### **metaSPAdes Assembly Loop**

* **`cd "${INPUT_DIR}"`**: Navigates into the de-hosted FASTQ read directory.
* **`for FORWARD_FILE in *"${FORWARD_SUFFIX}"; do`**: Iterates through every forward read file matching the specified suffix pattern.
* **`SAMPLE_NAME=$(basename "${FORWARD_FILE}" "${FORWARD_SUFFIX}")`**: Strips directory paths and the forward suffix to extract the clean sample identifier (e.g., `sample01`).
* **`REVERSE_FILE="${SAMPLE_NAME}${REVERSE_SUFFIX}"`**: Reconstructs the paired reverse read filename.
* **`SAMPLE_OUT_DIR="${ASSEMBLY_OUT_DIR}/${SAMPLE_NAME}"`**: Sets a unique subfolder per sample (e.g., `results/assemblies/sample01`), preventing output collisions.
* **`metaspades.py`**: Runs the metagenome-optimized SPAdes assembler designed to resolve uneven coverage depths.
* **`-1 "${FORWARD_FILE}"` / `-2 "${REVERSE_FILE}"**`: Passes the paired forward (`R1`) and reverse (`R2`) host-filtered reads.
* **`-o "${SAMPLE_OUT_DIR}"`**: Designates the output directory for assembly graphs, contigs, and logs.
* **`-t "${THREADS}"`**: Allocates 16 CPU threads to accelerate graph construction.
* **`-m "${MEM_GB}"`**: Limits memory usage to 64 GB of RAM to prevent out-of-memory errors during assembly.

### **MetaQUAST Assessment Loop**

* **`cd "${ASSEMBLY_OUT_DIR}"`**: Enters the base assembly results directory.
* **`for SAMPLE_DIR in */; do`**: Loops over each individual sample assembly directory.
* **`SAMPLE_NAME=$(basename "${SAMPLE_DIR}")`**: Extracts the sample name identifier from the directory path.
* **`CONTIGS_FILE="${SAMPLE_DIR}/contigs.fasta"`**: Identifies the primary assembled contig file produced by metaSPAdes.
* **`SAMPLE_QUAST_DIR="${QUAST_OUT_DIR}/${SAMPLE_NAME}"`**: Defines a dedicated report folder for the sample.
* **`if [ -f "${CONTIGS_FILE}" ]; then ... fi`**: Verifies that the contig assembly exists before initiating quality metrics.
* **`metaquast.py`**: Executes the metagenome-tailored version of QUAST to evaluate community assemblies.
* **`-m "${MIN_CONTIG_LEN}"`**: Sets the minimum contig size cutoff (500 bp) to exclude very short, uninformative fragments from length metrics.
* **`--threads "${THREADS}"`**: Parallelizes alignment and statistical computation.
* **`--max-ref-number 0`**: Prevents MetaQUAST from attempting to download thousands of reference genomes from NCBI, executing reference-free evaluation to save runtime.

</details>

#### Optional Assembly Branch: Contig-Level Functional & ARG Annotation

##### 1. Defining Variables & Paths

This optional branch predicts open reading frames (ORFs) and profiles antibiotic resistance genes (ARGs) directly across all raw assembled contigs. It catalogs functional genes from low-coverage or fragmented contigs that may not be recovered into final binned genomes.

```bash
# Define project directories
WORKING_DIR="/path/to/your/project"
ASSEMBLY_DIR="${WORKING_DIR}/results/assemblies"
GENES_OUT_DIR="${WORKING_DIR}/results/genes"
ARG_OUT_DIR="${WORKING_DIR}/results/arg"

# Reference database path (CARD or functional DB)
CARD_DB="${WORKING_DIR}/ref/CARD.dmnd"

# Parameters and resources
THREADS=16
MIN_ID=70
EVALUE="1e-5"

# Create output directories
mkdir -p "${GENES_OUT_DIR}" "${ARG_OUT_DIR}"

```

##### 2. Contig Gene Prediction & ARG Annotation Code

```bash
cd "${ASSEMBLY_DIR}"

for SAMPLE_DIR in */; do

    # Strip trailing slash to extract sample name
    SAMPLE_NAME=$(basename "${SAMPLE_DIR}")
    CONTIGS_FILE="${SAMPLE_DIR}/contigs.fasta"

    # Define output files
    PROTEIN_FAA="${GENES_OUT_DIR}/${SAMPLE_NAME}.proteins.faa"
    GENE_FNA="${GENES_OUT_DIR}/${SAMPLE_NAME}.genes.fna"
    ARG_TSV="${ARG_OUT_DIR}/${SAMPLE_NAME}.blastp.tsv"

    if [ -f "${CONTIGS_FILE}" ]; then
        echo "=========================================="
        echo "Predicting Genes (Prodigal) for: ${SAMPLE_NAME}"
        echo "=========================================="

        prodigal \
          -i "${CONTIGS_FILE}" \
          -a "${PROTEIN_FAA}" \
          -d "${GENE_FNA}" \
          -p meta

        echo "=========================================="
        echo "Searching ARGs (DIAMOND BLASTP) for: ${SAMPLE_NAME}"
        echo "=========================================="

        diamond blastp \
          -p "${THREADS}" \
          -q "${PROTEIN_FAA}" \
          -d "${CARD_DB}" \
          -o "${ARG_TSV}" \
          --max-target-seqs 1 \
          --evalue "${EVALUE}" \
          --id "${MIN_ID}"

    else
        echo "Warning: contigs.fasta not found in ${SAMPLE_DIR}, skipping..."
    fi

done

```

To understand how these codes work, see the details below:

<details> 

### **Variables & Path Setup**

* **`GENES_OUT_DIR="${WORKING_DIR}/results/genes"`**: Directory storing nucleotide gene sequences and translated protein files.
* **`ARG_OUT_DIR="${WORKING_DIR}/results/arg"`**: Directory storing tabular alignment results from ARG searches.
* **`CARD_DB="${WORKING_DIR}/ref/CARD.dmnd"`**: Path to the DIAMOND-formatted reference database (e.g., CARD).
* **`MIN_ID=70`**: Percentage sequence identity threshold required to keep a protein alignment hit.
* **`EVALUE="1e-5"`**: Statistical significance cutoff to remove spurious matches.


### **Gene Prediction & Alignment Loop **

* **`cd "${ASSEMBLY_DIR}"`**: Navigates into the assembly directory containing sample folders.
* **`prodigal`**: Calls the prokaryotic gene prediction software.
* **`-i "${CONTIGS_FILE}"`**: Passes the input contigs FASTA file.
* **`-a "${PROTEIN_FAA}"`**: Directs predicted protein translations to an amino acid FASTA file.
* **`-d "${GENE_FNA}"`**: Directs predicted gene sequences to a nucleotide FASTA file.
* **`-p meta`**: Runs Prodigal in metagenomic mode, allowing gene calls on fragmented sequences without single-organism training models.
* **`diamond blastp`**: Runs high-speed protein-versus-protein alignment against the reference database.
* **`-p "${THREADS}"`**: Allocates CPU cores to parallelize alignment searches.
* **`-q "${PROTEIN_FAA}"`**: Uses Prodigal's predicted protein FASTA as the alignment query.
* **`-d "${CARD_DB}"`**: Sets the target reference database binary.
* **`-o "${ARG_TSV}"`**: Outputs tabular BLAST results (BLAST format 6).
* **`--max-target-seqs 1`**: Retains only the single top-scoring hit per protein query.

</details>

#### Binning Workflow: MAG Recovery Pipeline

##### Pre-Processing Contigs & Coverage Preparation

Before binning, contigs shorter than 2,000 bp are filtered out to improve binning accuracy and remove sequence noise. Reads are then mapped to these filtered contigs to generate coordinate-sorted BAM files and calculate contig coverage depth tables.

```bash

mamba activate binning_env

# Define project directories
WORKING_DIR="/path/to/your/project"
ASSEMBLY_DIR="${WORKING_DIR}/results/assemblies"
CLEAN_READS_DIR="${WORKING_DIR}/data/dehost"
BINNING_OUT_DIR="${WORKING_DIR}/results/binning"

FILTERED_CONTIGS_DIR="${BINNING_OUT_DIR}/1_filtered_contigs"
MAP_OUT_DIR="${BINNING_OUT_DIR}/2_mapping"
COV_OUT_DIR="${BINNING_OUT_DIR}/3_coverage"

# File suffixes & thresholds
FORWARD_SUFFIX="_R1.dehost.fq.gz"
REVERSE_SUFFIX="_R2.dehost.fq.gz"
MIN_CONTIG_LEN=2000
THREADS=16

# Create output directories
mkdir -p "${FILTERED_CONTIGS_DIR}" "${MAP_OUT_DIR}" "${COV_OUT_DIR}"

```

> ##### 2. Contig Filtering, Mapping & Depth Calculation Code


```bash

# --- Step 1: Filter Contigs (>= 2000 bp) ---
cd "${ASSEMBLY_DIR}"

for SAMPLE_DIR in */; do

    SAMPLE_NAME=$(basename "${SAMPLE_DIR}")
    INPUT_CONTIGS="${SAMPLE_DIR}/contigs.fasta"
    OUTPUT_FILTERED="${FILTERED_CONTIGS_DIR}/${SAMPLE_NAME}.contigs.ge2kb.fasta"

    if [ -f "${INPUT_CONTIGS}" ]; then
        echo "=========================================="
        echo "Filtering contigs (>= ${MIN_CONTIG_LEN} bp) for: ${SAMPLE_NAME}"
        echo "=========================================="

        awk -v MINLEN="${MIN_CONTIG_LEN}" '
            BEGIN {
                RS=">"; 
                FS="\n"; 
                OFS=""
            }
            NR > 1 {
                id = $1;
                seq = "";
                for (i = 2; i <= NF; i++) seq = seq $i;
                if (length(seq) >= MINLEN) {
                    gsub(/ /, "_", id);
                    print ">" id "\n" seq "\n";
                }
            }
        ' "${INPUT_CONTIGS}" > "${OUTPUT_FILTERED}"
    fi

done

# --- Step 2: Read Mapping (Bowtie2 + Samtools) ---
cd "${FILTERED_CONTIGS_DIR}"

for CONTIG_FILE in *.contigs.ge2kb.fasta; do

    SAMPLE_NAME=$(basename "${CONTIG_FILE}" ".contigs.ge2kb.fasta")
    R1="${CLEAN_READS_DIR}/${SAMPLE_NAME}${FORWARD_SUFFIX}"
    R2="${CLEAN_READS_DIR}/${SAMPLE_NAME}${REVERSE_SUFFIX}"

    SAMPLE_INDEX_DIR="${MAP_OUT_DIR}/${SAMPLE_NAME}/bt2index"
    INDEX_PREFIX="${SAMPLE_INDEX_DIR}/${SAMPLE_NAME}"
    SORTED_BAM="${MAP_OUT_DIR}/${SAMPLE_NAME}.sorted.bam"

    if [ -f "${R1}" ] && [ -f "${R2}" ]; then
        echo "=========================================="
        echo "Mapping reads to filtered contigs for: ${SAMPLE_NAME}"
        echo "=========================================="

        mkdir -p "${SAMPLE_INDEX_DIR}"

        # Build Bowtie2 index
        bowtie2-build "${CONTIG_FILE}" "${INDEX_PREFIX}"

        # Align paired reads and sort BAM directly
        bowtie2 -x "${INDEX_PREFIX}" \
          -1 "${R1}" \
          -2 "${R2}" \
          --sensitive \
          --no-unal \
          -p "${THREADS}" | \
        samtools view -@ "${THREADS}" -bS - | \
        samtools sort -@ "${THREADS}" -o "${SORTED_BAM}"

        # Index sorted BAM
        samtools index "${SORTED_BAM}"
    fi

done

# --- Step 3: Coverage Depth Calculation ---
cd "${MAP_OUT_DIR}"

for BAM_FILE in *.sorted.bam; do

    SAMPLE_NAME=$(basename "${BAM_FILE}" ".sorted.bam")
    CONTIG_FILE="${FILTERED_CONTIGS_DIR}/${SAMPLE_NAME}.contigs.ge2kb.fasta"
    DEPTH_FILE="${COV_OUT_DIR}/${SAMPLE_NAME}_coverage.tsv"

    if [ -f "${CONTIG_FILE}" ]; then
        echo "=========================================="
        echo "Calculating Contig Depths for: ${SAMPLE_NAME}"
        echo "=========================================="

        jgi_summarize_bam_contig_depths \
          --outputDepth "${DEPTH_FILE}" \
          --referenceFasta "${CONTIG_FILE}" \
          "${BAM_FILE}"
    fi

done

```

To understand how these codes work, see the details below:

<details> 
  
### **Variables & Path Setup**

* **`FILTERED_CONTIGS_DIR="${BINNING_OUT_DIR}/1_filtered_contigs"`**: Directory storing assemblies filtered to contigs $\ge 2\text{ kb}$.
* **`MAP_OUT_DIR="${BINNING_OUT_DIR}/2_mapping"`**: Directory holding Bowtie2 index files, coordinate-sorted BAM files, and BAM indexes (`.bai`).
* **`COV_OUT_DIR="${BINNING_OUT_DIR}/3_coverage"`**: Directory holding contig coverage depth tables used by downstream binners.
* **`MIN_CONTIG_LEN=2000`**: Discards short contigs ($<2{,}000\text{ bp}$) lacking sufficient $k$-mer composition signals.


### **Pre-Processing Step**

* **`awk -v MINLEN="${MIN_CONTIG_LEN}" ...`**:
* `RS=">"`: Parses FASTA headers as record boundaries.
* `gsub(/ /, "_", id)`: Replaces whitespace in contig header IDs with underscores to prevent parsing failures in binning tools.
* `if (length(seq) >= MINLEN)`: Retains only sequences meeting or exceeding 2,000 bp.
* **`bowtie2-build "${CONTIG_FILE}" "${INDEX_PREFIX}"`**: Builds FM-indexes from length-filtered contigs to allow rapid read alignment.
* **`bowtie2 -x "${INDEX_PREFIX}" -1 "${R1}" -2 "${R2}"`**: Maps host-filtered forward and reverse paired reads against the sample-specific contig index.
* **`--sensitive`**: Uses standard sensitive preset parameters for short-read alignment accuracy.
* **`--no-unal`**: Omits unaligned read records from the stream to minimize processing overhead.
* **`samtools view -@ "${THREADS}" -bS - | samtools sort -@ "${THREADS}" -o "${SORTED_BAM}"`**: Converts the SAM data stream to BAM format on the fly and sorts records by genomic coordinates.
* **`samtools index "${SORTED_BAM}"`**: Generates a `.bai` index file required for fast coordinate retrieval.
* **`jgi_summarize_bam_contig_depths`**: Parses alignment depths from the coordinate-sorted BAM and produces the contig abundance matrix required by MetaBAT2 and MaxBin2.
* **`--outputDepth "${DEPTH_FILE}"`**: Designates the final tab-separated coverage output path.
* **`--referenceFasta "${CONTIG_FILE}"`**: References the length-filtered contigs to verify sequence lengths and headers during coverage estimation.

</details>


##### Multi-Tool Binning (MetaBAT2, MaxBin2, CONCOCT)

##### 1. Defining Variables & Paths

Running multiple binners with complementary statistical models maximizes the recovery of distinct Metagenome-Assembled Genomes (MAGs). The raw bin outputs from each tool are saved into dedicated subdirectories before consensus integration with DAS Tool.

```bash
# Define project directories
WORKING_DIR="/path/to/your/project"
BINNING_OUT_DIR="${WORKING_DIR}/results/binning"

FILTERED_CONTIGS_DIR="${BINNING_OUT_DIR}/1_filtered_contigs"
MAP_OUT_DIR="${BINNING_OUT_DIR}/2_mapping"
COV_OUT_DIR="${BINNING_OUT_DIR}/3_coverage"

METABAT2_OUT_DIR="${BINNING_OUT_DIR}/4_metabat2"
MAXBIN2_OUT_DIR="${BINNING_OUT_DIR}/5_maxbin2"
CONCOCT_OUT_DIR="${BINNING_OUT_DIR}/6_concoct"

# Hardware resources
THREADS=16

# Create output directories for all three binners
mkdir -p "${METABAT2_OUT_DIR}" "${MAXBIN2_OUT_DIR}" "${CONCOCT_OUT_DIR}"

```

##### 2. Multi-Tool Binning Script

```bash
cd "${FILTERED_CONTIGS_DIR}"

for CONTIG_FILE in *.contigs.ge2kb.fasta; do

    SAMPLE_NAME=$(basename "${CONTIG_FILE}" ".contigs.ge2kb.fasta")
    DEPTH_FILE="${COV_OUT_DIR}/${SAMPLE_NAME}_coverage.tsv"
    BAM_FILE="${MAP_OUT_DIR}/${SAMPLE_NAME}.sorted.bam"

    # Define sample-specific output directories
    SAMPLE_METABAT="${METABAT2_OUT_DIR}/${SAMPLE_NAME}"
    SAMPLE_MAXBIN="${MAXBIN2_OUT_DIR}/${SAMPLE_NAME}"
    SAMPLE_CONCOCT="${CONCOCT_OUT_DIR}/${SAMPLE_NAME}"

    mkdir -p "${SAMPLE_METABAT}" "${SAMPLE_MAXBIN}" "${SAMPLE_CONCOCT}"

    echo "=========================================="
    echo "Running Multi-Tool Binning for: ${SAMPLE_NAME}"
    echo "=========================================="

    # --- Tool 1: MetaBAT2 ---
    echo "[1/3] Running MetaBAT2..."
    metabat2 \
      -i "${CONTIG_FILE}" \
      -a "${DEPTH_FILE}" \
      -o "${SAMPLE_METABAT}/${SAMPLE_NAME}.bin" \
      -m 2000 \
      -t "${THREADS}" \
      --unbinned

    # --- Tool 2: MaxBin2 ---
    echo "[2/3] Running MaxBin2..."
    run_MaxBin.pl \
      -contig "${CONTIG_FILE}" \
      -abund "${DEPTH_FILE}" \
      -out "${SAMPLE_MAXBIN}/${SAMPLE_NAME}.maxbin" \
      -thread "${THREADS}"

    # --- Tool 3: CONCOCT ---
    echo "[3/3] Running CONCOCT..."
    # 1. Cut contigs into 10kb chunks for accurate compositional modeling
    cut_up_fasta.py "${CONTIG_FILE}" \
      -c 10000 \
      -o 0 \
      --merge_last \
      -b "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.bed" > "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.10kb.fa"

    # 2. Generate contig coverage table for the split contigs
    concoct_coverage_table.py \
      "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.bed" \
      "${BAM_FILE}" > "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.concoct_depth.tsv"

    # 3. Run CONCOCT clustering
    concoct \
      --composition_file "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.10kb.fa" \
      --coverage_file "${SAMPLE_CONCOCT}/${SAMPLE_NAME}.concoct_depth.tsv" \
      -b "${SAMPLE_CONCOCT}/" \
      -t "${THREADS}"

    # 4. Merge sub-contig clustering back to original contig IDs
    merge_cut_up_clustering.py \
      "${SAMPLE_CONCOCT}/clustering_gt1000.csv" > "${SAMPLE_CONCOCT}/${SAMPLE_NAME}_clustering_merged.csv"

    # 5. Extract FASTA bins from clustering results
    extract_fasta_bins.py \
      "${CONTIG_FILE}" \
      "${SAMPLE_CONCOCT}/${SAMPLE_NAME}_clustering_merged.csv" \
      --output_path "${SAMPLE_CONCOCT}/bins"

done

```

To understand how these codes work, see the details below:

<details> 

### **Variables & Path Setup**

* **`METABAT2_OUT_DIR` / `MAXBIN2_OUT_DIR` / `CONCOCT_OUT_DIR**`: Specific root directories that keep each binner's FASTA files and clustering tables isolated.
* **`DEPTH_FILE="${COV_OUT_DIR}/${SAMPLE_NAME}_coverage.tsv"`**: The contig abundance matrix generated by `jgi_summarize_bam_contig_depths` in the previous step.
* **`BAM_FILE="${MAP_OUT_DIR}/${SAMPLE_NAME}.sorted.bam"`**: Coordinate-sorted alignment file required directly by CONCOCT's coverage script.

### **Tool-by-Tool Execution**

* **MetaBAT2:**
* `-i "${CONTIG_FILE}"`: Target length-filtered FASTA assembly.
* `-a "${DEPTH_FILE}"`: Contig depth/coverage summary table.
* `-o "${SAMPLE_METABAT}/${SAMPLE_NAME}.bin"`: Output prefix for generated bin FASTA files (`*.bin.1.fa`, `*.bin.2.fa`, etc.).
* `-m 2000`: Explicit minimum contig length cutoff (2,000 bp).
* `--unbinned`: Saves leftover contigs that failed binning thresholds to a dedicated FASTA file.


* **MaxBin2:**
* `-contig "${CONTIG_FILE}"`: Assembly input FASTA.
* `-abund "${DEPTH_FILE}"`: Abundance depth file.
* `-out "${SAMPLE_MAXBIN}/${SAMPLE_NAME}.maxbin"`: Base prefix for MaxBin2 output files (produces `*.maxbin.001.fasta`, `*.maxbin.summary`, etc.).
* `-thread "${THREADS}"`: Allocates CPU threads for Expectation-Maximization clustering.


* **CONCOCT:**
* `cut_up_fasta.py`: Splits long contigs into uniform 10 kb chunks to avoid length-biased composition estimation.
* `concoct_coverage_table.py`: Computes sample coverage over the 10 kb split fragments using the sorted BAM file.
* `concoct`: Runs Gaussian mixture model clustering combining tetranucleotide frequency and coverage depth.
* `merge_cut_up_clustering.py`: Resolves clustered 10 kb chunks back into the original intact contig identifiers.
* `extract_fasta_bins.py`: Reads the original contig FASTA and writes individual `.fa` bin files into the designated `bins/` folder.

</details>











