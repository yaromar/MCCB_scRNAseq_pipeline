# AGENTS.md

## Repository purpose

`MCCB_scRNAseq_pipeline` is a reusable internal pipeline for single-cell RNA-seq analysis in the MCCB Bioinformatics group.

This repository is for generalized pipeline development only. It mirrors stable, reusable steps from exploratory collaborator analyses (/MengJuWu-Wu-202606-scRNAseq/) into a portable Nextflow pipeline.

## Development model

Yaro performs exploratory analysis in /MengJuWu-Wu-202606-scRNAseq/, but this repository must remain project-agnostic.

When Yaro asks to mirror an analysis step into the pipeline:

1. Read the described exploratory step.
2. Separate reusable workflow logic from project-specific analysis.
3. Define the generalized input files, output files, parameters, and assumptions.
4. Implement the reusable version in this repo.
5. Do not copy raw data, results, notebooks, or biological interpretation.
6. Do not hard-code collaborator paths, sample IDs, or conditions.
7. Document how the module can be run on future projects.

## Module development checklist

When adding or modifying a module:

1. Identify stage name.
2. Define input contract.
3. Define output contract.
4. Add/update Nextflow module in `modules/`.
5. Add helper scripts in `scripts/` or `bin/` as needed.
6. Add config parameters if needed.
7. Add resource labels for local and HPCC/LSF execution.
8. Wire into `main.nf` only if requested.
9. Update `README.md` and/or `docs/output_contract.md`.
10. Keep implementation minimal, portable, and testable.

## Expected repository layout

```text
assets/
bin/
conf/
docs/
modules/
scripts/
templates/
test_data/
main.nf
nextflow.config
README.md

Runtime outputs should go to:
results/
work/
logs/

These should be ignored by Git.

## Design principles

* Contract-first design.
* Small modules.
* Explicit inputs and outputs.
* Reproducible execution.
* Portable across users.
* Compatible with UMass HPCC / LSF.
* Human-readable reports.
* Machine-readable summary tables.
* Project-specific biology stays outside executable pipeline code.

## When uncertain

Ask for:

* what exploratory command/script was run
* expected input files
* expected output files
* whether this belongs in the reusable pipeline or project-specific repo
* software availability on HPCC
* required resource profile
* whether the step should be wired into main.nf