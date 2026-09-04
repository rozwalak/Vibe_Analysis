Please modify the existing pipeline/workflow according to the following requirements. Preserve the current behavior and output structure wherever possible, and make sure the changes do not break existing functionality.

### 1. Save mapping statistics plots in both PNG and SVG formats

The pipeline currently generates:

`{sample}_mapping_stats.png`

Keep generating this PNG file, but additionally save the same plot as:

`{sample}_mapping_stats.svg`

The SVG should contain the same information and visualization as the PNG.

---

### 2. Create a consolidated `all_consensus_breadth50_plots` directory

Create a new directory inside the main `results` directory:

`results/all_consensus_breadth50_plots/`

This directory should contain copies of the mapping statistics plots for genomes that meet the required filtering criteria.

For every sample/genome combination:

* The genome must be present in `all_consensus_breadth50`.
* The genome must satisfy:

`breadth3 > 50`

Here, `breadth3` is the percentage of the genome covered by **at least 3 reads**.

Therefore:

`breadth3 > 50`

means that **more than 50% of the genome is covered at a sequencing depth of at least 3 reads**.

For every genome that satisfies these conditions, copy/save both:

* `{sample}_mapping_stats.png`
* `{sample}_mapping_stats.svg`

into:

`results/all_consensus_breadth50_plots/`

The consolidated directory should therefore contain the PNG and SVG mapping statistics plots for **all genomes from `all_consensus_breadth50` that have `breadth3 > 50`**, across all samples.

Do not include plots for genomes that:

* are not part of `all_consensus_breadth50`, or
* have `breadth3 <= 50`.

Please use the existing naming convention where possible and avoid overwriting files from different samples/genomes if there is any possibility of filename collisions.

---

### 3. Add `n_reads_fastq` and `relative_abundance` to individual sample statistics tables

Update the per-sample statistics output tables, such as:

`SRS473742_stats.tsv`

to include these additional columns:

* `n_reads_fastq`
* `relative_abundance`

These columns should be present in the individual sample `.tsv` output and correctly populated for each relevant genome.

`n_reads_fastq` should contain the number of reads in the corresponding FASTQ/sample data.

`relative_abundance` should contain the relative abundance value calculated by the existing workflow/methodology. Do not introduce a new abundance calculation if an appropriate value is already calculated elsewhere in the pipeline; instead, reuse the existing value/source to keep the results consistent.

---

### 4. Filtering and consistency

The filtering logic for the consolidated plots must use the same genome/sample data already used to generate `all_consensus_breadth50`.

The key condition is:

`genome ∈ all_consensus_breadth50 AND breadth3 > 50`

Interpret `breadth3` as **the percentage of the genome covered by at least 3 reads**.

Make sure the filtering is applied consistently and that the plots copied into `all_consensus_breadth50_plots` correspond exactly to the genomes passing this condition.

---

### 5. Preserve the existing pipeline

Please inspect the existing code to determine:

* where `{sample}_mapping_stats.png` is generated;
* where the mapping statistics, including `breadth3`, are calculated;
* where `all_consensus_breadth50` is generated;
* where the individual `{sample}_stats.tsv` files are generated;
* where `n_reads_fastq` and `relative_abundance` are already available, if they are calculated elsewhere.

Implement the changes at the appropriate points in the existing workflow rather than duplicating calculations unnecessarily.

After making the changes, verify that:

1. Every mapping statistics plot is generated as both PNG and SVG.
2. `results/all_consensus_breadth50_plots/` is created automatically.
3. Only genomes satisfying `genome ∈ all_consensus_breadth50` and `breadth3 > 50` have their plots copied into this directory.
4. Both PNG and SVG versions are present for every qualifying plot.
5. Individual sample statistics files such as `SRS473742_stats.tsv` contain `n_reads_fastq` and `relative_abundance`.
6. Existing outputs and functionality remain unchanged except for these requested additions.

