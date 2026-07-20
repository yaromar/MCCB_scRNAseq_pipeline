# Copilot Instructions

This repository is `MCCB_scRNAseq_pipeline`, a reusable MCCB single-cell RNA-seq pipeline.

The goal is reusable, project-agnostic pipeline logic.

## Development workflow

Pipeline development is driven by real analysis work.

Yaro performs exploratory and semi-interactive analyses in collaborator-specific project folder /MengJuWu-Wu-202606-scRNAseq/. When Yaro asks to “mirror,” “package,” “turn this step into a module,” or “add a module based on this analysis,” convert the stable reusable logic from that project-specific analysis into this pipeline repository.

The exploratory analysis folder is the source of analysis prototypes. This pipeline repo is the destination for generalized, reusable implementations.

Do not copy collaborator-specific biological interpretation, notebooks, results, hard-coded paths, sample names, or assumptions into the pipeline. Extract only reusable workflow logic, parameters, input/output contracts, scripts, and documentation.

When mirroring a step:
1. Identify what the exploratory step does.
2. Define a reusable input/output contract.
3. Generalize paths and parameters.
4. Add or update the appropriate Nextflow module.
5. Add reusable helper scripts if needed.
6. Wire into `main.nf` only if requested.
7. Update documentation and output contracts.
8. Keep project-specific context in docs only, never in executable logic.

## Repository scope

This repo should contain:

- Nextflow DSL2 workflows and modules
- reusable R/Python/bash helper scripts
- samplesheet/config schemas and validation
- documentation
- report templates
- small portable examples

This repo must not contain:

- raw FASTQs
- Cell Ranger outputs
- Seurat/AnnData objects
- collaborator-specific results
- exploratory project notebooks
- large data files

## Architecture

Use four layers:

1. Contract layer: samplesheets, configs, schemas, validation.
2. Execution layer: Nextflow DSL2 workflows/modules.
3. Analysis layer: reusable R/Python/bash scripts.
4. Reporting layer: MultiQC/Quarto/HTML reports.

## HPC environment

Target environment is UMass HPCC using LSF/bsub.

Support:

- local development profile
- HPCC profile with `executor = 'lsf'`
- configurable work directory
- configurable output directory
- configurable references and samplesheets

Never hard-code user-specific or collaborator-specific absolute paths.

## Coding rules

- Keep modules small and composable.
- Prefer Nextflow DSL2.
- Use explicit inputs and outputs.
- Add sensible resource labels.
- Helper scripts must use CLI arguments.
- Fail with informative error messages.
- Avoid hidden global state.
- Update documentation when adding modules.
- Keep examples portable with placeholder paths.
- Never commit large outputs or raw data.

## scRNA-seq analysis principles

- FASTQ QC is pre-count QC.
- Cell-level QC requires count matrices and comes after quantification.
- Do not perform DE on integrated or batch-corrected expression values; instead model batch effects in the DE model.
- Prefer pseudobulk DE.
- Treat cell-level DE as exploratory unless justified.
- Keep project-specific biological interpretation outside reusable pipeline code.