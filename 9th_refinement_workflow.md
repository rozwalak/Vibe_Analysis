Review and optimize the existing pipeline for **runtime, I/O efficiency, and resource usage**, but **do not change the quality, resolution, dimensions, formatting, styling, or content of any plots**. Plot outputs must remain visually identical in quality to the current implementation.

**Important: do not implement changes yet.** First inspect the existing workflow and scripts, then propose a concrete optimization plan with the affected files/rules, expected benefit, and any risks or trade-offs. Wait for approval before modifying anything.

Focus the investigation on the following areas, roughly in this priority order:

### 1. Eliminate repeated FASTQ scans

Identify every place where FASTQ files are opened/decompressed merely to count reads or obtain metadata.

The goal should be to calculate read counts **once per sample**, store them in a small persistent metadata/cache file, and reuse those values in:

* `mapping_statistics.py`
* `summarize_project.py`
* other scripts/rules that need read counts.

Do not repeatedly decompress and scan large FASTQ files if the same information is already available.

Consider whether manifest/sample metadata can be precomputed once during DAG construction or as a dedicated metadata step.

### 2. Reduce unnecessary BAM rereading

Trace every complete or substantial BAM pass in the workflow, including:

* `samtools calmd`
* mapping statistics
* `pydamage`
* indexing/validation
* any other BAM-consuming scripts.

Determine which scans are genuinely necessary and which are duplicated.

Where technically safe:

* reuse metadata already calculated during an existing BAM pass;
* avoid recalculating statistics;
* run independent post-BAM analyses concurrently once the final BAM is available;
* minimize unnecessary decompression, sorting, indexing, and validation passes.

Do not compromise correctness.

### 3. Improve sample-level parallelism and resource allocation

Review the Snakemake resource/thread declarations and determine whether the current configuration can cause CPU or memory oversubscription.

Recommend sensible allocations for:

* CPU threads;
* memory;
* temporary disk;
* I/O-heavy operations.

The objective is not simply to assign more threads, but to find the point where overall throughput is maximized.

Consider the difference between:

* sample-level parallelism;
* multithreaded individual tools;
* I/O-bound versus CPU-bound rules.

### 4. Reduce unnecessary intermediate BAM I/O

Investigate why both:

* `sample.panmap.bam`
* `sample.bam`

are retained.

Determine whether both are genuinely required downstream, for debugging, or for reproducibility.

If the first BAM is only an intermediate, propose whether it can safely be removed/compressed/treated as temporary after successful creation of the canonical final BAM.

Do not remove an output without verifying that no downstream rule depends on it.

### 5. Use faster temporary storage

Identify rules that are particularly I/O-heavy, especially:

* BAM sorting;
* BAM indexing;
* `pydamage`;
* plotting;
* compressed FASTQ/BAM processing.

Recommend using Snakemake temporary/shadow directories or configurable local SSD storage where appropriate.

The optimization should avoid relying on network-mounted storage for large temporary/intermediate files when local temporary storage is available.

### 6. Keep plot quality completely unchanged

**Do NOT optimize plotting by reducing DPI, resolution, image dimensions, SVG quality, font quality, or any other visual quality parameter.**

Do not suggest lower-quality PNG/SVG output as a performance optimization.

Instead investigate:

* avoiding unnecessary plot regeneration;
* correct Snakemake dependencies;
* generating plots only when their inputs have actually changed;
* separating expensive reporting from computational processing if appropriate;
* skipping plotting only for genuinely invalid/empty samples where this is already logically acceptable.

Existing plot appearance and quality must remain unchanged.

### 7. Make summary generation incremental

Inspect how project/global summary files are generated.

Recommend a design where unchanged sample/project results do not cause unnecessary recomputation.

Consider:

* precise Snakemake input/output dependencies;
* timestamps/checksums where appropriate;
* separating per-sample summary generation from project-level aggregation;
* aggregating only projects whose underlying inputs changed.

The final summary must remain deterministic and identical in content and ordering, apart from the explicit column-order correction described below.

### 8. Precompute and cache manifest metadata

Investigate whether the workflow repeatedly parses the same sample sheet/manifest or repeatedly derives:

* sample layout;
* FASTQ paths;
* read counts;
* file sizes;
* compressed-file metadata;
* validation information.

If so, propose a persistent metadata/cache mechanism so this work happens once and is reused.

### 9. Reduce Conda/environment startup overhead

Review the number of Conda environments and the size/frequency of small rules.

Consider whether compatible utility/reporting scripts could share an environment without creating dependency conflicts.

The goal is to reduce startup overhead for many small jobs, while preserving reproducibility.

### 10. Simplify unnecessary process nesting

Inspect `run_safe_command.py` and determine where it adds meaningful fault isolation versus unnecessary process overhead.

In particular, look for chains such as:

Snakemake shell → Python wrapper → shell command → Python script → external tool

Where safe, recommend using Snakemake's native:

* logging;
* benchmarking;
* error handling;
* shadow/temp directories;
* resource management.

Keep `run_safe_command.py` where it provides important sample-level fault containment, but avoid unnecessary wrapper layers for trivial operations.

### 11. Add benchmarking before making major optimizations

Recommend adding Snakemake benchmark outputs to the major expensive rules.

Capture, where available:

* wall-clock time;
* CPU usage/time;
* maximum memory;
* number of threads;
* input/output sizes;
* compression-related time;
* potentially I/O characteristics.

Use these measurements to identify the actual bottleneck before making invasive changes.

The likely candidates to compare are:

* panmap;
* FASTQ decompression/scanning;
* BAM processing;
* `pydamage`;
* plotting;
* filesystem/network I/O.

### 12. Improve reuse/caching across repeated runs

For identical inputs and parameters, investigate whether the workflow can reliably preserve and reuse:

* panmap indexes;
* successful per-sample outputs;
* expensive intermediate computation results;
* report-independent computational outputs.

Avoid deleting valid outputs unnecessarily during partial reruns.

Consider stable parameter/input fingerprints where useful so that expensive calculations are rerun only when their actual dependencies change.

---

## Summary-file column ordering requirement

There is also a **current output-format issue that must be corrected**.

In the current summary files, the `pydamage` result columns appear in the middle of the other mapping/statistics columns.

Change the summary column ordering so that **all pydamage-related result columns appear at the end of the summary statistics, immediately after `breadth10`**.

The intended ordering is therefore conceptually:

`... [other existing summary columns] → breadth10 → [all pydamage result columns]`

Do not otherwise reorder, rename, remove, or alter summary columns unless necessary for the performance changes.

The pydamage values themselves, calculations, formatting, and meaning must remain unchanged. Only their position in the output columns should change.

## Also fix the rerun CSV naming convention:

* The current `csv_to_rerun.csv` naming is too generic.
* It should be project-specific and named **`{project}_to_rerun.csv`**.
* Every rule/script/path that creates, reads, references, or expects `csv_to_rerun.csv` should be inspected and updated consistently.
* The `{project}` value must come from the existing project identifier used by the workflow; do not invent a second naming convention.
* Make sure different projects cannot overwrite each other's rerun CSV files.
* Do not otherwise change the contents or semantics of the rerun CSV.
---

### Constraints

* **Do not change plot quality.**
* Do not reduce PNG DPI.
* Do not reduce SVG quality.
* Do not change plot dimensions or styling.
* Do not sacrifice correctness for speed.
* Do not remove outputs until all dependencies have been verified.
* Preserve reproducibility.
* Preserve existing scientific calculations and result values.
* Preserve the existing summary information.
* Only move pydamage columns to the end, after `breadth10`.
* Prefer optimizations that reduce redundant computation/I/O rather than reducing analytical quality.
* Do not implement anything yet.

### What I want from you now

First inspect the complete workflow and identify the actual bottlenecks and redundant operations.

Then provide:

1. A ranked list of proposed optimizations by expected speedup.
2. Which files/rules/scripts would be affected by each optimization.
3. An estimate of whether each optimization is likely to have **high, medium, or low** impact.
4. Any risks to correctness or reproducibility.
5. A proposed implementation order.
6. Specifically identify every place where FASTQ/BAM files are currently reread.
7. Explain how the summary-column ordering should be changed so that pydamage follows `breadth10`.

**Do not modify files yet. Wait for my approval after presenting the plan.**
