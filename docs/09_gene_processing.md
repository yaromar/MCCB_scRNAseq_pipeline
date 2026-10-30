## Purpose

Prepare high-quality expression values and feature annotations for downstream
dimensionality reduction.

This stage performs:

* gene filtering;
* within-library normalization;
* log-transformation;
* library-aware variance modeling;
* highly variable gene (HVG) selection;
* assembly of the canonical combined SingleCellExperiment.

No dimensionality reduction, batch correction, integration, clustering, or
visualization is performed.

⸻

## Philosophy

Stage 09 prepares expression data while minimizing technical artifacts without
removing biological differences.

Normalization should remove differences caused by sequencing depth and capture
efficiency while preserving genuine biological variation.

Feature selection should prioritize genes representing biological variability
within libraries rather than technical differences between libraries.

The complete filtered gene expression matrix is preserved. HVG selection
identifies informative subsets for downstream analyses but never discards genes
from the canonical dataset.

⸻

## Input contract

### Required assays

* counts
* soupx_counts

Normalization is performed using the SoupX-corrected soupx_counts assay.

The original counts assay must remain unchanged.

### Counts-only fallback when soupx_counts is unavailable

If `soupx_counts` is unavailable because upstream ambient-RNA correction
cannot be run (for example, missing raw droplet matrices), Stage 09 may proceed
using `counts` as the normalization input **only if this adaptation is
explicitly documented**.

When this fallback is used, record all of the following in stage metadata and
stage documentation:

* why `soupx_counts` is unavailable;
* which assay was used for normalization (`counts`);
* what adaptation checks were applied before proceeding;
* that this is a project-specific adaptation, not a silent default.

Required adaptation checks for counts-only fallback:

* counts assay exists and is non-negative;
* feature and cell identifiers are unique;
* required sample/library metadata are present;
* stage outputs preserve provenance of the fallback decision.

⸻

### Required metadata

Stage 08 filtered objects must contain:

* unique feature identifiers;
* unique cell identifiers;
* QC annotations;
* sample identifiers;
* preserved feature annotations:
    * Symbol
    * is_mitochondrial
    * is_ribosomal
    * is_hemoglobin

⸻

## Output contract

### Preserved assays

* counts
* soupx_counts

### Added assays

* logcounts

computed from

soupx_counts

using

scran::computeSumFactors()
↓
scuttle::logNormCounts()

⸻

### Added colData

* scran_precluster
* size_factor_within_sample

⸻

### Added rowData

Combined variance statistics:

* mean
* total_variance
* technical_variance
* biological_variance
* variance_p_value
* variance_fdr

Feature-selection annotations:

* hvg_rank
* hvg_primary
* hvg_primary_rank
* hvg_bio_gt_0
* n_samples_hvg_primary

Per-library statistics:

* detected_cells_
* biological_variance_
* hvg_rank_
* hvg_top_primary_

⸻

### Added row subsets

HVG_primary
HVG_top_2000
HVG_top_3000
HVG_top_4000
HVG_top_5000
HVG_bio_gt_0

These subsets provide reusable feature sets for downstream analyses while
retaining all genes in the expression matrix.

⸻

### Added metadata

metadata(sce)$gene_preprocessing

containing

* filtering parameters
* normalization parameters
* feature-selection parameters
* reproducibility information
* execution metadata

⸻

## Algorithm

Load Stage 08 filtered SCEs
        ↓
Validate inputs
        ↓
Synchronize feature space
        ↓
Gene filtering
        ↓
Normalize each library independently
        ↓
Estimate gene variance independently
        ↓
Combine variance models
        ↓
Select HVGs
        ↓
Attach feature metadata
        ↓
Assemble combined SCE
        ↓
Generate summaries
        ↓
Write outputs

⸻

## Configuration

Variable	Meaning	Default
min_detected_per_sample	Minimum detected cells within any library	10
scran_min_mean	computeSumFactors() minimum mean	0.1
primary_hvgs	Primary HVG set	5000
hvg_sets	Additional HVG sets	2000,3000,4000,5000
workers	BiocParallel workers	configurable
seed	Random seed	configurable

⸻

## Pipeline decisions

### Gene filtering precedes normalization

Genes detected in extremely few cells contribute negligible biological
information while increasing runtime, memory usage, and multiple-testing burden.

Filtering is performed before normalization and variance modeling.

The current implementation retains genes detected in at least

min_detected_per_sample

cells within any library.

This intentionally preserves genes expressed only in one biological condition.

Previous support for a combined total-cell threshold is retained in the code for
future flexibility but is not currently used.

⸻

### Normalize each library independently

Each sequencing library is normalized separately using

quickCluster()
↓
computeSumFactors()
↓
logNormCounts()

rather than normalizing the combined dataset.

This follows current OSCA/Bioconductor recommendations for heterogeneous
multi-library datasets.

Independent normalization avoids estimating size factors across biologically
distinct conditions that may differ substantially in cellular composition.

Optional: if independently estimated per-library size factors are later
rescaled to a shared reference (for example, a global median) to improve
comparability in the combined object, this must be documented as **technical
scale alignment**. It must not be described as integration or batch correction.

⸻

### Use scran deconvolution normalization

The default normalization method is scran deconvolution.

Compared with simple library-size normalization it reduces composition bias by

* pooling cells;
* estimating pool size factors;
* deconvolving pool estimates into cell-specific size factors.

This approach performs particularly well for heterogeneous 10x UMI datasets.

Alternative normalization methods such as SCTransform, analytic Pearson
residuals, Dino, bayNorm, or SCnorm are intentionally outside the scope of the
current Bioconductor-first pipeline.

⸻

### Log-normalized expression

Normalization estimates size factors.

Log transformation produces

logcounts

using

log2(normalized counts + pseudo-count)

These values are intended for

* PCA;
* clustering;
* visualization;
* neighborhood graph construction;
* most downstream exploratory analyses.

The corrected counts remain available in

soupx_counts

for methods requiring count data.

When counts-only fallback is used, replace the above assay reference with the
documented fallback assay (`counts`) in project-stage outputs, and preserve the
original integer counts unchanged.

⸻

### Variance modeling

Feature selection is based on biological variance rather than expression level.

Each library is modeled independently using

modelGeneVar()

This estimates

* total variance;
* technical variance;
* biological variance.

Separate modeling avoids fitting a single mean–variance relationship across
libraries that may differ in sequencing depth or composition.

⸻

### Combine variance models

Per-library variance models are combined using

combineVar(
    method = "fisher",
    equiweight = TRUE
)

This produces a single library-aware ranking while giving equal influence to
each sequencing library.

Important interpretation note:

`combineVar(method = "fisher", equiweight = TRUE)` combines **significance
evidence** across per-library variance models. It is not equivalent to
averaging biological variance magnitudes, and these two operations should not
be treated as interchangeable.

Genes variable in only one library are not automatically discarded because they
may represent condition-specific or rare populations.

⸻

### Preserve multiple feature sets

Rather than storing only one HVG list, the pipeline records several reusable
feature subsets.

Current defaults are

* top 2000
* top 3000
* top 4000
* top 5000
* biological variance > 0

The primary production feature set consists of the top 5000 genes.

Selecting slightly more features is generally safer for heterogeneous tumor
datasets than selecting too few.

⸻

### Preserve all genes

Feature selection does not remove genes.

All retained genes remain in

* counts
* soupx_counts
* logcounts

HVG subsets only identify preferred features for downstream dimensionality
reduction.

Differential expression, marker discovery, pathway analyses, and visualization
continue to use the complete expression matrix.

⸻

### No cross-library normalization

The combined object contains independently normalized libraries.

No additional cross-library depth rescaling is performed.

Methods intentionally not used include

* multiBatchNorm()
* Harmony
* fastMNN()
* Seurat integration
* Scanorama

The experiment contains one pooled library per biological condition.
Condition, library, and batch are therefore perfectly confounded.

Cross-library normalization or integration would risk removing genuine
condition-associated biological differences while offering no principled way to
separate technical and biological effects.

⸻

### No PCA

Stage 09 intentionally ends after feature selection.

PCA is deferred to Stage 10.

This separation keeps preprocessing independent from downstream dimensionality
reduction and allows alternative HVG sets to be evaluated without repeating
normalization.

⸻

## Validation

The stage must verify

* all input files exist;
* required assays exist;
* gene identifiers are unique;
* cell identifiers are unique;
* no negative expression values exist;
* all libraries share an identical feature space;
* gene filtering retains at least one feature;
* all size factors are finite and positive;
* if any post-normalization scale-alignment step is applied, aligned size
        factors are also finite and positive;
* variance-model rows remain synchronized;
* inherited feature annotations are preserved;
* row metadata remain synchronized across libraries;
* stored rowSubset annotations match rowData membership columns;
* combined object preserves feature order;
* combined object preserves cell uniqueness;
* required assays and metadata are written successfully.

Recommended additional validation for robustness:

* targeted normalization sensitivity check (for example, one alternative
        `computeSumFactors` setting);
* HVG stability diagnostics between primary and alternative settings using:
        * overlap metrics (for example, Jaccard);
        * rank concordance (for example, Spearman).

If sensitivity results indicate materially unstable HVG behavior without a clear diagnostic basis, document a decision checkpoint before proceeding.

### Extreme size-factor assessment

Cells with rescaled scran size factors below 0.05 were examined to assess potential normalization instability.

A total of 50 cells met this criterion. Most had low UMI counts (approximately 500–800), with correspondingly low estimated size factors. Several cells exhibited unusually low gene complexity relative to their UMI counts, including:

* PT1_TECHREP1_ATGTCCCCATGGTCCT-1: 3,363 UMIs, 181 detected genes, rescaled size factor 0.0201.
* PT1_TECHREP1_AACCCAATCAGCAAGG-1: 3,547 UMIs, 155 detected genes, rescaled size factor 0.0259.
* PT1_TECHREP3_GTGCGGTCAAACCGTG-1: 5,079 UMIs, 158 detected genes, rescaled size factor 0.0422.

These observations suggest that some extreme size factors may reflect low RNA content or highly concentrated expression profiles. However, the underlying causes cannot be established from these metrics alone.

No additional cells were excluded at Stage 9. The established Stage 8 QC criteria remain unchanged.

The extreme-size-factor cells will be flagged for inspection during Stage 10 dimensionality reduction, particularly to determine whether they disproportionately influence principal components or form technical clusters.

The normalization sensitivity analysis demonstrated high HVG stability, but this does not independently establish the accuracy of individual size-factor estimates.

⸻

## Recommended outputs

Objects

* combined Stage 09 SCE
* per-library Stage 09 SCEs
* per-library variance models
* combined variance model

Tables

* feature universe summary
* gene filtering summary
* normalization summary
* feature-selection table
* primary HVG table
* HVG summary

Plots

* library size versus size factor
* combined biological variance curve

Metadata

* stage metadata JSON
* sessionInfo
* stage summary

⸻

## Acceptance criteria

* All libraries processed successfully.
* Gene filtering completed.
* Independent normalization completed.
* Valid size factors generated.
* Variance models successfully combined.
* Primary and alternative HVG sets generated.
* Canonical combined SCE assembled.
* Feature metadata preserved.
* Original assays retained.
* Metadata recorded.
* Outputs written successfully.

⸻

## Future improvements

Potential future extensions include

* configurable gene-filtering strategies;
* optional two-pass scran normalization;
* support for alternative normalization methods (SCTransform, Pearson residuals);
* optional spike-in normalization;
* optional multiBatchNorm() for experiments with true biological replicates;
* support for deviance-based feature selection (scry);
* automated HVG sensitivity analyses;
* configurable feature-selection strategies for multi-sample studies.

## Implementation caution

When deriving per-library HVG ranks and top-gene flags, ensure that logical
filters and rank-based row subsets are applied in the **same row order**.
Do not apply a logical vector from an unsorted table to a separately sorted
rank table.

Also, name per-library columns according to their true semantics. For example,
do not label mean-expression columns as detected-cell counts.