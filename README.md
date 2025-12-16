# ChIP-seq Analysis Pipeline

A comprehensive Snakemake workflow for processing single-end ChIP-seq data from raw FASTQ files to peak calling and visualization. Workflow is very similar to paired-end ChIP-seq workflow (https://github.com/gynecoloji/SnakeMake_ChIPseq)

## 📚 Table of Contents

- [Overview](#overview)
- [Pipeline Components](#pipeline-components)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Input Requirements](#input-requirements)
- [Output Description](#output-description)
- [Parameters](#parameters)
- [Conda Environments](#conda-environments)
- [License](#license)
- [Contact](#contact)

## 🔍 Overview

This pipeline performs standard ChIP-seq data processing and analysis through a unified workflow, providing comprehensive results and quality metrics. The pipeline supports both narrow and broad peak calling modes and can use input controls when available.

The workflow processes paired-end sequencing data through multiple steps of quality control, alignment, filtering, and peak calling, generating detailed reports at each stage to ensure reliable results.

## 🧩 Pipeline Components

The workflow consists of several key components:

### Quality Control and Preprocessing
- **Quality Control**: FastQC for raw reads quality assessment
- **Read Preprocessing**: Fastp for adapter trimming and quality filtering

### Alignment and Filtering
- **Alignment**: HISAT2 for efficient read alignment to the reference genome
- **BAM Processing**: Samtools for filtering, sorting, and indexing
- **Duplication Removal**: Picard for identifying and removing PCR duplicates
- **Blacklist Filtering**: BEDTools for filtering against known problematic regions

### Peak Calling
- **Narrow Peaks**: MACS2 for transcription factor binding site detection
- **Broad Peaks**: MACS2 with broad peak options for histone mark analysis
- **Input Control Support**: Optional input sample integration for more accurate peak calling

### QC Reporting
- **Blacklist Filtering Statistics**: Custom reporting on blacklist filtering effectiveness

## 🛠️ Requirements

- Snakemake
- Python 3.6+ with pandas
- FastQC
- Fastp
- HISAT2
- Samtools
- Picard
- BEDTools
- MACS2 (installed via conda environment)
- 16GB+ RAM recommended
- 8+ CPU cores recommended for optimal performance

## 📥 Installation

Clone the repository:

```bash
git clone https://github.com/gynecoloji/SnakeMake_ChIPseq_single_end.git
cd SnakeMake_ChIPseq_single_end
```

Create the snakemake conda environment:

```bash
conda env create -f envs/snakemake.yaml
```

## 🚀 Usage

1. Prepare your sample information table in CSV format (samples.csv) with the following columns:
   - `sample_id`: Unique identifier for each sample
   - `condition`: Experimental condition of the sample
   - `replicate`: Integer indicating biological replicate number
   - `input_control`: Sample ID of the input control to use
   - `peak_mode`: "narrow" (default) or "broad" for peak calling mode
   - `notes`: Additional information about the sample

2. Place paired-end FASTQ files in the `data/` directory following the naming convention:
   - `{sample}_R1_001.fastq.gz`
   - `{sample}_R2_001.fastq.gz`

3. Configure reference paths in `ref/config.yaml`:
   - HISAT2 index
   - Blacklist regions BED file

4. Run the workflow:

```bash
# Run with 8 cores
snakemake --cores 8 --use-conda -s snakefile_tolerant_ChIPseq

```

## 📁 Input Requirements

- **FASTQ files**: Single-end reads with naming pattern `{sample}.fastq.gz` 
- **Sample information table**: CSV file (samples.csv) with the following structure:

  | sample_id | condition | replicate | input_control | peak_mode | notes |
  |-----------|-----------|-----------|---------------|-----------|-------|
  | sample1   | treatment | 1         | input1        | narrow    | ...   |
  | sample2   | treatment | 2         | input1        | narrow    | ...   |
  | sample3   | control   | 1         | input2        | broad     | ...   |
  | ...       | ...       | ...       | ...           | ...       | ...   |

- **Reference files**:
  - HISAT2 genome index
  - Blacklist regions BED file
  - Picard JAR file in `ref/picard.jar`
  - Python script for blacklist statistics in `ref/blacklist-stats-script.py`

## 📊 Output Description

The pipeline generates organized outputs in the `results/` directory:

```
results/
├── fastqc/                   # Raw read quality reports
├── fastp/                    # Trimmed reads and QC reports
├── aligned/                  # Alignment files and summaries
├── filtered/                 # Filtered BAM files and flagstat reports
├── dedup/                    # Deduplicated BAM files
├── blacklist_filtered/       # Blacklist-filtered BAM files
├── bigwig/                   # RPGC normalized bigwig files
├── normalized_bigwig/        # normalized to input bigwig files
├── peaks/                    # Peak calling results (narrow and broad)
└── qc/                       # QC reports including blacklist filtering stats
```

## ⚙️ Parameters

Key configurable parameters in the workflow:

### Sample Configuration

The pipeline uses a CSV file (samples.csv) with 6 columns to manage samples and their properties:
- **sample_id**: Unique identifier for each sample
- **condition**: Experimental condition (e.g., treatment, control)
- **replicate**: Integer indicating biological replicate number
- **input_control**: Sample ID of the input control to use for peak calling
- **peak_mode**: Peak calling mode ("narrow" for transcription factors, "broad" for histone marks)
- **notes**: Additional sample information or metadata

- **FastQC**:
  - Threads: 4 (configurable)

- **Fastp**: 
  - Quality requirement: 30+ base quality
  - Adapter detection: automatic for single-end
  - PolyG trimming: enabled
  - Quality trimming: 4bp windows, Q20 threshold

- **HISAT2**:
  - Spliced alignment: disabled (ChIP-seq specific)
  - Tolerant mode (added parameters): --score-min L,0,-0.6 \
                                      --mp 4,2 \
                                      --rdg 5,3 --rfg 5,3

- **Samtools**:
  - Filtering: Properly paired reads (0x2) and primary alignments (-F 0x100)
  - Uniquely mapped: Only reads with NH:i:1 tag

- **MACS2**:
  - Genome: hs (human)
  - Q-value cutoff: 0.05
  - No model mode for samples without input controls
  - Shift: -75 (for samples without input controls)
  - Extsize: 150 (for samples without input controls)

## 🧪 Conda Environments

The workflow uses the following Conda environment:

1. **macs2.yaml**: MACS2 for peak calling

Other tools are assumed to be available in the base environment or path.

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📧 Contact

For questions or feedback, please open an issue on the GitHub repository or contact the author.

---

Last updated: Dec 15, 2025  
Created by: gynecoloji