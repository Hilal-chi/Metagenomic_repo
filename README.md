# Metagenomic_repo
Bu repo bitene kadar kimseye açık değil. Farklı cihazlarda çalışabilmek için çevrimiçi bir dosya oluşturuyorum sadece.

# **SHOTGUN METAGENOMICS GUIDELINE**

## **Purpose Of This Repository**

This repository is a coding guideline for starters in shotgun metagenomics data analysis. It includes codes of installing tools, creating environments and running analyses. If you're new to metagenomic analyses, you can see the [Metagenomic guideline](https://docs.google.com/document/d/1kU-FlgxgB0zssmJtbSXxJsxBNo2GGQfwhAVUqOSdKl4/edit?tab=t.0) for more information.

> Note: This repository is for Ubuntu/Debian systems only. If you're using other oprating systems such as Mac or Windows, you can either install WSL for Windows or use Mac terminal instead.

## Installation & Environment Setup

This pipeline requires specific bioinformatics tools. To ensure all tools run smoothly without library conflicts, using **Conda** or **Mamba** is suggested.

> Mamba or Conda?
> Mamba works exactly like Conda, yet resolves package dependencies in parallel, completing your installations faster. If Conda is not installed in your computer yet, Mamba is recommended.
> If you have Conda, you can use the same script with writing "conda" isntead of "mamba".


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
To understand how these codes work, you can look at the details.

<details>

1. **`curl -L -O "..."`**
   * **`curl`** is a command-line tool used to transfer data from a server.
   * **`-L`** tells curl to follow any redirects if the webpage has moved.
   * **`-O`** (capital letter O) saves the downloaded file on your computer using its original online name (`Miniforge3-Linux-x86_64.sh`).

3. **`bash Miniforge3-Linux-x86_64.sh`** * This executes the downloaded shell script (`.sh`) using the Bash interpreter to start the interactive installation wizard on your Ubuntu system.

4. **`source ~/.bashrc`** * This reloads your terminal's configuration file (`.bashrc`). It registers the newly installed `mamba` command so you can use it immediately without needing to close and reopen your terminal window.

</details>

### Installing Miniforge


#### Pre processing

For running the analysis properly, clean and complete data is needed. To achieve this; quality control, trimming and dehosting is necessary.


















