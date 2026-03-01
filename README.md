## **Denovo Long-Read Sequencing Analysis Pipeline**

### **Overview**
This project implements a workflow for the analysis of long-read sequencing data from the NCBI SRA database. The pipeline is designed to process raw reads, assess sequence quality, assemble the genome, and evaluate assembly completeness using BUSCO. The workflow is implemented using Snakemake, providing reproducibility and scalability.

### **Purpose of the Analysis**
The main goals of this analysis are:

* Obtain high-quality sequencing reads from SRA and convert them to FASTQ format.

* Assess the quality of raw reads using FastQC.

* Perform de novo assembly of long-read sequences with HiFiASM.

* Convert assembly files from GFA to FASTA format for downstream analyses.

* Evaluate genome assembly quality with QUAST.

* Check assembly completeness using BUSCO, referencing the appropriate lineage dataset.

* Ensure that datasets and results are organized in a reproducible directory structure.

This type of analysis is commonly used in:

* Functional genomics studies
* Comparitive genome analysis

### **Data Description**
#### *Input Data*
* Input Data: Raw long-read sequencing data (SRA ID specified in config/config.yaml).

*Note:* The user can change the SRR id, based on the study.

### **Workflow**
The pipeline consists of the following major steps:

*1. Download Raw Reads:*

* Purpose: Download raw sequencing reads from the NCBI SRA database.

* Input: SRA ID (config/config.yaml).

* Output: FASTQ file stored in resources.

* Tool: fasterq-dump from the SRA Toolkit.

* Details:
The pipeline creates a directory resources/ and uses fasterq-dump to convert the SRA data to FASTQ format. Multithreading is used for faster downloads.

*2. Quality Assessment:*

* Purpose: Assess the quality of raw reads.

* Input: FASTQ file from generate_fastq.

* Output:

    * HTML report: results/01_fastqc/*_fastqc.html

    * ZIP archive: results/01_fastqc/*_fastqc.zip

* Tool: FastQC via Snakemake wrapper.

* Details:
FastQC checks per-base quality, GC content, sequence length distribution, and adapter contamination. These reports help determine if preprocessing or trimming is required.

*3. De novo Assembly:*

* Purpose: Assemble long-read sequences into contigs.

* Input: FASTQ reads.

* Output: GFA file: results/02_assembly/*.bp.p_ctg.gfa

* Tool: HiFiASM.

* Details:
HiFiASM is optimized for high-fidelity long reads (PacBio HiFi). The -o parameter specifies the output prefix, and threads are used for parallel computation.

*4. Convert GFA to FASTA:*

* Purpose: Convert assembly graph (GFA) to FASTA for downstream analyses.

* Input: GFA assembly from de_novo.

* Output: FASTA file: results/02_assembly/*.bp.p_ctg.fa

* Tool: awk.

* Details:
The GFA format contains contig sequences as segments. The awk command extracts these sequences and writes them in standard FASTA format (>contig_name followed by the sequence).

*5. Assembly Quality Check:*

* Purpose: Evaluate genome assembly metrics such as N50, total length, and contig statistics.

* Input: FASTA assembly from convert_gfa.

* Output: QUAST report: results/03_quast/{Read_id}/report.html  (Read_id = SRR id)

* Tool: QUAST.

* Details:
QUAST provides summary statistics that help identify assembly completeness, fragmentation, and potential misassemblies. Useful for comparing different assembly strategies.

*6. Genome Completeness Assessment:*

* Purpose: Check the completeness of the genome assembly using conserved single-copy orthologs.

* Input: FASTA assembly.

* Output: BUSCO results in results/04_busco/{Read_id}.

* Tool: BUSCO.

* Parameters:

    *  -l specifies the lineage dataset (e.g., saccharomycetes_odb10).

    * -m genome indicates genome mode.

    * --download_path points to resources/busco_downloads to reuse datasets.

* Details:
BUSCO searches for lineage-specific single-copy orthologs in the assembly to estimate completeness. The --force flag allows overwriting existing runs. The lineage dataset is downloaded once and stored locally to avoid repeated downloads.

**The pipeline is organized as follows:**
``` text
.
├── config/
│   └── config.yaml            # Contains SRA ID and BUSCO dataset
├── resources/                 # Storage for downloaded dataset    
│   ├── busco_downloads/       # BUSCO lineage datasets
│   └── SRR13577846/
├── results/
│   ├── 01_fastqc/             # FastQC output
│   ├── 02_assembly/           # Genome assembly files (GFA & FASTA)
│   ├── 03_quast/              # QUAST reports
│   └── 04_busco/              # BUSCO results
├── workflow/
│   ├── Snakefile              # Snakemake pipeline
│   └── envs/                  # Conda environments
└── README.md                  # This file
```
### **Required Software**
This workflow uses conda environments for reproducibility.

Main tools:

* Snakemake (9.16.0)
* SRA Toolkit (3.2.1)
* FastQC (wrapper 0.85.0)
* HiFiASM (0.25.0)
* QUAST (5.3.0)
* BUSCO (6.0.0)

All dependencies are managed through Conda environments. Each tools were specified under the worflow/envs/

### **How to Run the Workflow**
*1. Install Requiremnets:*

Install Conda (Miniconda) and Snakemake.

*2. Configure Input Files:*

Edit the configuration file to specify:
* SRR id
* lineage dataset

*3. Run the Pipeline:*

```
To perform a dry run:
snakemake -n

To run:
snakemake -p --software-deployment-method conda -j 20
```

### **Output File**
Key outputs include:
* Fastq file
* fastqc analysis report 
* denovo genome assembly of long read sequence fasta file (contig file)
* quast report
* BUSCO report

### **People Involved**
* **Name:** Karnesh Sampath
* **Date:** 25.02.2026

### **Notes:**
* The workflow is reproducible and modular. Modifying samples or the lineage dataset is straightforward via config/config.yaml.

* All outputs are stored in a structured results/ directory for easy access.

* Ensure sufficient disk space, especially for long-read FASTQ files and BUSCO datasets.

* The pipeline works for one SRR id only.








