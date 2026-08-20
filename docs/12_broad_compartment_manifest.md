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
- reviewed cluster-to-compartment decision table;
- optional cell-level diagnostic table for explicitly heterogeneous global
  clusters;
- sample metadata.

All upstream assays, embeddings, and cluster labels must remain unchanged.

---

## Core invariant

Every canonical cell must receive exactly one Stage 12 broad-compartment status.

A Stage 12 assignment must be reproducible from explicit reviewed inputs.

Do not infer compartment membership ad hoc from expression inside this stage.

---

## Assignment modes

Supported assignment levels should include:

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

- original global cluster;
- assigned compartment;
- assignment level;
- assignment source;
- assignment confidence;
- assignment reason.

Special diagnostic fields may also be propagated where relevant.

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

---

## Outputs

Required:

### Manifest-annotated canonical object

Expression values and upstream reduced dimensions remain unchanged.

Only Stage 12 routing metadata are added.

### Cell-level manifest

One row per canonical cell.

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

---

## Downstream contract

Downstream modules must select cells from the Stage 12 manifest.

They must not independently reconstruct broad compartments from global cluster
IDs.

This is especially important when a global cluster has been partitioned at
cell level.

---

## Reproducibility requirements

Record:

- canonical upstream object;
- cluster column;
- sample column;
- reviewed cluster decision table;
- cell-level diagnostic tables;
- compartment policy;
- software version;
- completion date.

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