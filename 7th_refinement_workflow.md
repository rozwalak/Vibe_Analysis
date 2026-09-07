Modify the pipeline to improve project-level outputs and make the entire analysis resilient to sample-level and project-level errors.

### 1. Generate project-level breadth50 plots

Currently, an output similar to `all_consensus_breadth50_plots` is generated at the overall/global level.

Add equivalent output for every project:

* For each project, generate:
  `{project}_breadth50_plots`
* The folder must contain all corresponding **PNG and SVG** plot files for that project.
* Keep the existing `all_consensus_breadth50_plots` output unchanged.
* Project-level plots should be generated from all successfully completed analyses, even if some samples failed.

### 2. Implement robust error handling

I encountered crashes related to the **BWA implementation**. Currently, such an error can terminate the pipeline and prevent later steps, including summary and plot generation.

Redesign the pipeline so that **an error in one sample or analysis step does not terminate the entire pipeline**.

The desired behavior is:

* Catch failures at the appropriate sample/analysis-step level.
* Record the error with sufficient diagnostic information.
* Continue processing other samples.
* Continue all downstream steps that can still be performed.
* Generate summaries from successful analyses.
* Generate plots from available successful results.
* Never silently ignore an error.

For example, if BWA fails for one sample, that sample should be marked as failed/to-rerun, while the remaining samples continue through the pipeline.

### 3. Implement `errors` folders at TWO levels

Error reporting must exist at both the **project level** and the **overall/global level**.

#### Project level

For every project, create an `errors/` directory inside that project's output structure.

It should contain:

* Logs/details for all errors encountered while processing that project.
* Information identifying the:

  * project
  * sample
  * analysis/step that failed
  * error or exception message
  * relevant command/context needed for debugging
* CSV file(s) containing the original input rows for failed analyses that need to be rerun.

The rerun file should follow:

`{project}_{sample}_to_rerun.csv`

If multiple samples fail, make sure their information is retained without overwriting previous failures. Prefer a design that allows all failed rows for the project to be rerun.

#### Overall/global level

Also create an overall/global `errors/` directory in the top-level output structure.

This directory should contain:

* A consolidated record of **all errors from all projects**.
* Logs/details allowing failures to be traced back to their project and sample.
* A consolidated CSV containing the input rows that need to be rerun across projects.

The global error information must not depend on the successful completion of individual projects.

### 4. Rerun CSV contents

The rerun CSV must contain the **original/selected input rows required to rerun the failed analysis**, not merely sample names.

Preserve the relevant original columns and values.

At project level, use:

`{project}_{sample}_to_rerun.csv`

At overall/global level, provide a consolidated rerun CSV covering all affected projects/samples.

Avoid overwriting rerun information when multiple samples fail.

### 5. Pipeline execution requirements

The pipeline should distinguish between:

* **sample-level failures** — one sample fails but the project continues;
* **project-level failures** — one project has problems but other projects continue;
* **global/pipeline failures** — errors that genuinely prevent further processing.

Where possible, errors should be isolated to the smallest possible scope.

A failure should only stop execution when continuing would be technically impossible or would produce invalid results.

### 6. Summary and plot generation

Summary and plotting steps must operate on the successfully completed analyses.

If some samples failed:

* summaries should still be generated;
* project-level plots should still be generated;
* overall/global plots should still be generated where sufficient data is available;
* failed samples should be clearly represented in the error/rerun information rather than causing the output-generation steps to crash.

### 7. Implementation

Before modifying the code:

1. Inspect the existing pipeline structure and identify where sample, project, and global processing occur.
2. Identify all places where exceptions — especially BWA-related exceptions — can currently terminate execution.
3. Implement error handling at the appropriate scope.
4. Centralize error recording where practical instead of duplicating error-handling logic.
5. Preserve the existing output structure and behavior wherever possible.
6. Do not hide or suppress exceptions without recording them.

### 8. Tests / validation

Add or update tests to verify at minimum:

* One sample fails during BWA → remaining samples continue.
* A failed sample is recorded in the project `errors/` directory.
* The failed sample's original input row is written to the project rerun CSV.
* Errors from multiple samples are all retained.
* Errors from multiple projects are consolidated in the global `errors/` directory.
* A project with failed samples still produces its summaries.
* A project with failed samples still produces `{project}_breadth50_plots`.
* PNG and SVG project-level plots are generated.
* Overall summaries/plots are still generated when possible.
* A failure in one project does not unnecessarily terminate processing of other projects.
* Error logs contain enough information to diagnose the failure.

After implementation, run the relevant test suite and perform a representative pipeline run with at least one intentionally/realistically failing sample to verify that the pipeline completes and produces the expected outputs.

Finally, provide a concise summary of:

* files changed,
* error-handling architecture,
* project-level outputs added,
* project-level and global `errors/` outputs,
* tests performed and their results.
