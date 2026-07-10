# Experimental design considerations for scRNA-seq

## Protocol classes

Single-cell RNA-seq protocols differ in throughput, transcript coverage, molecular counting strategy, and flexibility.

### Droplet-based protocols

Examples include 10x Genomics Chromium, Drop-seq, and inDrop.

Advantages:

- high cell throughput
- relatively low cost per cell
- UMI-based molecular counting
- well suited to cell-type discovery and composition analysis

Limitations:

- sparse expression measurements
- limited transcript coverage, usually near the 3' or 5' end
- elevated doublet rates at high loading concentrations
- limited ability to detect isoforms and sequence variants

Droplet-based methods are currently the dominant approach for large-scale cell atlas and tissue-composition studies.

### Plate-based protocols

Examples include Smart-seq2 and related full-length methods.

Advantages:

- higher transcript coverage
- greater sensitivity per cell
- more suitable for splicing, isoform analysis, and sequence-variant detection
- easier integration with cell sorting, imaging, and morphology

Limitations:

- lower throughput
- higher cost per cell
- often no UMIs
- stronger sensitivity to amplification bias

### UMI-based versus read-based quantification

UMI-based protocols tag original RNA molecules before PCR amplification. Reads sharing the same cell barcode, gene assignment, and UMI can therefore be collapsed into one observed molecule.

This reduces PCR amplification bias and makes molecule counts more interpretable than raw read counts.

Full-length read-based protocols generally rely on read or fragment counts rather than UMI counts and therefore require greater care when handling amplification and transcript-length effects.

## Cell number and sequencing depth

Experimental design requires balancing the number of profiled cells against sequencing depth per cell.

More cells are generally preferable when the goal is:

- detecting rare cell populations
- estimating cell-type composition
- characterizing heterogeneous tissues
- resolving branching or transitional states

Greater sequencing depth is more useful when the goal is:

- detecting weakly expressed genes
- distinguishing subtle transcriptional states
- improving pathway-level estimates
- identifying sequence variants or splice isoforms

There is no universally optimal depth. Required depth depends on:

- tissue complexity
- RNA content per cell
- expected rarity of populations
- protocol
- downstream analysis goals

UMI count per cell is often more informative than raw reads per cell when evaluating effective sequencing depth.

## Biological replication

Individual cells are not independent biological replicates.

The appropriate experimental unit is usually:

- donor
- animal
- culture
- tissue sample
- independently processed biological specimen

Multiple biological replicates per condition are required for formal condition-level inference.

Conditions should not be confounded with:

- sequencing batch
- library preparation batch
- capture run
- tissue-processing date
- operator

Pooling several biological specimens before library preparation does not preserve replicate-level information unless cells can later be reassigned to their specimen of origin through:

- sample hashing
- genetic demultiplexing
- lipid-tag multiplexing
- other sample-barcoding methods

## Pipeline implications

The pipeline should require sample-level metadata that distinguishes:

- library
- biological replicate
- condition
- batch
- capture technology
- chemistry
- sequencing lane
- species
- reference build

The pipeline should never assume that each cell is an independent replicate.

Formal differential expression should preferentially use replicate-aware pseudobulk methods when biological replicates are available.