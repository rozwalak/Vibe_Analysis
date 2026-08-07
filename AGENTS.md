# AGENTS.md

## Project purpose

This directory is a scientific analysis workspace.

The primary objective is to answer the scientific questions defined in `SCIENTIFIC_BRIEF.md`. Code is an instrument used to obtain, test, and reproduce the scientific answer. Producing sophisticated, reusable, or extensively engineered software is not the primary objective unless explicitly requested.

Before beginning substantial work, read:

1. `SCIENTIFIC_BRIEF.md`
2. `ANALYSIS_PLAN.md`
3. `STATUS.md`
4. `DECISIONS.md`
5. Relevant files under `inputs/`

## Priority order

When making trade-offs, use this priority order:

1. Scientific validity
2. Correct interpretation of the data
3. Transparent evidence for conclusions
4. Reproducibility
5. Computational efficiency
6. Code quality and generality

Do not improve code architecture at the expense of completing or validating the scientific analysis.

## Before writing code, identify:

1. the precise scientific claim this analysis could support;
2. the evidence needed to support it;
3. the main confounders or alternative explanations;
4. the smallest analysis that would discriminate among them.

Then perform the analysis. Treat code as supporting material and update the
scientific report as soon as results are available.

## Scientific workflow

For each substantial task:

1. Restate the scientific question being addressed.
2. Identify the data and evidence required to answer it.
3. Check data structure, units, missingness, sample counts, and relevant metadata.
4. State important assumptions before relying on them.
5. Perform the smallest adequate analysis.
6. Validate unexpected or important findings using an independent check where practical.
7. Save durable tables and figures under `results/`.
8. Update the scientific interpretation in `reports/scientific_report.md`.
9. Update `STATUS.md` with findings, limitations, and unresolved questions.
10. Record consequential methodological choices in `DECISIONS.md`.

Do not stop merely because a script runs successfully. Completion is determined by whether the scientific question has been answered and supported.

## Directory rules

* `inputs/` contains source data, reference data, and metadata.
* Treat source data as read-only.
* Never modify files reached through `inputs/raw-data`.
* `work/` contains temporary files, caches, logs, and intermediates.
* Files under `work/` may be regenerated or deleted.
* `scripts/` contains analysis and plotting code.
* `notebooks/` contains exploratory analyses.
* `results/` contains durable evidence used in the report.
* `reports/` contains the scientific conclusions and methods.
* Do not place generated outputs in the project root.
* Do not treat a notebook as the sole record of an important result; export important tables, figures, parameters, and conclusions.

## Analysis standards

* Preserve sample identifiers exactly.
* Verify joins explicitly and report unmatched records.
* Report the number of samples, observations, and features included in each major analysis.
* Distinguish biological absence from missing data and technical failure.
* Avoid pseudoreplication.
* Account for repeated measures, batches, cohorts, and other dependencies when relevant.
* Do not imply causation from association.
* Report effect sizes and uncertainty, not only significance values.
* Apply multiple-testing correction where appropriate.
* Examine sensitivity to important thresholds and filtering choices.
* Identify results driven by very small numbers of samples.
* Clearly separate observed results from hypotheses and speculation.
* Describe negative and inconclusive results rather than hiding them.

## Figures and tables

Every important figure must:

* Answer a stated scientific question.
* Have labelled axes, units, sample sizes, and an informative caption.
* Use biologically meaningful ordering.
* Avoid unnecessary decoration.
* Show underlying observations where practical.
* Be accompanied by the table or data used to generate it.

Store final figures in `results/figures/` and supporting tables in `results/tables/`.

## Code policy

Prefer short, readable, task-specific scripts over frameworks.

Do not:

* Build command-line interfaces unless they are useful for reproducing the analysis.
* Add packaging infrastructure unless explicitly required.
* Refactor functioning analysis code merely for elegance.
* Introduce a database when ordinary files are adequate.
* Add dependencies without a clear analytical benefit.
* Rewrite a working analysis in another language without a scientific reason.
* spend substantial effort optimizing code that already runs adequately.

Code should include enough validation and error handling to prevent silent scientific errors.

## Reproducibility

For each durable result, retain:

* The script or notebook that generated it.
* Input filenames or dataset versions.
* Parameters and thresholds.
* Software and package versions where material.
* Random seeds for stochastic analyses.
* A concise command or workflow for regeneration.

Prefer relative paths inside scripts. Large source datasets may be represented by read-only symbolic links.

## Reporting

`reports/scientific_report.md` is a primary deliverable.

Organize it around:

1. Scientific question
2. Data examined
3. Methods
4. Results
5. Interpretation
6. Limitations
7. Conclusions
8. Recommended next analyses

Do not organize the main report as a chronological account of code development.

Every substantive conclusion should point to a supporting figure, table, diagnostic, or quantitative result.

## Communication during work

When reporting progress, prioritize:

* New scientific findings
* Problems with data quality
* Assumptions that affect interpretation
* Alternative explanations
* Analyses that materially change the conclusions
* Remaining uncertainty

Do not lead with lists of files created or code refactored unless those changes affect the scientific result.

## Definition of done

A task is complete when:

* The requested scientific question has been addressed.
* The relevant evidence has been checked.
* Important limitations and alternative explanations are stated.
* Durable figures and tables have been saved.
* The report contains an interpretable scientific conclusion.
* The analysis can be reproduced from the retained code and documented inputs.

Working code alone does not constitute completion.

