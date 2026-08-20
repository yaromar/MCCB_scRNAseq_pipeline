# Stage 13 — Compartment-Specific Analysis
## Pipeline/agent implementation specification

## Pipeline role

Stage 13 resolves within-lineage transcriptional structure after broad
compartment assignment.

It consists of three logically separate modules:

1. compartment-specific preprocessing;
2. canonical uncorrected dimensionality reduction and clustering;
3. optional integration/batch-correction sensitivity analysis.

The corrected representation must not automatically replace the uncorrected
representation.

---
# Integration philosophy

## Experimental design versus computational correction

Experimental multiplexing and computational integration are distinct concepts.

Multiplexing methods such as sample hashing, MULTI-seq, genetic demultiplexing,
or other experimental barcoding strategies reduce technical differences by
allowing samples to be processed together.

They are experimental-design strategies and are not post hoc batch-correction
methods.

The pipeline should record multiplexing/demultiplexing information upstream but
must not treat it as equivalent to computational integration.

## Integration is optional

Integration should not be treated as a mandatory preprocessing step.

The default sequence is:

counts
→ QC
→ optional ambient-RNA correction
→ doublet handling
→ within-sample normalization
→ within-sample variance modelling
→ batch-aware/combined feature selection
→ uncorrected PCA
→ optional integration
→ neighborhood graph
→ clustering
→ UMAP
→ annotation

The uncorrected representation should therefore always be generated and retained
before any integration step.

Integration should be enabled only when there is a clear analytical reason to
align datasets.

## What integration attempts to do

Integration does not simply "remove technical noise."

Its practical objective is to construct a representation in which biologically
corresponding cells from different samples can occupy comparable neighborhoods.

Any method that forces datasets into closer agreement can also remove genuine
biological differences.

The pipeline must therefore treat integration as a modelling choice rather than
as universally correct preprocessing.

## Method families and output contracts

Integration backends differ in what they modify.

### Corrected latent-coordinate methods

Examples:

- Harmony;
- scVI latent representations.

These methods generate an alternative latent representation while leaving the
original expression assays unchanged.

### Neighborhood/graph correction methods

Example:

- BBKNN.

These modify neighborhood relationships without necessarily changing the
underlying expression matrix or PCA coordinates.

### Matching/alignment methods

Examples:

- fastMNN;
- anchor-based methods;
- Scanorama.

These identify corresponding cells or neighborhoods and estimate alignment
between datasets.

Depending on implementation, they may provide corrected coordinates and/or
reconstructed expression-like values.

### Global expression correction methods

Examples:

- ComBat;
- linear batch-removal methods.

These directly alter expression values and make stronger assumptions about the
relationship between batch and biology.

They should not be a default for heterogeneous scRNA-seq datasets.

The pipeline should expose integration methods through a common interface while
retaining method-specific output metadata.

## Corrected embedding versus corrected expression

The pipeline must distinguish explicitly between:

- corrected latent coordinates;
- corrected neighborhood graphs;
- corrected/reconstructed expression.

These objects are not interchangeable.

Corrected latent coordinates or graphs may be used for:

- neighborhood construction;
- clustering;
- visualization;
- annotation sensitivity;
- trajectory or local-neighborhood analyses when justified.

Original normalized expression should remain the default for:

- marker detection;
- differential expression;
- gene-level effect estimates;
- pathway ranking;
- GSEA;
- ligand/receptor expression summaries;
- other gene-level biological interpretation.

Corrected or reconstructed expression must never silently replace the canonical
normalized assay.

## Canonical representation policy

For every integration run, store separately:

- canonical uncorrected PCA;
- canonical uncorrected graph;
- canonical uncorrected clusters;
- corrected latent representation;
- corrected graph, if applicable;
- corrected sensitivity clusters;
- method and parameters;
- correction covariates.

An integration method must not overwrite canonical upstream representations.

## Integration-design guardrail

Before running integration, evaluate whether the proposed correction covariate
is confounded with biology.

Examples include:

- sample = treatment;
- batch = genotype;
- sequencing run = disease status.

When technical and biological variables are perfectly or strongly confounded,
the pipeline must warn that the removed structure cannot be classified as
technical from the available experiment.

Under such designs, integration should normally be treated as a sensitivity
analysis rather than as automatic correction.

## Integration evaluation

Integration quality must be evaluated along two separate axes:

1. alignment of comparable populations;
2. preservation of biological structure.

Greater sample mixing alone is not sufficient evidence of good integration.

Recommended structural diagnostics include:

- uncorrected versus corrected PCA/UMAP;
- sample distribution;
- cluster correspondence;
- adjusted Rand index;
- normalized mutual information, optionally;
- mapping purity;
- mapping entropy;
- original-cluster splitting and merging;
- neighborhood preservation.

Recommended biological-conservation diagnostics include:

- broad-lineage consistency;
- marker conservation;
- genome-wide marker-ranking correlation;
- curated program conservation;
- canonical marker overlays;
- annotation stability.

Where practical, quantitative integration metrics may be supplemented with
packages such as scIB, but no single metric should determine the canonical
representation.

## Interpretation rule

A corrected representation should be preferred only when it improves the
analysis objective without unacceptable loss of biological structure.

Possible outcomes include:

- uncorrected representation remains canonical;
- corrected representation becomes canonical for clustering/annotation;
- both are retained for different downstream purposes;
- integration is judged inappropriate for the experimental design.

The decision and rationale must be recorded explicitly.

# Stage 13A — Compartment-specific preprocessing

## Inputs

Required:

- canonical manifest-annotated SingleCellExperiment;
- cell-level compartment assignment;
- downstream-inclusion flag;
- raw/corrected count assay designated for analysis;
- sample identifier;
- technical-gene exclusion set.

Each compartment is processed independently.

## Cell selection

Select cells exclusively from the upstream routing manifest.

Do not reconstruct compartments from global cluster IDs.

## Gene filtering

Gene filtering should be configurable.

Default strategy:

retain a gene if detected above the configured threshold in at least the
configured number of cells in any biological sample/condition.

The implementation should permit condition-specific genes to survive filtering.

## Normalization

Normalize independently within biological samples inside each compartment.

Preferred method:

- scran pooled deconvolution.

Fallback:

- library-size normalization when pooled-deconvolution normalization cannot be
  estimated reliably.

Fallback use must be recorded explicitly.

Do not silently substitute normalization methods.

## Variance modelling

Estimate biological variance within each biological sample when feasible.

Combine sample-specific variance evidence using a predefined reproducible
strategy appropriate to feature selection.

This step identifies genes useful for within-compartment representation and
must not be interpreted as replicated biological variance inference when the
study lacks independent biological replicates.

Do not allow a numerically dominant sample to define the HVG space solely
because it contains more cells unless that behavior is explicitly intended.

## HVG selection

Select HVGs independently for each compartment.

Exclude configured technical feature classes from HVG selection.

The target number of HVGs should be configurable and may be reduced for small
compartments.

Record:

- genes tested;
- genes retained;
- technical genes excluded;
- HVG target;
- actual HVGs selected.

## Outputs

For each compartment:

- preprocessed SCE;
- compartment-specific normalized expression;
- variance statistics;
- HVG annotations;
- normalization diagnostics;
- preprocessing summary.

---

# Stage 13B — Canonical uncorrected clustering

## Principle

The default canonical representation is uncorrected unless the experimental
design provides sufficient evidence that a correction variable represents
technical rather than biological variation.

## Inputs

Per-compartment Stage 13A object and primary HVGs.

## Workflow

For each compartment:

1. PCA;
2. PC selection;
3. neighborhood/SNN graph;
4. graph clustering;
5. UMAP;
6. cluster composition;
7. separation/stability diagnostics;
8. provenance summaries.

## Configurable parameters

Expose at minimum:

- number of HVGs;
- number of PCs computed;
- number of PCs used;
- graph k;
- graph weighting;
- clustering algorithm;
- clustering parameters/resolution where applicable;
- UMAP neighbors;
- UMAP minimum distance;
- random seed.

Defaults should be overridable per compartment.

## Validation

Check:

- PCA convergence;
- finite coordinates;
- valid graph;
- nonempty clusters;
- cluster sizes;
- sample composition;
- low-dimensional diagnostics.

Do not reject a solution solely because silhouette values are low.

Continuous biological state spaces may legitimately produce low silhouette
values.

## Outputs

For each compartment:

- canonical clustered SCE;
- PCA;
- graph;
- clusters;
- UMAP;
- cluster/sample summaries;
- QC diagnostics.

Also generate a downstream manifest describing all compartment objects.

---

# Stage 13C — Integration sensitivity

## Activation

Stage 13C is optional.

It should be enabled when the analyst wishes to determine whether
sample-associated structure materially affects clustering or annotation.

## Design check

Before correction, explicitly determine whether the correction covariate is
confounded with a biological variable of interest.

If:

`batch/sample = biological condition`

the corrected representation must be labelled as a sensitivity analysis unless
there is independent evidence supporting technical correction.

## Corrected representation

Integration/correction should operate on a latent representation when possible.

Do not overwrite:

- counts;
- normalized expression;
- canonical uncorrected coordinates;
- canonical clusters.

Store corrected coordinates and clusters separately.

Integration may generate corrected latent coordinates, corrected neighborhood
graphs, or reconstructed expression depending on the selected backend.

The pipeline must preserve the output type explicitly and must not assume that
all integration methods return equivalent objects.

## Structural comparison

Compare corrected and uncorrected solutions using multiple complementary
metrics.

Recommended outputs include:

- adjusted Rand index;
- cluster correspondence;
- mapping purity;
- mapping entropy;
- silhouette statistics;
- sample composition.

Cluster-number differences alone are insufficient.

## Biological conservation

Evaluate biological identity using original, uncorrected expression.

Possible metrics include:

- top-marker overlap;
- marker-ranking correlation;
- gene-program conservation;
- dominant program identity.

Never calculate biological marker statistics from artificially corrected
expression unless the downstream method explicitly requires and justifies it.

Marker and program conservation should normally be evaluated from the original
uncorrected normalized expression while using corrected cluster assignments or
matched populations only to define the groups being compared.

## Visual review

Generate:

- corrected UMAP by cluster;
- corrected UMAP by sample;
- cluster-correspondence heatmaps;
- marker-conservation diagnostics;
- program-conservation diagnostics.

Visual review complements but does not replace quantitative diagnostics.

## Decision framework

Do not define corrected representations as superior solely because they show
greater sample mixing.

Review jointly:

- structural conservation;
- biological conservation;
- known experimental design;
- sample imbalance;
- expected biological continua;
- visual diagnostics.

Possible final decisions:

- retain uncorrected representation;
- adopt corrected representation;
- retain both for different purposes;
- require targeted manual review.

The decision and rationale must be recorded explicitly.

---

# Canonical-data policy

Maintain distinct concepts for:

- canonical counts;
- canonical normalized expression;
- canonical uncorrected latent representation;
- sensitivity/corrected latent representations;
- canonical clustering;
- alternative sensitivity clustering.

Downstream modules must know which representation is canonical.

---

# Reproducibility

For every compartment record:

- upstream object;
- cell-selection rule;
- sample variable;
- gene-filter rule;
- normalization method;
- fallback normalization events;
- variance-modelling method;
- HVGs;
- PCA dimensions;
- graph parameters;
- clustering parameters;
- UMAP parameters;
- integration method and version, if used;
- integration covariates;
- random seeds;
- canonical/sensitivity status.

---

# Experimental-design guardrail

Integration cannot solve confounding.

When the correction variable is perfectly confounded with biological condition,
the pipeline must warn that removed structure cannot be classified as technical
from the available data.

The pipeline should not automatically optimize sample mixing under such a
design.

---

# Downstream contract

Fine annotation should begin from the canonical Stage 13B representation while
using Stage 13C results as sensitivity evidence.

Expression-based downstream analyses should use the designated original counts
or normalized expression rather than corrected latent coordinates.

Alternative corrected clusters should never silently replace canonical cluster
IDs.