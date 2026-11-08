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

- `counts`

### Optional assays

- `soupx_counts`

### Normalization input

The normalization input assay is configurable.

When suitable SoupX-corrected counts are available, the workflow
may use `soupx_counts`.

Otherwise, normalization may use the original `counts` assay.

The selected assay and rationale must be recorded in stage metadata.

All original assays must be preserved without modification.

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
- `size_factor_rescaled` (when cross-library scale alignment is applied)

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

The original counts assay is always preserved. When available, soupx_counts is also retained. The assay used for normalization is recorded explicitly.

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

When donors contribute unequal numbers of technical libraries, equal library weighting does not provide equal donor weighting. This should be considered when interpreting combined HVG rankings, particularly when donor and batch effects are confounded.

Genes variable in only one library are not automatically discarded because they
may represent condition-specific or rare populations.

Library-specific normalization and variance modeling should retain biological donor, technical replicate, and processing batch identifiers. Multiple technical libraries from the same donor must not be interpreted as independent biological replicates.

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

The primary HVG count is configurable and should reflect dataset complexity, cell-state heterogeneity, and downstream stability diagnostics. The default of 5,000 genes reflects the original pipeline application and is not intended as a universal recommendation.

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

### Cross-library normalization and integration

Size factors are estimated independently within each sequencing library.

Because independently estimated size factors may have arbitrary
relative scales, a documented scale-alignment step may be applied
before constructing the combined normalized expression matrix.

For example, per-library size factors may be rescaled to a shared
reference median.

This adjustment establishes a common numerical normalization scale.
It does not remove batch effects, correct donor-associated variation,
or guarantee biological comparability between libraries.

Importantly, such alignment assumes that differences in the
library-wide reference scale should not be preserved as biological
differences. This assumption should be evaluated in the context of
the experimental design.

Batch correction and integration are separate decisions and are not
performed automatically during Stage 09.

When biological conditions, donors, and processing batches are
confounded, technical and biological effects cannot be reliably
separated from the available data.

Any integration or batch-correction strategy must therefore be
evaluated and justified for the specific dataset.

One methodological nuance: Aligning medians across libraries is not guaranteed to improve biological comparability. It imposes a reference-scale assumption. 
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

Inspect the distributions of estimated size factors within each
library and identify extreme outliers.

For unusually small or large size factors, examine:

- total UMI counts;
- detected-gene counts;
- transcriptomic complexity;
- mitochondrial percentage;
- library identity;
- potential dominance by a small number of highly expressed genes.

Extreme size factors may reflect low RNA content, composition bias,
technical artifacts, or legitimate biological differences.

They should not automatically trigger cell exclusion.

If substantial outliers are present, document their prevalence,
characteristics, and potential implications for downstream analyses.

Retain relevant QC annotations for Stage 10 evaluation.

Normalization sensitivity analyses can assess the stability of
feature selection but do not independently validate individual
size-factor estimates.


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