# SCIENTIFIC BRIEF: Bacteriophage codiversification with humans

## Project summary

We have identified a bacteriophage family, provisionally named **calcyviruses**, that is broadly detectable across a collection of human dental calculus metagenomes. Our initial analysis with vContact2 identified this family as very distinct from other Caudoviricetes bacteriophages, and iPhoP predictions indicate the host is the Desulfobulbus oralis pathobiont. Preliminary search across millions of phage genomes from MetaVR and collection of proteins from 34 mammalian species suggests a direct switch from environmental sources to the human oral cavity, as we did not find any signal from these phage sequences in mammals other than Neanderthals and Anatomically Modern Humans. 

The scientific objective is now to move beyond genomic description of the new family and conduct a **phylogenomic analysis of ancient bacteriophages from the last 100,000 years**. The atlas should describe the diversity, persistence, genome structure, gene content, and evolutionary relationships of phage-like phages within this cohort. It should also provide a validated framework for discovering more divergent phage relatives in other assembled metagenomes and public viral-genome collections.

The primary objective is the **scientific answer**, not the software itself. Code should be reproducible, modular, testable, and documented, but implementation decisions should be driven by their ability to answer the biological questions.

---

## Central scientific hypothesis

> phage is not a single invariant genome, but the reference member of a diverse, persistent, and potentially widespread phage population whose members can be recognised at multiple levels of evolutionary divergence.

We expect the phage population to include:

- closely related strains detectable by nucleotide read mapping;
- patient-specific or longitudinal lineages;
- recombinant or mosaic genomes;
- partial or divergent relatives detectable through shared proteins and genome architecture;
- more remote family members that may no longer align well at the nucleotide level but occupy a related region of protein-set embedding space.

---

## Primary scientific questions

### 1. Is phage one phage or a population of related phages?

Determine whether the mapped signal represents:

- a nearly invariant genome present in many samples;
- multiple closely related strains;
- several discrete phage lineages;
- a broader family of mosaic phages sharing only part of the reference genome.

### 2. Does phage persist within individual patients?

For patients with longitudinal samples, determine whether:

- the same lineage persists through time;
- within-patient evolution is detectable;
- lineages are replaced;
- multiple lineages coexist;
- highly similar genomes occur in unrelated patients.

### 3. Which genomic regions are conserved and which are variable?

Identify:

- core and accessory genes;
- conserved genome modules;
- hypervariable loci;
- recombination breakpoints;
- gene gain and loss;
- structural variants;
- candidate host-range and defence-evasion genes.

### 4. Can protein-set embeddings recover remote phage relatives?

Evaluate whether a pretrained Protein Set Transformer or related genome-level protein embedding method can identify phage-family members when nucleotide similarity becomes weak.

### 5. When does AI add information beyond conventional methods?

Benchmark the contribution of protein language models and Protein Set Transformers against:

- nucleotide read mapping;
- whole-genome nucleotide similarity;
- translated protein homology;
- gene-content similarity;
- mean-pooled protein embeddings;
- gene-order and synteny measures.

The project must quantify where AI improves sensitivity or biological resolution, rather than assuming that it does.

---

## Available data

The project directory contains or links to the following resources:

```text
inputs/phage.fasta
inputs/work/MGI/
inputs/ONT_MinION/
inputs/ONT_PromethION/
inputs/Metadata/
```

Expected meanings:

- `inputs/phage.fasta`: the current 45.65-kb phage reference contig.
- `inputs/MGI/no_human`: paired-end MGI short-read metagenomes after human-read removal.
- `inputs/ONT_MinION/no_human`: Oxford Nanopore MinION metagenomes after human-read removal.
- `inputs/ONT_PromethION/no_human`: Oxford Nanopore PromethION metagenomes after human-read removal.
- `inputs/Metadata/`: cohort and sample metadata.

Existing mapping outputs may include:

```text
inputs/vijiphage_mapping_summary.tsv
results/bam/
results/coverage/
results/stats/
```

Do not assume that every optional output exists. Discover inputs explicitly and fail clearly when required data are absent.

### Initial cohort size

The first mapping analysis included:

- 127 MGI paired-end metagenomes;
- 61 ONT MinION metagenomes;
- 10 ONT PromethION metagenomes;
- 198 platform-specific datasets in total.

Many ONT samples have matched MGI samples. Sample identifiers should be normalised carefully, preserving the original identifiers and recording all transformations.

---

## Scope

The initial project should produce two connected scientific products:

1. **phage Population Atlas**
   - reconstruct and compare sample-specific phage genomes;
   - characterise lineages, variants, gene content, recombination, and persistence.

2. **phage Remote-Family Discovery Benchmark**
   - construct a protein-based representation of the phage population;
   - determine whether Protein Set Transformer embeddings improve recovery of divergent or fragmented relatives;
   - create a scalable search strategy for assembled metagenomes and viral catalogues.

---

## Explicit non-goals for the first implementation

Do not initially attempt to:

- train a large foundation model from scratch;
- replace read mapping with a Protein Set Transformer for raw FASTQ screening;
- infer a bacterial host solely from abundance correlations;
- claim active infection or prophage induction from mapping breadth alone;
- classify a candidate as phage-family solely because it is close in one embedding space;
- build a polished public web application;
- optimise every step before obtaining scientifically interpretable results.

A pretrained model should be used where possible. Simpler baselines must be implemented before or alongside more complex AI methods.

---

# Work package 1: Establish a high-confidence phage sample set

## Objective

Define which samples contain sufficient and sufficiently distributed phage-like sequence for genome reconstruction.

## Required analyses

For every platform-specific sample, retain or calculate:

- Q20 primary mapped reads;
- reference breadth at 1×, 5×, and 10×;
- mean depth;
- maximum depth;
- coverage evenness;
- fraction of the reference in anomalously high-coverage peaks;
- number and distribution of uncovered intervals;
- platform;
- patient and sampling date, when available.

Generate per-base coverage profiles for all samples passing a configurable minimum breadth threshold.

## Provisional evidence categories

The thresholds below are working definitions and must remain configurable:

- **Strong**: at least 80% reference breadth at 1×.
- **Probable**: 30–79.9% breadth at 1×.
- **Weak/localised**: greater than 0% but less than 30% breadth.
- **Not detected**: 0% breadth after filtering.

For genome reconstruction, prioritise samples with:

- at least 80% breadth at 5× for short reads; or
- at least 80% breadth at 1× plus informative long reads; or
- compelling complementary evidence from matched platforms.

These are starting criteria, not biological truths.

## Important control

Localised mapping to one conserved phage gene must not be treated as evidence for the complete phage genome. Coverage distribution must be inspected.

## Deliverables

```text
results/sample_qc/vijiphage_sample_evidence.tsv
results/sample_qc/coverage_plots/
results/sample_qc/high_confidence_samples.tsv
```

---

# Work package 2: Recover sample-specific phage genomes

## Objective

Construct the best-supported phage sequence or sequences from each high-confidence sample.

## Recommended strategy

Use a tiered reconstruction workflow.

### A. Read recruitment

For each selected sample:

- extract reads mapping to the current phage reference;
- for paired-end data, include mapped reads and their mates;
- for ONT data, retain complete reads, supplementary alignments, and reads with substantial partial alignment;
- optionally recruit additional reads by iterative mapping to the emerging sample-specific consensus.

Avoid uncontrolled iterative expansion that recruits unrelated phages. Record the criteria at every recruitment round.

### B. Assembly

Evaluate one or more of:

- reference-guided consensus;
- assembly of recruited short reads;
- assembly of recruited long reads;
- hybrid assembly from matched MGI and ONT reads;
- polishing long-read assemblies using matched short reads.

Prefer a scientifically defensible sequence over forcing every sample into a complete circular genome.

### C. Validation

For every reconstructed genome:

- remap all relevant reads;
- calculate breadth, depth, and coverage uniformity;
- inspect unsupported regions;
- report ambiguous bases;
- detect structural inconsistencies;
- compare with the original reference;
- retain assembly provenance.

Do not label a genome complete or circular without explicit supporting evidence.

### D. Multiple haplotypes

Where variant patterns or long reads support multiple phage haplotypes:

- report the evidence;
- attempt haplotype separation only when justified;
- avoid collapsing clearly incompatible variants into an artificial consensus.

## Deliverables

```text
results/genomes/sample_specific_genomes.fasta
results/genomes/genome_manifest.tsv
results/genomes/read_support.tsv
results/genomes/assembly_qc/
results/genomes/rejected_or_partial_candidates.fasta
```

The genome manifest should include at minimum:

```text
genome_id
sample_id
patient_id
sampling_date
platforms_used
assembly_method
length
breadth_after_remapping
mean_depth
ambiguous_bases
circularity_evidence
completeness_status
notes
```

---

# Work package 3: Build the phage Population Atlas

## Objective

Describe the genetic organisation and population structure of reconstructed phage genomes.

## Analyses

### 3.1 Nucleotide relationships

Calculate:

- pairwise ANI or equivalent whole-genome identity;
- aligned fraction;
- SNP and indel distances;
- core-genome alignment;
- phylogenetic or distance-based clustering;
- lineage assignments with uncertainty.

Do not construct a conventional tree without checking whether recombination or genome mosaicism invalidates a simple tree-like interpretation.

### 3.2 Genome variation

Identify:

- SNPs;
- small indels;
- large insertions and deletions;
- inversions or other structural rearrangements;
- variable genome termini;
- recombination breakpoints;
- repeated regions;
- possible terminal repeats.

### 3.3 Gene annotation

Call genes consistently across all genomes and annotate using appropriate phage-focused resources.

At minimum, produce:

- protein sequences;
- coordinates and strands;
- gene families;
- predicted functions;
- confidence or evidence source;
- structural or protein-language-model annotations where available.

Use existing tools such as Phold, PHROGs, Foldseek, ProstT5, or equivalent methods when available and appropriate.

### 3.4 Pangenome

Classify protein families as:

- core;
- near-core;
- shell;
- rare/accessory.

Measure:

- gene presence/absence;
- module conservation;
- synteny;
- gene gain and loss;
- lineage-specific proteins.

### 3.5 Longitudinal population structure

Using patient and date metadata, test:

- whether within-patient genomes are more similar than between-patient genomes;
- persistence versus replacement;
- within-patient evolutionary rate;
- recurrent mutations;
- evidence of multiple simultaneous lineages;
- near-identical genomes shared between patients.

Patient identifiers must be handled consistently and all longitudinal analyses must account for repeated sampling.

## Deliverables

```text
results/atlas/vijiphage_genome_alignment.fasta
results/atlas/pairwise_genome_similarity.tsv
results/atlas/variant_matrix.tsv
results/atlas/proteins.faa
results/atlas/gene_annotations.tsv
results/atlas/gene_presence_absence.tsv
results/atlas/lineage_assignments.tsv
results/atlas/longitudinal_summary.tsv
results/atlas/figures/
```

## Core atlas figures

Generate publication-quality figures for:

1. genome alignment and variable regions;
2. genome map with core and accessory genes;
3. genome-similarity heat map;
4. population network or phylogeny, as appropriate;
5. patient-by-time lineage plot;
6. pangenome or gene-presence/absence plot;
7. recombination or mosaic-block summary;
8. platform and assembly-quality diagnostics.

All figures must be reproducible from scripts or notebooks and exported as both high-resolution PNG and vector SVG/PDF where practical.

---

# Work package 4: Construct phage protein representations

## Objective

Represent each phage-family genome in protein space so that remote relatives can be retrieved despite reduced nucleotide similarity.

## Important principle

A Protein Set Transformer should complement, not replace, nucleotide mapping.

- Raw-read mapping is expected to remain the fastest and most sensitive approach for close phage strains and low-abundance occurrences.
- Protein-set methods require assembled sequence and predicted proteins.
- Their potential advantage is detection of remote, mosaic, or highly diverged relatives.

## Candidate representations

Implement and benchmark at least the following:

### Baseline 1: Protein homology profile

Represent each genome by:

- fraction of phage protein families detected;
- alignment scores;
- coverage of query and target proteins;
- distribution across genome modules.

A candidate should require evidence from multiple modules, not only one common protein.

### Baseline 2: Gene-content vector

Represent each genome using:

- presence/absence or abundance of protein families;
- Jaccard, weighted Jaccard, or cosine similarity;
- optional weighting that downweights common phage proteins.

### Baseline 3: Mean-pooled protein language-model embeddings

For each protein:

- compute a pretrained protein embedding;
- pool protein embeddings to produce a genome representation;
- test mean, median, attention-free weighted mean, and module-aware pooling.

### Model 4: Pretrained Protein Set Transformer

Use a pretrained viral Protein Set Transformer when feasible.

The genome representation should retain, when supported by the model:

- protein embeddings;
- protein order or positional context;
- coding strand;
- contextualised protein embeddings;
- genome-level embedding.

Do not train a large PST from scratch unless the pretrained model is demonstrably unsuitable and enough independent genomes exist to support training.

## phage reference set

The phage search representation should not consist only of one centroid.

Retain:

- individual genome embeddings;
- lineage centroids;
- the global population centroid;
- within-population distance distribution;
- outlier genomes;
- partial-genome embeddings used only for fragment benchmarking.

This allows the search to recognise multiple phage lineages and prevents a single averaged representation from obscuring population structure.

## Deliverables

```text
results/embeddings/protein_embeddings/
results/embeddings/genome_embeddings.tsv
results/embeddings/vijiphage_embedding_index/
results/embeddings/embedding_metadata.tsv
results/embeddings/embedding_visualisations/
```

---

# Work package 5: Benchmark remote-family discovery

## Objective

Determine where protein-based AI becomes more useful than conventional sequence searching.

## Benchmark design

The benchmark must include controlled positives, realistic negatives, and strict prevention of information leakage.

### Positive examples

Use:

- reconstructed phage genomes;
- held-out phage lineages;
- artificially fragmented phage genomes;
- simulated divergent genomes, where scientifically defensible;
- experimentally or manually validated remote relatives discovered during the project.

### Negative examples

Use a broad and difficult negative set containing:

- unrelated phages of similar genome length;
- phages sharing common structural or replication proteins;
- phages infecting the predicted or suspected host;
- bacterial and plasmid contigs where relevant;
- close non-phage neighbours from embedding or protein-search space.

Random unrelated viruses alone are not an adequate negative set.

### Fragmentation benchmark

Fragment complete or near-complete phage genomes to simulate metagenomic contigs.

Test, for example:

- 100%;
- 75%;
- 50%;
- 25%;
- 10%;
- fixed contig sizes such as 5, 10, 20, and 30 kb.

Use multiple random breakpoints and replicates.

Measure performance as a function of:

- retained genome fraction;
- number of predicted proteins;
- which genome modules remain;
- sequence divergence;
- lineage;
- model or search method.

### Divergence benchmark

Where possible, create or identify positives spanning:

- high nucleotide identity;
- moderate nucleotide identity;
- protein-level homology without strong nucleotide similarity;
- conserved genome architecture with substantial protein divergence;
- mosaic genomes sharing only some phage modules.

Avoid unrealistic random mutation models that destroy biologically meaningful protein and genome structure.

### Leakage prevention

Train/validation/test splits must be made by independent genome or lineage clusters, never by random protein fragments from the same genome.

No fragments from a held-out genome may appear in training or threshold calibration.

## Methods to compare

At minimum:

1. nucleotide mapping or nucleotide alignment;
2. whole-genome ANI/aligned fraction;
3. MMseqs2 or equivalent protein homology;
4. gene-content similarity;
5. mean-pooled protein language-model embeddings;
6. pretrained Protein Set Transformer embeddings;
7. an integrated evidence score.

## Metrics

Report:

- sensitivity;
- specificity;
- precision;
- recall;
- F1 score;
- area under the precision–recall curve;
- ROC-AUC where informative;
- top-k retrieval recall;
- rank of the nearest known phage member;
- calibration of similarity scores;
- performance versus fragment length;
- performance versus divergence;
- CPU/GPU time;
- peak memory;
- storage requirements;
- amortised query time after indexing.

For heavily imbalanced searches, prioritise precision–recall metrics over accuracy.

## Key scientific benchmark question

> At what combination of nucleotide divergence, genome completeness, and protein count does the Protein Set Transformer outperform nucleotide and conventional protein searches?

This answer is more important than obtaining the highest possible headline metric.

## Deliverables

```text
results/benchmark/benchmark_dataset_manifest.tsv
results/benchmark/fragmentation_results.tsv
results/benchmark/divergence_results.tsv
results/benchmark/performance_summary.tsv
results/benchmark/runtime_summary.tsv
results/benchmark/figures/
```

---

# Work package 6: Search external metagenomes and viral catalogues

## Objective

Apply the validated search hierarchy to discover the ecological and evolutionary distribution of phage-like viruses outside the initial CF cohort.

## Search hierarchy

Use a tiered approach.

### Tier 1: close relatives

For raw reads or assembled contigs:

- map to the phage population reference panel;
- report breadth, depth, identity, and best-matching lineage.

### Tier 2: protein-family screening

For assembled contigs:

- predict proteins;
- search against phage protein families;
- retain candidates supported by multiple genome modules.

This is intended as a computationally efficient filter.

### Tier 3: remote embedding retrieval

For candidates and precomputed viral catalogues:

- generate or load protein embeddings;
- calculate genome embeddings;
- retrieve nearest neighbours to all phage lineage representatives;
- use an approximate nearest-neighbour index where appropriate.

### Tier 4: biological validation

For every high-priority candidate:

- compare nucleotide similarity;
- compare shared proteins;
- assess synteny;
- inspect genome completeness and viral evidence;
- identify which phage modules are retained;
- remap source reads where available;
- assess whether similarity could arise from one common phage module alone.

## Candidate classification

Use evidence categories such as:

- **phage strain**: strong whole-genome nucleotide relationship.
- **phage lineage member**: strong genome-wide protein and gene-content relationship, with reduced nucleotide similarity.
- **Probable phage-family member**: multi-module protein and embedding support with coherent genome organisation.
- **Module-sharing phage**: shares one or more modules but lacks genome-wide family support.
- **Unresolved candidate**: insufficient or conflicting evidence.
- **Rejected**: similarity explained by common proteins, contamination, or non-viral sequence.

The precise definitions must be developed from benchmark performance.

## Ecological questions

For validated external relatives, record:

- biome;
- geography;
- host or predicted host;
- human body site;
- disease context;
- sampling date;
- genome completeness;
- nearest phage lineage;
- nucleotide and protein similarity;
- retained and variable modules.

The key potential discoveries include:

- restriction to CF or respiratory environments;
- presence in healthy respiratory metagenomes;
- oral, gut, wastewater, or environmental reservoirs;
- association with a particular bacterial host across biomes;
- a globally distributed but previously unrecognised phage family;
- conserved structural modules with rapidly changing host-range genes.

---

# Integrated evidence model

A candidate phage-family score may combine:

```text
PST or genome-embedding similarity
protein-family similarity
gene-content similarity
synteny or module-order conservation
nucleotide similarity
viral-classification confidence
presence of discriminative phage proteins
candidate completeness
```

Start with interpretable rules and simple statistical models.

Do not introduce a complex classifier until:

- baselines exist;
- the benchmark dataset is sufficiently large;
- independent train/test splits are possible;
- the complex model demonstrates measurable improvement.

All components of the integrated score must remain separately visible.

---

# Scientific validity requirements

## Reproducibility

- Every result must be generated from version-controlled code.
- Record software versions, model versions, database versions, and parameters.
- Preserve input manifests.
- Use deterministic seeds where applicable.
- Separate raw data, intermediate data, results, and reports.
- Do not modify source data.
- Cache expensive protein embeddings with checksums and metadata.
- Make interrupted analyses resumable.

## Traceability

Every genome, protein, embedding, and candidate must be traceable back to:

- source sample or database;
- source contig;
- coordinates;
- gene-calling version;
- protein sequence;
- embedding model;
- analysis version.

## Uncertainty

Report uncertainty and conflicting evidence explicitly.

Do not convert continuous similarity into a family label without calibrated thresholds or supporting biological evidence.

## Controls

Include:

- unrelated phages;
- difficult negatives sharing common phage modules;
- synthetic or artificial fragments;
- shuffled protein orders where informative;
- reduced protein sets;
- candidate contigs with one strong but isolated homolog.

## Avoiding circular reasoning

Do not define phage-family membership solely using the same embedding distance that is later reported as evidence of successful family discovery.

Validation must include independent evidence such as protein homology, synteny, whole-genome similarity, source-read support, or consistent genome architecture.

---

# Suggested repository structure

```text
.
├── SCIENTIFIC_BRIEF.md
├── AGENTS.md
├── README.md
├── pyproject.toml
├── config/
│   ├── default.yaml
│   └── cluster.yaml
├── data/
│   ├── manifests/
│   ├── references/
│   └── external/
├── docs/
│   ├── methods.md
│   ├── decisions.md
│   └── data_dictionary.md
├── notebooks/
│   ├── 01_mapping_qc.ipynb
│   ├── 02_population_atlas.ipynb
│   ├── 03_embeddings.ipynb
│   └── 04_benchmark.ipynb
├── src/
│   └── vijiphage_atlas/
│       ├── io.py
│       ├── manifests.py
│       ├── mapping.py
│       ├── coverage.py
│       ├── recruitment.py
│       ├── assembly.py
│       ├── variants.py
│       ├── annotation.py
│       ├── pangenome.py
│       ├── longitudinal.py
│       ├── embeddings.py
│       ├── retrieval.py
│       ├── benchmark.py
│       ├── plotting.py
│       └── cli.py
├── workflows/
│   ├── mapping/
│   ├── assembly/
│   ├── annotation/
│   ├── embeddings/
│   └── benchmark/
├── scripts/
├── tests/
├── results/
│   ├── sample_qc/
│   ├── genomes/
│   ├── atlas/
│   ├── embeddings/
│   ├── benchmark/
│   └── external_search/
└── reports/
```

This is a suggested structure, not a requirement. Avoid unnecessary restructuring if an existing repository already has a clear organisation.

---

# Implementation priorities for Codex

## Priority 1: inspect and document the current state

Before writing substantial new code:

1. inspect the repository and linked data;
2. identify existing mapping outputs and scripts;
3. validate sample naming and matched-platform relationships;
4. produce a concise data inventory;
5. document assumptions and unresolved issues.

## Priority 2: produce scientifically useful intermediate results

The first useful deliverables are:

1. a high-confidence sample list;
2. per-base coverage plots;
3. reconstructed sample-specific genomes for the best-supported samples;
4. a pairwise similarity matrix;
5. a first lineage or population-structure analysis;
6. consistent protein calls and annotations.

Do not postpone scientific analysis while building a general framework.

## Priority 3: establish baselines before AI

Implement:

1. nucleotide similarity;
2. protein homology;
3. gene-content similarity.

Only then add protein language models and the Protein Set Transformer.

## Priority 4: build a rigorous benchmark

The AI component is successful only if it is tested against:

- held-out genomes;
- fragmented genomes;
- difficult negatives;
- conventional methods;
- runtime and resource use.

## Priority 5: search external collections

Do this after thresholds and failure modes are understood.

---

# Milestones

## Milestone 1: data and mapping audit

Completion criteria:

- all sample inputs catalogued;
- sample identifiers normalised;
- mapping outputs validated;
- coverage profiles generated;
- high-confidence samples selected;
- report summarising data quality and reconstruction candidates.

## Milestone 2: initial population atlas

Completion criteria:

- sample-specific genomes recovered for a scientifically useful subset;
- genome QC completed;
- pairwise nucleotide relationships calculated;
- proteins called consistently;
- preliminary pangenome and lineage analysis generated;
- longitudinal examples identified.

## Milestone 3: protein representation atlas

Completion criteria:

- protein embeddings cached;
- genome-level baseline embeddings produced;
- pretrained PST evaluated;
- phage lineage representatives indexed;
- embedding-space visualisations generated.

## Milestone 4: remote-discovery benchmark

Completion criteria:

- benchmark positives and negatives defined;
- fragmentation benchmark complete;
- conventional and AI methods compared;
- calibrated candidate thresholds proposed;
- runtime and accuracy trade-offs documented.

## Milestone 5: external discovery

Completion criteria:

- selected external catalogues searched;
- candidates ranked;
- high-priority candidates independently validated;
- ecological distribution summarised;
- candidate genomes and metadata released internally for review.

---

# Success criteria

The project will be scientifically successful if it can answer most of the following:

1. How many distinct phage lineages occur in the cohort?
2. Are lineages primarily patient-specific or shared?
3. Does phage persist and evolve longitudinally within patients?
4. Which genes and genome modules are conserved or variable?
5. Is there evidence of recombination or mosaic evolution?
6. Which samples contain sufficient support for complete or near-complete genomes?
7. At what evolutionary distance does protein-based AI outperform nucleotide methods?
8. How incomplete can a contig be before PST or protein-set retrieval becomes unreliable?
9. Which proteins contribute most strongly to remote-family recognition?
10. Are phage-family members restricted to CF airways or distributed across other environments?
11. Can remote candidates be validated independently by gene sharing, synteny, and source-read support?
12. Does the final search strategy improve biological discovery enough to justify its computational cost?

---

# Expected scientific outputs

Potential outputs include:

- a phage population-genomics manuscript;
- a curated set of sample-specific phage genomes;
- a phage pangenome and lineage nomenclature;
- a benchmark of nucleotide, protein-homology, protein-language-model, and PST approaches;
- identification of remote phage-family members;
- hypotheses about host range, ecological reservoirs, and variable host-interaction genes;
- a reusable workflow for protein-set discovery of phage families.

The highest-value scientific result would be evidence that phage is the first characterised member of a widespread but previously unrecognised phage family whose members retain related protein architecture despite extensive nucleotide divergence.

---

# Reporting expectations

At the end of each major work package, produce:

- a Markdown report;
- tabular outputs in TSV or Parquet;
- publication-quality figures;
- a reproducible notebook or script;
- a short section distinguishing observations, interpretations, and unresolved questions;
- explicit recommendations for the next scientific step.

Reports should emphasise biological conclusions and evidence. Do not fill reports with implementation detail that belongs in developer documentation.

---

## Out of scope

- Production software
- General-purpose workflow development
- Unrelated exploratory analyses

---

# Final instruction to the coding agent

Treat this as a scientific investigation supported by software development.

When choosing between:

- a sophisticated implementation that delays interpretation; and
- a simpler, validated analysis that answers the biological question,

prefer the simpler validated analysis.

Do not merely execute the proposed plan mechanically. Inspect the data, test assumptions, identify unexpected patterns, and update the analysis when the evidence suggests a better scientific direction.
