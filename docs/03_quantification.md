# Alignment, barcode processing, and quantification

## Overview

Raw droplet-based scRNA-seq data typically contain:

- cell barcode sequences
- UMI sequences
- cDNA-derived biological reads
- sample-index reads

The quantification workflow must:

1. identify the assay chemistry
2. parse cell barcodes and UMIs
3. correct barcode errors
4. map biological reads
5. assign reads to genes
6. correct or collapse UMI observations
7. call cell-associated barcodes
8. generate gene-by-cell count matrices

## Quantification tools

Common tools include:

- Cell Ranger
- STARsolo
- alevin-fry
- kallisto|bustools
- zUMIs
- scPipe
- scruff

### Cell Ranger

Cell Ranger is the vendor-supported workflow for 10x Genomics data.

It performs:

- chemistry detection
- barcode parsing and correction
- STAR-based genome alignment
- gene assignment
- UMI collapsing
- cell calling
- count-matrix generation
- summary reporting
- optional BAM generation
- optional secondary analysis

### STARsolo

STARsolo provides a configurable open implementation of STAR-based single-cell processing.

Advantages:

- high alignment speed
- flexible chemistry definitions
- customizable barcode and UMI handling
- support for gene, gene-full, and velocity-related features

### alevin-fry and kallisto|bustools

These use lightweight mapping or pseudoalignment-style strategies.

Advantages:

- lower runtime
- lower memory requirements
- modular barcode and UMI processing
- efficient matrix generation

Tradeoffs:

- different handling of multimapping
- different reference requirements
- less direct access to nucleotide-level alignments
- greater responsibility for user configuration

## Mapping strategies

Mapping methods can be grouped into:

- spliced genome alignment
- contiguous transcriptome alignment
- selective alignment
- lightweight mapping

### Spliced genome alignment

Reads are aligned to the genome while allowing large intronic gaps.

Examples:

- STAR
- STARsolo
- Cell Ranger

Advantages:

- supports exon-junction reads
- captures intronic reads
- suitable for scRNA-seq and snRNA-seq
- supports BAM output
- allows post hoc inspection of genomic alignments

Limitations:

- high memory use
- large indexes
- substantial temporary disk usage
- longer runtime

### Transcriptome-based mapping

Reads are mapped against annotated transcript sequences.

Advantages:

- smaller reference
- reduced runtime
- reduced memory use
- no need to infer splice junctions during mapping

Limitations:

- misses reads outside annotated transcripts
- limited handling of unspliced RNA
- less suitable for single-nucleus data unless the reference is augmented

### Augmented transcriptomes

An augmented transcriptome contains annotated spliced transcripts plus representations of unspliced or intronic sequence.

This can improve recovery of:

- intronic reads
- nascent RNA
- single-nucleus signal
- velocity-related information

The exact construction must be documented because different tools define augmented references differently.

## Alignment terminology

### Global alignment

The complete query is aligned across the complete target sequence.

### Local alignment

Only the best-matching subsequences are aligned.

### Semi-global or fitting alignment

Most or all of the query is aligned to a substring of the reference.

### Soft clipping

Bases at the ends of a read may remain unaligned without contributing full mismatch penalties.

Soft clipping is represented in the CIGAR string using `S`.

### CIGAR strings

CIGAR strings encode the structure of a sequence alignment.

Common operations include:

- `M`: alignment match or mismatch
- `=`: exact match
- `X`: mismatch
- `I`: insertion relative to the reference
- `D`: deletion relative to the reference
- `N`: skipped reference region, commonly an intron
- `S`: soft clipping
- `H`: hard clipping

Example:

30M1000N60M represents a read aligned across two exonic segments separated by a 1000-base intron.

## Reference choices

The pipeline should require explicit reference metadata.

At minimum, record:

* species
* genome assembly
* annotation source
* annotation release
* FASTA checksum
* GTF checksum
* index-building tool
* index-building parameters
* reference package version

### Genome reference

A genome reference supports spliced alignment and detection of intronic or intergenic mappings.

### Annotated transcriptome

A transcriptome reference contains only known transcript sequences.

### Augmented transcriptome

An augmented transcriptome includes additional unspliced or decoy sequences.

### Spike-ins and exogenous sequences

If the experiment contains:

* ERCC spike-ins
* viral sequences
* transgenes
* CRISPR constructs
* synthetic reporters
* knock-in cassettes

those sequences should be added to the reference if their expression or alignment is relevant.

## Cell barcode correction

Cell barcodes identify droplets or cells.

For 10x chemistries, candidate barcodes come from a known whitelist.

Barcode correction commonly considers:

* whether the observed barcode is in the whitelist
* Hamming or edit distance to valid barcodes
* barcode quality scores
* barcode abundance
* ambiguity among candidate corrections

Reads with unresolvable barcodes may be discarded.

## Cell calling

The set of observed barcodes is much larger than the number of real cells because many droplets are empty.

Cell calling attempts to distinguish cell-associated barcodes from:

* empty droplets
* low-RNA droplets
* ambient-RNA-containing droplets
* barcode errors

Common strategies include:

### Barcode-rank knee or inflection methods

Barcodes are ranked by UMI count. Abrupt changes in the rank curve help identify high-count cell-associated barcodes.

### Expected-cell priors

A user-provided expected cell count can influence the cell-calling model.

This should guide, not force, the number of called cells.

### Forced cell count

A fixed number of top-ranked barcodes may be selected.

This should be avoided unless there is strong experimental justification and the barcode-rank distribution has been inspected.

### Empty-droplet statistical models

Methods such as EmptyDrops model ambient RNA and test whether a barcode’s expression profile is inconsistent with an empty droplet.

## UMI processing

UMIs identify original pre-amplification molecules.

Reads are usually grouped by:

* corrected cell barcode
* gene or transcript assignment
* UMI sequence

Reads in the same group are collapsed to estimate molecule counts.

### UMI sequencing and PCR errors

A true UMI may generate nearby erroneous sequences through:

* PCR substitution errors
* sequencing miscalls

Tools may merge related UMIs using:

* adjacency graphs
* directional graphs
* edit-distance rules
* abundance-aware correction

### UMI collisions

A convergent collision occurs when two independent molecules from the same cell and gene receive the same UMI.

Collision probability increases with:

* short UMI length
* high transcript abundance
* high molecule count

A divergent pattern occurs when reads from one original molecule appear under multiple UMI observations because of error or protocol-specific behavior.

### Multimapping reads and UMIs

Reads may map to:

* multiple transcripts of the same gene
* multiple genes
* repeated genomic regions

Tools differ in whether they:

* discard ambiguous reads
* assign them conservatively
* use equivalence classes
* resolve assignments probabilistically
* apply expectation-maximization

These differences can lead to modest differences in gene-count matrices between tools.

## Count matrix construction

The main output is a sparse gene-by-cell matrix.

Recommended outputs include:

* filtered count matrix
* raw count matrix
* barcode metadata
* feature metadata
* cell-calling metrics
* molecule-level metadata where available
* alignment summary
* software and reference provenance

## Gene identifiers

Stable identifiers should be retained internally whenever possible.

Preferred identifiers include:

* Ensembl gene IDs
* Entrez Gene IDs where appropriate

Gene symbols are useful for interpretation but may be:

* duplicated
* renamed
* deprecated
* non-unique
* annotation-release dependent

The pipeline should preserve both:

* stable gene identifier
* display symbol

Any symbol conversion should be documented with:

* source annotation
* release/version
* mapping date
* handling of duplicated symbols

## Non-gene rows and special features

Count matrices may contain non-gene entries such as:

* alignment summary rows
* spike-ins
* antibody features
* CRISPR guide features
* multiplexing tags
* custom feature barcodes

These should be separated by feature type rather than treated automatically as gene expression.

ERCC spike-ins should not be confused with human genes such as ERCC1.

## Output retention

### Retain

* filtered matrix
* raw matrix
* metrics summary
* web summary
* molecule-level file if required downstream
* feature metadata
* barcode metadata
* reference manifest
* command-line and version metadata

### Optional

* BAM and index
* Loupe file
* secondary clustering output

### Temporary

* aligner temporary files
* stage-level execution directories
* temporary sorting files
* workflow-engine bookkeeping

Temporary files may be deleted after successful completion if:

* final outputs are validated
* checksums are recorded
* logs are retained
* rerun instructions are documented