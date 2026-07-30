## Purpose

Perform uncorrected dimensionality reduction, graph construction, clustering,
visualization, and clustering diagnostics on the Stage 09 gene-preprocessed
SingleCellExperiment.

This stage performs:

* PCA on the primary HVG set;
* retention of a defined PC subset for neighborhood analysis;
* shared nearest-neighbor graph construction;
* graph-based community detection;
* UMAP generation;
* optional t-SNE generation;
* export of cell embeddings and PCA loadings;
* cluster-size, sample-composition, silhouette, and PCA-association diagnostics;
* assembly of the canonical Stage 10 SingleCellExperiment.

No batch correction, integration, imputation, marker detection, differential
expression, or annotation is performed.

⸻

## Philosophy

Stage 10 summarizes the expression manifold without removing sample-associated
structure that may contain the biological condition of interest.

Dimensionality reduction should compact the data and reduce noise while
preserving interpretable transcriptional structure.

Clustering should be treated as an exploratory, resolution-dependent summary,
not as an estimate of a true number of biological cell types.

Clusters are computational communities that require validation using marker
coherence, sample and QC composition, cluster separation, and biological
interpretability.

No clustering metric should be used as an automatic criterion for selecting the
biologically optimal partition.

⸻

## Input contract

### Required object

A Stage 09 combined SingleCellExperiment.

### Required assays

* logcounts

PCA is performed using the independently normalized Stage 09 logcounts assay.

### Required row annotations

A logical primary HVG subset stored as

* HVG_primary

The subset must:

* align with SCE rows;
* contain no missing values;
* contain at least two genes.

### Required colData

* sample_id

The sample identifier must be complete and aligned with cells.

### Required identifiers

* unique feature identifiers;
* unique cell identifiers.

### Preserved assays and metadata

All Stage 09 assays, rowData, colData, row subsets, and metadata must remain
unchanged unless this stage explicitly adds new annotations.

⸻

## Output contract

### Preserved assays

All input assays, including:

* counts
* soupx_counts
* logcounts

No assay values are modified.

### Added reduced dimensions

* PCA_uncorrected
* PCA_clustering
* UMAP_uncorrected
* optionally TSNE_uncorrected

`PCA_uncorrected` contains all computed PCs.

`PCA_clustering` contains the subset of PCs used for graph construction and
clustering.

UMAP and t-SNE are visualizations only and must not be used as clustering input.

### Added colData

* cluster_primary

The same cluster assignments must also be stored in:

* colLabels(sce)

### Added metadata

```r
metadata(sce)$dimensionality_reduction_clustering
```

containing:

* input assay;
* feature subset;
* PCA settings;
* graph settings;
* clustering settings;
* UMAP settings;
* t-SNE settings;
* omitted analysis steps;
* interpretation constraints;
* seed;
* worker configuration;
* execution metadata.

### Saved auxiliary objects

* PCA rotation/loadings matrix;
* primary clustering result;
* primary SNN graph.

⸻

## Algorithm

Load Stage 09 combined SCE
        ↓
Validate assays, metadata, identifiers, and HVG subset
        ↓
Perform PCA on logcounts using HVG_primary
        ↓
Retain first N PCs for neighborhood analysis
        ↓
Export PCA variance and loadings
        ↓
Construct rank-weighted SNN graph
        ↓
Run graph community detection
        ↓
Store cluster labels
        ↓
Generate UMAP
        ↓
Optionally generate t-SNE
        ↓
Export cell embeddings
        ↓
Calculate clustering diagnostics
        ↓
Generate plots and summaries
        ↓
Attach metadata
        ↓
Save canonical Stage 10 SCE

⸻

## Configuration

| Variable | Meaning | Default |
|---|---|---:|
| `assay_name` | Expression assay used for PCA | `logcounts` |
| `sample_column` | Sample identifier in `colData` | `sample_id` |
| `hvg_subset` | Row subset used for PCA | `HVG_primary` |
| `pca_name` | Full PCA representation name | `PCA_uncorrected` |
| `clustering_pca_name` | PC subset used for graph construction | `PCA_clustering` |
| `n_pcs_compute` | Number of PCs to compute | 50 |
| `n_pcs_cluster` | Number of PCs used for graph construction | 30 |
| `graph_k` | Number of nearest neighbors | 20 |
| `snn_type` | SNN weighting method | `rank` |
| `cluster_function` | Community-detection algorithm | `walktrap` |
| `cluster_column` | Primary cluster label column | `cluster_primary` |
| `run_umap` | Generate UMAP | `TRUE` |
| `umap_neighbors` | UMAP neighbors | 30 |
| `umap_min_dist` | UMAP minimum distance | 0.3 |
| `umap_spread` | UMAP spread | 1 |
| `run_tsne` | Generate t-SNE | `FALSE` |
| `tsne_perplexity` | t-SNE perplexity | 30 |
| `diagnostic_pcs` | PCs included in diagnostics | 20 |
| `top_loadings_per_pc` | Positive/negative loadings exported per PC | 30 |
| `workers` | Parallel workers | configurable |
| `seed` | Random seed | configurable |

⸻

## Pipeline decisions

### Use uncorrected expression structure

PCA, graph construction, and clustering are performed on uncorrected
log-normalized expression values.

No Harmony, fastMNN, Seurat integration, Scanorama, or other batch-correction
method is applied.

The experiment contains one pooled library per biological condition.
Sample, library, and condition are therefore perfectly confounded.

Removing sample-associated variation would also risk removing the biological
condition of interest.

The pipeline must preserve sample-associated structure and describe it
explicitly rather than classify it as purely technical or biological.

⸻

### Use only selected HVGs for PCA

PCA is performed on the Stage 09 `HVG_primary` subset.

The complete expression matrix remains in the SCE.

HVG restriction is used only to reduce noise and computational cost in
dimensionality reduction.

Marker detection, differential expression, pathway analysis, and visualization
of individual genes must continue to use the complete retained gene set.

⸻

### Center genes but do not scale to unit variance

PCA should use:

```text
center: TRUE
scale: FALSE
```

Stage 09 variance modeling and HVG selection already account for mean-variance
behavior.

Unit-variance scaling would give low-variance genes the same influence as genes
with strong biological variability.

Not scaling is therefore the preferred default for this Bioconductor-first
workflow.

⸻

### Use approximate SVD

PCA should use:

```r
BiocSingular::IrlbaParam()
```

with a fixed seed.

Approximate SVD is efficient for large sparse single-cell matrices and avoids
computing unnecessary components.

The pipeline must validate the returned dimensions and verify that all PC scores
are finite.

⸻

### Compute more PCs than are used for clustering

The default strategy is:

```text
compute 50 PCs
use first 30 PCs for graph construction
```

There is no uniquely correct PC count.

Computing additional PCs enables:

* elbow inspection;
* loading interpretation;
* sensitivity analyses;
* downstream use without recomputing PCA.

A fixed transparent primary choice is preferred over automatically selecting a
PC count from heterogeneous biological variance estimates.

Potential sensitivity analyses include:

* 20 PCs;
* 30 PCs;
* 40 PCs;
* 50 PCs.

These analyses should be treated as robustness checks rather than automatic
optimization.

⸻

### Preserve PCA loadings

PCA produces:

* cell scores;
* gene loadings.

Loadings must be retained and exported.

They are required to:

* interpret transcriptional programs represented by PCs;
* identify technical or QC-associated PCs;
* distinguish biological structure from library-size or mitochondrial effects;
* document which genes drive major axes.

For each diagnostic PC, export both top positive and top negative loadings.

⸻

### Cluster in PCA space

Clustering must operate on the retained PCA representation.

The required sequence is:

```text
logcounts
    ↓
HVG_primary
    ↓
PCA
    ↓
nearest-neighbor graph
    ↓
community detection
    ↓
cluster labels
```

UMAP and t-SNE coordinates must never be used as clustering input.

Clustering directly on all genes is also outside the intended design of this
stage.

⸻

### Use graph-based clustering

Graph-based clustering is appropriate for large single-cell datasets because it:

* scales well;
* accommodates irregular cluster shapes;
* does not require Gaussian or spherical clusters;
* summarizes local transcriptomic neighborhoods.

Each cell is represented as a node.

Edges connect nearby cells in PCA space.

Community-detection algorithms identify groups with stronger internal
connectivity than external connectivity.

Graph-based clustering retains only local neighborhood structure. Differences
in sampling density can therefore affect the apparent resolution of abundant
and rare populations.

⸻

### Use a rank-weighted shared nearest-neighbor graph

The default graph is:

```text
shared nearest-neighbor
k = 20
weighting = rank
```

Two cells receive a stronger connection when their local neighbor sets overlap.

Rank weighting reduces reliance on raw Euclidean distance and emphasizes local
neighborhood agreement.

Increasing `k` generally produces broader, more interconnected communities.

Decreasing `k` generally produces finer, more local communities.

There is no universally optimal `k`.

Reasonable sensitivity settings may include:

* 20;
* 30;
* 40.

Parameter selection should be based on marker coherence and predictable cluster
splitting or merging, not on one summary metric alone.

⸻

### Use Walktrap as the primary community detector

The default clustering method is:

```text
Walktrap
```

through:

```r
bluster::clusterRows()
```

with:

```r
SNNGraphParam(
    k = 20,
    type = "rank",
    cluster.fun = "walktrap"
)
```

Walktrap is a valid OSCA/Bioconductor clustering strategy.

Leiden may be used as a sensitivity analysis but should not automatically
replace Walktrap solely because it is common in Scanpy or Seurat workflows.

Community-detection algorithms are not directly interchangeable, and their
resolution controls are implementation-specific.

⸻

### Treat clustering as exploratory and resolution-dependent

Clusters are computational communities, not automatically biological cell
types.

After marker analysis they may represent:

* broad cell types;
* cell subtypes;
* activation states;
* differentiation states;
* regions of a continuum;
* technical or QC-associated populations.

The pipeline must not claim that the number of clusters is the true number of
cell types.

The relevant question is:

```text
How well does this partition resolve the biological populations or states
needed for the study?
```

not:

```text
What is the true number of clusters?
```

Different partitions can be simultaneously useful at different biological
resolutions.

⸻

### Preserve small clusters

Small clusters must not be removed based only on size.

They may represent:

* rare immune populations;
* proliferating cells;
* transient states;
* endothelial or stromal populations;
* contaminants;
* residual doublets;
* damaged or high-mitochondrial cells.

Small clusters should instead be evaluated using:

* marker coherence;
* sample composition;
* QC distributions;
* doublet scores;
* cluster separation;
* reproducibility across clustering settings.

⸻

### Generate UMAP from retained PCs

UMAP should be generated from `PCA_clustering`.

Default parameters:

```text
n_neighbors = 30
min_dist = 0.3
spread = 1
components = 2
```

UMAP is a stochastic nonlinear visualization.

Different seeds or parameters may change the geometry while preserving major
neighborhood structure.

Do not interpret:

* axis values;
* disconnected-island distances;
* visual island area as abundance;
* trajectory direction from UMAP geometry alone.

Use UMAP cautiously to inspect:

* local neighborhoods;
* cluster overlap;
* internal heterogeneity;
* sample-associated regions;
* concentration of QC metrics.

⸻

### Keep t-SNE optional

t-SNE is optional and disabled by default.

It may be used as an independent visualization sensitivity analysis.

It must use the retained PCA representation as input.

t-SNE must not be used for clustering or inferential analysis.

⸻

## Clustering diagnostics

Diagnostics guide interpretation but do not define a biologically optimal
partition automatically.

### Cluster size

Export the number and percentage of cells in each cluster.

Cluster size identifies:

* dominant populations;
* intermediate populations;
* rare populations.

Size alone does not establish biological validity.

⸻

### Cluster centroids

Calculate mean coordinates for each cluster in retained PCA space.

Centroids provide a compact representation for:

* cluster-level visualization;
* nearest-cluster comparisons;
* downstream correspondence analyses.

Centroid distance alone is not a complete measure of separation because it
ignores cluster shape and dispersion.

⸻

### Approximate silhouette width

Exact silhouette calculation requires an all-by-all distance matrix and is
impractical for large datasets.

Use:

```r
bluster::approxSilhouette()
```

in retained PCA space.

Export:

* cell-level silhouette width;
* nearest alternative cluster;
* cluster mean silhouette;
* cluster median silhouette;
* fraction of negative silhouette values;
* overall summaries.

Interpretation:

* large positive values indicate stronger separation;
* values near zero indicate overlap or transitions;
* negative values indicate that cells may fit another cluster better.

Low silhouette width can indicate:

* over-clustering;
* continuous biology;
* related subtypes;
* heterogeneous parent populations.

Low silhouette width must not automatically trigger merging or exclusion.

⸻

### Sample composition

For each cluster and sample, export:

* cell count;
* percentage within cluster;
* percentage within sample.

Strong sample enrichment may reflect:

* true compositional differences;
* condition-associated states;
* technical library effects;
* a combination of these factors.

Because sample and condition are confounded, sample-enriched clusters must be
described as sample-associated or condition-associated rather than as
statistically validated perturbation effects.

⸻

### PCA association with sample

For each diagnostic PC:

* fit a descriptive linear model against sample identity;
* calculate sample-associated R²;
* export omnibus p-values and FDR.

These statistics are diagnostic only.

They do not separate biological condition effects from technical sample effects.

No PC should be removed based only on sample association in this experimental
design.

⸻

### PCA association with QC covariates

For each diagnostic PC, calculate Spearman correlations with available numeric
cell metadata, including where present:

* total counts;
* detected genes;
* mitochondrial percentage;
* complexity;
* size factor;
* doublet score.

These correlations help identify residual technical structure.

They must be interpreted together with:

* PCA loadings;
* embedding plots;
* known biology;
* cluster composition.

Strong QC association does not automatically imply that a PC is technical.
Different cell types can genuinely differ in RNA content, complexity, and
mitochondrial proportion.

No PC should be removed or regressed out based only on these correlations.

⸻

### Marker coherence

Marker coherence is the primary biological clustering diagnostic, but marker
testing belongs to Stage 11.

Stage 10 should only flag clusters for later review.

Potential over-clustering indicators include:

* adjacent clusters with nearly identical markers;
* differences driven only by expression magnitude;
* low silhouette width;
* cluster boundaries driven by mitochondrial, ribosomal, stress, or depth
  signatures.

Potential under-clustering indicators include:

* incompatible canonical markers within one cluster;
* broad internal expression gradients;
* reproducible substructure after lineage-specific reclustering;
* one cluster spanning several neighborhoods.

⸻

## Diagnostics not required in the primary implementation

The primary Stage 10 implementation does not need to include:

* neighborhood purity;
* within-cluster RMSD or dispersion;
* graph modularity;
* pairwise modularity matrices;
* PC and `k` parameter sweeps;
* Leiden comparison;
* adjusted Rand index;
* cluster-pair Jaccard correspondence;
* clustree-style plots;
* resampling-based stability.

These are valid future sensitivity analyses.

They should be added only when required to resolve uncertain cluster boundaries
or document robustness.

No future metric should be used alone to select a preferred clustering.

⸻

## Iterative subclustering

Lineage-specific subclustering belongs after broad annotation.

Recommended workflow:

```text
whole dataset
    ↓
broad lineage annotation
    ↓
subset one lineage
    ↓
repeat gene filtering if needed
    ↓
repeat HVG selection
    ↓
repeat PCA
    ↓
rebuild graph
    ↓
recluster
```

Global HVGs and global PCs should generally not be reused for fine subclustering.

Large between-lineage signals can obscure subtle within-lineage heterogeneity.

Subclustering should use broad, well-supported lineage assignments rather than a
single uncertain global cluster.

⸻

## Validation

The stage must verify:

* input file exists;
* input is a SingleCellExperiment;
* required assay exists;
* sample metadata exist and contain no missing values;
* feature identifiers are unique;
* cell identifiers are unique;
* the requested HVG subset exists;
* the HVG subset is logical, non-missing, and aligned to rows;
* enough genes and cells exist for the requested PCA dimensions;
* graph `k` is smaller than the number of cells;
* UMAP neighbors are smaller than the number of cells;
* t-SNE perplexity is valid for the number of cells;
* PCA scores have expected dimensions;
* PCA scores are finite;
* retained clustering PCs have expected dimensions;
* cluster assignments match the number of cells;
* cluster assignments contain no missing values;
* graph vertex count matches the number of cells;
* UMAP and t-SNE outputs are finite and correctly aligned;
* cluster labels are stored in both `colData` and `colLabels`;
* required reduced dimensions exist;
* saved cluster labels match the clustering result;
* expected outputs are written successfully.

⸻

## Recommended outputs

### Objects

* canonical combined Stage 10 SCE;
* primary clustering result;
* primary SNN graph;
* PCA rotation matrix.

### Tables

* PCA variance explained;
* PCA top loadings;
* cell embeddings;
* cluster sizes;
* cluster centroids;
* cell-level silhouette widths;
* cluster-level silhouette summary;
* cluster-by-sample composition;
* PCA-sample associations;
* PCA-QC associations.

### Plots

* PCA variance explained;
* PCA by sample;
* PCA by cluster;
* PCA by available QC covariates;
* UMAP by sample;
* UMAP by cluster;
* UMAP by available QC covariates;
* optional t-SNE by sample and cluster;
* cluster sample-composition plot.

### Metadata

* stage metadata JSON;
* sessionInfo;
* stage summary;
* execution log.

⸻

## Acceptance criteria

* Stage 09 input loads successfully.
* Primary HVG subset is available and valid.
* PCA completes successfully.
* Full and clustering PCA representations are stored.
* PCA loadings are retained when available.
* SNN graph construction completes.
* Cluster assignments are generated and aligned to cells.
* Cluster labels are written to `colData` and `colLabels`.
* UMAP completes when enabled.
* t-SNE completes when enabled.
* Cluster and PCA diagnostics are generated.
* Original assays and metadata are preserved.
* Stage metadata are recorded.
* Canonical Stage 10 SCE is saved.
* All required outputs are written.
* The stage completes without validation errors.

⸻

## Future improvements

Potential future extensions include:

* configurable PC-count sensitivity analysis;
* configurable `k` sensitivity analysis;
* Leiden and Louvain comparison;
* neighborhood purity;
* within-cluster dispersion;
* graph modularity summaries;
* cluster correspondence using Jaccard indices;
* adjusted Rand index across solutions;
* clustree-style resolution tracking;
* cell-resampling stability analysis;
* automated identification of parameter-sensitive clusters;
* optional PHATE for continuous-state visualization;
* optional density-preserving embeddings;
* reusable lineage-specific subclustering modules.
