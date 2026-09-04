# Codex Implementation Specification: Add FASTQ Read Counts, Library Layout, and Relative Abundance to Summary Files

## Objective

Update the pipeline so that information from the raw FASTQ input files is incorporated into the summary TSV files.

For each sample, determine:

1. The total number of reads in the raw FASTQ file(s) used as input.
2. The library layout (`PAIRED` or `SINGLE`).
3. The relative abundance represented by `reads_fastq` compared with the total number of raw FASTQ reads.

This information must be added consistently to both project-level and combined summary files.

---

## Required Changes

### 1. Add `n_reads_fastq` to summary files

For each sample, extract the total number of reads from the raw FASTQ input file(s) used by the pipeline.

Add a new column named:

```text
n_reads_fastq
```

to both:

```text
{project}_summary.tsv
all_summary.tsv
```

`n_reads_fastq` must be the second column, immediately after `sample`.

The column order should begin as follows:

```text
sample	n_reads_fastq	library_layout	...
```

---

### 2. Add `library_layout`

Add a new column named:

```text
library_layout
```

This column must be placed immediately after `n_reads_fastq`.

The value must be one of:

```text
PAIRED
SINGLE
```

Determine the value based on the raw FASTQ input files associated with each sample:

* Use `SINGLE` when the sample uses a single-end library and has one FASTQ input.
* Use `PAIRED` when the sample uses paired-end sequencing and has paired FASTQ inputs, such as R1 and R2.

The resulting column order must therefore begin with:

```text
sample	n_reads_fastq	library_layout	...
```

---

### 3. Handle single-end and paired-end FASTQ files correctly

The implementation must support both single-end and paired-end sequencing data.

#### Single-end

For a single-end sample:

```text
library_layout = SINGLE
```

`n_reads_fastq` should be the number of reads in the single raw FASTQ file.

Example:

```text
sample1	1000000	SINGLE
```

#### Paired-end

For a paired-end sample:

```text
library_layout = PAIRED
```

`n_reads_fastq` must represent the total number of reads across all raw FASTQ files associated with that sample.

For example, if a sample has:

```text
sample1_R1.fastq.gz
sample1_R2.fastq.gz
```

and each file contains 1,000,000 reads, then:

```text
n_reads_fastq = 2000000
library_layout = PAIRED
```

The implementation should use the actual FASTQ files provided as input to the pipeline and correctly associate them with their corresponding samples.

---

### 4. Calculate relative abundance

Add a new column named:

```text
calcy_relative_abundance
```

This column should contain the percentage of `n_reads_fastq` represented by `reads_fastq`.

Calculate it as:

```text
calcy_relative_abundance = (reads_fastq / n_reads_fastq) * 100
```

Place `calcy_relative_abundance` immediately after the existing `n_reads` column.

For example:

```text
n_reads_fastq = 2000000
reads_fastq = 500000

calcy_relative_abundance = 25.0
```

The value represents the percentage of total raw FASTQ reads accounted for by `reads_fastq`.

---

## Required Column Placement

The beginning of the summary files must follow this order:

```text
sample	n_reads_fastq	library_layout	...
```

Additionally, the relevant section containing the abundance calculation must follow this order:

```text
...
n_reads	calcy_relative_abundance	...
```

Do not remove, rename, or reorder existing columns except as required to insert these new columns.

---

## Expected Output

Both:

```text
{project}_summary.tsv
all_summary.tsv
```

must include the following new columns:

1. `n_reads_fastq` — second column, immediately after `sample`.
2. `library_layout` — immediately after `n_reads_fastq`, with values `PAIRED` or `SINGLE`.
3. `calcy_relative_abundance` — immediately after `n_reads`.

The relevant structure should be:

```text
sample	n_reads_fastq	library_layout	...	n_reads	calcy_relative_abundance	...
```

For every row:

```text
calcy_relative_abundance = (reads_fastq / n_reads_fastq) * 100
```

---

## Edge Cases and Validation

Ensure the implementation correctly handles:

1. **Single-end FASTQ input**

   * Set `library_layout` to `SINGLE`.
   * Count reads from the single FASTQ file.

2. **Paired-end FASTQ input**

   * Set `library_layout` to `PAIRED`.
   * Count reads across both paired FASTQ files and sum them.

3. **Compressed FASTQ files**

   * Support compressed FASTQ files, such as `.fastq.gz`, if they are already supported by the pipeline.

4. **Missing FASTQ files**

   * Handle missing or inaccessible input files gracefully and provide an appropriate error or missing value according to the pipeline's existing conventions.

5. **Zero read counts**

   * Prevent division-by-zero errors when calculating `calcy_relative_abundance`.
   * Use an appropriate missing value or defined fallback consistent with the pipeline's existing behavior.

6. **Consistency**

   * Ensure the new columns and values are correctly propagated to both `{project}_summary.tsv` and `all_summary.tsv`.

7. **Correct sample association**

   * Ensure FASTQ files are associated with the correct sample, especially when multiple samples and a mixture of single-end and paired-end libraries are processed in the same pipeline run.
