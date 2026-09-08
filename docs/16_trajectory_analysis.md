# Stage 16: Targeted trajectory and state-continuum analysis

## Stage structure

Stage 16 evaluates whether selected annotated cellular compartments contain
transcriptional continua that are sufficiently coherent to justify directional
pseudotime analysis.

Trajectory analysis is intentionally targeted rather than applied
automatically to every compartment.

The stage is divided into five complementary operations.

### Stage 16A — Trajectory eligibility and initial topology assessment

Identifies candidate compartments and evaluates whether their transcriptional
geometry supports one or more plausible continuous state structures.

### Stage 16B — Connectivity and topology sensitivity

Evaluates whether candidate topology edges are supported by cell-level
neighborhood structure and whether topology is robust to sample/condition
composition.

### Stage 16C — Biological orientation and trajectory specification

Integrates validated topology with frozen upstream annotations and predefined
biological programs to determine whether a biologically justified root and
direction can be assigned.

### Stage 16D — Rooted pseudotime reconstruction

Fits the accepted rooted trajectory using the canonical uncorrected
dimensional representation and validates lineage structure, pseudotime
ordering, branch weights, and projection behavior.

### Stage 16E — Biological and genome-wide trajectory characterization

Characterizes accepted trajectories using curated biological programs,
selected markers, and genome-wide pseudotime modeling.

---

# Core purpose

Trajectory analysis asks a different question from clustering.

Clustering asks:

> Which transcriptionally distinct populations or states are present?

Trajectory analysis asks:

> Do some of these states form a sufficiently coherent transcriptional
> continuum that can be represented by an ordering coordinate?

A cluster graph or UMAP arrangement alone does not establish a biological
trajectory.

The pipeline must distinguish:

transcriptional proximity
        |
        v
candidate state continuum

from:

candidate state continuum
        |
        +-- topology support
        +-- cell-level connectivity
        +-- sample-composition sensitivity
        +-- biological orientation
        |
        v
accepted descriptive trajectory

Only the latter should advance to rooted pseudotime.

---

# Conceptual boundary

Pseudotime is a mathematical ordering of cells along an inferred transcriptional
manifold.

It does not by itself establish:

* measured chronological time;
* cellular ancestry;
* lineage tracing;
* physical differentiation;
* irreversible developmental progression;
* causal movement from one experimental condition to another.

Preferred terminology includes:

* transcriptional state continuum;
* maturation/state trajectory;
* pseudotime coordinate;
* inferred state progression;
* branching transcriptional manifold.

Avoid unless independently justified:

* developmental trajectory;
* differentiation pathway;
* lineage progression;
* cells transition from condition A to condition B.

---

# Experimental-design contract

Trajectory inference and treatment inference are different statistical
problems.

A trajectory can sometimes be estimated from many cells within one biological
sample because the objective is to describe transcriptional geometry.

However, biological replication is still required for treatment-level
inference about trajectory differences.

For the current project:

sgCtrl:
4 tumors pooled
        |
        v
one scRNA-seq library

sgEzh2:
4 tumors pooled
        |
        v
one scRNA-seq library

Therefore:

effective biological replication
=
1 library per condition

Cells may support estimation of within-library transcriptional structure.

They do not provide independent biological replication for comparing
conditions.

This distinction must be preserved throughout Stage 16.

---

# Hard statistical boundary

Do not infer that a treatment:

* advances cells along pseudotime;
* delays maturation;
* changes trajectory shape;
* redirects a branch;
* increases transition probability;
* alters lineage fate;

solely because cells from different conditions occupy different regions of an
inferred trajectory when each condition consists of one pooled library.

Condition-specific occupancy may reflect:

* biological treatment effects;
* biological sample differences;
* state-composition differences;
* technical library effects;
* selective recovery;
* condition-confounded transcriptional states.

Without replicated biological samples, these possibilities cannot be separated
statistically.

---

# Canonical upstream inputs

Stage 16 should consume finalized upstream products rather than recreate them.

Typical input sequence:

Stage 13A
canonical uncorrected PCA / normalized expression
        |
        v
Stage 13B
canonical clustering
        |
        v
Stage 14 / 14B
reviewed biological annotations
        |
        v
Stage 15
condition-associated programs and pathway context
        |
        v
Stage 16

Required upstream information should include:

* canonical uncorrected dimensional representation;
* canonical cluster assignments;
* finalized biological annotations;
* sample/condition metadata;
* uncorrected normalized expression;
* raw or corrected raw counts;
* gene identifiers and symbols;
* optional frozen biological-program definitions.

---

# Representation contract

Trajectory inference should operate on a biologically interpretable,
uncorrected transcriptional representation whenever possible.

For the current pipeline:

`PCA_stage13b_clustering`

is the canonical trajectory representation.

Use PCA or another validated uncorrected latent representation for:

* neighborhood geometry;
* cluster centroids;
* trajectory fitting;
* pseudotime estimation;
* topology sensitivity.

UMAP may be used for:

* visualization;
* qualitative inspection;
* plotting fitted curves.

UMAP should not be treated as the primary trajectory-inference substrate.

The general principle is:

high-dimensional canonical representation
        ->
trajectory inference

low-dimensional visualization
        ->
display only

---

# Integration-corrected representations

Harmony or another integration-derived representation may be useful for:

* sensitivity analysis;
* visualization;
* evaluating whether sample-driven structure dominates.

However, integration-corrected coordinates should not automatically replace the
canonical uncorrected representation for trajectory inference.

This is especially important when biological condition and sample identity are
confounded.

Integration may deliberately remove the same variation that the biological
question aims to characterize.

---

# Stage 16A: Candidate selection

## Purpose

Identify compartments where trajectory analysis is biologically plausible and
technically meaningful.

Trajectory analysis should not be run indiscriminately on every cell type.

Useful candidate compartments typically contain:

* multiple related transcriptional states;
* sufficient cell numbers;
* plausible biological continuity;
* nontrivial within-compartment structure;
* enough support for neighborhood-based topology estimation.

Poor candidates include:

* clearly unrelated cell classes;
* compartments dominated by isolated condition-specific states;
* very small populations;
* populations without plausible continuous biology;
* compartments where topology is entirely determined by technical/sample
  separation.

---

# Candidate eligibility manifest

Recommended output:

`trajectory_candidate_manifest.tsv`

Suggested fields:

compartment
cells
clusters
n_pcs_used
condition_exclusive_clusters
shared_between_conditions_clusters
median_condition_cell_ratio
max_condition_cell_ratio
centroid_mst_median_edge_length
centroid_mst_max_edge_length
slingshot_lineages
slingshot_centroid_edge_jaccard
alternative_topology_edge_jaccard
trajectory_direction_assigned
trajectory_interpretation_status

This manifest should summarize eligibility rather than make an automatic
biological decision.

---

# Stage 16A: Initial topology

At least one primary topology method should be used.

Useful approaches include:

* Slingshot;
* cluster-centroid minimum spanning tree;
* TSCAN;
* another graph-based trajectory method appropriate to the data.

Using multiple methods is only useful if they provide meaningfully independent
information.

---

# Method redundancy

Agreement between two methods is not automatically independent validation.

For example:

Euclidean cluster-centroid MST
and
TSCAN cluster MST

may produce identical topology because both are based on closely related
centroid-distance MST logic.

Therefore:

method agreement
!=
independent validation

The pipeline should document whether topology-comparison methods are genuinely
different in their assumptions and construction.

Do not add methods solely to increase method count.

---

# Unrooted first-pass rule

Stage 16A should generally fit topology without imposing a biological root.

The objective is initially to ask:

> What topology does the transcriptional geometry support?

rather than:

> Can the data be forced to match an expected biological sequence?

Root assignment should occur only after topology and connectivity have been
evaluated.

---

# Stage 16B: Cell-level connectivity

Cluster-centroid topology can create edges between clusters whose centroids are
near each other even when few cells directly occupy the corresponding boundary.

Therefore candidate edges should be evaluated using cell-level connectivity.

Useful diagnostics include:

* cross-cluster k-nearest-neighbor connectivity;
* mutual-nearest-neighbor connectivity;
* boundary-cell connectivity;
* neighborhood overlap;
* cluster-pair distance distributions.

No single metric should automatically determine biological validity.

---

# kNN connectivity

For each cluster pair, quantify the extent to which cells in one cluster have
nearest neighbors in the other cluster.

Possible summary fields include:

cluster_a
cluster_b
cells_a
cells_b
a_to_b_knn_fraction
b_to_a_knn_fraction
minimum_directional_fraction
mean_directional_fraction

Highly asymmetric support may still be biologically meaningful, especially
when population sizes differ.

---

# Mutual-nearest-neighbor connectivity

Pairwise MNN support may complement direct kNN connectivity.

MNN should be treated as a topology diagnostic rather than as an integration
or batch-correction operation unless explicitly used for that purpose.

Possible output fields:

cluster_a
cluster_b
mnn_pairs
fraction_a_with_mnn
fraction_b_with_mnn
minimum_mnn_fraction

Avoid universal hard thresholds.

Connectivity metrics should inform manual or rule-based review in combination
with topology, biology, and sample composition.

---

# Geometric-bridge detection

A global topology may contain edges that arise because cluster centroids are
geometrically convenient rather than because the corresponding populations
form a plausible cell-level continuum.

Possible warning signs include:

* unusually long centroid edges;
* negligible direct kNN connectivity;
* very weak MNN support;
* edges linking condition-exclusive states;
* failure to recur within sample-restricted topology.

Such edges should be flagged as potential geometric bridges.

They should not automatically be interpreted as biological transitions.

---

# Stage 16B: Sample-stratified topology sensitivity

If multiple samples or conditions contribute to the candidate compartment,
trajectory topology should be reconstructed within individual samples when
sufficient states are represented.

Possible analyses include:

* sample-specific centroid MST;
* sample-specific Slingshot;
* sample-specific connectivity;
* comparison of global and sample-specific edge sets.

These analyses answer:

> Is the inferred topology present within a biological library, or does it
> emerge primarily because different libraries occupy different states?

---

# Topology sensitivity metrics

Possible comparison metric:

edge Jaccard similarity

For edge sets A and B:

J(A,B)
=
|A intersection B|
/
|A union B|

Useful comparisons include:

global centroid MST
vs
sample-specific centroid MST

global Slingshot
vs
sample-specific Slingshot

These are sensitivity diagnostics rather than hypothesis tests.

---

# Sample-restricted analyses are not biological replication

When each condition contains one pooled library:

sample-specific topology
!=
replicated trajectory inference

It is a structural sensitivity analysis.

For example:

global topology
        |
        +-- reproduced within sgCtrl
        +-- partially reproduced within sgEzh2
        |
        v
greater structural confidence

This does not establish a replicated treatment effect.

---

# Two-state sample subsets

A condition may contain only two sufficiently represented states.

In that case:

* local connectivity can still be characterized;
* a local state continuum may be retained;
* a multi-state trajectory or branching model should not be forced.

Failure to fit Slingshot because only two supported clusters exist is not
negative evidence against local connectivity.

---

# Stage 16C: Biological orientation

## Purpose

Determine whether an accepted topology can be assigned a biologically justified
direction.

Biological orientation should occur only after topology has been evaluated
without directional constraints.

Evidence may include:

* known maturation markers;
* proliferation programs;
* differentiation-associated programs;
* independently defined biological programs;
* metabolic or functional gradients;
* prior experimental knowledge;
* cell-cycle state;
* developmental markers;
* external reference information.

---

# Frozen annotation rule

Final upstream biological annotations may be used to interpret topology.

They should not normally be used to construct topology.

Recommended separation:

Stage 13B clusters
        ->
topology units

Stage 14/14B labels
        ->
biological interpretation

This reduces circularity.

---

# Frozen program rule

Where biological programs were defined upstream, reuse them unchanged for
trajectory orientation.

Do not silently redefine a program after seeing the trajectory unless the new
program is explicitly labeled post hoc.

This preserves the distinction between:

pre-specified biological evidence

and:

trajectory-motivated exploratory evidence

---

# Composite orientation scores

Avoid collapsing biologically distinct axes into one arbitrary
"early-to-late" score unless there is a clear biological justification.

For example, macrophage programs such as:

* resident-like;
* recruited/inflammatory;
* TAM/remodeling;
* antigen presentation;
* interferon response;

may represent partially orthogonal state dimensions.

A single weighted orientation index may obscure real branching or multidimensional
biology.

Prefer examining individual program and marker gradients.

---

# Root selection

A root should be selected using explicit biological evidence.

Appropriate evidence may include:

* proliferative/precursor identity;
* immature marker expression;
* known progenitor state;
* early differentiation program;
* experimentally known starting population.

Do not root a trajectory solely because a cluster:

* is sgCtrl;
* has the lowest first principal component;
* appears at one end of UMAP;
* has the smallest cluster number;
* is visually convenient.

---

# Terminal-state selection

Terminal states should preferably emerge from topology rather than being forced.

If known terminal states are supplied to a trajectory algorithm, this should be
documented explicitly.

The preferred default is:

biological root supplied
+
terminal structure inferred

when topology supports this design.

---

# Directionality decision classes

Recommended machine-readable statuses include:

accept_directed_descriptive_trajectory
retain_undirected_state_manifold
retain_undirected_branching_state_manifold
retain_undirected_local_continuum
reject_trajectory
insufficient_evidence

Suggested output:

`stage16c_trajectory_decision_manifest.tsv`

---

# Directed versus undirected biological structure

Not every coherent topology should receive pseudotime.

A compartment may be biologically meaningful as an undirected state manifold.

Examples:

macrophage state structure
        |
        +-- resident-like
        +-- TAM/remodeling
        +-- APC/IFN
        +-- inflammatory/recruited

may represent connected state relationships without one defensible universal
developmental axis.

In such cases the correct decision is to retain the manifold without assigning
a root.

Not forcing pseudotime is a valid analytical result.

---

# Stage 16D: Rooted trajectory reconstruction

Once a trajectory is accepted:

1. subset the exact intended cells;
2. use the canonical upstream representation;
3. specify the biologically justified root;
4. refit or reconstruct the trajectory;
5. extract lineage structure;
6. validate topology;
7. extract pseudotime and lineage weights.

---

# Rooted topology validation

The fitted rooted topology should be compared with the accepted Stage 16C
working structure.

Recommended outputs include:

`stage16d_rooted_slingshot_lineage_manifest.tsv`

`stage16d_rooted_slingshot_edges.tsv`

`stage16d_expected_vs_observed_topology.tsv`

Useful metrics include:

* expected edge recovery;
* lineage count;
* recovered terminal states;
* cluster ordering.

Topology should not be accepted merely because pseudotime values were
successfully returned.

---

# Pseudotime summaries

For each cluster and lineage summarize:

cluster
lineage
cells
finite_pseudotime_cells
median_pseudotime
mean_pseudotime
pseudotime_q25
pseudotime_q75

Expected biological ordering should generally be reflected in median
pseudotime.

Minor cell-level non-monotonicity is expected.

---

# Lineage weights

Branching trajectory methods may assign each cell a lineage weight.

These weights should be inspected.

Recommended summaries include:

cluster
lineage
mean_weight
median_weight
fraction_weight_gt_0
fraction_weight_gt_0_5

Expected pattern for a bifurcation:

shared trunk
        ->
high membership in both lineages

branch region
        ->
fractional lineage membership

terminal branch A
        ->
high lineage-A weight
low/zero lineage-B weight

terminal branch B
        ->
high lineage-B weight
low/zero lineage-A weight

Fractional weights near a branchpoint are expected and can support the
branching interpretation.

---

# Shared-trunk concordance

For branching trajectories, pseudotime estimates from the two lineages should
be compared across cells assigned to the common trunk.

Useful metrics include:

* Pearson correlation;
* Spearman correlation;
* median absolute difference;
* mean absolute difference;
* upper quantiles of absolute difference.

Strong agreement along the pre-branch trunk and lower agreement near the
branchpoint is expected in a coherent bifurcating model.

---

# Projection anomalies

Principal-curve fitting may occasionally assign unusual pseudotime coordinates
to otherwise valid cells.

A cell should not be removed solely because of an extreme trajectory
coordinate.

Investigate:

* raw library complexity;
* detected genes;
* mitochondrial fraction;
* doublet/QC flags;
* biological annotation;
* PCA position;
* UMAP position;
* neighboring cells;
* lineage weights.

If independent QC is normal, retain the cell and treat the trajectory value as
a projection anomaly.

Trajectory coordinates must not become a circular cell-filtering criterion.

---

# Stage 16E: Curated biological characterization

## Purpose

Determine whether the accepted trajectory exhibits biologically coherent
program and marker changes.

The preferred sequence is:

accepted topology
        |
        v
predefined biological programs
        |
        v
selected marker genes
        |
        v
gene-detection QC
        |
        v
genome-wide trajectory modeling

This order reduces the risk of constructing the biological interpretation only
after seeing genome-wide statistical rankings.

---

# Program scoring

Stage 16E should reuse the same normalized expression substrate and program
definitions used upstream where possible.

For the current pipeline:

`logcounts_stage13a`

is used for descriptive program and marker summaries.

Program scores may use:

mean normalized expression
of program genes present

or another documented scoring approach.

Program definitions and feature coverage must be recorded.

---

# Program-coverage contract

Recommended fields:

program
requested_genes
genes_present
genes_missing
coverage_fraction

Do not silently replace missing genes.

Do not redefine upstream programs simply because one or more genes are absent.

Coverage below a configurable threshold should produce a warning.

---

# Pseudotime scaling for visualization

For lineage-specific visualization, pseudotime may be rescaled:

scaled_pseudotime
=
(pseudotime - minimum)
/
(maximum - minimum)

yielding values from 0 to 1.

Scaled pseudotime is useful for comparing trajectory shapes across lineages.

It must be labeled as a normalized visualization coordinate rather than the
native fitted pseudotime.

---

# Weighted pseudotime-bin summaries

For visualization, cells may be grouped into pseudotime bins.

For each lineage and bin, summarize:

* mean program score;
* median program score;
* mean marker expression;
* detection fraction.

Lineage weights should be incorporated where appropriate so that branch-region
cells contribute proportionally.

The number of bins should be configurable.

Binned plots are descriptive visual summaries, not statistical tests.

---

# Marker-level validation

Selected biological markers should be chosen before genome-wide modeling when
possible.

Useful marker categories may include:

* proliferative;
* immature;
* lineage maturation;
* recruitment;
* inflammatory;
* interferon-responsive;
* terminal-state context;
* post hoc QC programs.

Marker trends should be evaluated jointly rather than relying on one gene.

---

# Sparse-marker QC

Genes with low detection should not be interpreted from smoothed expression
alone.

For low-expression markers summarize:

lineage
pseudotime_bin
cells
fraction_detected
mean_expression
median_expression

A sparse gene may support an interpretation but should not dominate it.

Prefer aggregate program behavior when several related genes are individually
sparse.

---

# Post hoc programs

Unexpected upstream findings may justify exploratory trajectory programs.

Such programs should be explicitly labeled as post hoc.

They should not be presented as independent validation of a trajectory that
motivated their creation.

Example categories might include:

* ECM/mesenchymal-like;
* stress-response;
* specialized inflammatory signatures.

Interpretation should consider:

* absolute score magnitude;
* gene-level detection;
* coordinated versus isolated expression;
* trajectory-wide consistency.

---

# Stage 16E: Genome-wide modeling

For accepted directed trajectories, genome-wide expression dynamics may be
modeled using tools such as:

* tradeSeq;
* generalized additive models;
* another pseudotime-aware expression framework.

The model should consume the already accepted trajectory.

Do not silently refit topology while performing gene-level trajectory analysis.

---

# Count substrate

Genome-wide GAM fitting should use raw or corrected raw gene-level counts.

Normalized expression should be used for descriptive visualization rather than
as the count likelihood substrate unless the chosen method explicitly requires
otherwise.

For the current pipeline:

SoupX-adjusted counts
        ->
trajectory gene filtering
        ->
tradeSeq GAM

---

# Gene identity

Feature rownames may contain stable identifiers such as Ensembl IDs while
biological gene symbols are stored separately.

The pipeline must explicitly identify the gene-symbol field.

Do not assume:

rownames(object)
=
gene symbols

Recommended validation:

* unique stable feature identifiers;
* biological symbol column present;
* duplicate biological symbols handled deterministically.

---

# Duplicate gene symbols

If duplicate biological symbols are collapsed for genome-wide modeling, the
aggregation rule must be documented.

For raw counts, a defensible rule is:

duplicate symbol
        ->
sum count rows

Do not average raw counts across duplicated identifiers.

If no collapsing is performed, outputs should preserve stable feature IDs to
avoid ambiguity.

---

# Gene filtering for GAM fitting

Low-information genes should be removed before genome-wide trajectory fitting.

Configurable criteria may include:

* minimum detected cells;
* minimum detection fraction;
* minimum total count;
* combination of detection and count thresholds.

Filtering is computational and statistical preprocessing.

It must not be tuned after seeing desired biological results.

---

# tradeSeq pseudotime contract

Slingshot may represent cells outside a given lineage with missing pseudotime:

`slingPseudotime(..., na = TRUE)`

This is useful for descriptive interpretation.

Some downstream tools such as `tradeSeq::fitGAM()` require a complete
pseudotime matrix.

In that case:

descriptive pseudotime
=
`slingPseudotime(..., na = TRUE)`

GAM input pseudotime
=
`slingPseudotime(..., na = FALSE)`

with:

canonical Slingshot lineage weights

This preserves lineage contribution through the weights while satisfying the
complete-matrix requirement.

Do not manually impute missing pseudotime values.

Do not overwrite the canonical descriptive pseudotime representation.

---

# tradeSeq input QC

Before fitting, record:

cells
genes
lineages
descriptive_pseudotime_missing
GAM_pseudotime_missing
zero_weights_by_lineage
finite_pseudotime_range
weight_distribution

Terminal cells belonging exclusively to the opposite lineage should normally
retain zero weight for that lineage.

---

# Number of knots

The number of GAM knots should be configurable.

Where computationally practical, `evaluateK()` may be used to inspect candidate
values.

The goal is not to optimize until a preferred biological result appears.

A reasonable stable value should be selected and recorded.

Knot selection is a modeling choice, not a biological discovery.

---

# Model convergence

Convergence must be evaluated explicitly.

Recommended output:

`stage16e_tradeseq_convergence.tsv`

Fields may include:

gene
converged
detected_cells
detection_fraction
total_counts

The pipeline should report:

genes_tested
genes_converged
convergence_fraction

Failed fits should be retained in metadata and excluded from downstream
ranking.

---

# Genome-wide trajectory tests

Different tests answer different biological questions.

## Association test

Question:

> Does modeled gene expression vary along pseudotime?

Useful for identifying genes associated with the trajectory.

## Start-versus-end test

Question:

> Does modeled expression differ between the beginning and end of a lineage?

Useful for coarse root-to-terminal characterization.

## Differential-end test

Question:

> Do the fitted endpoints of two lineages differ?

Useful for comparing terminal branches.

## Pattern test

Question:

> Do fitted expression trajectories differ across the full shapes of two
> lineages?

This may be useful when the biological question concerns branch-wide dynamics.

It is optional.

Do not add tests solely to increase method count.

---

# Test-output normalization

Package APIs may return inconsistent column names across versions.

The pipeline should validate and normalize output schemas before downstream
processing.

For example, if `associationTest()` returns one unnamed terminal column that
corresponds to the package's mean log-fold-change summary, rename it explicitly
after validating its position.

Do not rely on unnamed columns downstream.

---

# Numerical p-value underflow

Strong cell-level associations may produce p-values numerically equal to zero.

This is a floating-point result.

It should not be interpreted as infinite certainty.

For descriptive ranking, prioritize a combination of:

* successful convergence;
* valid Wald tests;
* Wald-statistic magnitude;
* expression prevalence;
* effect-size context.

Do not rank solely by raw p-value.

---

# Descriptive ranking fields

Useful derived fields include:

min_descriptive_pvalue
max_wald
n_valid_wald_tests
detection_fraction
detected_cells

Recommended sorting may prioritize:

1. successful convergence;
2. number of valid Wald tests;
3. maximum Wald statistic;
4. minimum descriptive p-value;
5. expression prevalence.

Exact ranking logic should be documented.

---

# Terminal-branch direction

The sign convention of endpoint log-fold-change statistics should be verified
once against observed or fitted endpoint expression.

For example:

positive `logFC1_2`
=
higher fitted terminal expression in lineage 1

negative `logFC1_2`
=
higher fitted terminal expression in lineage 2

The pipeline should not assume sign direction without validating the installed
method implementation.

A small set of genes spanning positive and negative values should be checked
against observed terminal-state expression before final interpretation.

---

# Statistical interpretation of genome-wide trajectory models

Cell-level GAM statistics describe expression structure conditional on the
inferred trajectory.

They do not create biological replication.

With one pooled library:

strong cell-level trajectory association
!=
replicated biological effect

Preferred language:

* trajectory-associated gene;
* gene varies along pseudotime;
* fitted terminal states differ transcriptionally;
* descriptive genome-wide trajectory characterization.

Avoid:

* significantly altered by treatment;
* significantly induced by knockout;
* replicated differential trajectory expression.

---

# Biological synthesis

Stage 16E should integrate:

trajectory topology
+
program trends
+
marker trends
+
genome-wide trajectory genes
+
terminal-branch contrasts

The genome-wide model should ideally recapitulate major biological themes
identified independently by curated programs and markers.

This increases internal biological coherence.

It does not convert the trajectory into lineage-tracing evidence.

---

# Relationship to condition-associated analysis

Stage 15 and Stage 16 answer different questions.

Stage 15 asks:

> How do observed populations, pathways, and expression profiles differ between
> conditions?

Stage 16 asks:

> Which biological programs vary along a supported transcriptional state
> continuum?

These should not be conflated.

For example:

Stage 15
        ->
sgEzh2-associated neutrophil state redistribution

Stage 16
        ->
supported sgCtrl neutrophil maturation/state continuum

does not imply:

sgEzh2 drives cells forward along sgCtrl pseudotime

unless additional replicated evidence supports that conclusion.

---

# Project-specific neutrophil example

The current project accepted one directed sgCtrl neutrophil trajectory:

cluster 5
proliferating immature / granule-biogenesis-high
        |
        v
cluster 1
immature / granule-biogenesis-high
        |
        v
cluster 3
classical / recruited
        |
        v
cluster 4
inflammatory branch region
       / \
      v   v
cluster 2     cluster 6
Ccrl2-        inflammatory
associated    terminal state
inflammatory  with stronger late
terminal      IFN/ISG activity

Two lineages were recovered:

lineage 1:
5 -> 1 -> 3 -> 4 -> 2

lineage 2:
5 -> 1 -> 3 -> 4 -> 6

This should be treated as a project-specific implementation example rather than
a generic pipeline expectation.

---

# Project-specific macrophage example

The current macrophage compartment contained biologically interpretable local
state relationships but did not support one sufficiently robust directed
trajectory.

The correct Stage 16 decision was therefore:

sgCtrl macrophages
        ->
undirected state manifold

sgEzh2 macrophages
        ->
undirected branching state manifold

This illustrates an important pipeline principle:

trajectory analysis is allowed to conclude that no rooted pseudotime should be
assigned.

---

# Acceptance criteria

Stage 16 should not be considered complete until:

* candidate compartments were selected intentionally rather than automatically;
* the canonical uncorrected representation was used for topology inference;
* UMAP was not used as the sole trajectory substrate;
* integration-corrected coordinates were not substituted without justification;
* sample/condition support was summarized for all candidate clusters;
* condition-exclusive or highly imbalanced states were identified;
* initial topology was fit without biologically forcing direction;
* topology-method redundancy was documented;
* candidate edges were evaluated using cell-level connectivity;
* possible geometric bridges were identified;
* sample-restricted topology sensitivity was evaluated where feasible;
* sample-specific topology was not interpreted as biological replication;
* biological orientation used explicit evidence;
* frozen upstream annotations were not used circularly to construct topology;
* frozen upstream biological programs were reused where appropriate;
* roots were biologically justified;
* compartments without defensible direction remained undirected;
* rooted topology matched or adequately explained the accepted specification;
* cluster-level pseudotime ordering was inspected;
* lineage-weight behavior was inspected;
* shared-trunk concordance was evaluated for branching trajectories;
* trajectory-coordinate anomalies were evaluated against independent QC;
* cells were not filtered solely because of pseudotime anomalies;
* program-gene coverage was recorded;
* sparse trajectory markers received detection-level QC;
* genome-wide GAM modeling used raw/corrected raw counts;
* gene symbols were resolved explicitly;
* duplicate-symbol handling was documented;
* trajectory genes were filtered using a predefined rule;
* the exact accepted trajectory was retained during genome-wide modeling;
* complete pseudotime input requirements were handled without manual
  imputation;
* model convergence was recorded;
* trajectory test outputs were schema-validated;
* p-value underflow was handled appropriately;
* terminal-branch effect direction was verified;
* genome-wide statistics were interpreted descriptively when biological
  replication was absent;
* final metadata records all major analytical decisions;
* the distinction between transcriptional pseudotime and biological lineage
  tracing is stated explicitly.

---

# Interpretation language

Preferred:

* transcriptional state continuum;
* maturation/state trajectory;
* inferred pseudotime;
* state ordering;
* branch region;
* terminal transcriptional state;
* trajectory-associated gene;
* fitted terminal endpoint;
* consistent with;
* supports the state interpretation;
* descriptive.

Avoid without independent evidence:

* lineage tracing;
* developmental fate;
* differentiation proven;
* cells transition from control to treatment;
* treatment pushes cells along pseudotime;
* treatment redirects differentiation;
* significantly altered trajectory by treatment.

A durable caveat is:

Because pseudotime is inferred from transcriptional geometry, the resulting
ordering should be interpreted as a state coordinate rather than direct
evidence of cellular ancestry or measured temporal progression. When biological
conditions are represented by one pooled library each, differences in trajectory
occupancy or state distribution are descriptive and cannot establish replicated
treatment-level effects.

---

# Metadata contract

Recommended Stage 16 metadata should encode both trajectory status and
inferential limits.

Example:

{
  "stage": "16_trajectory_analysis",
  "trajectory_representation": "PCA_stage13b_clustering",
  "visualization_representation": "UMAP_stage13b_uncorrected",
  "trajectory_method": "Slingshot",
  "candidate_compartments": [
    "monocyte_macrophage",
    "neutrophil_granulocytic"
  ],
  "accepted_directed_trajectories": [
    "neutrophil_sgCtrl"
  ],
  "undirected_structures": [
    "macrophage_sgCtrl",
    "macrophage_sgEzh2",
    "neutrophil_sgEzh2"
  ],
  "genome_wide_model": "tradeSeq",
  "inference": "descriptive_only",
  "reason": "one pooled sequencing library per biological condition"
}

For replicated projects, metadata may additionally contain:

* biological sample identifier;
* number of samples per condition;
* trajectory-comparison design;
* sample-level inferential model;
* replicated trajectory-difference method.

---

# Recommended output structure

results/
└── 16_trajectory_analysis/
    ├── 16a_eligibility_topology/
    │   ├── objects/
    │   ├── plots/
    │   ├── tables/
    │   ├── metadata/
    │   └── stage16a_summary.md
    │
    ├── 16b_connectivity_sensitivity/
    │   ├── plots/
    │   ├── tables/
    │   ├── metadata/
    │   └── stage16b_summary.md
    │
    ├── 16c_biological_orientation/
    │   ├── plots/
    │   ├── tables/
    │   └── metadata/
    │
    ├── 16d_rooted_pseudotime/
    │   ├── objects/
    │   ├── plots/
    │   ├── tables/
    │   └── metadata/
    │
    └── 16e_trajectory_characterization/
        ├── figures/
        ├── tables/
        ├── objects/
        ├── logs/
        └── stage16e_metadata.json

---

# Recommended core outputs

## Stage 16A

trajectory_candidate_manifest.tsv
trajectory_cluster_manifest.tsv
slingshot_lineage_manifest.tsv
trajectory_topology_method_agreement.tsv

Per-compartment outputs may include:

*_centroid_mst_edges.tsv
*_slingshot_lineages.tsv
*_tscan_mst_edges.tsv
*_cluster_sample_annotation_summary.tsv

## Stage 16B

stage16b_all_cluster_pair_connectivity.tsv
stage16b_candidate_edge_connectivity.tsv
stage16b_sample_specific_centroid_mst_edges.tsv
stage16b_sample_specific_slingshot_edges.tsv
stage16b_sample_topology_sensitivity_summary.tsv

## Stage 16C

stage16c_trajectory_orientation_review.tsv
stage16c_trajectory_decision_manifest.tsv
stage16c_accepted_directed_trajectories.tsv
stage16c_undirected_state_structures.tsv
stage16c_completion_summary.tsv

## Stage 16D

stage16d_cell_pseudotime.tsv
stage16d_cluster_pseudotime_summary.tsv
stage16d_cluster_lineage_weight_summary.tsv
stage16d_rooted_slingshot_edges.tsv
stage16d_rooted_slingshot_lineage_manifest.tsv
stage16d_expected_vs_observed_topology.tsv
stage16d_shared_trunk_concordance.tsv
stage16d_root_orientation_sanity.tsv
stage16d_terminal_orientation_sanity.tsv

## Stage 16E

stage16e_program_coverage.tsv
stage16e_program_pseudotime_bins.tsv
stage16e_marker_pseudotime_bins.tsv
stage16e_marker_manifest.tsv
stage16e_tradeseq_gene_filter.tsv
stage16e_tradeseq_convergence.tsv
stage16e_tradeseq_association_annotated.tsv
stage16e_tradeseq_start_vs_end_annotated.tsv
stage16e_tradeseq_diff_end_annotated.tsv
stage16e_top_association_genes.tsv
stage16e_top_start_vs_end_genes.tsv
stage16e_top_terminal_branch_genes.tsv
stage16e_completion_summary.tsv

---

# Configuration recommendations

Project-specific decisions should be stored in configuration rather than
hard-coded wherever practical.

Example:

stage16:
  representation:
    trajectory: PCA_stage13b_clustering
    visualization: UMAP_stage13b_uncorrected

  candidate_compartments:
    - monocyte_macrophage
    - neutrophil_granulocytic

  topology:
    method: slingshot
    compare_centroid_mst: true
    compare_tscan: true

  connectivity:
    knn_k: configurable
    calculate_mnn: true

  sample_sensitivity:
    enabled: true
    min_clusters_for_slingshot: configurable

  orientation:
    use_final_annotations: true
    use_frozen_programs: true
    allow_automatic_rooting: false

  rooted_pseudotime:
    terminal_clusters_forced: false

  characterization:
    normalized_assay: logcounts_stage13a
    count_assay: soupx_counts
    pseudotime_bins: configurable

  tradeseq:
    enabled: true
    knots: configurable
    evaluate_k: optional
    min_detected_cells: configurable
    min_detection_fraction: configurable

Exact thresholds are project-level decisions and should not be treated as
universal defaults.

---

# Architectural recommendation

Stage 16 should separate four software layers.

## 1. Eligibility and geometry layer

Responsible for:

* candidate-compartment selection;
* loading canonical PCA;
* cluster-centroid topology;
* unrooted trajectory fitting;
* topology comparison;
* sample/condition support summaries.

## 2. Connectivity and sensitivity layer

Responsible for:

* kNN connectivity;
* MNN support;
* geometric-bridge diagnostics;
* sample-specific topology;
* topology-agreement metrics.

## 3. Biological-orientation layer

Responsible for:

* loading frozen annotations;
* loading frozen biological programs;
* marker/program summaries;
* root review;
* directed-versus-undirected decision;
* trajectory specification.

## 4. Characterization layer

Responsible for:

* rooted trajectory reconstruction;
* pseudotime extraction;
* lineage weights;
* topology sanity;
* program trends;
* marker trends;
* sparse-marker QC;
* genome-wide trajectory modeling;
* final completion metadata.

Biological interpretation should remain outside generic topology and GAM
functions.

---

# Reusable versus project-specific logic

The reusable pipeline should implement:

* topology inference;
* cluster-edge extraction;
* connectivity metrics;
* sample-specific sensitivity;
* root/terminal manifests;
* pseudotime summaries;
* lineage-weight summaries;
* program-scoring machinery;
* GAM fitting;
* result validation;
* output schemas.

Project-specific configuration should define:

* candidate compartments;
* biologically plausible roots;
* marker panels;
* biological programs;
* state interpretations;
* specific hypotheses;
* communication to collaborators.

Do not hard-code project-specific biological labels into reusable trajectory
functions.

---

# Recommended analysis hierarchy for agents

When implementing or extending Stage 16, use this decision sequence:

1. Read experimental design
        |
        v
2. Identify independent biological unit
        |
        v
3. Load canonical uncorrected clustering representation
        |
        v
4. Select biologically plausible candidate compartments
        |
        v
5. Summarize cluster and sample support
        |
        v
6. Infer unrooted candidate topology
        |
        v
7. Compare topology constructions
        |
        v
8. Determine whether method agreement is independent
        |
        v
9. Quantify cell-level edge connectivity
        |
        v
10. Reconstruct topology within samples where possible
        |
        v
11. Identify composition-sensitive/geometric bridges
        |
        v
12. Decide whether the structure is:
        directed candidate /
        undirected manifold /
        insufficient
        |
        v
13. Review biological orientation using frozen annotations
    and predefined programs
        |
        v
14. Assign biological root only when justified
        |
        v
15. Refit accepted rooted trajectory
        |
        v
16. Validate expected topology and lineage structure
        |
        v
17. Inspect pseudotime ordering and lineage weights
        |
        v
18. Evaluate shared-trunk concordance and projection anomalies
        |
        v
19. Characterize predefined biological programs
        |
        v
20. Validate selected marker trajectories
        |
        v
21. Perform sparse-marker detection QC
        |
        v
22. Fit genome-wide trajectory model if warranted
        |
        v
23. Validate model convergence and output schema
        |
        v
24. Interpret association, start/end, and branch tests
        descriptively
        |
        v
25. Integrate topology, programs, markers, and
    genome-wide results
        |
        v
26. Write completion summary and metadata

A failed directionality decision should stop rooted pseudotime analysis for that
candidate without being treated as pipeline failure.

---

# Hard rules for pipeline-development agents

1. Do not run trajectory analysis automatically on every compartment.
2. Treat trajectory eligibility as a biological and geometric decision.
3. Use the canonical uncorrected high-dimensional representation for primary
   trajectory inference.
4. Do not infer trajectory directly from UMAP geometry.
5. Do not substitute Harmony or another corrected representation without
   explicit justification.
6. Fit initial topology without forcing a biological root.
7. Do not treat method agreement as independent validation when methods share
   the same underlying graph logic.
8. Evaluate cluster-level topology with cell-level connectivity.
9. Flag unusually long or weakly connected topology edges as possible geometric
   bridges.
10. Perform sample-restricted topology sensitivity when multiple samples are
    available.
11. Do not interpret sample-restricted topology as biological replication.
12. Explicitly record condition-exclusive and severely imbalanced states.
13. Do not interpret a cross-condition bridge as a biological transition merely
    because a trajectory algorithm connects it.
14. Use frozen upstream annotations for interpretation rather than circular
    topology construction.
15. Reuse frozen upstream biological programs whenever possible.
16. Do not collapse orthogonal biological programs into one arbitrary
    early/late score without justification.
17. Choose roots using explicit biological evidence.
18. Do not root based solely on cluster number, UMAP endpoint, principal
    component sign, or experimental condition.
19. Do not force terminal states unless scientifically justified.
20. Permit the valid outcome that a compartment is an undirected state
    manifold.
21. Rooted topology must be validated against the accepted trajectory
    specification.
22. Inspect lineage weights, not pseudotime alone.
23. For branching models, verify shared-trunk pseudotime concordance.
24. Expect reduced lineage concordance near a true branch region.
25. Do not delete cells solely because of anomalous pseudotime projections.
26. Evaluate trajectory anomalies against independent QC and biological
    evidence.
27. Use canonical normalized expression for descriptive marker/program trends.
28. Record program-gene coverage.
29. Treat sparse genes cautiously and inspect detection fractions.
30. Label trajectory-motivated exploratory programs as post hoc.
31. Use raw or corrected raw counts for count-based genome-wide trajectory
    models.
32. Resolve stable feature IDs and gene symbols explicitly.
33. Document duplicate-symbol handling.
34. Filter low-information genes using predefined criteria.
35. Do not refit or modify the accepted topology silently during gene-level
    modeling.
36. Preserve canonical lineage weights.
37. When a modeling package requires complete pseudotime coordinates, use the
    trajectory method's supported complete representation rather than manual
    imputation.
38. Record GAM input QC and model convergence.
39. Validate package-specific test output schemas before downstream use.
40. Do not interpret numerical p-values of zero as infinite evidence.
41. Rank trajectory genes using convergence, Wald magnitude, valid tests, and
    expression prevalence rather than p-value alone.
42. Verify terminal-branch log-fold-change sign against actual endpoint
    expression before final interpretation.
43. Distinguish genes associated with pseudotime from genes differing between
    biological conditions.
44. Never infer biological replication from cell number.
45. Do not claim that a treatment moves cells along pseudotime without
    replicated evidence.
46. Pseudotime is a transcriptional state coordinate, not lineage tracing.
47. Genome-wide trajectory statistics should support characterization, not
    retroactively define the trajectory.
48. Biological interpretation should integrate topology, programs, markers,
    and genome-wide evidence.
49. Project-specific biological conclusions belong in project documentation,
    not generic pipeline code.
50. A conservative decision not to assign pseudotime is preferable to forcing
    an unsupported trajectory.