Please review and modify the workflow with the following requirements.

### 1. Fix `all_summary.tsv`

`all_summary.tsv` must contain the complete information from every `{project}_summary.tsv`.

It is currently incomplete, likely because mapping statistics are missing or incompletely propagated for at least one sample.

* Inspect `aDNA_panmap_snakemake` to determine the expected structure and content of `all_summary.tsv`.
* Trace the workflow to identify why some project/sample statistics are missing.
* Make sure all samples and all relevant mapping statistics are correctly represented in the final `all_summary.tsv`.
* Verify this with test data, including a project containing multiple samples.

### 2. Redesign error reporting

Remove the current `results/errors` structure.

Errors should be reported at two levels:

**Global level**

* Create a global `.txt` error report.
* The report should list the **project name and sample name** for every sample/project that became corrupted or failed at any point in the workflow.
* Keep this report concise and suitable for quickly identifying affected samples.

**Project/sample level**

* Detailed error information should remain inside the relevant project/sample-specific folders, close to the logs/results that caused the problem.

**Important:**

* If a project has **no errors**, do **not** create an `errors` folder for that project.
* Avoid creating empty error directories or placeholder error files.

### 3. Fix pydamage log duplication

The pydamage log is currently reported twice.

Only report:

`pydamage.log.pydamage`

Do not duplicate the same pydamage log under another name/path.

### 4. Move `status` under `logs`

The `status` directory should be a subdirectory of `logs`.

Expected structure:

```text
logs/
└── status/
```

rather than having `status` as a separate top-level directory.

### 5. `csv_to_rerun` location

`csv_to_rerun` files should exist **only at the project level**.

Remove sample-specific/project-sample files such as:

```text
PacificCalculus_ERSX0004058_to_rerun.csv
```

There should be a single appropriate `csv_to_rerun` output per project, rather than separate files for individual samples.

### 6. Simplify and harden the workflow

While implementing the above changes, carefully review the workflow architecture.

Look for opportunities to:

* remove redundant steps or duplicated outputs;
* simplify file/path handling;
* reduce unnecessary intermediate files;
* make error propagation more reliable;
* make the workflow easier to understand from its directory structure and filenames;
* make failures easier to diagnose;
* avoid implicit assumptions about which files exist;
* make the workflow more robust when one sample fails while others succeed.

Do not simplify merely for the sake of reducing code: preserve the existing functionality and outputs that are actually required.

### 7. Testing and validation

After making the changes, test the workflow thoroughly.

At minimum, verify:

1. `all_summary.tsv` contains the expected information from all `{project}_summary.tsv` files.
2. Mapping statistics are not silently lost when a sample has incomplete/failed mapping statistics.
3. A failed/corrupted sample is listed in the global error `.txt` report with both project and sample names.
4. Detailed error information is available in the appropriate project/sample folder.
5. Projects without errors do not receive an `errors` folder.
6. Only `pydamage.log.pydamage` is reported.
7. `logs/status/` is used instead of a separate `status/` directory.
8. `csv_to_rerun` exists only at project level.
9. Sample-specific files such as `*_to_rerun.csv` are no longer generated.
10. Partial failures do not incorrectly mark unrelated samples/projects as failed.
11. The final directory structure is consistent and self-explanatory.

Please inspect the existing workflow and `aDNA_panmap_snakemake` before making changes, and prefer a robust underlying fix over adding special-case handling for the current symptoms.
