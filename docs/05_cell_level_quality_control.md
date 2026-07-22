## Purpose
Perform conservative per-library quality-control assessment while preserving all
cells for downstream review.

The objective is to identify candidate low-quality cells without prematurely
excluding biologically valid populations.

## Scope
This stage performs QC metric calculation and candidate identification only.

It does not:
- remove cells
- perform ambient RNA correction
- detect doublets
- normalize counts
- filter genes

Those operations are performed in later pipeline stages.

## Inputs
Per-library pre-QC SingleCellExperiment objects.

Required:
- counts assay
- feature annotations
- sample metadata

## Metrics 
At minimum:
- total UMIs or counts
- detected features
- mitochondrial percentage
- ribosomal percentage
- hemoglobin percentage (when relevant)
- library complexity

## Feature identification

Identify biologically relevant feature classes before QC metric calculation.

At minimum, support:

- mitochondrial genes
- ribosomal protein genes
- hemoglobin genes

The identification strategy should be species-aware and configurable. For standard
mouse and human annotations, pattern matching of gene symbols is sufficient,
while alternative annotation sources may use gene biotypes or feature metadata.

The exact regular expressions or annotation rules should be documented by the
implementation.

## QC metric calculation
The default implementation uses `scuttle::perCellQCMetrics()`.

Metrics should be stored in `colData()`.

Sparse count matrices should remain sparse throughout QC metric calculation.

## Threshold generation
* adaptive
* per library
* MAD
* log transform counts
* 5 MAD default
* configurable

Thresholds should be stored alongside the QC annotations to ensure complete
reproducibility.

## Candidate QC flags
Recommended flags include:
qc_low_library
qc_low_features
qc_high_mito
qc_low_complexity
qc_high_library
qc_high_features
qc_candidate_low_quality

## Diagnostic plots
### Required plots:
* histogram UMIs
* histogram genes
* histogram mito
* histogram complexity
* ribosomal histogram
* hemoglobin histogram
* genes vs UMIs
* mito vs UMIs
* complexity vs mito

### Candidate-vs-retained diagnostics
* MA plot
* non-mito MA plot
* save full expression table

## Candidate-vs-retained diagnostics
Interpretation should focus on coordinated biological programs rather than
individual genes.

Particular attention should be given to:
- coherent lineage-marker enrichment
- coordinated stress-response programs
- unexpected metabolic signatures

Mitochondrial genes are expected to be enriched when mitochondrial percentage
contributes to candidate selection and should not be interpreted as evidence of cell-type loss.

## Outputs
results/05_qc_metrics/

*_qc_annotated_sce.rds
*_cell_qc_metrics.tsv.gz
*_qc_thresholds.tsv
*_candidate_vs_retained_expression.tsv.gz

qc_flag_summary.tsv
qc_thresholds_all_samples.tsv
qc_feature_class_summary.tsv
sessionInfo.txt

plots/

## Metadata stored
The output object should record sufficient metadata to reproduce the QC
decision independently of the project directory.

metadata(sce)

qc_thresholds

qc_nmads

qc_status

qc_date

qc_feature_counts

qc_candidate_expression_diagnostic

## Acceptance criteria
Before proceeding, verify:
- QC metrics computed successfully
- adaptive thresholds generated
- candidate flags recorded
- no cells removed
- diagnostic plots generated
- expression diagnostics generated when both candidate and retained cells exist
- sparse representation preserved
- raw counts unchanged
- QC metadata recorded

