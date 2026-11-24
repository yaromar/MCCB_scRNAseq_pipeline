# Stage 12 — Broad Compartment Manifest
## Pipeline/agent implementation specification

## Pipeline role

Stage 12 converts reviewed global annotation decisions into a deterministic
cell-level routing manifest.

Its purpose is to define exactly which cells enter each downstream
compartment-specific analysis while preserving all upstream provenance.

This stage is declarative rather than analytical.

---

## Required upstream state

Expected inputs:

- canonical globally clustered SingleCellExperiment;
- immutable canonical cell IDs;
- canonical global cluster labels;
- cluster-to-compartment decision table with explicit review status;
- optional cell-level diagnostic table for explicitly heterogeneous global
  clusters;
- compartment routing policy table;
- sample and experimental-design metadata.

The cluster decision table must distinguish:

- proposed broad-compartment assignment;
- biological annotation confidence;
- supporting biological evidence;
- competing biological evidence;
- technical concerns;
- clustering-stability concerns;
- assignment rationale;
- review status and reviewer provenance.

Biological annotation confidence must not be conflated with technical
quality or clustering stability.

All upstream assays, embeddings, cluster labels, and existing cell metadata
must remain unchanged.

---

## Core invariant

Every canonical cell must receive exactly one Stage 12 broad-compartment status.

A Stage 12 assignment must be reproducible from explicit reviewed inputs.

Do not infer compartment membership ad hoc from expression inside this stage.

---

## Assignment modes

Supported assignment levels should include:

## Review-gated assignment workflow

Stage 12 supports two execution states:

### Provisional

Used when one or more annotation decisions remain under review.

The pipeline must:

1. Validate the proposed cluster assignments and routing policy.
2. Generate a provisional per-cell manifest and supporting evidence tables.
3. Generate a provisional manifest-annotated object when appropriate.
4. Identify all assignments requiring reviewer confirmation.
5. Prevent provisional outputs from being mistaken for canonical downstream
   inputs.

Provisional assignments may be used for exploratory review but must not
silently become final canonical assignments.

### Final

Used when all required biological assignment decisions have been explicitly
reviewed and approved.

The pipeline must:

1. Verify that every required decision has an approved review status.
2. Validate assignment completeness and routing-policy coverage.
3. Generate the final canonical manifest-annotated object.
4. Record the reviewed decision-table and routing-policy versions.
5. Emit the final downstream input manifest.

An assignment may be approved as `unresolved` when evidence is insufficient.
Approval means that the decision has been reviewed, not necessarily that a
specific biological identity has been established.

The pipeline must prevent stale canonical outputs from being reused after
the decision table or routing policy changes.

Projects with fully reviewed input decisions may execute directly in final
mode without a separate provisional run.

### Global-cluster assignment

Use when a reviewed global cluster has one sufficiently coherent broad lineage.

### Cell-level assignment

Use only for clusters that have already been shown by upstream diagnostics to
contain multiple broad lineages.

Cell-level assignment must consume an explicit reviewed diagnostic table.

### Unresolved

Cells without sufficient evidence for a broad lineage remain unresolved.

Do not force lineage assignment merely to maximize downstream coverage.

---

## Required provenance

For every cell, record:

### Assignment provenance

- immutable canonical cell ID;
- original global cluster;
- assigned broad compartment;
- assignment level;
- assignment source;
- biological annotation confidence;
- assignment reason;
- review status.

### Technical and clustering context

Preserve, where available:

- technical concerns;
- clustering-stability concerns;
- relevant upstream QC flags;
- doublet-detection annotations.

These fields must remain separate from biological annotation confidence.

### Experimental-design metadata

Carry forward relevant existing metadata, including:

- sample ID;
- donor or biological replicate ID, when applicable;
- library ID, when applicable;
- technical replicate ID, when applicable;
- batch and condition, when applicable.

Do not assume that libraries, technical replicates, and biological
replicates are interchangeable.

### Routing provenance

Record:

- downstream subclustering eligibility;
- primary analysis scope;
- downstream priority;
- routing-policy source.

Special diagnostic fields may also be propagated where relevant.

All original canonical cell metadata must remain available in the
manifest-annotated object.

---

## Compartment policy

Routing behavior must be configured separately from biological identity.

For every compartment define:

- `enter_downstream_subclustering`;
- `primary_analysis_scope`;
- `downstream_priority`;
- `policy_reason`.

This allows valid biological populations to remain in the canonical object
without necessarily entering the primary workflow branch.

Biological compartment assignments and downstream routing decisions must
be maintained in separate version-controlled input tables.

The compartment decision table defines biological identity.

The compartment policy table defines analytical routing.

Routing priorities must not influence biological annotation decisions.

A biologically valid compartment may be excluded from the primary
downstream branch without being excluded from the canonical object.

---

## Canonical compartment categories

The pipeline should not hard-code project-specific compartment names.

Compartment labels and policies should be supplied through a configuration or
reviewed manifest.

Typical examples may include:

- epithelial/tumor;
- fibroblast/stromal;
- monocyte/macrophage;
- dendritic/APC;
- neutrophil/granulocytic;
- T/NK/ILC;
- B cell;
- erythroid;
- off-target tissue;
- unresolved/technical.

---

## Heterogeneous-cluster handling

For any cluster routed at cell level:

1. identify the canonical cells belonging to that global cluster;
2. verify exact agreement with the diagnostic table;
3. reject duplicate cell IDs;
4. reject missing canonical cells;
5. reject unexpected cells;
6. validate diagnostic class vocabulary;
7. map reviewed diagnostic classes to broad compartments;
8. route mixed or weak evidence to unresolved unless explicitly configured
   otherwise.

The pipeline should fail rather than silently drop mismatched cells.

---

## Validation

Fail if:

- canonical object is invalid;
- required metadata columns are missing;
- cell IDs are duplicated;
- any canonical cell lacks an assignment;
- any compartment lacks a policy;
- any heterogeneous-cluster diagnostic table mismatches the canonical cell
  universe;
- unexpected diagnostic classes are encountered;
- manifest order differs from the canonical object;
- source global-cluster cell counts do not match upstream counts.

### Additional validation requirements

The pipeline must verify:

- the cluster decision table covers the complete canonical cluster universe;
- decision-table cluster IDs are unique;
- review-status values belong to an explicitly defined vocabulary;
- every assigned compartment maps to exactly one routing-policy entry;
- routing-policy values have valid types and permitted values;
- all required biological and routing provenance fields are populated;
- provisional assignments cannot silently produce final canonical outputs;
- final canonical outputs are generated only from approved decisions;
- previously generated canonical outputs cannot be mistaken for outputs
  from a newer provisional or modified decision set.

### Canonical object preservation

Stage 12 must verify that the output preserves:

- original cell and gene identifiers and ordering;
- original assay names and assay values;
- original reduced-dimension names and values;
- original global cluster assignments;
- all pre-existing cell metadata.

Only explicitly defined Stage 12 metadata may be added.

Validation results must be computed from actual input/output comparisons.
Do not report hard-coded successful validation results.

Validation failures must stop execution rather than produce apparently
successful canonical outputs.

---

## Outputs

Required:

### Manifest-annotated canonical object

Expression values and upstream reduced dimensions remain unchanged.

Only Stage 12 routing metadata are added.

### Provisional review outputs

When decisions are not fully approved, generate:

- provisional annotated object or equivalent review artifact;
- provisional per-cell manifest;
- cluster-level decision table with review status;
- supporting biological and technical evidence summaries;
- explicit list of decisions requiring review.

Provisional outputs must be clearly distinguished from final canonical
outputs in filenames and metadata.

### Cell-level manifest

One row per canonical cell.

The per-cell manifest must contain the assignment, technical,
experimental-design, and routing provenance described above.

It must preserve canonical cell ordering and contain exactly one row
per canonical cell.

### Compartment policy table

Explicit routing policy for every compartment.

### Compartment summary

Counts and sample composition.

### Source-cluster summary

Contribution of upstream global clusters to each compartment.

### Downstream input manifest

For every downstream compartment:

- cell count;
- source clusters;
- special cell-level assignments;
- canonical input object;
- exact selection rule.

### Review and QC figures

Generate figures appropriate for evaluating broad-compartment assignments,
such as:

- global embedding colored by broad compartment;
- unresolved-cell highlighting;
- source-cluster-to-compartment mapping;
- compartment composition by sample;
- compartment composition by donor or library, when applicable.

Prefer separate donor/library panels when combined overlays obscure
sample-specific patterns.

Figures are descriptive annotation diagnostics and must not imply
independent biological replication from technical libraries.

---

## Downstream contract

Downstream modules must select cells from the Stage 12 manifest.

They must not independently reconstruct broad compartments from global cluster
IDs.

This is especially important when a global cluster has been partitioned at
cell level.

Downstream analyses must consume only finalized Stage 12 assignments unless
an explicitly documented exploratory run permits provisional inputs.

Changes to approved compartment assignments or routing policies require
regeneration and validation of the affected downstream manifests.

A compartment assignment does not establish the biological homogeneity
of its constituent global clusters or the existence of distinct
within-compartment cell states.

---

## Reproducibility requirements

Record:

- canonical upstream object path and identifier;
- canonical cluster column;
- canonical cell ID column;
- sample and experimental-design columns used;
- cluster-to-compartment decision table;
- cell-level diagnostic tables, when applicable;
- compartment routing-policy table;
- assignment mode;
- provisional or final execution state;
- reviewer and approval provenance;
- software and relevant package versions;
- execution date;
- output object and manifest paths.

Record file checksums or equivalent content identifiers for the reviewed
decision table and routing-policy table.

Where practical, also record the canonical upstream object identifier or
checksum.

The recorded provenance must be sufficient to determine which exact
reviewed biological decisions and routing policies generated a given
canonical Stage 12 object.

---

## Separation of concerns

Stage 12 must not:

- normalize;
- select HVGs;
- recompute PCA;
- recluster;
- perform marker detection;
- annotate fine states;
- compare conditions.

Its only role is to convert reviewed biological decisions into a validated,
versioned routing manifest.