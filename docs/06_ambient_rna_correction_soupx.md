# Stage 06 — Ambient RNA Correction (SoupX)

## Purpose

Estimate and remove ambient RNA contamination from each independently processed 10x library while preserving the original count matrix.

The objective is to reduce contamination originating from lysed cells and extracellular RNA prior to normalization and downstream biological analyses.

## Scope

This stage performs ambient RNA estimation and correction only.

It does not:

* remove cells
* remove genes
* perform normalization
* identify highly variable genes for downstream analysis
* perform dimensionality reduction
* perform biological clustering
* detect doublets
* annotate cell types

Temporary normalization, PCA, and clustering are generated solely to support contamination estimation and are discarded after correction.

## Inputs

Per-library QC-annotated SingleCellExperiment objects from Stage 05.

Corresponding Cell Ranger outputs:

* raw feature-barcode matrix
* filtered feature-barcode matrix

Required:

* counts assay
* unique feature identifiers
* gene symbols
* cell barcodes
* Stage-05 QC annotations

## Input validation

Before contamination estimation, verify that:

* Stage-05 object is a valid SingleCellExperiment
* counts assay exists
* count matrix contains no missing or negative values
* feature identifiers are unique
* cell barcodes are unique
* Cell Ranger raw and filtered matrices are present
* Stage-05 counts exactly match the filtered Cell Ranger matrix after alignment

The pipeline should terminate if any inconsistency is detected.

## Feature selection

Only Gene Expression features should be used.

Other feature types (e.g. antibody capture, CRISPR guides, multiplexing tags) should be excluded prior to contamination estimation.

Feature-type selection should be configurable.

## Temporary preprocessing

Temporary preprocessing is performed only to generate coarse transcriptional clusters required by SoupX.

Recommended workflow:

* library-size normalization
* log transformation
* gene variance modelling
* highly variable gene selection
* PCA
* shared nearest-neighbor graph construction
* Louvain clustering

These intermediates are not retained as downstream analysis results.

## Temporary feature filtering

To improve clustering stability, genes expressed in very few cells may be excluded from the temporary clustering workflow.

This filtering applies only to temporary preprocessing and must not alter:

* Stage-05 counts
* SoupX input matrices
* final corrected count matrices

The minimum detection threshold should be configurable.

## Contamination estimation

The default implementation uses SoupX.

Recommended workflow:

* construct SoupChannel
* provide temporary cluster assignments
* estimate global contamination fraction using automatic estimation
* correct counts using integer rounding

The estimated contamination fraction should be recorded for each library.

## Validation

Following correction, verify that:

* corrected matrix dimensions are unchanged
* feature order is unchanged
* barcode order is unchanged
* corrected counts contain no missing values
* corrected counts are non-negative
* sparse matrix representation is preserved

## Summary statistics

At minimum, calculate:

* estimated contamination fraction
* total UMIs before correction
* total UMIs after correction
* total UMIs removed
* fraction of UMIs removed
* per-cell removed UMIs
* per-cell contamination fraction

Generate summaries of genes with the largest ambient RNA correction.

## Outputs

results/06_soupx/
*_soupx_sce.rds
*_ambient_genes.tsv
*_cell_summary.tsv
soupx_summary.tsv
sessionInfo.txt
*_soupx_rho.pdf

## Metadata stored

The corrected object should retain both the original counts and corrected counts.

Recommended metadata include:

metadata(sce)
soupx
method
version
date
estimated_contamination
total_raw_umis
total_corrected_umis
total_removed_umis
overall_removed_fraction
preliminary_clustering

The corrected counts should be stored as a separate assay (e.g. soupx_counts) without modifying the original counts assay.

## Acceptance criteria

Before proceeding, verify:

* Stage-05 and Cell Ranger matrices align exactly
* temporary clustering completed successfully
* contamination estimate generated
* corrected count matrix validated
* original counts preserved
* corrected counts stored separately
* sparse representation preserved
* summary statistics generated
* ambient gene summaries generated
* per-cell summaries generated
* metadata recorded