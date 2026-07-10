# Data import and single-cell object model

## Purpose

Import quantified single-cell count matrices into a structured R object while preserving sparse representation, sample provenance, feature identifiers, and all information needed for quality control and downstream analysis.

## Supported inputs

The pipeline should support:

- Cell Ranger Matrix Market directories
- Cell Ranger HDF5 count matrices
- H5AD
- Loom
- generic sparse count matrices
- selected tool-specific outputs such as alevin-fry and kallisto|bustools

For 10x Genomics data, the preferred input is the Cell Ranger matrix directory or HDF5 output rather than CSV or Excel.

## Sparse representation

Single-cell matrices contain a large proportion of zero values.

Matrices should therefore be imported and stored in sparse form. Dense conversion should be avoided unless required by a specific downstream method and the expected memory cost has been evaluated.

## Canonical object

The pipeline uses `SingleCellExperiment` as the canonical Bioconductor object.

Relevant components include:

- `assays`: count and normalized expression matrices
- `colData`: cell-level metadata and QC metrics
- `rowData`: feature identifiers and annotations
- `rowRanges`: optional genomic coordinates
- `reducedDims`: PCA, corrected embeddings, UMAP, and related representations
- `altExps`: alternate feature spaces such as spike-ins, antibody tags, or CRISPR guides
- `metadata`: project-level provenance and analysis parameters
- `sizeFactors`: per-cell normalization factors
- `colLabels`: cluster or annotation labels

Accessor functions should be used instead of direct slot access.

## Import strategy for multiple libraries

Each library should initially be imported separately.

Before combining libraries:

1. verify that feature identifiers and feature order are compatible
2. add sample and condition metadata
3. preserve the original barcode
4. create globally unique cell names
5. record the reference build and annotation source

A recommended globally unique cell identifier is:

<sample_id>_<cell_barcode>

The original 10x barcode should remain available in colData.

## Required cell metadata

Minimum required fields:

* cell_barcode
* sample_id
* library_id
* condition
* biological_replicate
* batch

Additional fields may include:

* pool_id
* donor_id
* capture_id
* sequencing_run
* chemistry
* tissue
* species

Missing replicate information should be represented explicitly rather than inferred.

## Required feature metadata

Recommended fields:

* stable gene identifier
* gene symbol
* feature type
* gene biotype where available
* chromosome
* mitochondrial flag
* ribosomal flag

Stable identifiers, such as Ensembl gene IDs, should remain the canonical row identifiers.

Gene symbols should be retained as annotations but should not replace stable identifiers without documenting duplicate and missing symbols.

## Filtered and raw matrices

Both Cell Ranger matrices should be retained:

### Filtered matrix

Contains barcodes called as cells by Cell Ranger.

Used as the starting point for:

* cell-level QC
* normalization
* clustering
* annotation

### Raw matrix

Contains a much broader set of droplet barcodes.

Useful for:

* barcode-rank inspection
* EmptyDrops
* ambient-RNA estimation
* SoupX
* CellBender-style workflows
* alternative cell calling

Raw matrices should generally be processed separately by library.

## Synchronization and subsetting

One advantage of SingleCellExperiment is synchronized subsetting.

Subsetting cells automatically updates:

* assays
* cell metadata
* dimensional reductions
* alternate experiments

Subsetting genes automatically updates:

* assays
* feature metadata
* genomic ranges

This reduces bookkeeping errors caused by separately managing matrices and annotations.

## Combining libraries

Libraries may be combined by columns after feature compatibility has been confirmed.

The combined object should retain:

* sample identity
* condition
* replicate identity
* batch
* original barcode
* globally unique cell name

Combining objects does not imply batch correction or integration.

## Normalized assays

Raw counts must remain unchanged.

Derived assays may include:

* log-normalized expression
* corrected counts
* residuals
* imputed or denoised values

Each assay should have a clear name and documented method.

## Reduced dimensions

The following may be stored in reducedDims:

* PCA
* batch-corrected embeddings
* UMAP
* t-SNE
* diffusion maps

For reproducibility, record:

* input assay
* selected features
* number of components
* random seed
* software version
* method-specific parameters

## Annotation fields

Cell annotation should be stored at multiple levels rather than in one overloaded field.

Recommended fields:

* broad_lineage
* major_cell_type
* subtype
* state
* annotation_method
* annotation_confidence

Cell identity and cell state should remain separate when possible.

## Batch-aware preprocessing

Upstream QC and technical modeling should generally be performed within library or batch.

Examples include:

* outlier-based QC thresholds
* ambient RNA estimation
* doublet detection
* library-size normalization diagnostics
* mean-variance modeling

Objects should only be combined after library-specific metadata and QC decisions have been recorded.

## Condition–batch confounding

If each condition is represented by one library, batch and condition are perfectly confounded.

In this situation:

* aggressive integration may remove real biological differences
* corrected embeddings should be used cautiously
* uncorrected expression should be retained for condition comparisons
* downstream inference should acknowledge the lack of biological replication
* cell-level p-values should not be interpreted as replicate-supported condition effects

## Recommended outputs

The import stage should produce:
results/03_cell_qc/
├── cellranger_objects_by_sample.rds
├── combined_filtered_pre_qc_sce.rds
└── import_summary.tsv

The per-sample object collection should retain both filtered and raw matrices.

## Acceptance checks

Before proceeding, verify:

* matrix dimensions match Cell Ranger-reported cell counts
* all cells have sample metadata
* cell names are globally unique
* feature identifiers are consistent across libraries
* count matrices remain sparse
* raw counts are integer-valued
* reference and annotation provenance are recorded