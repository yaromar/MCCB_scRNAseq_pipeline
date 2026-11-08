# Stage 07 – Doublet Detection Specification

## Purpose

Identify likely doublets in each independently processed 10x Genomics library using scDblFinder. This stage annotates cells only and does not remove cells or modify expression values.

---

## Input contract

### Required assays

- `counts`

### Optional assays

- `soupx_counts`

Doublet detection uses a configurable input assay.

The preferred input is `soupx_counts` when ambient-RNA correction has
been performed and the corrected assay is appropriate for scDblFinder.

If ambient-RNA correction was not performed or `soupx_counts` is
unavailable, the original `counts` assay may be used.

The selected assay must be recorded in
`metadata(sce)$doublet_detection`.

All original assays must remain unchanged.

### Required metadata

- Unique feature identifiers
- Unique cell identifiers
- Library/sample identifiers when processing a combined object

### Optional feature annotations

- Gene symbols, when available

Gene-symbol column names should be configurable rather than
hard-coded.

Common examples include:

- `Symbol`
- `gene_symbol`
- `gene_name`

Gene-symbol annotations may be used for reporting and interpretation
but should not be treated as a universal prerequisite for
scDblFinder execution.

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
| DOUBLET_INPUT_ASSAY | Assay used for doublet detection | soupx_counts if available; otherwise counts |
| SCDBLFINDER_CLUSTER_MODE | Artificial doublet generation mode | none |
| EXPECTED_DOUBLET_RATE | Expected doublet fraction | automatic |
| N_WORKERS | BiocParallel workers | 4 |

One additional methodological qualification: scDblFinder expects count-like data. If an ambient-corrected assay contains noninteger estimates, its compatibility should be verified rather than assuming every corrected matrix is suitable.

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

### Library-aware doublet detection

Doublet detection should ordinarily be performed separately for each
independently loaded 10x capture/library.

When the input is a combined SCE, split cells by the appropriate
capture/library identifier before running scDblFinder, then restore
annotations to the original combined object using unique cell IDs.

Biological donors and technical replicate libraries must remain
distinguishable.

Multiple technical libraries from one donor should not automatically
be pooled for doublet detection.

For multiplexed captures, the relevant unit is the original capture
rather than the demultiplexed biological sample. This distinction
must be preserved when selecting the grouping variable.

Validate that the merged annotations match the original cell IDs
and ordering.

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

This is a current implementation default, not a universal methodological rule.
For new datasets, reassess whether cluster-aware mode materially improves
doublet annotation quality and document any project-specific override.

### Annotation-only philosophy

This stage **never removes cells**.

Doublet predictions are stored as annotations and evaluated jointly with QC metrics during the downstream integrated QC review stage.

---

## Acceptance criteria

- Stage completes successfully for each intended capture/library.
- All original cells are retained.
- Original assays and expression values remain unchanged.
- Required scDblFinder annotations are present and finite where applicable.
- Output cell IDs and ordering match the input.
- Doublet classifications contain no unexpected missing values.
- Predicted doublet fractions are reported and reviewed against
  experimental expectations.
- Score distributions and QC relationships are examined for
  obvious anomalies.
- Diagnostic plots and summary tables are generated.
- Input assay, grouping variable, parameters, package version,
  and random seeds are recorded.

### Expected doublet rate and interpretation

By default, scDblFinder estimates the expected doublet rate
automatically (`dbr = NULL`).

The expected rate depends on the experimental capture configuration
and cell loading.

When reliable experimental loading information is available, a
project-specific expected rate may be supplied.

Predicted doublet fractions should be reviewed against the expected
range for the capture configuration, but agreement with the prior
does not independently validate the classifications.

Evaluate score distributions, UMI counts, detected features,
QC overlap, and biological plausibility.

Doublet classifications remain computational predictions rather
than experimentally verified multiplets.

### Doublet and cell-QC overlap

When upstream QC annotations are available, report:

- predicted doublets per library;
- fraction of doublets also flagged by upstream cell QC;
- fraction of upstream QC candidates classified as doublets;
- union of QC candidates and predicted doublets.

These summaries describe complementarity between QC methods.

Neither the overlap nor the union should automatically determine
cell exclusion. Final filtering decisions belong to the
integrated-QC stage.

---

## Future improvements

Potential future enhancements:

- support per-sample expected doublet rates from `samples.tsv`
- benchmark cluster-aware mode on additional datasets
- support multiplexed datasets (HTO, MULTI-seq)
- support experimental doublet labels for benchmarking
- evaluate additional doublet detection methods