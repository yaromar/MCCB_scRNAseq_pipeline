# Stage 07 – Doublet Detection Specification

## Purpose

Identify likely doublets in each independently processed 10x Genomics library using scDblFinder. This stage annotates cells only and does not remove cells or modify expression values.

---

## Input contract

### Required assays

- counts
- soupx_counts

### Required metadata

- rownames(sce): unique feature IDs
- rowData(sce)$Symbol
- colnames(sce): unique cell barcodes

---

## Output contract

### Preserved assays

- counts
- soupx_counts

### Added colData

Required:

- scDblFinder.score
- scDblFinder.class

Additional:

- all other `scDblFinder.*` columns returned by the package

### Added metadata

```
metadata(sce)$doublet_detection
```

containing:

- method
- package version
- input assay
- cluster mode
- expected doublet rate
- random seed
- workers
- summary statistics

---

## Algorithm

```
Validate input
        ↓
Build temporary analysis SCE
        ↓
Expose selected assay as "counts"
        ↓
Run scDblFinder
        ↓
Validate outputs
        ↓
Copy all scDblFinder annotations
        ↓
Generate per-cell summaries
        ↓
Generate diagnostic plots
        ↓
Save outputs
```

---

## Configuration

| Variable | Meaning | Default |
|----------|---------|---------|
| PROJECT_DIR | Project root | ... |
| INPUT_DIR | Stage 06 outputs | ... |
| OUTPUT_DIR | Stage 07 outputs | ... |
| DOUBLET_INPUT_ASSAY | Assay used for doublet detection | soupx_counts |
| SCDBLFINDER_CLUSTER_MODE | Artificial doublet generation mode | none |
| EXPECTED_DOUBLET_RATE | Expected doublet fraction | automatic |
| N_WORKERS | BiocParallel workers | 4 |

---

## Validation

The stage must verify:

- required assays exist
- unique feature IDs
- unique cell barcodes
- `rowData(sce)$Symbol` exists
- no negative counts
- no missing values
- minimum number of cells
- output cell order is unchanged
- output feature order is unchanged
- required scDblFinder columns were returned

---

## Pipeline decisions

### Cluster-independent artificial doublet generation

Artificial doublets are generated using `clusters = NULL`.

**Rationale**

Cluster-aware simulation (`clusters = TRUE`) was evaluated during development but exhibited unstable runtime on heterogeneous tumor datasets while providing no clear improvement in downstream diagnostics.

Cluster-independent simulation produced:

- stable execution
- substantially shorter runtime
- expected doublet fractions
- clear separation between singlets and doublets

and was therefore adopted as the pipeline default.

### Annotation-only philosophy

This stage **never removes cells**.

Doublet predictions are stored as annotations and evaluated jointly with QC metrics during the downstream integrated QC review stage.

---

## Acceptance criteria

- Stage completes successfully.
- Expected doublet fractions are observed.
- Score distributions are clearly separated.
- Diagnostic plots are generated.
- Output object preserves all existing assays.
- Doublet annotations are stored successfully.

---

## Future improvements

Potential future enhancements:

- support per-sample expected doublet rates from `samples.tsv`
- benchmark cluster-aware mode on additional datasets
- support multiplexed datasets (HTO, MULTI-seq)
- support experimental doublet labels for benchmarking
- evaluate additional doublet detection methods