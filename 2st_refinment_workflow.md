# Codex Implementation Specification: Workflow Refinement

## Objective

Inspect the current workflow implementation and fix the issues identified in the latest run.

**First priority:** inspect:

```text
.snakemake/log/2026-09-03T143624.734039.snakemake.log
```

The `pydamage` step ran unexpectedly long and was manually cancelled. Diagnose why it is taking so long before making workflow changes.

Also investigate why the generated `SRS473742_mapping_stats` output differs substantially from the expected result.

---

## 1. Diagnose `pydamage` before modifying the workflow

### Required actions

1. Read and inspect:

   ```text
   .snakemake/log/2026-09-03T143624.734039.snakemake.log
   ```
2. Identify:

   * the exact `pydamage` command that was executed;
   * input files and their sizes;
   * parameters/configuration used;
   * whether the process was CPU-, memory-, disk-, or I/O-bound;
   * whether the command appears to be processing substantially more data than intended;
   * whether Snakemake is invoking `pydamage` redundantly or with unnecessarily expensive settings.
3. Inspect the corresponding Snakefile/rule, configuration, scripts, and helper code responsible for the `pydamage` invocation.
4. **Before integrating any optimization into Snakemake, run the relevant `pydamage` command directly outside Snakemake** on the same input(s), using the intended parameters.
5. Confirm that the standalone execution:

   * starts correctly;
   * produces valid output;
   * does not exhibit an obvious pathological runtime;
   * uses the expected input data and parameters.

### Optimization goal

Optimize `pydamage` execution without changing the scientific meaning of the analysis.

Prefer the smallest safe change that addresses the diagnosed bottleneck. Do not blindly reduce computation, filter data, or change statistical parameters simply to make the rule faster.

After the standalone test succeeds, integrate the optimized invocation into Snakemake.

### Verification

Run the optimized `pydamage` workflow/rule and verify that:

* it completes successfully;
* expected output files are generated;
* outputs are structurally valid;
* the command actually executed by Snakemake matches the intended optimized configuration.

---

# 2. Fix `SRS473742_mapping_stats`

Inspect the current implementation and determine why the output differs from the expected result.

## 2.1 Remove the third row

The mapping-stats visualization/table currently contains a third row that should not be present.

**Requirement:**

* Remove the third row entirely.
* Do not include genome annotation-related information.
* Do not add genome annotation functionality as part of this change.

The final output should contain only the intended mapping-stat rows.

---

## 2.2 Change the mean indicator

The mean currently uses a green line.

**Requirement:**

* Replace the green mean line with a **red dashed line**.
* Use a dashed line style, e.g. `--`.
* Do not use green for the mean indicator.

This is specifically to improve accessibility for users who may have difficulty distinguishing red and green.

---

## 2.3 Fix edit-distance histogram spacing

The edit-distance histogram currently has excessive gaps between columns.

**Requirement:**

* Histogram bars/columns must **touch each other**.
* Remove the visible gaps between adjacent histogram bins.
* Preserve the existing binning semantics unless the current implementation is demonstrably incorrect.

For matplotlib-style plotting, this will generally mean configuring the bar width/alignment so adjacent bins occupy the full bin width.

---

# 3. Fix `SRS473742_read_stats.tsv` and Read ANI

## 3.1 Add Read ANI to the TSV

`SRS473742_read_stats.tsv` currently does not contain Read ANI.

**Requirement:**

* Add a Read ANI value/column to the generated TSV.
* The value must be derived from the actual read-level ANI data used by the visualization.
* Do not introduce placeholder, guessed, or hard-coded values.

## 3.2 Fix the current Read ANI visualization

The current Read ANI visualization appears to be calculated from unknown or invalid values.

**Requirement:**

* Trace the complete data flow for Read ANI:

  1. source data;
  2. parsing;
  3. calculation;
  4. TSV generation;
  5. visualization.
* Identify why unknown values are currently being used.
* Ensure the visualization uses the same validated Read ANI values that are written to `SRS473742_read_stats.tsv`.
* Handle missing/undefined ANI values explicitly rather than silently converting them into misleading numerical values.

### Consistency requirement

The TSV and visualization must use the **same underlying Read ANI calculation/data**.

There must be no separate, inconsistent implementation of Read ANI for the table and plot.

---

# 4. Redesign horizontal coverage visualization

The current horizontal coverage plot contains random purple areas and does not represent the desired visualization.

## Required behavior

### 4.1 Remove purple regions

Remove the existing random purple areas completely.

Do not preserve them as a fallback or overlay.

### 4.2 Use 100 bp bins

Instead of plotting every individual position:

* divide the reference/genome into **100 bp bins**;
* calculate coverage for each 100 bp bin;
* plot one coverage value per bin.

The binning must be deterministic and correctly handle the final partial bin if the reference length is not divisible by 100.

### 4.3 Color each bin by mean Read ANI

For every 100 bp coverage bin:

1. determine the reads/ANI observations contributing to that region;
2. calculate the **mean Read ANI for that region**;
3. use that mean Read ANI to determine the bin's color.

Do not use arbitrary/random colors.

### 4.4 Use a yellow-to-red color scale

The color mapping must be a **yellow → red** scale.

Avoid green/purple color scales for this visualization.

The colorbar/legend should clearly communicate that color represents mean Read ANI.

### 4.5 Missing ANI values

Define explicit behavior for bins with no valid Read ANI observations.

Do not silently interpret missing/unknown ANI as a real numerical ANI value.

Use an appropriate missing-data representation that does not conflict with the yellow-red scale.

---

# 5. Data-flow and implementation requirements

Before editing code, locate and understand:

* the workflow/Snakefile rules;
* configuration files;
* scripts producing `*_mapping_stats`;
* scripts producing `*_read_stats.tsv`;
* code responsible for Read ANI calculation;
* code responsible for coverage calculation;
* plotting functions;
* `pydamage` invocation and its inputs/outputs.

Prefer reusing existing data-processing functions rather than implementing duplicate calculations.

If Read ANI is currently calculated in multiple places, consolidate the calculation where practical so the TSV and visualizations share one source of truth.

Do not introduce genome annotation dependencies or functionality.

---

# 6. Validation

After implementing the changes, run the relevant workflow for `SRS473742`.

Verify all of the following:

### `pydamage`

* [ ] Original long-running behavior has been diagnosed.
* [ ] `pydamage` has been tested directly outside Snakemake.
* [ ] Standalone execution succeeds.
* [ ] The optimized command is integrated into Snakemake.
* [ ] Snakemake execution succeeds.

### Mapping stats

* [ ] Third row is absent.
* [ ] No genome annotation row/data is included.
* [ ] Mean is shown as a red dashed line.
* [ ] Edit-distance histogram columns touch with no visible gaps.

### Read stats

* [ ] `SRS473742_read_stats.tsv` contains Read ANI.
* [ ] Read ANI is based on real/validated data.
* [ ] Unknown values are handled explicitly.
* [ ] TSV and visualization use the same Read ANI calculation.

### Horizontal coverage

* [ ] Random purple regions are gone.
* [ ] Coverage is aggregated into 100 bp bins.
* [ ] Every reference position is represented through its corresponding bin, including the final partial bin where applicable.
* [ ] Each bin is colored according to mean Read ANI for that region.
* [ ] The color scale runs yellow → red.
* [ ] Missing ANI values are not treated as valid ANI measurements.
* [ ] A clear colorbar/legend identifies the Read ANI scale.

---

# 7. Regression checks

Do not only inspect the final image manually.

Add or update tests where the repository already has a testing framework.

At minimum, validate programmatically that:

1. the mapping-stats output has the expected number of rows;
2. the Read ANI column exists in `SRS473742_read_stats.tsv`;
3. Read ANI values are valid or explicitly missing;
4. coverage is binned at 100 bp;
5. the number of coverage bins matches the expected reference-length/binning calculation;
6. the plotting code uses the required red dashed mean indicator;
7. the coverage color mapping uses the intended yellow-red scale.

Avoid brittle tests that depend on exact pixel positions or rendered-image internals unless image regression testing already exists in the project.

---

# 8. Final Codex report

When finished, report:

* files changed;
* root cause of the slow `pydamage` execution;
* standalone `pydamage` test command and result;
* optimization applied and why it is scientifically safe;
* root cause of the incorrect Read ANI behavior;
* how Read ANI is now calculated and propagated;
* changes made to mapping-stat visualization;
* changes made to horizontal coverage visualization;
* tests/checks executed and their results;
* any remaining uncertainty or issue that could not be verified.

Do not claim the workflow is fixed unless the relevant workflow/tests have actually been executed successfully.


