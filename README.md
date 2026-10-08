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

---

## Read Alignment

Paired-end reads were aligned to the chromosome 8 hg19 reference using
BWA-MEM.

Sample-specific read groups were included for each member of the family trio:

- Father
- Mother
- Proband

The aligned reads were directly sorted into BAM format using SAMtools.

### Alignment Commands

#### Father

```bash
bwa mem -t 4 -R '@RG\tID:000\tSM:father\tPL:ILLUMINA' \
reference/hg19_chr8.fa \
raw_data/father_R1.fq.gz \
raw_data/father_R2.fq.gz \
| samtools sort -o aligned_data/father.sorted.bam
```

#### Mother

```bash
bwa mem -t 4 -R '@RG\tID:001\tSM:mother\tPL:ILLUMINA' \
reference/hg19_chr8.fa \
raw_data/mother_R1.fq.gz \
raw_data/mother_R2.fq.gz \
| samtools sort -o aligned_data/mother.sorted.bam
```
#### Proband

```bash
bwa mem -t 4 -R '@RG\tID:002\tSM:proband\tPL:ILLUMINA' \
reference/hg19_chr8.fa \
raw_data/proband_R1.fq.gz \
raw_data/proband_R2.fq.gz \
| samtools sort -o aligned_data/proband.sorted.bam
```

Why BWA-MEM?

BWA-MEM was used to align the sequencing reads to the reference genome and
produce coordinate-sorted BAM files for downstream variant analysis.

---

## BAM Processing

After read alignment, SAMtools was used to process the aligned reads before
variant calling.

The processing workflow included:

1. Filtering mapped read pairs
2. Name collating
3. Mate information processing
4. Coordinate sorting
5. Duplicate removal
6. BAM indexing

### Filtering

Reads were filtered using SAMtools to retain records where both the read and
its mate were mapped.

```bash
samtools view -b -F 12 aligned_data/father.sorted.bam \
-o aligned_data/father.filtered.bam
```

The same filtering approach was applied to the mother and proband.

### Duplicate Removal

A modern SAMtools duplicate-removal workflow was used.

For each sample, the steps were:

```bash
samtools collate -o aligned_data/father.namecollate.bam \
aligned_data/father.filtered.bam

samtools fixmate -m aligned_data/father.namecollate.bam \
aligned_data/father.fixmate.bam

samtools sort -o aligned_data/father.fixmate.sorted.bam \
aligned_data/father.fixmate.bam

samtools markdup -r aligned_data/father.fixmate.sorted.bam \
aligned_data/father.dedup.bam
```

The same workflow was applied to the mother and proband.

### BAM Quality Assessment

SAMtools `flagstat` was used to evaluate the final BAM files.

```bash
samtools flagstat aligned_data/father.dedup.bam
samtools flagstat aligned_data/mother.dedup.bam
samtools flagstat aligned_data/proband.dedup.bam
```

The final BAM files were indexed:

```bash
samtools index aligned_data/father.dedup.bam
samtools index aligned_data/mother.dedup.bam
samtools index aligned_data/proband.dedup.bam
```

### Final BAM Statistics

| Sample | Primary Reads | Mapped | Properly Paired | Singletons |
|---|---:|---:|---:|---:|
| Father | 2,879,684 | 100% | ~99.67% | 0 |
| Mother | 2,528,804 | 100% | ~99.75% | 0 |
| Proband | 3,197,114 | 100% | ~99.69% | 0 |

The processed BAM files were used as input for downstream joint variant
calling.

---

## Variant Calling

After BAM processing, FreeBayes was used to perform joint variant calling
across the father, mother, and proband.

Joint calling was performed using the processed and deduplicated BAM files.

```bash
freebayes --genotype-qualities \
-f reference/hg19_chr8.fa \
aligned_data/father.dedup.bam \
aligned_data/mother.dedup.bam \
aligned_data/proband.dedup.bam \
> variants/family.raw.gq.vcf
```
The --genotype-qualities option was used to include genotype quality (GQ)
information in the resulting VCF file.

The initial variant calling produced 35,551 raw variant records.

The resulting VCF file was:

variants/family.raw.gq.vcf

---

## Variant Normalization

The raw VCF file was normalized using BCFtools to standardize variant
representation and split multiallelic variants into separate records.

```bash
bcftools norm \
-f reference/hg19_chr8.fa \
-m -any \
variants/family.raw.gq.vcf \
-o variants/family.gq.norm.vcf
```

The normalization process:

* Split multiallelic variants into separate records.
* Realigned variants where necessary.
* Standardized variant representation using the reference genome.

The normalization process produced 38,027 variant records from the
original 35,551 raw records.

The increase in record count is mainly due to splitting multiallelic
variants into separate records.

The normalized VCF file was:

variants/family.gq.norm.vcf
