# Refinement of `plan_workflow.md`: Codex Implementation Specification

## Objective

First, inspect and understand the current workflow before making any changes.

Review:

* The current pipeline structure.
* All Snakemake rules.
* Configuration files.
* Input sample sheets.
* Helper scripts.
* Conda/environment definitions.
* Current output directory structure.
* Current output files and their dependencies.

Before implementation, identify where each requested change belongs and preserve existing functionality unless explicitly changed below.

## General Implementation Requirements

* Modify the existing workflow rather than creating duplicate or parallel functionality where possible.
* Reuse existing rules, scripts, and intermediate files when practical.
* Prefer integrating additional functionality into existing rules when this keeps the workflow maintainable.
* Create new rules only when integration into an existing rule would make the workflow unclear or technically inappropriate.
* Ensure all rules work for multiple projects and multiple samples.
* Avoid hard-coded project names or sample IDs such as `Warinner2014` or `SRS473742`. These names are examples only.
* Ensure that output paths are generated dynamically from project and sample identifiers.
* Preserve compatibility with existing workflow outputs unless their replacement is explicitly requested below.
* Define appropriate dependencies so downstream rules wait for all required upstream outputs.
* Update configuration handling, rule inputs/outputs, scripts, and environment dependencies as required.
* Use temporary files safely and avoid partially written final output files.
* Validate that BAM files are indexed after they are modified.
* Do not silently omit samples from summary tables because they have low or zero mapping.

---

# Change #1: Update Sample Sheet and Library Type Handling

## Current Configuration

The current configuration contains something similar to:

```text
# The supplied files are tab-delimited despite their .csv extension.
projects:
  Warinner2014:
    samplesheet: input/metagenome_assembly-Warinner2014.csv
    library_type: single
```

## Required Implementation

### 1. Support both `.csv` and `.tsv` sample sheets

Remove the assumption that `.csv` files are tab-delimited.

The workflow must correctly support both:

```text
*.csv
```

and:

```text
*.tsv
```

files.

Determine the delimiter appropriately based on the file format or otherwise implement reliable delimiter handling.

### 2. Remove project-level `library_type`

Remove the project-level parameter:

```text
library_type: single
```

Library type must instead be determined independently for each sample from the sample sheet.

### 3. Determine library type from read columns

Use the following logic:

* A single-end library has data in column `R0`.
* A paired-end library has data in columns `R1` and `R2`.

The workflow should determine the appropriate library type for each sample automatically from these columns.

Do not require a separate `library_type` parameter in `config.yaml`.

## Expected Inputs

Example sample sheet columns:

```text
sample    R0    R1    R2
```

The exact delimiter may be comma-separated or tab-separated.

## Expected Behaviour

For each sample:

* If `R0` is populated, process the sample as single-end.
* If `R1` and `R2` are populated, process the sample as paired-end.
* The implementation should use the appropriate input files and commands for that sample.

## Acceptance Criteria

* [ ] `.csv` sample sheets work correctly.
* [ ] `.tsv` sample sheets work correctly.
* [ ] No project-level `library_type` parameter is required.
* [ ] Single-end samples are detected using `R0`.
* [ ] Paired-end samples are detected using `R1` and `R2`.
* [ ] A single project may contain samples with different library types.
* [ ] No hard-coded assumptions are made about all samples having the same library type.

---

# Change #2: Update `panmap` Parameters

## Required Implementation

Add the following parameters to the relevant current `panmap` commands:

```text
--trim-start 7 --trim-end 7
```

Also add:

```text
--min-depth 3
```

These parameters should be incorporated into the existing `panmap` workflow.

The goal is to improve placement against the reference and improve the quality of the consensus sequence.

### Optional `--force-leaf`

Also support:

```text
--force-leaf
```

This option must be configurable.

Add an appropriate Boolean or equivalent on/off setting in the configuration so that:

* When enabled, `--force-leaf` is included in the `panmap` command.
* When disabled, `--force-leaf` is not included.

Do not require users to manually edit the Snakemake rule to change this behaviour.

## Expected Input

A configuration option controlling `--force-leaf`.

The exact configuration structure can be chosen based on the current workflow, but it should be clear and documented.

## Expected Output

Existing `panmap` outputs should continue to be produced, including the consensus sequence and BAM files.

## Acceptance Criteria

* [ ] `--trim-start 7` is passed to the appropriate `panmap` command.
* [ ] `--trim-end 7` is passed to the appropriate `panmap` command.
* [ ] `--min-depth 3` is passed to the appropriate `panmap` command.
* [ ] `--force-leaf` can be enabled through configuration.
* [ ] `--force-leaf` can be disabled through configuration.
* [ ] The workflow behaves correctly in both configurations.
* [ ] Existing `panmap` outputs remain available.

---

# Change #3: Add MD Tags to `panmap` BAM Files

## Problem

MD tags are currently missing from:

```text
panmap/*.bam
```

This limits downstream analyses.

## Required Implementation

Add MD tags using `samtools calmd` or an equivalent reliable method.

A command similar to the following can be used:

```text
samtools calmd -@ 8 -b Warinner2014/panmap/SRS473742.bam \
Warinner2014/panmap/SRS473742.ref.fa \
> Warinner2014/panmap/SRS473742.tmp.bam && \
mv Warinner2014/panmap/SRS473742.tmp.bam \
Warinner2014/panmap/SRS473742.bam && \
samtools index -@ 8 Warinner2014/panmap/SRS473742.bam
```

The paths and thread count above are examples only. Use workflow variables and configured resources rather than hard-coded values.

## Preferred Design

Ideally, incorporate MD tag generation into the existing `panmap` rule.

If this would be technically inappropriate, unsafe, or difficult to maintain, create a separate Snakemake rule.

If a separate rule is created, all downstream rules requiring MD tags must depend on the corrected BAM rather than the original BAM.

## Important Requirements

* Avoid corrupting or partially replacing the final BAM file.
* Use an appropriate temporary output and atomic replacement strategy.
* Ensure the final BAM is indexed.
* Ensure the MD-tagged BAM is the BAM used by pyDamage and mapping statistics.

## Expected Inputs

* `panmap` BAM file.
* Corresponding reference FASTA file.

## Expected Outputs

* BAM file containing MD tags.
* Corresponding BAM index.

## Acceptance Criteria

* [ ] BAM files contain MD tags after this step.
* [ ] BAM files remain valid and readable by `samtools`.
* [ ] BAM files are indexed.
* [ ] pyDamage receives a BAM containing MD tags.
* [ ] Mapping statistics receive a BAM containing MD tags.
* [ ] The workflow does not expose downstream rules to a partially written BAM.

---

# Change #4: Replace MapDamage2 with pyDamage

## Required Implementation

Replace MapDamage2 with pyDamage.

Remove MapDamage2 from the active workflow rules and incorporate pyDamage instead.

Use a command equivalent to:

```text
pydamage --outdir results/Warinner2014/pydamage \
analyze results/Warinner2014/panmap/SRS473742.bam --plot
```

The exact paths must be generated dynamically.

## Important Dependency

The BAM file provided to pyDamage must already contain MD tags.

Therefore, pyDamage must depend on the MD-tag correction step described in Change #3.

## Required Outputs

For each sample, generate pyDamage output in the expected sample-specific output directory, including:

```text
pydamage_results.csv
```

and the requested plots.

The final workflow structure should support paths similar to:

```text
results/{project}/{sample}/pydamage/pydamage_results.csv
```

## Environment Requirements

Update the workflow environment definitions as necessary:

* Remove MapDamage2 dependencies if they are no longer used.
* Add pyDamage and its required dependencies.
* Ensure the environment can reproduce the pyDamage analysis.

## Acceptance Criteria

* [ ] MapDamage2 is no longer used by the workflow.
* [ ] pyDamage is executed for each applicable sample.
* [ ] pyDamage receives a BAM containing MD tags.
* [ ] `pydamage_results.csv` is produced.
* [ ] pyDamage plots are produced.
* [ ] Output paths are generated dynamically for project and sample.
* [ ] The required software environment is updated accordingly.

---

# Change #5: Create Comprehensive `mapping_statistics`

## Objective

Create comprehensive statistics for mapped reads in each BAM file.

The functionality should be similar to analyses performed by:

[bam-filter repository](https://github.com/genomewalker/bam-filter?utm_source=chatgpt.com)

However, do not run `bam-filter`.

Because of problems with calculating some values, reimplement the required statistics directly in this workflow.

## Rename and Expand the Existing Rule

The current version partially performs this analysis in the `coverage` rule.

Expand this functionality and rename the rule:

```text
coverage
```

to:

```text
mapping_statistics
```

The output directory should be:

```text
mapping_statistics
```

The existing coverage plot and:

```text
covered_positions.tsv
```

should still be generated.

Additional changes to the coverage visualisation are described in Change #6.

## Raw Per-Read Statistics

Create or retain a raw per-read table containing sufficient information to generate the required summary statistics and histograms.

The implementation must retain at least:

* Read identifier.
* Read length.
* Edit distance.
* Read ANI.
* Mapping quality, if required for downstream visualisation or validation.
* Genomic position or other information required to calculate per-window statistics for the coverage visualisation.

The following command is only an example of the type of calculation required:

```text
samtools view -F 2308 input.bam | \
awk '{
    cigar=$6
    nm=-1

    for(i=12;i<=NF;i++)
        if($i ~ /^NM:i:/) {
            split($i,a,":")
            nm=a[3]
        }

    len=0
    while(match(cigar, /[0-9]+[MIDNSHP=X]/)) {
        op=substr(cigar, RSTART+RLENGTH-1, 1)
        n=substr(cigar, RSTART, RLENGTH-1)

        if(op=="M" || op=="=" || op=="X")
            len += n

        cigar=substr(cigar, RSTART+RLENGTH)
    }

    if(nm >= 0 && len > 0)
        print $1, len, nm, (1-nm/len)*100
}'
```

This is only an example. Implement the calculations as necessary to produce the outputs specified below.

## Required Summary Output

For each sample, create:

```text
results/{project}/{sample}/mapping_statistics/{sample}_stats.tsv
```

For example:

```text
results/Warinner2014/SRS473742/mapping_statistics/SRS473742_stats.tsv
```

The file must contain the following columns:

| Column                  | Definition                                                     |
| ----------------------- | -------------------------------------------------------------- |
| `reference`             | ID of the reference genome                                     |
| `reference_length`      | Length of the reference genome                                 |
| `n_reads`               | Number of aligned reads in the BAM file                        |
| `read_length_mean`      | Mean of all read lengths                                       |
| `read_length_std`       | Standard deviation of read lengths                             |
| `read_length_min`       | Minimum observed read length                                   |
| `read_length_max`       | Maximum observed read length                                   |
| `read_length_median`    | Median observed read length                                    |
| `mapping_quality_mean`       | Mean mapping quality                                           |
| `edit_distance_mean`        | Mean edit distance                                             |
| `read_ani_mean`         | Mean average nucleotide identity of reads                      |
| `read_ani_std`          | Standard deviation of average nucleotide identity of reads     |
| `read_ani_median`       | Median average nucleotide identity of reads                    |
| `coverage_mean`         | Mean coverage value for mapped reads                           |
| `bases_covered`         | Number of reference positions covered by at least 1 read       |
| `bases_covered_depth3`  | Number of reference positions covered by at least 3 reads      |
| `bases_covered_depth5`  | Number of reference positions covered by at least 5 reads      |
| `bases_covered_depth10` | Number of reference positions covered by at least 10 reads     |
| `breadth`               | Percentage of reference positions covered by at least 1 read   |
| `breadth3`              | Percentage of reference positions covered by at least 3 reads  |
| `breadth5`              | Percentage of reference positions covered by at least 5 reads  |
| `breadth10`             | Percentage of reference positions covered by at least 10 reads |

For `mapping_quality_mean`, calculate the mean MAPQ for mapped reads. For example:

```text
samtools view -F 4 input.bam | \
awk '{sum+=$5; n++} END {if(n>0) print sum/n}'
```

For `edit_distance_mean`, the summary table should contain the mean value, but the raw per-read table must retain the edit distance of each read for histogram generation.

## Empty and Low-Coverage Samples

The implementation must handle samples with:

* Very few mapped reads.
* Zero mapped reads.
* Zero covered bases.

The workflow must still produce the expected summary output.

Use appropriate values for statistics that cannot be calculated from an empty set, rather than causing the workflow to fail.

Ensure these samples remain represented in project-level and global summary files.

## Acceptance Criteria

* [ ] The rule is named `mapping_statistics`.
* [ ] The output directory is named `mapping_statistics`.
* [ ] The coverage plot is still generated, but integrated with other plots.
* [ ] `covered_positions.tsv` is still generated.
* [ ] A per-sample `{sample}_stats.tsv` file is generated.
* [ ] Per-read data extracted from bam file and used for _stats.tsv should be kept in '{sample}_read_stats.tsv' in the same folder as `{sample}_stats.tsv`.
* [ ] All required columns are present.
* [ ] Per-read edit distances are retained in raw data.
* [ ] Per-read data contain sufficient information for all requested histograms and genomic-window visualisations.
* [ ] Coverage thresholds of 1, 3, 5, and 10 are calculated correctly.
* [ ] Breadth values are calculated correctly from the reference length.
* [ ] Samples with low or zero mapped reads do not cause the workflow to fail.
* [ ] Such samples remain present in the summary output.

---

# Change #6: Update `mapping_statistics` Visualisation

## Objective

Replace the previous visualisation, which only showed horizontal coverage (breadth), with a combined multi-panel figure.

Use the data generated by `mapping_statistics`.

## Figure Layout

### First Row: Three Histograms

Create three plots:

1. Read length distribution.
2. Edit distance distribution.
3. Read ANI distribution.

For each histogram:

* Plot the distribution of the corresponding per-read values.
* Display both the mean and median within the plot.
* Do not use dashed lines.

The exact visual implementation of the mean and median indicators can be chosen, but they must be clearly distinguishable.

### Second Row: Horizontal Coverage

Create one horizontal coverage plot, similar to the previous version, with the following additional information.

#### Background Histogram

Add a slightly transparent histogram in the background with:

```text
1,000 bp bins
```

Each bin should represent the number of reads mapped within the corresponding genomic region.

#### Mean ANI by Genomic Window

Colour each genomic bin according to the mean ANI of reads mapped to that bin.

The visualisation should make it possible to identify genomic regions where:

* Mapping is lower.
* Read ANI is lower.
* Poorly mapped regions may also be more distantly related to the reference.

Use the per-read or derived window-level data generated by `mapping_statistics` rather than recalculating unrelated statistics elsewhere in the workflow.

## Future/Additional Genomic Annotation Component

Add genomic visualisation with functional annotation below the coverage plot.

The intended result is an annotated genome representation below the coverage visualisation.

Functional annotation should be displayed in genomic coordinates aligned with the coverage plot so that regions of:

* High or low coverage.
* High or low ANI.
* Functional genomic features.

can be visually compared.

The implementation should use the available annotation data already produced or available to the workflow, as appropriate.

## Acceptance Criteria

* [ ] A single combined figure is produced.
* [ ] The first row contains read length, edit distance, and read ANI histograms.
* [ ] Mean and median are shown in all three histograms.
* [ ] Dashed lines are not used for mean or median indicators.
* [ ] The second row shows horizontal coverage.
* [ ] A slightly transparent background histogram is included.
* [ ] Background bins are 1,000 bp.
* [ ] The histogram represents mapped-read counts by genomic region.
* [ ] Bins are coloured according to mean ANI for reads mapped to that region.
* [ ] The plot allows comparison between mapping patterns and ANI.
* [ ] Genomic functional annotation is shown below the coverage plot and aligned to genomic coordinates.
* [ ] The visualisation is generated from data produced by `mapping_statistics`.

---

# Change #7: Update Project-Level Summary Files

## Objective

Update project-level summary files such as:

```text
Warinner2014_summary.tsv
```

The new summary must combine information from:

1. pyDamage output:

```text
results/{project}/{sample}/pydamage/pydamage_results.csv
```

2. Mapping statistics:

```text
results/{project}/{sample}/mapping_statistics/{sample}_stats.tsv
```

## Required Columns

The first column must be:

```text
sample
```

For example:

```text
SRS473742
```

Then include:

### All columns from `{sample}_stats.tsv`

Include all mapping statistics columns.

### Selected columns from `pydamage_results.csv`

Include only:

```text
CtoT-0
CtoT-1
CtoT-2
CtoT-3
CtoT-4
CtoT-5
CtoT-6
CtoT-7
CtoT-8
CtoT-9
predicted_accuracy
pvalue
```

Do not include additional pyDamage columns unless required internally.

## Required Output

For each project, generate a summary file similar to:

```text
results/Warinner2014/Warinner2014_summary.tsv
```

The exact dynamic path should be:

```text
results/{project}/{project}_summary.tsv
```

## Handling Missing or Empty Results

If a sample has very low or zero mapped reads:

* The sample must still appear in the project summary.
* Mapping statistics should still be represented.
* Missing pyDamage values should not cause the sample to be removed.
* Missing or undefined values should be represented appropriately.

## Acceptance Criteria

* [ ] One project-level summary file is produced per project.
* [ ] The first column is `sample`.
* [ ] All samples are represented, including low- and zero-mapping samples.
* [ ] All columns from `{sample}_stats.tsv` are included.
* [ ] Only the specified pyDamage columns are included.
* [ ] Rows are correctly matched by sample identifier.
* [ ] Missing pyDamage results do not cause samples to disappear from the summary.

---

# Change #8: Create Global Summary and Filtered Consensus FASTA Files

## Part A: Global Summary

### Required Implementation

When multiple projects are analysed, multiple project output folders may exist, for example:

```text
results/Warinner2014
results/Ottoni2021
```

As the final workflow step, concatenate all project-level summary files, for example:

```text
Warinner2014_summary.tsv
Ottoni2021_summary.tsv
```

into:

```text
results/all_summary.tsv
```

## Requirements

The global summary must:

* Include all project-level summary information.
* Include all samples.
* Include samples with very low numbers of mapped reads.
* Include samples with zero mapped reads.
* Preserve all required columns and data.

## Part B: Filter Samples by `breadth3`

Filter:

```text
results/all_summary.tsv
```

for samples satisfying:

```text
breadth3 > 50%
```

Only samples passing this threshold should be used for the consensus FASTA aggregation described below.

Do not remove non-passing samples from `all_summary.tsv`. The filtering is only for selecting consensus sequences.

breadth3 > 50%: both parameters should be editable in the config.

## Part C: Per-Project Consensus FASTA

For each sample passing:

```text
breadth3 > 50%
```

find its consensus sequence in a sample-specific directory such as:

```text
results/{project}/{sample}/panmap/
```

For example:

```text
SRS473742.consensus.fa
```

Combine all qualifying consensus FASTA files from each project into:

```text
results/{project}/{project}_consensus_breadth50.fna
```

For example:

```text
results/Warinner2014/Warinner2014_consensus_breadth50.fna
```

## Part D: Global Consensus FASTA

Combine all consensus FASTA files from all projects that pass:

```text
breadth3 > 50%
```

into:

```text
results/all_consensus_breadth50.fna
```

## Important Requirements

* Use the project and sample information from the workflow and summary tables dynamically.
* Do not hard-code project names.
* Do not hard-code sample IDs.
* Ensure each consensus sequence can be associated with its original sample.
* Avoid duplicate sequences caused by repeated file discovery.
* Handle projects with no samples passing the threshold without causing the workflow to fail.
* Handle runs with only one project as well as runs with multiple projects.

## Acceptance Criteria

### Global Summary

* [ ] `results/all_summary.tsv` is created.
* [ ] All project-level summary files are included.
* [ ] All samples are retained, including samples with low or zero mapped reads.
* [ ] The global summary contains all required information.

### Per-Project Consensus FASTA

* [ ] Only samples with `breadth3 > 50%` are selected.

* [ ] A filtered consensus FASTA is created for each project.

* [ ] Output naming follows:

  ```text
  results/{project}/{project}_consensus_breadth50.fna
  ```

* [ ] No samples failing the threshold are included.

* [ ] Projects with no passing samples do not cause the workflow to fail.

### Global Consensus FASTA

* [ ] All qualifying consensus sequences from all projects are combined.

* [ ] The output is:

  ```text
  results/all_consensus_breadth50.fna
  ```

* [ ] Only samples with `breadth3 > 50%` are included.

* [ ] Duplicate inclusion of the same consensus sequence is avoided.

---

# Final Validation Requirements

After implementing all changes, validate the workflow end-to-end.

At minimum, verify the following:

## Configuration and Input Handling

* [ ] Both `.csv` and `.tsv` sample sheets are supported.
* [ ] Single-end and paired-end samples are detected independently.
* [ ] Mixed library types can be processed within the same project.

## `panmap`

* [ ] `--trim-start 7` is applied.
* [ ] `--trim-end 7` is applied.
* [ ] `--min-depth 3` is applied.
* [ ] `--force-leaf` can be switched on and off through configuration.

## BAM Processing

* [ ] BAM files contain MD tags.
* [ ] BAM files are indexed.
* [ ] Downstream rules use the corrected BAM files.

## pyDamage

* [ ] MapDamage2 has been replaced.
* [ ] pyDamage runs successfully.
* [ ] pyDamage receives BAM files containing MD tags.
* [ ] `pydamage_results.csv` is generated.

## Mapping Statistics

* [ ] Per-read statistics are generated.
* [ ] `{sample}_stats.tsv` contains all requested columns.
* [ ] Coverage and breadth thresholds are calculated correctly.
* [ ] Zero-mapping samples are handled without workflow failure.

## Visualisation

* [ ] The combined multi-panel figure is generated.
* [ ] All three histograms are present.
* [ ] Mean and median are displayed.
* [ ] Horizontal coverage is displayed.
* [ ] Read-count information is shown in 1,000 bp genomic bins.
* [ ] Mean ANI is represented by genomic window.
* [ ] Functional annotation is displayed below the genomic coverage plot.

## Summaries and Consensus FASTA Files

* [ ] Project-level summaries are generated.
* [ ] `all_summary.tsv` is generated.
* [ ] Low- and zero-mapping samples remain represented.
* [ ] `breadth3 > 50%` filtering is applied only for consensus FASTA selection.
* [ ] Per-project filtered consensus FASTA files are generated.
* [ ] `all_consensus_breadth50.fna` is generated.

## Final Deliverables

In addition to implementing the changes, provide a concise summary of:

1. Which files were modified.
2. Which files or rules were added, if any.
3. How library type is now detected.
4. How the `--force-leaf` option is configured.
5. Where MD tags are added.
6. How pyDamage is integrated.
7. Which files contain raw per-read statistics and summary statistics.
8. How the combined mapping statistics figure is generated.
9. How project-level and global summaries are generated.
10. How consensus sequences are selected and combined.
11. Any implementation decisions where the existing workflow required a different technical solution than the examples provided above.
12. Any assumptions that were necessary because the requested functional annotation or existing workflow structure did not provide enough information directly.
