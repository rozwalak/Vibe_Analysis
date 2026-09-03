# Codex Implementation Specification: Workflow Refinement

## Objective

Inspect the current workflow implementation and refine the existing outputs and plotting logic. **Do not rewrite the workflow from scratch.** Preserve the current data-processing logic and correct only the identified issues, while improving the visual design for publication-quality output.

## Required Changes

### 1. `all_summary.tsv`: project identifier

Update the generated `all_summary.tsv` file so that:

* The **first column is named `project`**.
* The value in this column must identify the project, e.g. `Warinner2014`.
* `all_summary.tsv` summarises results across multiple projects, so the project identifier must be explicitly retained in every row.
* Ensure the change is propagated consistently through the code that generates and writes this table.
* Do not infer or hard-code `Warinner2014`; use the appropriate project identifier from the workflow's existing metadata/input structure.

### 2. `SRS473742_mapping_stats.png`: investigate and correct the plot

The current `SRS473742_mapping_stats.png` does **not** match the expected visualization, although the **Read ANI values are correct**.

* Inspect the existing plotting code and underlying data transformation.
* Preserve the currently correct Read ANI values.
* Identify why the other plotted quantities/layout are incorrect.
* Correct the plotting logic rather than modifying the underlying values simply to make the figure look right.
* Verify that the resulting figure accurately represents the source data.

### 3. Mean and median lines

For all relevant plots containing mean and median reference lines:

* Render both the **mean** and **median** lines in **dark gray**.
* Keep them visually distinct from the data distribution while maintaining a restrained publication-style appearance.
* Apply this consistently wherever these reference lines occur.

### 4. Edit-distance plot

The current edit-distance visualization is incorrect because the distances between columns are visually represented.

Change the edit-distance plot to a **true histogram**:

* Use contiguous histogram bins.
* Bin the edit-distance values according to their numeric distribution.
* The x-axis should represent edit distance.
* The y-axis should represent the number/frequency of observations (or the workflow's existing normalized equivalent, if normalization is explicitly required).
* Do **not** use separated bars/columns that imply categorical values.
* Ensure the binning is statistically and visually appropriate for the range of edit-distance values.

### 5. Horizontal coverage plot

The current horizontal coverage plot is incorrect: all bins have the same height.

Correct the implementation so that:

* The x-axis represents genomic/reference position or the relevant horizontal coordinate.
* The plot is divided into appropriate positional bins.
* **Each bin's height must be proportional to the number of reads mapped to that region**, i.e. the coverage.
* Regions with more mapped reads must therefore have taller bins than regions with fewer mapped reads.
* Do not normalize all bins to identical heights.
* Verify the binning/aggregation logic against the underlying mapped-read data before plotting.
* If the workflow already calculates coverage values, use those values directly rather than recomputing them incorrectly in the plotting layer.

### 6. Publication-quality visual design

The current upper-panel layout is intentionally simple and should remain conceptually simple, but it needs to be refined for publication in a **high-impact scientific journal**.

Improve the figure design with emphasis on:

* clean and balanced panel layout;
* consistent typography and font sizing;
* clear axis labels and units;
* appropriate whitespace and margins;
* consistent line widths;
* restrained, publication-appropriate styling;
* clear visual hierarchy;
* consistent alignment of panels and axes;
* legible tick labels;
* appropriately sized legends/annotations;
* no unnecessary decorative elements;
* export settings suitable for high-resolution publication.

Do not make the figure unnecessarily complex. The goal is a **minimal, clean, scientifically precise figure with polished journal-quality typography and layout**.

## Implementation Requirements

1. First inspect the existing workflow, plotting functions, data structures, and output-generation code.
2. Trace each incorrect figure back to its source data and transformation step.
3. Fix the underlying implementation rather than applying cosmetic workarounds.
4. Preserve existing correct behavior, especially the Read ANI calculation/values.
5. Keep the workflow reproducible and compatible with its existing input/output conventions.
6. Avoid introducing unnecessary dependencies.
7. Follow the existing project coding style.
8. If tests exist, update them where appropriate and run the relevant test suite.
9. Run the workflow or the relevant plotting/output steps after modification.
10. Verify the generated:

* `all_summary.tsv`
* `SRS473742_mapping_stats.png`
* edit-distance plot
* horizontal coverage plot

11. Check that the figures contain no misleading visual encodings and that plotted values correspond to the underlying data.

## Acceptance Criteria

The implementation is complete when:

* `all_summary.tsv` has `project` as its first column and correctly identifies the originating project for every row.
* Read ANI values remain unchanged/correct.
* Mean and median lines are dark gray.
* Edit distance is displayed as a proper histogram with contiguous numeric bins.
* Horizontal coverage bin heights reflect the actual number of mapped reads in each region.
* The upper figure layout remains simple but has a polished, high-impact-journal publication aesthetic.
* The workflow runs successfully without regressions.
* Generated outputs have been inspected and verified against the underlying data.
