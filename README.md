# trio-wes-osteopetrosis-analysis
Command-line trio WES analysis for candidate variant identification in osteopetrosis
# Trio WES Variant Analysis for Osteopetrosis

## Overview

This project presents a command-line bioinformatics workflow for analyzing
trio whole-exome sequencing (WES) data from an affected proband and both
unaffected parents.

The objective was to identify candidate genetic variants consistent with an
autosomal-recessive inheritance model and prioritize variants with potentially
relevant molecular consequences.

The analysis was performed in a Linux/Ubuntu environment using commonly used
NGS and variant-analysis tools.

> *Important:* Although the underlying dataset represents whole-exome
> sequencing data, this training analysis was performed against a chromosome 8
> reference. Therefore, this project represents a chromosome 8-restricted WES
> analysis rather than a complete genome-wide or unrestricted whole-exome
> analysis.

---

## Project Origin

This project is a *command-line reimplementation and adaptation* of the
Galaxy Training Network (GTN) tutorial:

*Exome sequencing data analysis for diagnosing a genetic disease*

Original tutorial:

https://training.galaxyproject.org/training-material/topics/variant-analysis/tutorials/exome-seq/tutorial.html

The original tutorial presents a family-trio WES analysis involving an affected
proband with osteopetrosis and two unaffected parents. The tutorial demonstrates
quality control, read mapping, BAM processing, variant calling, functional
annotation, and inheritance-based candidate variant prioritization.

Instead of performing the analysis entirely through the Galaxy graphical
interface, I reproduced the workflow directly in a Linux/Ubuntu command-line
environment.

The goal of this project was not simply to follow the tutorial, but to
understand each stage of the analysis, execute the underlying bioinformatics
commands independently, and document the resulting workflow.

Where appropriate, the original workflow was adapted to use a current
command-line software stack and updated analysis practices.

---

## Original Tutorial vs. This Implementation

| Analysis Stage | Galaxy Training Tutorial | This Project |
|---|---|---|
| Environment | Galaxy graphical interface | Linux / Ubuntu |
| Quality Control | Galaxy tools | FastQC + MultiQC |
| Read Alignment | BWA | BWA-MEM |
| BAM Processing | Galaxy/SAMtools | SAMtools command line |
| Variant Calling | FreeBayes | FreeBayes |
| Variant Normalization | Galaxy tools | BCFtools |
| Functional Annotation | SnpEff | SnpEff |
| Inheritance Filtering | Family-based filtering | Slivar |
| Variant Prioritization | Galaxy workflow | Command-line filtering + biological interpretation |

This implementation was developed as a hands-on learning and portfolio
project based on the concepts and dataset structure presented by the original
GTN training material.

---

## Project Objectives

The main objectives were to:

- Perform sequencing quality control
- Align paired-end reads to a reference genome
- Process and deduplicate BAM files
- Perform joint variant calling
- Normalize variants
- Functionally annotate variants
- Apply an autosomal-recessive inheritance model
- Prioritize high-impact candidate variants
- Investigate the biological relevance of prioritized candidates
- Reproduce a Galaxy-based educational workflow from the command line

---

## Workflow

```text
FASTQ
  │
  ▼
FastQC / MultiQC
  │
  ▼
BWA-MEM
  │
  ▼
SAMtools
Filtering + Deduplication
  │
  ▼
FreeBayes
Joint Variant Calling
  │
  ▼
BCFtools
Variant Normalization
  │
  ▼
SnpEff
Functional Annotation
  │
  ▼
Slivar
Autosomal-Recessive Filtering
  │
  ▼
292 Candidate Variants
  │
  ▼
HIGH-Impact Prioritization
  │
  ▼
4 HIGH-Impact Candidates
  │
  ▼
Biological Interpretation
  │
  ▼
CA2 Prioritized Candidate
```
---

## Dataset

The dataset consists of paired-end sequencing data from a family trio:

| Sample | Role |
|---|---|
| Father | Unaffected parent |
| Mother | Unaffected parent |
| Proband | Affected individual |

The project uses a chromosome 8 reference derived from the hg19 assembly.

Raw FASTQ and BAM files are not included in this repository.

---

## Reference Genome

Reference:

hg19_chr8.fa

The reference was obtained from the training dataset resources.

Because the reference contains chromosome 8 only, the analysis is restricted to
chromosome 8.

---

## Software and Tools

| Tool | Purpose |
|---|---|
| FastQC | Sequencing quality control |
| MultiQC | QC report aggregation |
| BWA-MEM | Read alignment |
| SAMtools | BAM processing and duplicate removal |
| FreeBayes | Joint variant calling |
| BCFtools | Variant normalization and querying |
| SnpEff | Functional annotation |
| Slivar | Family-based inheritance filtering |

---

## Quality Control

FastQC was used to assess the quality of all six FASTQ files, followed by
MultiQC to summarize the results.

The sequencing data showed generally good quality, with:

- Approximately 101 bp read length
- Approximately 44% GC content
- Duplication levels of approximately 21–25%
- Predominantly high Phred quality scores

A bimodal per-sequence GC-content distribution was observed. This pattern is
compatible with exome-capture sequencing and was therefore not treated as
sufficient reason for trimming.

### QC Summary

| Sample | Read Length | GC Content | Duplication |
|---|---:|---:|---:|
| Father R1 | 101 bp | 44% | 23.2% |
| Father R2 | 101 bp | 44% | 21.4% |
| Mother R1 | 101 bp | 44% | 23.3% |
| Mother R2 | 101 bp | 44% | 21.9% |
| Proband R1 | 101 bp | 44% | 25.4% |
| Proband R2 | 101 bp | 44% | 23.1% |

The MultiQC report was generated as:

`qc/multiqc_report.html`