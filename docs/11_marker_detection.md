# Stage 11 — Marker Detection and Annotation Evidence

## Pipeline role

Stage 11 converts unsupervised global clusters into quantitative biological
annotation evidence.

The stage answers:

> What transcriptional features distinguish each computational cluster, and is
> each cluster biologically coherent enough to support broad compartment
> assignment?

The output is annotation evidence, not final biological truth.

The stage must preserve uncertainty and must not force every cluster into a
specific lineage or subtype.

---

## Required upstream state

Expected upstream products:

- QC-filtered cells;
- ambient-corrected counts if ambient correction is part of the workflow;
- normalized uncorrected expression;
- canonical global clusters;
- canonical PCA/UMAP for visualization and neighborhood review;
- sample metadata;
- cluster-separation diagnostics.

Marker statistics must be computed from measured normalized expression, not
from integrated coordinates or reconstructed expression values.

---

## Canonical assay

Default:

`logcounts`

or the project-specific equivalent generated from the canonical corrected count
assay.

Do not use:

- Harmony coordinates;
- fastMNN reconstructed expression;
- integrated Seurat assay values;
- imputed expression;
- denoised expression;

for primary marker inference.

Integrated latent spaces may be used later for sensitivity analysis only.

---

## Primary marker engine

Default implementation:

`scran::scoreMarkers()`

Required characteristics:

- pairwise cluster comparisons;
- retention of full pairwise statistics;
- effect-size summaries;
- expression-detection summaries;
- ability to rank markers without relying on inferential p-values.

Equivalent implementations may be substituted only if they preserve these
properties.

---

## Statistical interpretation

Cluster-marker statistics are descriptive.

Two reasons prevent marker-test p-values from being interpreted as independent
biological inference:

1. clusters were derived from the same expression data being tested;
2. cells from a shared biological sample are not independent biological
   replicates.

Marker p-values must therefore not be used to claim:

- cluster existence;
- treatment effects;
- biological replication.

The stage should prioritize effect sizes and marker-program coherence.

---

## Primary marker quantities

Retain at minimum:

- target-cluster mean expression;
- target-cluster detection fraction;
- comparison-cluster detection fraction;
- AUC;
- Cohen's d or equivalent standardized effect;
- detection-fraction difference;
- pairwise effect summaries.

Recommended cluster-level ranking summaries:

- median AUC;
- median Cohen's d;
- mean AUC;
- detection difference;
- minimum pairwise rank.

No single ranking metric should be treated as universally optimal.

Complete pairwise statistics may be stored in serialized marker-result objects, with a manifest identifying their location and contents. Annotation-facing TSV exports may contain summarized statistics, provided the underlying full pairwise results remain accessible and reproducible.

---

## Primary versus sensitivity rankings

Canonical marker analysis:

- comprehensive;
- no or minimal fold-change threshold.

Optional sensitivity analysis:

- modest nonzero threshold such as `lfc = 0.5`.

The sensitivity run exists to determine whether biological interpretation is
stable to ranking choices.

It must not automatically overwrite the primary result.

---

## Biological marker filtering

Complete marker results must remain unfiltered.

For annotation-facing summaries, configurable exclusion lists may remove:

- mitochondrial genes;
- ribosomal genes;
- globin genes;
- MALAT1;
- predicted genes;
- other explicitly configured technical genes.

A configurable minimum target-cluster detection fraction may be used when
constructing top-marker tables.

These filters affect presentation only.

---

## Cluster review workflow

For every cluster:

1. inspect data-driven top markers;
2. inspect curated positive markers;
3. inspect negative/competing lineage evidence;
4. examine expression prevalence;
5. compare with neighboring clusters;
6. review sample composition;
7. review QC characteristics;
8. identify whether the dominant distinction is:
   - lineage;
   - subtype;
   - functional state;
   - proliferation;
   - stress;
   - technical artifact;
   - unresolved continuum.

The stage must allow a cluster to remain unresolved.

When available, incorporate upstream cluster-separation, parameter-sensitivity, sample-composition, and QC diagnostics into the Stage 11 evidence summary. These should identify clusters requiring additional review but should not independently determine biological identity, annotation confidence, or exclusion.

---

## Weak-cluster policy

Weak cluster geometry does not trigger automatic merging.

For clusters with poor silhouette or high neighborhood overlap:

- identify nearest neighboring clusters;
- perform explicit pairwise marker comparison;
- determine whether differences reflect coherent biological programs;
- distinguish lineage differences from state gradients or technical effects.

Possible outcomes:

- retain as separate lineage;
- retain as separate state;
- carry both into the same downstream compartment;
- mark unresolved;
- recommend later merge;
- exclude from primary biology if technical.

The module should produce evidence, not automatically execute destructive
merging.

---

## Rare-cluster policy

Small clusters require explicit QC.

Review:

- lineage-program coherence;
- doublet evidence;
- mitochondrial/stress signatures;
- ambient-RNA signatures;
- transcript complexity;
- sample restriction;
- cell-cycle status.

Unexpected tissue identities should not be discarded merely because they are
unexpected.

When unexpected highly expressed genes are present, evaluate whether the
program is:

- coordinated across multiple genes;
- present within individual cells;
- compatible with QC;
- retained after ambient-RNA correction.

---

## Positive and negative annotation evidence

Do not annotate from one positive marker.

Prefer coordinated programs.

Examples of general logic:

### T/NK boundary

T-cell identity requires coordinated TCR/T-cell support.

NK-associated genes alone should not imply conventional NK identity when TCR
genes are strongly expressed.

### DC/macrophage boundary

MHC-II expression alone does not define dendritic cells.

Require DC-specific identity programs and evaluate competing macrophage
evidence.

### Neutrophil/monocyte boundary

Inflammatory genes shared across myeloid cells should not define lineage.

Use coordinated neutrophil- or monocyte-specific programs.

### Fibroblast/endothelial boundary

Perivascular or vascular-associated markers may occur in fibroblast or mural
populations.

Require a coherent endothelial identity program before assigning endothelial
identity.

---

## Identity versus state

Identity and state must be represented separately.

Recommended ontology:

- broad lineage;
- cell class;
- cell type;
- subtype;
- functional state.

Examples of state programs that should not automatically define lineage:

- proliferation;
- interferon response;
- inflammatory response;
- antigen presentation;
- hypoxia;
- stress.

---

## Annotation confidence

Each cluster should support explicit confidence metadata.

Recommended values:

- high;
- moderate;
- low;
- unresolved.

Confidence should be based on:

- positive-marker coherence;
- negative evidence;
- marker prevalence;
- consistency with cluster neighbors;
- QC characteristics;
- reference agreement;
- stability across sensitivity analyses.

Computational stability and cluster separation are not sufficient to assign biological annotation confidence. Before marker-based biological review, annotation confidence should remain unresolved or pending. Assign confidence only after evaluating positive and competing lineage markers, marker prevalence, QC characteristics, and relevant reference evidence.

---

## Cross-sample marker consistency

Optional when a cluster contains sufficient cells in multiple samples.

For each eligible sample:

- rerun cluster marker detection within that sample;
- compare top marker sets and/or marker effect rankings.

Use this only as robustness evidence.

Do not call the results statistically replicated conserved markers unless the
experimental design contains genuine biological replication.

Minimum cells per sample/cluster should be configurable.

Level 1 — Expression consistency

* Mean expression.
* Detection fraction.
* Presence of marker genes across samples.

Level 2 — Discrimination consistency

* Within-sample cluster-versus-rest detection difference.
* Within-sample AUC or equivalent effect size.
* Agreement in marker direction/ranking.

Level 2 provides stronger evidence that a marker remains cluster-specific.

Also  missing cluster–sample combinations count as zero cells when determining eligibility.

---

## External annotation tools

Automated or reference-based annotation is optional supporting evidence.

Possible methods include:

- SingleR;
- reference correlation;
- reference mapping;
- atlas-based classifiers;
- marker-database classifiers.

These tools must not automatically overwrite query-derived annotations.

Store external labels separately from manual/consensus labels.

Recommended metadata:

- reference label;
- pruned/reference-confidence label;
- assignment score;
- ambiguity score;
- reference source.

Reference compatibility should consider:

- species;
- tissue;
- disease context;
- perturbation context;
- annotation resolution.

---

## Batch-correction sensitivity

If biological condition and library are confounded, integration must not be
used as the canonical marker representation.

Optional integration sensitivity may ask:

> Do broad marker-defined identities remain recognizable after cross-library
> alignment?

Marker statistics for the sensitivity clusters should still be calculated from
the original uncorrected normalized expression.

Never infer that improved sample mixing proves successful technical correction
when sample and biology are confounded.

---

## Required outputs

At minimum:

### Complete markers

One complete table containing all cluster-gene marker statistics.

### Ranked marker tables

Recommended:

- median AUC;
- median standardized effect;
- detection difference;
- minimum-rank or pairwise-discrimination ranking.

### Cluster evidence summary

For each cluster:

- cell count;
- sample composition;
- top data-driven markers;
- broad candidate lineage;
- competing lineage evidence;
- technical concerns;
- annotation confidence;
- recommended downstream action.

### Curated marker coverage

Report which reference genes are:

- present;
- absent;
- excluded by feature filtering.

### Weak-cluster review

Explicit table of weak-cluster pairwise comparisons and decisions.

### Object metadata

Attach canonical cluster labels and any finalized Stage 11 broad evidence to the
SCE without overwriting upstream cluster provenance.

---

## Validation checks

Fail the stage if:

- required assay is absent;
- cluster labels contain missing values;
- gene identifiers are duplicated unexpectedly;
- marker calculations fail for a cluster;
- marker effect values contain invalid unexpected values;
- target detection fractions fall outside `[0, 1]`;
- output cluster universe differs from the canonical upstream cluster universe.

Warn but do not necessarily fail for:

- very small clusters;
- sample-specific clusters;
- technical-marker-dominated clusters;
- low marker specificity.

---

## Reproducibility requirements

Record:

- marker engine and version;
- assay used;
- cluster column used;
- marker-ranking definitions;
- technical-gene filters;
- detection thresholds;
- optional `lfc` threshold;
- within-sample eligibility threshold;
- date and software environment.

---

## Downstream contract

Stage 11 should not produce final fine cell types.

Its primary downstream contract is a broad compartment decision table suitable
for Stage 12.

Typical downstream actions:

- retain;
- subcluster;
- exclude from primary biology;
- unresolved;
- special cell-level reassignment.

Fine analysis must recompute lineage-specific:

- gene filtering;
- normalization where required;
- HVGs;
- PCA;
- graph;
- clustering;
- marker detection.

Global marker results remain provenance but do not replace compartment-specific
marker discovery.



## Figures
For multi-sample datasets, prefer donor- or library-faceted visualizations when overlays obscure sample-specific expression or composition patterns.
