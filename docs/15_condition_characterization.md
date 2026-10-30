# Stage 15: Condition-associated characterization
## Stage structure
Stage 15 characterizes observed differences between biological conditions after
QC, clustering, clustering-sensitivity assessment, and final biological
annotation have been completed.
The stage combines four complementary operations:
### Stage 15A — Composition and state redistribution
Summarizes recovered cell abundance and relative composition across biological
annotation levels.
### Stage 15B — Expression contrasts and pathway enrichment
Generates population-specific descriptive expression contrasts from aggregated
uncorrected counts and performs preranked competitive gene-set enrichment where
appropriate.
### Stage 15C — Targeted biological programs and signaling-source characterization
Scores predefined biological programs and summarizes candidate cytokine,
chemokine, or other signaling-source genes across annotated populations.
### Stage 15D — Validation and integrated summary
Validates Stage 15 component outputs and generates a machine-readable and
human-readable summary of the completed condition-characterization stage.

## Scope note for reuse

Stage 15 guidance combines:

* general statistical principles that should hold across projects;
* configurable analysis choices;
* project-specific examples from the original non-replicated pooled design.

Named labels such as sgCtrl and sgEzh2 are illustrative case-study identifiers,
not required condition names or a required two-condition structure.

For new datasets, document which Stage 15 decisions are retained, overridden,
or replaced based on experimental design and biological replication.
---
# Core purpose
Once the cellular atlas and biological annotation framework are finalized, the
principal question changes from:
> What populations and states are present?
to:
> How does the observed cellular system differ between conditions?
Condition-associated differences can appear through at least two related
mechanisms:
1. **within-population transcriptional differences**;
2. **changes in the relative abundance of biological populations or states**.
These should be analyzed jointly.
A sufficiently strong transcriptional response may cause related cells to
separate into different states or clusters, making the biological difference
appear primarily as composition or differential abundance.
With coarser clustering, the same response may remain within a shared
population and appear primarily as differential expression.
Therefore Stage 15 should not treat expression and abundance as independent or
mutually exclusive analyses.
Conceptually:

final annotation
      |
      +----------------------------+
      |                            |
      v                            v
 composition                 expression
      |                            |
      v                            v
state redistribution        within-state contrast
      |                            |
      +--------------+-------------+
                     |
                     v
             pathway/program context
                     |
                     v
              source-of-signal
                     |
                     v
             biological synthesis

⸻

# Experimental-design contract

## Independent experimental unit

The pipeline must identify the independent biological/experimental unit before
performing any cross-condition analysis.

For the current project:

sgCtrl:
4 tumors pooled before library preparation
        |
        v
one 10x library
sgEzh2:
4 tumors pooled before library preparation
        |
        v
one 10x library

Mouse identities are not recoverable after pooling.

Therefore:

effective inferential sample size
=
1 library per condition

Cells within a library are observations from the same experimental unit and
must not be treated as independent biological replicates.

Increasing the number of cells improves measurement of that library.

It does not provide additional biological replication.

more cells
    ->
more precise characterization of one sample
more independently processed biological samples
    ->
estimation of biological variability
    ->
replicated treatment inference

This distinction is a hard statistical boundary for Stage 15.

⸻

# General statistical rule

The pipeline must never infer biological replication from the number of cells.

Do not manufacture replication by:

* treating cells as independent treatment replicates;
* randomly dividing one library into pseudo-replicates;
* treating clusters as independent biological replicates;
* treating pooled animals as individual replicates when their identities are
    not recoverable;
* using cell-level Wilcoxon, t-tests, generalized linear models, or similar
    procedures and interpreting the resulting p-values as treatment-level
    evidence.

Artificial pseudo-replication produces anti-conservative uncertainty estimates
and apparent precision unsupported by the experimental design.

⸻

# Replicated-design alternative

In a normally replicated multi-condition experiment, the preferred expression
workflow is sample-level pseudobulk analysis:

cells
  |
  v
biological sample x population
  |
  v
sum raw counts
  |
  v
sample-level pseudobulk profiles
  |
  v
edgeR / DESeq2 / limma-voom
  |
  v
replicated treatment inference

For example:

Ctrl_1
Ctrl_2
Ctrl_3
vs
KO_1
KO_2
KO_3

would permit population-specific estimation of between-sample biological
variability.

## Current-project behavior

The current project instead contains:

sgCtrl_1
vs
sgEzh2_1

Therefore Stage 15 uses pseudobulk aggregation for effect description and gene
ranking, not replicated hypothesis testing.

⸻

# Measurement layers

Stage 15 must explicitly distinguish measurement substrates.

## Raw / corrected count layer

Use an uncorrected biological count assay such as:

soupx_counts

or the equivalent corrected raw-count assay.

Use for:

* pseudobulk aggregation;
* raw UMI detection;
* descriptive CPM/logCPM;
* gene-ranking construction;
* source-gene detection fractions.

## Normalized expression layer

Use canonical uncorrected normalized expression from Stage 13A:

logcounts_stage13a

Use for:

* visualization;
* per-cell program scoring;
* mean normalized expression;
* cluster/state expression summaries.

## Integration-corrected representation

Harmony or other integration-derived representations may be used for:

* alignment sensitivity;
* cluster-stability assessment;
* annotation support;
* visualization when clearly labeled.

They must not provide reconstructed expression values for Stage 15
condition-associated gene-expression analysis.

The general principle is:

integration
    ->
common geometry / annotation support
uncorrected measurements
    ->
biological condition characterization

This is especially important when sample/library and biological condition are
confounded.

⸻

# Canonical annotation source

Stage 15 must consume finalized biological annotations generated upstream.

For the current pipeline:

Stage 13B canonical uncorrected clusters
        +
Stage 13C Harmony sensitivity
        |
        v
Stage 14 reviewed annotations
        |
        v
Stage 14B propagated global annotation atlas
        |
        v
Stage 15

Downstream code should use stable annotation fields rather than re-deriving
biological identities from numeric cluster identifiers.

Recommended annotation levels include:

* broad compartment;
* Stage 13B canonical cluster;
* cell class;
* cell type;
* cell subtype;
* cell state;
* detailed final label;
* simplified plot label.

⸻

# Comparison classification

Every requested cross-condition population comparison must be classified before
expression or pathway analysis.

Recommended machine-readable field:

comparison_class

or:

contrast_class

Allowed values:

shared
severely_imbalanced
condition_exclusive
insufficient_cells

## Shared

Both conditions contain adequate cellular support.

Example:

sgCtrl >= minimum cells
sgEzh2 >= minimum cells

Supports:

* descriptive pseudobulk expression contrast;
* mean normalized-expression comparison;
* detection-fraction comparison;
* descriptive gene ranking;
* preranked GSEA;
* curated-program comparison.

## Severely imbalanced

Both conditions are represented, but cell support differs strongly.

Example:

sgCtrl = 60
sgEzh2 = 668

Such contrasts may reflect:

* within-state expression differences;
* state-selection effects;
* substate redistribution;
* instability from the smaller group.

Expression summaries may be retained but require explicit qualification.

## Condition-exclusive

One condition contains zero cells.

Example:

sgCtrl = 0
sgEzh2 = 500

There is no meaningful within-state condition contrast.

Do not manufacture differential expression.

Characterize instead using:

* state abundance;
* cluster markers;
* program activity;
* average expression;
* lineage/state identity;
* neighboring transcriptional states.

## Insufficient cells

At least one condition contains too few cells for reliable characterization.

Record this explicitly rather than forcing an analysis.

⸻

# Default comparison classifier

A reusable implementation may take the form:

classify_contrast <- function(
    ctrl_cells,
    ezh2_cells,
    min_cells,
    severe_ratio
) {
    if (ctrl_cells == 0 || ezh2_cells == 0) {
        return("condition_exclusive")
    }
    if (min(ctrl_cells, ezh2_cells) < min_cells) {
        return("insufficient_cells")
    }
    ratio <- max(ctrl_cells, ezh2_cells) /
        min(ctrl_cells, ezh2_cells)
    if (ratio >= severe_ratio) {
        return("severely_imbalanced")
    }
    "shared"
}

Thresholds must be configurable rather than embedded as biological truths.

Recommended configuration variables:

minimum_cells_per_condition
severe_imbalance_ratio

The classifier controls interpretation and analysis eligibility, not biological
identity.

⸻

# Stage 15A: Composition and state redistribution

## Purpose

Describe differences in recovered cellular composition between conditions.

This is not formal differential-abundance inference unless appropriate
biological replication exists.

## Required summaries

For each annotation level calculate, where meaningful:

population
condition
n_cells
fraction_of_sample
fraction_of_compartment

Cross-condition summaries may additionally contain:

sgCtrl_cells
sgEzh2_cells
sgCtrl_fraction
sgEzh2_fraction
delta_fraction
descriptive_ratio
descriptive_log2_ratio
comparison_class

Ratios require careful handling when one condition contains zero cells.

Avoid arbitrary pseudocounts unless explicitly documented.

⸻

# Compositional interpretation

Single-cell cell frequencies are compositional.

For any chosen denominator:

p1 + p2 + ... + pk = 1

Increasing one population necessarily alters the observed fraction of at least
one other population.

Therefore:

lower observed fraction
!=
proven absolute depletion

The appropriate interpretation is:

The observed relative composition differed between the recovered libraries.

not:

The treatment caused an absolute loss of this population.

Absolute abundance requires additional experimental information such as:

* tissue-level cell counts;
* flow-cytometric counts;
* counting beads;
* quantitative histology;
* another validated absolute-reference method.

⸻

# Within-compartment composition

When the question concerns redistribution among biologically related states,
within-compartment fractions are often preferable.

Example:

all macrophages
      |
      +-- recruited/inflammatory
      +-- resident-like
      +-- TAM/remodeling
      +-- antigen-presentation

This answers:

How is the recovered macrophage compartment internally distributed?

rather than allowing unrelated changes in epithelial, stromal, or lymphoid
abundance to dominate the denominator.

Within-compartment fractions remain compositional.

They simply use a more biologically relevant denominator.

⸻

# Differential-abundance alternatives

With adequate sample-level replication, possible methods include:

* edgeR models of cluster counts;
* scCODA;
* tascCODA;
* Milo;
* other compositional or neighborhood-based DA methods.

These methods should only be activated when the experimental design provides
sufficient independent sample-level information.

## scCODA

Useful when:

* compositions are the primary endpoint;
* multiple replicated samples are available;
* joint modeling of the sum-to-one constraint is desired.

Do not interpret scCODA effects from a one-sample-per-condition design as
replicated treatment effects.

## Milo

Useful when:

* changes may occur along transcriptional continua;
* cluster boundaries are not trusted as the only biological units;
* replicated samples are available.

Milo analyzes overlapping local neighborhoods rather than requiring fixed
cluster labels.

Artificially partitioning a library into pseudo-samples does not supply the
replication required for valid treatment inference.

## Current-project decision

Use Stage 14 populations/states together with descriptive abundance summaries.

Do not perform inferential scCODA or Milo testing.

⸻

# Stage 15B: Population-specific expression contrasts

## Biological question

For a population represented in both conditions:

What expression differences are observed between the two library-level
population profiles?

Expression analysis should be performed within biologically comparable
populations.

Examples:

macrophage sgCtrl vs sgEzh2
neutrophil sgCtrl vs sgEzh2
fibroblast sgCtrl vs sgEzh2
epithelial/tumor sgCtrl vs sgEzh2

Finer states may also be compared when adequately represented.

⸻

# Pseudobulk aggregation

For each population and condition:

cells
  |
  v
sum uncorrected raw counts
  |
  v
one pseudobulk profile

For the current project:

population x sgCtrl -> one profile
population x sgEzh2 -> one profile

These profiles permit descriptive effect estimation but not estimation of
within-condition biological variance.

⸻

# Pseudobulk normalization

For a replicated design, a typical edgeR workflow would be:

raw pseudobulk counts
        |
        v
filterByExpr()
        |
        v
calcNormFactors()
        |
        v
dispersion estimation
        |
        v
QL / GLM model

calcNormFactors() performs bulk-style library-composition normalization such as
TMM.

This serves a different purpose from cell-level scran normalization.

## Current-project workflow

Recommended descriptive workflow:

SoupX-adjusted raw counts
        |
        v
aggregate by population x condition
        |
        v
population-specific expression filtering
        |
        v
TMM normalization
        |
        v
log2 CPM
        |
        v
sgEzh2 - sgCtrl observed contrast

The resulting quantity is a descriptive expression difference.

It must not be labeled as a replicated statistical DE effect.

⸻

# Gene filtering

Low-information genes should be filtered before constructing population-level
rankings.

Reasons include:

* unstable fold changes from very low counts;
* zero-driven extreme values;
* unnecessary noise;
* poor gene-set ranking behavior;
* unnecessary multiple-testing burden in replicated extensions.

Filtering should be performed within the relevant population/contrast.

Do not derive one global gene filter solely from unrelated lineages.

## Replicated-design alternative

For conventional replicated pseudobulk DE:

edgeR::filterByExpr()

is appropriate when supplied with the relevant sample design.

## Non-replicated descriptive alternative

For descriptive contrasts, use a documented count/CPM-based expression rule.

Possible criteria include:

* minimum CPM in at least one condition;
* minimum pseudobulk count;
* minimum cellular detection fraction;
* a combination of count and detection criteria.

The exact rule should be configurable and recorded in the contrast manifest.

⸻

# Expression output contract

Recommended per-gene fields include:

compartment
annotation_level
population
gene
comparison_class
sgCtrl_cells
sgEzh2_cells
sgCtrl_pseudobulk_count
sgEzh2_pseudobulk_count
sgCtrl_logCPM
sgEzh2_logCPM
delta_logCPM
gene_passes_filter

Optional descriptive cell-level summaries:

fraction_detected_sgCtrl
fraction_detected_sgEzh2
mean_log_expression_sgCtrl
mean_log_expression_sgEzh2

These quantities provide complementary biological context.

⸻

# Terminology

Preferred:

* observed expression difference;
* sgEzh2-associated expression contrast;
* higher expression in the observed sgEzh2 profile;
* descriptive pseudobulk contrast.

Avoid without replication:

* significantly differentially expressed;
* treatment-induced expression change;
* EZH2-loss-dependent DE;
* statistically increased by knockout.

⸻

# Stage 15B: Gene-set enrichment

## Two distinct questions

### Gene-set enrichment

Question:

Are genes belonging to a biological pathway preferentially shifted in the
condition-associated gene ranking?

Examples:

* GSEA;
* fgsea;
* camera;
* Fisher/hypergeometric enrichment.

### Gene-set activity

Question:

How strongly is a predefined program represented in an individual cell or
population?

Examples:

* simple mean-expression scores;
* AUCell;
* VISION;
* UCell;
* GSVA-like approaches where technically appropriate.

These analyses are complementary but must not be conflated.

⸻

# Competitive versus self-contained testing

## Competitive enrichment

Competitive methods ask whether genes in a set rank differently from genes
outside the set.

Examples:

* preranked GSEA;
* fgsea;
* camera;
* overrepresentation analysis.

A supplied descriptive gene ranking can therefore be explored with preranked
GSEA even when replicated sample-level DE inference is unavailable.

Important:

significant enrichment under the gene-set ranking null
!=
replicated treatment significance across biological samples

The enrichment statistic describes the supplied ranking.

It does not restore missing biological replication.

## Self-contained methods

Self-contained methods ask whether the genes in a set collectively change
between biological conditions.

Examples include:

* fry;
* roast.

Their intended inferential advantages depend on replicated sample-level
information.

They should not be used as replicated treatment tests in a one-library-per-
condition experiment.

⸻

# Current-project enrichment strategy

For eligible shared populations:

uncorrected raw counts
        |
        v
condition pseudobulk
        |
        v
population-specific gene filtering
        |
        v
TMM / logCPM
        |
        v
signed descriptive gene ranking
        |
        v
preranked fgsea

The ranking statistic is currently based on the observed condition difference in
normalized pseudobulk expression.

Any alternative ranking statistic must be explicitly documented.

⸻

# Ranked-list ties

Two-profile descriptive comparisons can produce substantial numbers of tied
ranking values.

This occurs particularly when:

* genes have zero or near-zero counts;
* the ranking statistic is discretized;
* both aggregated profiles contain similar sparse expression.

The pipeline should record ranked-list QC such as:

genes_ranked
fraction_tied
unique_rank_values
positive_genes
negative_genes

fgsea may report arbitrary ordering within tied groups.

This does not automatically invalidate descriptive enrichment but reduces the
precision with which exact pathway rank or NES should be interpreted.

Undefined enrichment statistics should remain missing rather than being forced
to artificial values.

⸻

# Gene-set collections

Reference/database-driven collections may include:

* MSigDB Hallmark;
* Reactome;
* GO Biological Process.

These provide relatively broad biological discovery.

Project-specific curated programs may include:

* type-I IFN response;
* innate viral sensing;
* antigen presentation;
* macrophage inflammatory/recruited state;
* resident-like macrophage state;
* TAM/remodeling state;
* neutrophil recruitment;
* inflammatory neutrophil state;
* IFN-responsive neutrophil state;
* TAN/suppressive state;
* immature/granule-biogenesis state.

The pipeline must preserve provenance distinguishing:

reference / data-driven

from:

project-specific / hypothesis-motivated

Hypothesis-motivated sets are valuable but should not be presented as
independent pathway discovery.

⸻

# Gene-set coverage

Every tested or scored program should record feature coverage.

Recommended fields:

compartment
program
requested_genes
genes_present
coverage_fraction
genes_present_list
coverage_warning

Poorly represented sets should be flagged rather than silently interpreted.

A useful configurable rule for enrichment may include:

minimum overlap approximately 10-15 genes
maximum tested set size approximately 500 genes

These are implementation defaults, not universal biological constants.

Project-specific small programs may intentionally contain fewer genes but should
be interpreted according to their observed coverage.

⸻

#Gene-set redundancy

Reference databases contain overlapping pathways.

For example, multiple interferon-related pathways may share substantial leading
edges.

Therefore:

many enriched pathway names
!=
many independent biological phenomena

Interpretation should collapse related results into higher-level biological
themes.

Recommended workflow:

individual enriched pathways
        |
        v
overlap / leading-edge inspection
        |
        v
redundant pathway families
        |
        v
major biological themes

Leading-edge genes should be retained because they identify the genes actually
driving enrichment.

⸻

# Stage 15B targeted QC of unexpected pathway signals

Unexpected pathway-level results should trigger targeted QC rather than
automatic relabeling or cell removal.

Example pattern:

unexpected pathway
        |
        v
identify leading-edge genes
        |
        v
evaluate cell-level expression/detection
        |
        v
check lineage-marker support
        |
        v
check doublet / QC metrics
        |
        v
localize signal to clusters/states
        |
        v
perform sensitivity analysis

## Current-project neutrophil EMT/ECM example

Strong sgEzh2 Hallmark EMT enrichment was observed in the
neutrophil/granulocytic compartment.

Because the leading edge contained canonical ECM/stromal genes, targeted QC
examined:

* neutrophil lineage-marker support;
* stromal-marker detection;
* stromal-marker co-detection;
* Stage 08 doublet and high-library flags;
* library complexity;
* Stage 13B cluster localization;
* enrichment after removing cells with elevated stromal-marker co-detection.

The enrichment persisted after exclusion of cells expressing multiple stromal
markers.

Therefore the pipeline retained the neutrophil annotations and interpreted the
result as a distributed ECM/mesenchymal-like transcriptional feature rather than
automatic evidence of fibroblast contamination or literal EMT.

### General rule

Unexpected biological signals should not automatically trigger:

* cell deletion;
* cluster reassignment;
* lineage relabeling.

They should trigger a focused validation workflow.

⸻

# Stage 15C: Curated program scoring

## Purpose

Localize hypothesis-motivated biological programs across cells and finalized
states.

Program scoring answers:

Where is the biological program expressed?

It complements GSEA, which answers:

Is the program represented among genes shifted in the condition contrast?

⸻

# Program-score implementation

For each program and cell:

program score
=
mean Stage 13A normalized expression
of program genes present in the relevant feature set

Alternative scoring methods may be substituted if justified, including:

* UCell;
* AUCell;
* rank-based approaches;
* background-adjusted module scoring.

The scoring method must be recorded because different methods have different
scale and robustness properties.

The current simple mean-expression method is transparent, deterministic, and
adequate for targeted descriptive summaries.

⸻

# Program-score aggregation

Program scores should be summarized independently across annotation levels.

Recommended schema:

compartment
annotation_level
population
program
sample_id
cells
mean_program_score
median_program_score

Do not encode multiple biological annotation dimensions simultaneously as
grouping variables unless there is a clear analytical reason.

Instead use:

annotation_level
population

This creates a normalized table that is easy to query across:

* compartment;
* Stage 13B cluster;
* cell type;
* cell subtype;
* plot label;
* final label;
* cell state.

⸻

# Program-score interpretation

Program scores are descriptive measurements.

They are not treatment-level statistical tests.

Cell-level score distributions must not be used to infer biological replication
from the number of cells.

Program scores are best used to:

* localize biological programs;
* characterize states;
* support pathway interpretation;
* compare descriptive means/medians between sufficiently represented
    populations;
* generate mechanistic hypotheses.

⸻

# Stage 15C: Candidate source-of-signal analysis

## Purpose

Once pathway and state changes are identified, ask:

Which annotated populations plausibly express the candidate signaling
molecules associated with the observed phenotype?

Possible candidate genes include:

Cxcl1
Cxcl2
Cxcl5
Csf3
Il1b
Il6
Tnf
Ccl2
Ccl5
Cxcl9
Cxcl10
Cxcl11
Xcl1
Xcl2

The exact panel should remain configurable and project-specific.

⸻

## Source-signal measurements

For each candidate gene and population summarize:

mean normalized expression
fraction detected
mean raw count per cell

These measurements answer different questions.

### Mean normalized expression

Reflects average normalized expression across the population.

### Fraction detected

Reflects the proportion of cells with at least one nonzero corrected raw count.

This can be more interpretable than mean expression for sparse cytokines and
chemokines.

### Mean raw count per cell

Provides an additional count-scale summary and can help distinguish weak
widespread expression from a small number of strongly expressing cells.

⸻

# Source-signal table contract

Recommended schema:

compartment
annotation_level
population
gene
sample_id
cells
mean_log_expression
fraction_detected
mean_raw_count_per_cell
gene_present

Annotation levels should be summarized independently using the same
annotation_level / population convention used for program scores.

⸻

# Gene-absent sentinel rows

If a requested signal gene is absent from the compartment feature set, the
pipeline may emit one explicit sentinel record:

gene_present = FALSE
sample_id = ""
cells = 0
mean_log_expression = NA
fraction_detected = NA
mean_raw_count_per_cell = NA

This has two advantages:

1. absence is represented explicitly rather than silently dropping the gene;
2. downstream validation can distinguish unavailable features from biological
    zero expression.

Validation code must treat blank sample_id values as legitimate only for
gene_present = FALSE sentinel rows.

For example:

valid_sample <-
    signals$sample_id %in% expected_samples
valid_sentinel <-
    !signals$gene_present &
    signals$sample_id == "" &
    signals$cells == 0
if (any(!(valid_sample | valid_sentinel))) {
    stop("Invalid Stage 15C source-signal rows.")
}

Sentinel rows are metadata records, not biological observations.

⸻

# Program and source-signal condition comparisons

For populations represented in both conditions, Stage 15C may create paired
descriptive summaries containing:

cells_sgCtrl
cells_sgEzh2
mean_sgCtrl
mean_sgEzh2
delta_mean
fraction_detected_sgCtrl
fraction_detected_sgEzh2
delta_fraction_detected
comparison_class

Only comparisons passing the intended cell-support rules should be promoted as
clean shared-population contrasts.

Severely imbalanced populations should remain flagged even if effect summaries
are calculated.

⸻

# Population hierarchy and compositional confounding

Broad population contrasts can combine:

within-state transcriptional differences
+
changes in substate composition

For example, a broad macrophage compartment may differ because:

* the same macrophage states changed transcriptionally;
* different macrophage substates are represented in different proportions;
* both occurred.

Therefore pipeline-generated summaries should retain enough annotation
resolution to compare:

compartment
vs
cell state
vs
fine final label

A finer shared biological state may provide a cleaner within-state
characterization than a broad compartment.

Conversely, very fine labels can become unstable or strongly condition-specific.

The pipeline should not assume that the finest annotation is always the most
informative unit.

⸻
# Project-specific macrophage framework

Useful macrophage biological axes include:

recruited / inflammatory
        |
        v
resident-like
        |
        v
TAM / remodeling
        |
        v
MHC-II / antigen presentation

Representative curated markers may include:

## Recruited/inflammatory

Ccr2
Ly6c2
S100a8
S100a9
Sell
Vcan

## Resident-like

Folr2
C1qa
C1qb
C1qc
Pf4
Stab1
Maf
Mertk

## TAM/remodeling

Trem2
Spp1
Apoe
Gpnmb
Lpl
Arg1
Chil3
Lgals3

## Antigen presentation

Cd74
H2-Aa
H2-Ab1
H2-Eb1
B2m
Tap1
Tap2

These programs should characterize observed states rather than force mutually
exclusive macrophage identities.

⸻

## Project-specific neutrophil framework

Do not require every neutrophil state to map to a published named subtype.

Prefer:

data-driven cluster identity
+
curated biological programs

Relevant biological programs include:

* classical/recruited;
* inflammatory;
* IFN-responsive;
* suppressive/TAN-like;
* immature/granule-biogenesis;
* proliferative;
* project-specific post hoc programs when justified by data.

Published signatures provide context but should not override contradictory
data-driven evidence.

Terms such as PMN-MDSC should not be assigned from transcriptomic signatures
alone unless the project has additional functional evidence.

⸻
## Mechanistic-hypothesis boundary

A project may contain a mechanistic model such as:

EZH2 loss
   |
   v
TE / ERV derepression
   |
   v
innate sensing / TBK1 signaling
   |
   v
IRF / interferon response
   |
   v
antigen presentation / chemokines
   |
   v
immune recruitment / myeloid remodeling

Stage 15 may determine whether observed single-cell transcriptional data are
consistent with individual components of such a model.

It must not treat the proposed causal chain as established.

Separate:

REFERENCE / DATA-DRIVEN
Hallmark
Reactome
GO BP
cluster markers
unbiased expression rankings

from:

HYPOTHESIS-MOTIVATED
viral mimicry
innate sensing
TBK1-related programs
type-I IFN
specific chemokines
project-specific myeloid programs

This distinction should be preserved in output metadata and reporting.

Direct TE/ERV activity requires repeat-aware quantification and is not inferred
from ordinary gene-level scRNA-seq expression alone.

⸻

# Interpretation of sparse signaling genes

Genes such as cytokines may be sparsely detected.

The pipeline should avoid interpreting small changes in extremely sparse genes
without context.

Useful supporting evidence includes:

* detection fraction;
* normalized mean expression;
* raw count distribution;
* program-level behavior;
* related downstream response genes;
* consistency across related populations.

For example, an interferon-response program may be strong while Ifnb1 itself
is sparsely detected.

This supports:

an IFN-responsive or antiviral transcriptional state

but not necessarily:

abundant IFN-beta production by the measured cells.

⸻

# Stage 15D: Integrated validation and summary

## Purpose

Verify that all required Stage 15 components completed successfully and create
a concise technical summary.

Required upstream products typically include:

* Stage 15A composition table;
* Stage 15B contrast manifest;
* Stage 15B expression outputs;
* Stage 15B GSEA outputs;
* Stage 15C program summaries;
* Stage 15C source-signal summaries;
* Stage 15C program-coverage table.

Stage 15D should validate:

* expected files exist;
* required columns exist;
* sample identifiers are valid;
* sentinel rows satisfy the source-signal contract;
* component tables are nonempty where expected;
* generated expression/GSEA flags are internally consistent;
* program coverage is represented explicitly.

⸻

## Recommended Stage 15D outputs

stage15_summary.md
stage15_component_status.tsv
stage15_metadata.json

The component-status table may contain:

component
completed
records

Example components:

15a_composition
15b_expression
15b_gsea
15c_programs
15c_source_signals
15c_program_coverage

⸻

# Metadata contract

Recommended metadata should encode the inferential boundary explicitly.

Example:

{
  "stage": "15_condition_characterization",
  "analyses": {
    "composition": true,
    "descriptive_expression": true,
    "preranked_gsea": true,
    "curated_programs": true,
    "source_signal": true
  },
  "inference": "descriptive_only",
  "reason": "one pooled sequencing library per biological condition"
}

For a replicated project, the metadata should instead record:

* biological sample identifier;
* number of independent samples per condition;
* design formula;
* replicated DE method;
* replicated DA method where applicable.

⸻

# Acceptance criteria

Stage 15 should not be considered complete until:

* finalized Stage 14 annotations are used as input;
* all requested annotation levels are represented;
* composition tables reconcile with the upstream cell universe;
* no cells are treated as independent treatment replicates;
* no artificial pseudo-replication is introduced;
* expression comparisons use uncorrected count/expression assays;
* Harmony/integration-derived values are excluded from gene-level contrasts;
* all populations receive a comparison class;
* condition-exclusive populations do not receive faux within-state DE;
* low-expression genes are filtered according to a documented rule;
* expression-contrast eligibility is recorded;
* ranked-list QC is recorded where GSEA is performed;
* gene-set coverage is validated;
* poorly represented curated programs are flagged;
* source-signal feature absence is explicitly represented;
* Stage 15C sentinel rows pass validation;
* Stage 15D validates all expected outputs;
* the experimental-design limitation is present in metadata and human-readable
    reporting.

⸻

# Interpretation language

When biological replication is unavailable, preferred language includes:

* observed;
* condition-associated;
* higher in the observed sgEzh2 profile;
* lower in the observed sgCtrl profile;
* relative enrichment;
* state redistribution;
* pathway enriched in the observed gene ranking;
* consistent with;
* supports a hypothesis;
* hypothesis-generating.

Avoid:

* significantly increased by treatment;
* significantly differentially expressed following knockout;
* treatment caused;
* significantly depleted;
* significantly expanded;
* pathway significantly activated by EZH2 loss;
* replicated treatment effect.

A durable global caveat is:

Because tumors were pooled before library generation, each condition is
represented by one scRNA-seq library and mouse-level identities are not
recoverable. Condition-associated differences are therefore interpreted
descriptively and cannot establish population-level statistical significance,
biological reproducibility across mice, or causal effects of the perturbation.

⸻

# Alternative workflow when biological replication is available

The pipeline should be designed so that Stage 15 can generalize to replicated
experiments without replacing the entire stage.

## Expression

Current non-replicated mode:

pseudobulk
-> TMM/logCPM
-> descriptive contrast
-> preranked GSEA

Replicated mode:

sample x population pseudobulk
-> filterByExpr
-> TMM
-> edgeR / DESeq2 / limma-voom
-> replicated DE statistics
-> pathway analysis

Possible ranking statistics in replicated mode include:

* moderated t-statistic;
* likelihood-ratio statistic;
* quasi-likelihood F-derived signed statistic;
* signed log-fold change when appropriate.

## Composition

Current mode:

counts/fractions
-> descriptive differences

Replicated mode may support:

sample x population counts
-> edgeR abundance model

or:

sample compositions
-> scCODA / tascCODA

or:

sample x transcriptional neighborhoods
-> Milo

Selection should depend on the biological question rather than being hard-coded
to one method.

## Program activity

Current mode:

cell scores
-> descriptive population-condition summaries

Replicated mode could additionally aggregate program activity at the sample
level and fit:

* linear models;
* mixed models;
* pseudobulk-like sample summaries;
* other designs respecting biological replication.

Cells should still not be treated as independent treatment replicates by
default.

⸻

# Architectural recommendation

Stage 15 should separate three software layers.

## 1. Data preparation

Responsible for:

* loading finalized atlas;
* extracting annotations;
* locating raw and normalized assays;
* constructing population-condition groups;
* reconciling cell identities.

## 2. Statistical/descriptive engine

Responsible for:

* comparison classification;
* composition summaries;
* pseudobulk aggregation;
* normalization;
* gene filtering;
* expression ranking;
* pathway analysis;
* program scoring;
* source-signal summarization.

## 3. Interpretation/reporting layer

Responsible for:

* applying experimental-design caveats;
* distinguishing shared from imbalanced contrasts;
* summarizing pathway themes;
* generating metadata;
* creating completion reports.

Biological conclusions should not be hard-coded into reusable computational
functions.

⸻

# Configuration recommendations

Project-specific decisions should live in configuration rather than analysis
code whenever practical.

Useful configurable fields include:

stage15:
  sample_column: sample_id
  control_condition: sgCtrl
  comparison_condition: sgEzh2
  min_cells_per_condition: 20
  severe_imbalance_ratio: 10
  raw_count_assay: soupx_counts
  normalized_assay: logcounts_stage13a
  composition_levels:
    - compartment
    - stage13b_cluster
    - cell_type
    - cell_subtype
    - plot_label
    - final_label
    - cell_state
  gene_filter:
    method: configurable
  gsea:
    min_size: 15
    max_size: 500
  program_coverage_warning: 0.8

Exact parameter values are project decisions and should not be treated as
universal defaults.

⸻

# Output provenance

Every Stage 15 output should retain enough information to reconstruct:

* input object;
* assay used;
* annotation level;
* population identifier;
* sample/condition;
* cell count;
* comparison class;
* filtering rule;
* normalization method;
* gene-set resource/version;
* program definition/version;
* analysis eligibility;
* software version where relevant.

This is particularly important because Stage 15 combines multiple biological
resolutions and multiple types of descriptive evidence.

⸻

# Recommended directory structure

results/
└── 15_condition_characterization/
    ├── 15a_composition/
    │   ├── tables/
    │   └── figures/
    │
    ├── 15b_expression_gsea/
    │   ├── tables/
    │   │   ├── contrast_manifest/
    │   │   ├── expression/
    │   │   └── gsea/
    │   ├── figures/
    │   └── qc/
    │
    ├── 15c_programs_source_signals/
    │   ├── tables/
    │   └── figures/
    │
    └── 15d_summary/
        ├── stage15_summary.md
        ├── stage15_component_status.tsv
        └── stage15_metadata.json

Unexpected biological findings requiring targeted validation should be stored
under an explicit QC subdirectory rather than mixed with primary tables.

Example:

15b_expression_gsea/qc/neutrophil_mesenchymal/

⸻

# Recommended analysis hierarchy for agents

When implementing or extending Stage 15, use the following decision sequence:

1. Read experimental design
        |
        v
2. Identify independent biological unit
        |
        v
3. Determine whether replicated inference is supported
        |
        v
4. Load finalized Stage 14 annotation atlas
        |
        v
5. Generate descriptive composition
        |
        v
6. Classify every population contrast
        |
        v
7. Aggregate uncorrected counts
        |
        v
8. Filter and normalize population-level expression
        |
        v
9. Generate descriptive or replicated expression statistics
        |
        v
10. Run eligible pathway analysis
        |
        v
11. Score curated biological programs
        |
        v
12. Characterize candidate signaling sources
        |
        v
13. Investigate unexpected pathway/lineage conflicts
        |
        v
14. Integrate evidence across composition, expression,
    pathways, programs, and source signals
        |
        v
15. Validate outputs and write metadata

The decision in step 3 must control the inferential behavior of all downstream
steps.

⸻

# Hard rules for pipeline-development agents

1. The independent experimental unit is the biological sample/library defined
    by the experimental design, not the cell.
2. Never manufacture biological replication.
3. Determine replication before selecting DE, DA, pathway, or program-comparison
    methods.
4. Use uncorrected raw counts for pseudobulk expression analysis.
5. Use canonical uncorrected normalized expression for visualization and
    program scoring.
6. Do not use Harmony or another integration-corrected representation as the
    gene-expression substrate for condition comparisons.
7. Analyze both composition and expression because biological state changes may
    manifest through either or both.
8. Treat observed cell fractions as compositional measurements.
9. Prefer within-compartment proportions when asking about redistribution among
    related states.
10. Classify every population comparison before running expression contrasts.
11. Do not generate faux within-state DE for condition-exclusive populations.
12. Treat severely imbalanced populations as composition-sensitive descriptive
    comparisons.
13. Filter low-information genes within the relevant population before
    constructing gene rankings.
14. Keep descriptive effect estimation distinct from treatment-level inference.
15. Competitive preranked GSEA may characterize a descriptive ranking, but its
    enrichment statistics do not create biological replication.
16. Do not use self-contained replicated testing frameworks as though one
    library per condition were a replicated experiment.
17. Record gene-set overlap and coverage.
18. Flag poorly represented curated programs rather than silently interpreting
    them.
19. Inspect pathway redundancy and leading-edge genes before biological
    synthesis.
20. Curated program scores are complementary evidence, not independent
    treatment-level tests.
21. Keep database-driven discovery separate from project-motivated hypothesis
    interrogation.
22. Candidate cytokine/chemokine expression identifies plausible source
    populations; it does not establish cell-cell communication.
23. Unexpected pathway signals should trigger targeted QC before cell removal or
    annotation changes.
24. Sparse genes require cautious interpretation and should be assessed together
    with detection frequency and coordinated program behavior.
25. Gene absence from a feature set must be distinguished from biological zero
    expression.
26. Sentinel source-signal records must never be interpreted as biological
    samples.
27. Broad compartment-level contrasts may reflect both within-state expression
    and substate redistribution.
28. Fine annotation levels are not automatically superior when condition
    representation is sparse or unstable.
29. Biological interpretation should use conservative language matched to the
    experimental design.
30. Reusable pipeline code should compute evidence; project-specific biological
    conclusions belong in project documentation and reporting.