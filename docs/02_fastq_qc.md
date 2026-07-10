# FASTQ-level quality control for scRNA-seq

## Purpose

FASTQ-level QC evaluates sequencing quality and read structure before barcode processing, alignment, and quantification.

FASTQ QC cannot assess:

- cell viability
- empty droplets
- ambient RNA
- doublets
- per-cell mitochondrial content
- per-cell gene complexity

Those require the count matrix and are handled later.

## Expected read structure

Read structure depends on the assay chemistry.

For a typical 10x Genomics 3' gene-expression library:

- R1 contains the cell barcode and UMI
- R2 contains the cDNA-derived sequence
- I1 and I2 contain sample-index reads used during demultiplexing

The pipeline should record expected read lengths and chemistry-specific barcode/UMI layouts.

## FastQC metrics

### Per-base sequence quality

High-quality libraries should maintain high Phred scores across most read positions.

A moderate quality decline toward the end of a sequencing read is common in sequencing-by-synthesis platforms and is not automatically a reason to trim.

Trimming should only be considered if poor-quality bases are substantial enough to affect downstream mapping or barcode interpretation.

For 10x data, indiscriminate trimming of R1 is particularly risky because R1 contains fixed-position cell-barcode and UMI sequences.

### Per-tile sequence quality

This metric evaluates whether particular imaging tiles show systematically reduced quality.

Uniform performance across tiles is preferred.

Localized deviations may indicate:

- flow-cell imaging problems
- bubbles
- debris
- local sequencing failure

### Per-base sequence content

The fraction of A, C, G, and T at each read position is often non-random in scRNA-seq libraries.

This can result from:

- fixed barcode or UMI structure
- priming sequence
- low-complexity protocol-specific sequence
- non-random transcript start positions

A FastQC warning for per-base sequence content is therefore often expected and should be interpreted according to chemistry and read type.

### Per-sequence GC content

Observed GC distributions reflect the expressed transcriptome, not the genome as a whole.

The distribution depends on:

- transcript sequence composition
- gene-expression abundance
- cell-type composition
- library preparation protocol

Differences from FastQC's theoretical distribution are not automatically problematic.

Unexpected multimodality or extreme shifts may indicate:

- contamination
- mixed species
- adapter sequence
- technical artifacts
- abnormal library composition

### Per-base N content

N bases represent positions where the sequencer could not confidently assign a nucleotide.

N content should generally remain near zero.

Elevated N content may indicate:

- sequencing quality problems
- imaging problems
- low-complexity sequence
- instrument-specific artifacts

### Sequence length distribution

Reads from fixed-structure single-cell libraries are generally expected to have uniform lengths.

Variable lengths can arise from:

- preprocessing
- quality trimming
- mixed sequencing runs
- concatenated datasets
- non-standard library preparation

### Duplication levels

FastQC duplication metrics are not UMI-aware.

High duplication is common in droplet-based scRNA-seq because:

- highly expressed genes produce many similar fragments
- PCR amplification is extensive
- reads are concentrated near transcript ends
- index reads contain only a small set of sample-index sequences

High duplication should not be interpreted using bulk DNA-sequencing assumptions.

PCR duplicate removal should not be performed directly on raw 10x reads. UMI-aware deduplication is handled during quantification.

### Adapter content

Substantial adapter contamination may reduce mapping quality.

However, small amounts of adapter or protocol-specific sequence are often tolerated by Cell Ranger and similar tools.

Trimming should be chemistry-aware and should not alter barcode or UMI positions.

## QC decision logic

The pipeline should flag, but not automatically fail, libraries based only on a single FastQC warning.

QC decisions should integrate:

- read quality
- read lengths
- expected chemistry
- adapter content
- barcode quality
- mapping metrics
- cell-associated read fraction

## Recommended outputs

The FASTQ QC stage should produce:

- FastQC reports
- MultiQC report
- machine-readable summary table
- read-length summary
- file inventory
- checksum validation result
- sample-to-FASTQ mapping