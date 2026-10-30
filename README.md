# MCCB scRNA-seq pipeline

Reusable Nextflow/analysis scaffolding for MCCB single-cell RNA-seq workflows.

## Origin and adaptation scope

This repository was initially developed from a project with two biological
conditions, each represented by one pooled scRNA-seq library and no
independent biological replicates.

Some stage guidance therefore includes decisions motivated by that design
(for example, strong caution around condition-batch confounding and
non-replicated condition contrasts).

Treat stage documents as a reusable framework, not a universally fixed recipe.
For each new dataset, explicitly reassess:

* independent biological unit (donor/sample/specimen);
* relationship between donors, libraries, technical replicates, batches, and conditions;
* whether replicated, paired, longitudinal, or nested designs require different statistical models;
* which parameters and analysis branches should change and why.

Project-specific adaptations should be documented in project analysis notes and,
when generalized, promoted back into this repository with updated contracts.

## Current implementation status

Current executable implementation is focused on early pipeline stages
(samplesheet validation, FASTQ inventory, FastQC, MultiQC), while many later
stage documents represent analysis design specifications and expected contracts
for future or partial implementations.

Do not assume a documented stage is fully implemented unless corresponding
workflow/module code and configuration are present.
