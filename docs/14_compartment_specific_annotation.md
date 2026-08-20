# Stage 14: Compartment-specific biological characterization and annotation

## Stage structure

Stage 14 consists of two related but distinct operations:

### Stage 14A — Biological characterization and annotation

Generates marker/program evidence and produces reviewed biological labels for
the canonical compartment-specific clusters.

### Stage 14B — Annotation QC and propagation

Evaluates finalized annotations against the canonical transcriptional geometry
and propagates reviewed labels back to the global cell universe.

Stage 14B does not redefine the annotation framework established in Stage 14A.

## Purpose

Characterize and annotate the canonical Stage 13B compartment-specific
clusters using:

- data-driven cluster markers;
- curated lineage and functional-state gene programs;
- sample composition;
- Stage 13C clustering/integration sensitivity evidence;
- manual biological review.

This stage assigns biological identities and states to existing clusters.

It does not:

- recluster cells;
- modify Stage 13B cluster membership;
- perform condition-level differential expression;
- infer treatment effects;
- replace the canonical uncorrected representation with Harmony.

Stage 13B uncorrected clusters remain canonical.

## Inputs

Required:

- Stage 13B compartment-specific clustered SCE objects;
- Stage 13A normalized expression (`logcounts_stage13a`);
- optional Stage 13C integration-sensitivity evidence when integration was run;
- reviewed broad compartment assignments;
- curated lineage/state marker programs.

Compartments are processed independently.

## Marker detection

Cluster markers are calculated within each compartment using
`scran::scoreMarkers()`.

For each cluster:

1. compare the cluster against all other clusters in the same compartment;
2. rank genes using median AUC;
3. remove technical genes from the primary biological marker summary;
4. exclude markers detected in too few target cells;
5. retain top-ranked biological markers for annotation review.

Technical exclusions may include:

- mitochondrial genes;
- ribosomal genes;
- globin genes;
- MALAT1;
- selected predicted/uninformative genes.

Thresholds and exclusions should be configurable.

## Curated biological programs

Curated gene programs provide complementary annotation evidence.

Programs may represent:

- lineage identity;
- cellular subtype;
- functional state;
- biological context;
- competing lineage evidence.

For each cell:

program score =
mean normalized expression of genes from the program that are present
in the dataset.

Cluster-level mean and median program scores are calculated.

For visualization, each program may be standardized across clusters within
the same compartment:

relative Z =
(cluster mean - compartment mean) / compartment SD

Relative Z-scores indicate relative enrichment of one program across clusters.
They must not be used to compare absolute activity between different programs.

Program scores are annotation evidence, not hard classifiers.

## Annotation logic

Annotation should integrate:

1. data-driven cluster markers;
2. coordinated expression of canonical lineage genes;
3. curated program scores;
4. negative or competing lineage evidence;
5. transcriptional neighborhood structure;
6. Stage 13C clustering sensitivity;
7. sample composition;
8. biological context and external references where appropriate.

Single-marker assignments should be avoided.

When evidence is incomplete or multiple identities overlap, conservative labels
such as `-like`, `boundary`, or `unresolved` should be preferred.

Functional states should be stored separately from lineage identity whenever
possible.

## Annotation hierarchy

Recommended fields:

- `cell_class`
- `cell_type`
- `cell_subtype`
- `cell_state`
- `final_label`
- `plot_label`
- `annotation_confidence`
- `annotation_action`

`final_label` preserves the detailed reviewed annotation.

`plot_label` provides a simplified label suitable for visualization and
summaries.

## Annotation principles

Reusable annotation rules:

- coordinated multi-gene evidence should be preferred over single-marker
  assignments;
- positive identity evidence should be evaluated together with negative or
  competing-lineage evidence;
- lineage identity, subtype, and functional state should be represented
  separately whenever possible;
- proliferation, interferon response, stress, antigen presentation, and other
  cross-lineage programs should not automatically define a new lineage;
- reference or predefined program assignments should not override strong
  contradictory data-driven marker evidence;
- closely related or continuum populations may receive qualified labels such
  as `-like`, `boundary`, or `unresolved`;
- annotation confidence should reflect the strength and consistency of the
  underlying evidence;
- automated or reference-based annotation should be treated as supporting
  evidence unless explicitly configured as the primary annotation strategy.

Project- or tissue-specific annotation rules should be supplied through
version-controlled annotation references rather than hard-coded into the stage.

## Manual review boundary

The pipeline may generate annotation evidence and candidate labels
automatically, but final biological labels may require manual review.

The automated stage should therefore distinguish between:

- computed annotation evidence;
- candidate annotation;
- reviewed/final annotation.

Candidate labels must not silently become canonical final labels.

Recommended workflow:

computed markers/programs
→ candidate annotation table
→ optional manual/expert review
→ finalized annotation table
→ propagation to cell-level metadata

The finalized annotation table is the canonical biological decision record.

## Harmony sensitivity

Harmony-derived coordinates and clusters may be used as annotation-sensitivity
evidence.

They must not replace the canonical Stage 13B clustering when sample/library
and biological condition are confounded.

Marker statistics and biological interpretation remain based on the original
uncorrected normalized expression.

## Outputs

Per compartment:

- characterized SCE;
- marker tables;
- top-marker summaries;
- program-score tables;
- marker dotplots;
- program-score plots.

Combined:

- annotation-review table;
- curated program reference;
- finalized annotated SCE objects.

## Acceptance criteria

Before annotations are finalized:

- all canonical Stage 13B clusters are represented;
- marker detection completed successfully;
- curated program coverage is documented;
- technical genes are excluded from principal marker summaries;
- final labels are supported by coordinated biological evidence;
- ambiguous populations are labeled conservatively;
- lineage and state information are retained separately where appropriate;
- integration-sensitivity evidence has been reviewed when such an analysis was performed;
- no cluster membership has been modified implicitly;
- finalized annotations are recorded in a machine-readable table.

## Interpretation

This stage establishes biological labels for downstream descriptive
composition, expression, pathway, trajectory, and communication analyses.

Statistical interpretation of condition-associated differences is determined
by the experimental design and belongs to downstream comparison stages.



# Stage 14B: Annotation spatial consistency QC

## Purpose

Evaluate whether finalized compartment annotations are coherent within the
canonical transcriptional representation before downstream analyses.

This stage is diagnostic only.

It does not:

- recluster cells;
- alter canonical clusters;
- change annotations automatically;
- perform differential expression;
- infer treatment effects.

## Inputs

Required:

- finalized Stage 14 annotated compartment objects;
- canonical Stage 13B cluster assignments;
- canonical Stage 13B PCA;
- canonical Stage 13B UMAP;
- final annotation table;
- sample metadata.

Optional:

- Stage 13C Harmony representation for visualization;
- global pre-subclustering SCE for annotation propagation.

## Quantitative geometry

Annotation coherence is evaluated in canonical PCA space.

UMAP is used only for visualization.

A k-nearest-neighbor graph is constructed from the Stage 13B PCA coordinates.

Recommended default:

- k = 20

## Metrics

For each cell:

- canonical-cluster neighborhood purity;
- detailed-label neighborhood purity;
- grouped-label neighborhood purity.

For each cluster:

- mean and median cluster-neighborhood purity;
- final-label neighborhood purity;
- number of connected components;
- largest connected-component fraction;
- most frequent neighboring external annotations.

## QC classification

Clusters may be categorized as:

- coherent;
- mostly coherent / continuous;
- review;
- small-cluster review.

Thresholds are QC flags rather than biological decision rules.

Low cluster purity does not necessarily imply an incorrect annotation,
particularly when adjacent clusters share the same biological identity or
represent a transcriptional continuum.

Automatic QC classifications must not trigger reclustering or relabeling.

## Annotation propagation

When a global SCE is supplied:

1. verify that Stage 14 cells exactly match the cells designated for
   compartment-specific subclustering;
2. propagate finalized fine annotations to those cells;
3. retain broad upstream annotations for cells not subclustered;
4. preserve original cluster and compartment provenance.

Recommended propagated fields:

- cell class;
- cell type;
- cell subtype;
- cell state;
- detailed final label;
- grouped plot label;
- annotation confidence;
- annotation action;
- annotation resolution.

## Outputs

- per-cluster spatial-QC table;
- per-cell annotation table;
- sample-composition table;
- compartment annotation-QC SCEs;
- canonical and split-sample UMAP plots;
- optional global annotated atlas.

## Acceptance criteria

- finalized annotation table is complete;
- provisional annotations are disabled for final execution;
- all compartment cells map uniquely to annotations;
- canonical cluster membership is unchanged;
- PCA-based kNN graph is constructed successfully;
- all annotation-QC tables are generated;
- global annotation propagation matches the upstream cell manifest exactly;
- global atlas preserves cells not entering compartment subclustering.

## Interpretation

Spatial QC is supporting evidence for annotation validity.

Biological continua may legitimately have modest cluster-neighborhood purity.
Marker evidence, program scores, negative evidence, and biological context
remain the primary basis for final annotation.


## Cluster labeling
Cluster-neighborhood purity evaluates the geometric distinctness of the
canonical computational cluster.

Final-label or grouped-label neighborhood purity evaluates whether the
biological annotation is locally coherent even when multiple computational
clusters represent related states or the same broader identity.

These metrics answer different questions and should not be substituted for one
another.


## Annotation data contract

Final annotations should be machine-readable and keyed by stable identifiers.

At cluster level, the final annotation table should contain at minimum:

- compartment;
- canonical cluster identifier;
- cell class;
- cell type;
- cell subtype;
- cell state;
- detailed final label;
- grouped plotting label;
- confidence;
- annotation action;
- review notes or evidence summary.

At cell level, the same hierarchy should be propagated without altering
canonical cluster identifiers.

Downstream stages should consume these fields rather than re-deriving
biological identities from cluster numbers.