# ANALYSIS PLAN for this Project

## Purpose

This document translates `SCIENTIFIC_BRIEF.md` into an executable analysis plan for Codex.

The objective is to generate defensible scientific results about the diversity, persistence, genome structure, and broader distribution of phage-like phages. The plan deliberately prioritises interpretable biological outputs over software complexity.

Codex should work through the phases in order, but it should pause at the defined decision gates when results indicate that assumptions, thresholds, or methods need revision.

---

# 1. Guiding principles

1. **The scientific answer is the primary objective.**
   Code, workflows, tests, and documentation exist to support the analysis.

2. **Do not skip simple baselines.**
   Nucleotide mapping, protein homology, and gene-content similarity must be established before interpreting Protein Set Transformer results.

3. **Do not force complete genomes.**
   Partial, ambiguous, or mixed reconstructions should remain explicitly labelled as such.

4. **Do not treat one similarity score as proof of family membership.**
   Candidate relatives require multiple independent lines of evidence.

5. **Preserve provenance.**
   Every result must be traceable to the source sample, read set, contig, protein, model, database, and command.

6. **Avoid silent assumptions.**
   All thresholds and sample-name transformations must be recorded in configuration and reports.

7. **Make expensive analyses resumable.**
   Protein embeddings, assemblies, annotations, and external database searches must be cached safely.

8. **Separate observations from interpretations.**
   Reports must distinguish measured results, inferred explanations, and unresolved alternatives.

---

# 2. Immediate starting state

The project directory is expected to contain:

```text
inputs/phage.fasta
SCIENTIFIC_BRIEF.md
```

Potential existing outputs include:

```text
results/summary.tsv
results/bam/
results/coverage/
results/stats/
```

The linked read directories initially contained:

```
TODO: summarise the data here and replace this TODO
```

Do not assume that counts or paths remain unchanged. Reinventory them at the start of the analysis. If essential files are missing please report them and stop.



---

# 3. High-level workflow

```text
Data audit
    ↓
Mapping and coverage validation
    ↓
High-confidence sample selection
    ↓
Read recruitment and sample-specific reconstruction
    ↓
Genome QC and remapping
    ↓
Population structure, variants, pangenome, and longitudinal analysis
    ↓
Protein representations and embeddings
    ↓
Fragmentation and divergence benchmark
    ↓
External catalogue search
    ↓
Independent validation of candidate relatives
    ↓
Scientific synthesis
```

---

# 4. Phase 0: Repository and data audit

## Objective

Establish the exact inputs, existing analyses, computing environment, and unresolved data issues before substantial new computation.

## Tasks

### 4.1 Inspect the repository

Record:

- directory tree to a reasonable depth;
- existing scripts and notebooks;
- existing results and reports;
- configuration files;
- software environments;
- workflow definitions;
- symbolic links and their resolved targets.

Do not move or rename existing data during the audit.

### 4.2 Validate the reference

For `phage.fasta`, record:

- sequence identifier;
- number of records;
- sequence length;
- GC content;
- ambiguous-base count;
- whether the sequence appears circularised or contains terminal duplication;
- checksum.

Fail clearly if more than one record exists unless this is intentional and documented.

### 4.3 Build a canonical sample manifest

Generate a manifest containing at minimum:

```text
dataset_id
sample_id_original
sample_id_normalised
patient_id
sampling_date
platform
read1
read2
source_directory
file_size
checksum_or_fast_fingerprint
matched_mgi_sample
matched_minion_sample
matched_promethion_sample
metadata_status
notes
```

Requirements:

- preserve original sample identifiers;
- document every normalisation rule;
- detect duplicate identifiers;
- detect incomplete MGI pairs;
- verify that referenced files exist and are readable;
- do not infer patient identifiers from filenames without documenting the parsing rule;
- cross-check against metadata.

### 4.4 Audit existing mapping outputs

If mapping outputs exist:

- compare the number of BAMs and statistics files with the manifest;
- verify that BAMs are readable and indexed;
- verify reference sequence names;
- recalculate selected metrics independently for a small test set;
- check for malformed rows or column shifts in summary tables;
- compare sample identifiers across outputs.

### 4.5 Record computing resources

Document:

- cluster scheduler and partitions;
- available CPU and memory;
- available GPUs and accelerator type;
- installed software;
- container or conda support;
- scratch-space expectations;
- project storage limits.

## Outputs

```text
reports/00_data_audit.md
data/manifests/samples.tsv
data/manifests/sample_name_mapping.tsv
data/manifests/input_checksums.tsv
config/environment_inventory.txt
config/software_versions.tsv
```

## Decision gate 0

Proceed only when:

- the reference is validated;
- sample identities are unambiguous enough for platform matching;
- incomplete or duplicated inputs are explicitly resolved or excluded;
- existing mapping outputs are either validated or scheduled for regeneration.

If these conditions are not met, stop and report the blocking issues.

---

# 5. Phase 1: Mapping validation and sample evidence

## Objective

Create a trustworthy sample-by-sample evidence table for phage presence and reconstruction potential.

## Inputs

- `phage.fasta`;
- raw MGI, MinION, and PromethION reads;
- existing BAMs where validated;
- canonical sample manifest.

## Tasks

### 5.1 Standardise mapping parameters

Use a documented reference-mapping strategy:

- MGI paired-end reads: a short-read preset;
- ONT reads: an ONT long-read preset appropriate to the basecall quality;
- suppress secondary mappings for summary statistics;
- retain supplementary alignments in BAMs for later structural and junction analysis;
- apply mapping-quality and base-quality filters consistently.

Record exact commands and versions.

### 5.2 Calculate mapping metrics

For each platform-specific dataset calculate:

```text
primary_mapped
primary_mapped_q20
reference_length
mean_depth
median_depth
maximum_depth
breadth_1x_pct
breadth_5x_pct
breadth_10x_pct
breadth_20x_pct
coverage_cv
coverage_gini_or_equivalent
longest_uncovered_interval
number_uncovered_intervals
fraction_in_high_coverage_peaks
```

For ONT additionally calculate:

```text
longest_aligned_read
number_reads_covering_25pct_reference
number_reads_covering_50pct_reference
number_reads_covering_80pct_reference
number_near_full_length_reads
number_supplementary_alignments
```

### 5.3 Generate coverage profiles

Create per-base coverage files and plots.

Plots should show:

- coordinate along phage;
- depth;
- optional log-scaled depth;
- annotated genes once annotation is available;
- uncovered regions;
- high-coverage peaks;
- platform and sample metadata.

Generate individual plots for high-priority samples and a cohort-level coverage heat map.

### 5.4 Detect localised false-positive patterns

Flag samples where mapping is dominated by:

- one short conserved region;
- repeated or low-complexity sequence;
- a single high-coverage gene;
- highly soft-clipped alignments;
- alignments with poor identity;
- discordance between breadth and mapped-read count.

### 5.5 Define evidence classes

Use configurable working classes:

```text
strong
probable
weak_localised
not_detected
```

Also calculate a separate reconstruction-priority score based on:

- breadth;
- depth;
- evenness;
- matched-platform evidence;
- presence of informative long reads;
- absence of severe localised mapping.

Do not conflate detection class with reconstruction suitability.

## Outputs

```text
results/sample_qc/vijiphage_sample_evidence.tsv
results/sample_qc/reconstruction_priority.tsv
results/sample_qc/per_base_coverage/
results/sample_qc/coverage_plots/
results/sample_qc/coverage_heatmaps/
reports/01_mapping_and_sample_evidence.md
```

## Required figures

1. detection categories by platform;
2. breadth distribution by platform;
3. breadth at 1×, 5×, 10×, and 20×;
4. breadth versus mean depth;
5. matched-platform breadth comparison;
6. cohort coverage heat map;
7. representative strong, shallow, localised, and absent coverage profiles.

## Decision gate 1

Proceed to reconstruction when:

- mapping metrics are internally consistent;
- high-confidence samples are identified;
- localised mapping artefacts have been flagged;
- at least a scientifically useful subset has sufficient support for reconstruction.

If nearly all samples show identical complete coverage, first test whether reads are cross-contaminated, duplicated, or incorrectly assigned.

---

# 6. Phase 2: Pilot sample-specific reconstruction

## Objective

Determine the most reliable reconstruction strategy before scaling to the full cohort.

## Pilot sample selection

Select a deliberately diverse pilot set, for example:

- 5 high-depth MGI-only or effectively MGI-dominant samples;
- 5 matched MGI–MinION samples;
- all suitable matched MGI–PromethION samples, up to 5;
- 2–3 probable or heterogeneous samples;
- 1–2 localised-mapping negative controls.

Include longitudinal samples from at least two patients where possible.

## Candidate reconstruction methods

Evaluate:

### Method A: reference-guided consensus

Best suited to close phage strains with broad coverage.

### Method B: recruited short-read assembly

Extract mapped reads and mates, then assemble independently of the reference.

### Method C: recruited ONT assembly

Use informative long reads, retaining structural information.

### Method D: hybrid reconstruction

Combine ONT structure with MGI polishing for matched samples.

### Method E: iterative recruitment

Use only with strict controls. At each round:

- record newly recruited reads;
- quantify expansion in sequence space;
- stop if recruitment becomes unstable or pulls unrelated phages.

## Reconstruction validation

For every candidate reconstruction:

- remap original reads;
- calculate breadth, depth, and evenness;
- calculate reference identity and aligned fraction;
- inspect unsupported sequence;
- inspect assembly graph where available;
- detect duplicated ends;
- detect structural conflicts;
- compare with long-read evidence;
- record ambiguous bases;
- identify mixed-variant positions.

## Pilot comparison metrics

Compare methods using:

```text
assembly_length
number_contigs
ambiguous_bases
read_remapping_rate
breadth_after_remapping
coverage_evenness
unsupported_bases
structural_conflicts
agreement_with_long_reads
agreement_between_methods
runtime
memory
```

## Outputs

```text
results/pilot_reconstruction/
reports/02_reconstruction_pilot.md
```

## Decision gate 2

Choose the production reconstruction strategy separately for:

- MGI-only samples;
- matched MGI–MinION samples;
- matched MGI–PromethION samples;
- partial or mixed samples.

Do not impose one method on all samples if platform-specific strategies perform better.

---

# 7. Phase 3: Production reconstruction

## Objective

Recover the best-supported phage genome or partial genome from every suitable sample.

## Tasks

### 7.1 Run the selected workflow

For each high-priority sample:

- recruit reads;
- assemble or derive consensus;
- polish where appropriate;
- remap reads;
- calculate QC;
- retain logs and intermediate provenance.

### 7.2 Detect mixed populations

At variable sites calculate:

- allele frequencies;
- strand balance;
- base quality;
- platform concordance;
- linkage on long reads;
- co-occurring variant blocks.

Flag samples with:

- numerous intermediate-frequency variants;
- mutually incompatible long-read haplotypes;
- structural alternatives;
- evidence of more than one lineage.

Do not report a single consensus as biologically representative when a mixed population is strongly supported.

### 7.3 Standardise orientation and coordinates

Choose a reproducible coordinate system.

Options include:

- alignment to the original phage reference;
- rotation to a conserved marker gene;
- rotation to an inferred terminus;
- no rotation when circularity is uncertain.

Record any reverse-complementing or rotation.

### 7.4 Classify reconstruction status

Use explicit categories:

```text
complete_supported
near_complete
partial_high_confidence
mixed_population
ambiguous
failed
```

## Outputs

```text
results/genomes/sample_specific_genomes.fasta
results/genomes/partial_genomes.fasta
results/genomes/genome_manifest.tsv
results/genomes/read_support.tsv
results/genomes/variant_evidence/
results/genomes/assembly_qc/
reports/03_genome_reconstruction.md
```

## Decision gate 3

Proceed to the population atlas when:

- reconstructed sequences have passed remapping QC;
- failures and ambiguous samples are retained but excluded appropriately;
- coordinate normalisation is consistent;
- enough independent genomes exist to support population analysis.

---

# 8. Phase 4: phage population structure

## Objective

Determine whether phage comprises one strain, multiple lineages, or a broader mosaic population.

## Tasks

### 8.1 Whole-genome similarity

Calculate:

- pairwise ANI;
- aligned fraction;
- nucleotide distance;
- SNP distance on shared aligned regions;
- length differences;
- structural difference summaries.

Retain both identity and aligned fraction. High identity over a small shared segment is not equivalent to whole-genome relatedness.

### 8.2 Core alignment

Construct a core-genome alignment for genomes that meet a defined shared-region threshold.

Check:

- alignment quality;
- recombination;
- missing data;
- reference bias;
- duplicated regions.

### 8.3 Population graph and lineage definitions

Compare:

- hierarchical clustering;
- network representation;
- phylogenetic inference;
- recombination-aware analysis;
- pangenome-based clustering.

Do not force tree-like lineages if the genomes are strongly mosaic.

Lineage thresholds must be justified from the empirical distance distribution, not chosen solely for convenience.

### 8.4 Variant catalogue

Create a variant table containing:

```text
variant_id
reference_coordinate
variant_type
reference_allele
alternate_allele
affected_gene
predicted_effect
samples
patients
lineages
allele_frequency
long_read_support
```

### 8.5 Structural variation

Identify:

- insertions;
- deletions;
- inversions;
- duplications;
- variable termini;
- module replacements;
- recombination blocks.

### 8.6 Transmission-like clusters

Search for near-identical genomes in different patients.

Treat these as hypotheses requiring careful evaluation. Consider:

- sequencing batch;
- contamination;
- sampling dates;
- common laboratory processing;
- common environmental source;
- genuine transmission.

Do not use the word “transmission” without ruling out simpler explanations.

## Outputs

```text
results/atlas/pairwise_ani.tsv
results/atlas/aligned_fraction.tsv
results/atlas/core_alignment.fasta
results/atlas/genome_distance_matrix.tsv
results/atlas/lineage_assignments.tsv
results/atlas/variant_catalogue.tsv
results/atlas/structural_variants.tsv
results/atlas/recombination_blocks.tsv
results/atlas/figures/
reports/04_population_structure.md
```

## Required figures

1. pairwise ANI and aligned-fraction heat maps;
2. population network or justified phylogeny;
3. lineage composition by patient;
4. genome-length and structural-variation summary;
5. reference-coordinate variant density;
6. recombination or mosaic-block plot;
7. near-identical cross-patient genome investigation.

---

# 9. Phase 5: Longitudinal analysis

## Objective

Determine whether phage persists, evolves, disappears, or is replaced within patients.

## Tasks

### 9.1 Build a patient timeline

For each patient, plot:

- sampling dates;
- phage detection strength;
- lineage;
- genome reconstruction status;
- abundance proxy;
- within-sample diversity;
- important genotype changes.

### 9.2 Classify longitudinal patterns

Use evidence-based categories:

```text
persistent_same_lineage
persistent_evolving_lineage
lineage_replacement
intermittent_detection
multiple_lineages
insufficient_data
```

### 9.3 Within-patient versus between-patient distances

Test whether genomes from the same patient are more similar than genomes from different patients.

Use statistical methods that account for:

- repeated measures;
- unequal numbers of samples;
- temporal separation;
- non-independence.

### 9.4 Evolutionary rate

Estimate within-patient change only where:

- sampling dates are reliable;
- the same lineage persists;
- sequence quality is sufficient;
- mixed populations do not invalidate a simple consensus.

Report uncertainty and avoid overinterpreting short time spans.

### 9.5 Recurrent changes

Identify mutations or gene changes occurring independently in multiple patients, especially in:

- receptor-binding proteins;
- tail fibres;
- anti-defence proteins;
- integrase or excision functions;
- transcriptional regulators;
- lysis genes.

## Outputs

```text
results/longitudinal/patient_timelines.tsv
results/longitudinal/longitudinal_classification.tsv
results/longitudinal/within_between_distances.tsv
results/longitudinal/recurrent_changes.tsv
results/longitudinal/figures/
reports/05_longitudinal_population_dynamics.md
```

## Decision gate 5

Before claiming persistence or replacement, verify that sample identity, platform effects, coverage, and assembly quality do not explain the pattern.

---

# 10. Phase 6: Gene annotation and pangenome

## Objective

Characterise phage gene content, conserved modules, accessory genes, and candidate host-interaction functions.

## Tasks

### 10.1 Consistent gene calling

Run one primary gene-calling method consistently across all genomes.

Retain:

- nucleotide CDS;
- amino-acid sequence;
- coordinates;
- strand;
- partial-gene status;
- translation table;
- gene-caller version.

### 10.2 Functional annotation

Use phage-focused evidence where possible:

- sequence homology;
- profile HMMs;
- phage protein-family databases;
- structure similarity;
- protein language-model annotation;
- gene-neighbourhood context.

Record evidence separately rather than collapsing everything into one unsupported label.

### 10.3 Protein families

Cluster proteins using a documented approach.

Explore multiple clustering stringencies if family boundaries are unstable.

### 10.4 Pangenome classification

Define:

```text
core
near_core
shell
rare
singleton
```

Thresholds must be configurable and reported.

### 10.5 Module analysis

Identify coherent genome modules such as:

- DNA replication;
- packaging;
- head morphogenesis;
- tail morphogenesis;
- host recognition;
- lysogeny;
- lysis;
- defence and counter-defence;
- transcriptional regulation.

### 10.6 Variable gene prioritisation

Rank proteins based on:

- lineage association;
- patient association;
- recurrent mutation;
- gene gain or loss;
- location in recombination blocks;
- predicted host-interaction function;
- structural novelty;
- contribution to embedding similarity.

## Outputs

```text
results/pangenome/proteins.faa
results/pangenome/cds.fna
results/pangenome/gene_annotations.tsv
results/pangenome/protein_families.tsv
results/pangenome/gene_presence_absence.tsv
results/pangenome/module_assignments.tsv
results/pangenome/priority_variable_genes.tsv
results/pangenome/figures/
reports/06_gene_content_and_pangenome.md
```

## Required figures

1. comparative genome maps;
2. gene-presence/absence heat map;
3. core/accessory accumulation plot;
4. lineage-associated genes;
5. variable-module and recombination plot;
6. ranked candidate host-interaction proteins.

---

# 11. Phase 7: Protein representation baselines

## Objective

Create interpretable protein-level representations before applying a Protein Set Transformer.

## Tasks

### 11.1 Protein homology baseline

For each genome or candidate contig calculate:

- number of phage protein-family matches;
- fraction of phage families recovered;
- fraction of candidate proteins matched;
- query and target coverage;
- sequence identity;
- bit score;
- number of represented genome modules.

### 11.2 Gene-content baseline

Represent genomes using:

- binary family presence/absence;
- family copy number;
- weighted presence/absence;
- module-level features.

Calculate:

- Jaccard similarity;
- weighted Jaccard;
- cosine similarity;
- shared-family count.

### 11.3 Mean-pooled protein embeddings

Using a documented pretrained protein language model:

- generate one embedding per protein;
- cache embeddings;
- test mean pooling;
- test median or robust pooling;
- test length-weighted pooling;
- test inverse-family-frequency weighting;
- test module-aware pooling.

### 11.4 Embedding QC

Check:

- duplicate sequences;
- truncated proteins;
- extreme lengths;
- inconsistent model versions;
- numerical instability;
- storage format;
- reproducibility.

## Outputs

```text
results/representations/protein_homology_features.tsv
results/representations/gene_content_vectors/
results/representations/protein_embeddings/
results/representations/pooled_genome_embeddings.tsv
reports/07_protein_representation_baselines.md
```

---

# 12. Phase 8: Pretrained Protein Set Transformer

## Objective

Determine whether a pretrained Protein Set Transformer provides useful genome-level representations of phage diversity and remote relationships.

## Tasks

### 12.1 Confirm model compatibility

Document:

- exact model and checkpoint;
- expected protein embedding model;
- positional encoding requirements;
- strand requirements;
- maximum proteins or genome length;
- handling of fragmented genomes;
- licensing and redistribution restrictions.

### 12.2 Generate PST inputs

For every genome retain:

```text
genome_id
protein_id
protein_sequence_checksum
start
end
strand
order_index
source_contig
fragment_id
protein_embedding_path
```

### 12.3 Generate embeddings

Produce:

- contextualised protein embeddings, if available;
- genome-level embeddings;
- fragment-level embeddings;
- lineage centroids;
- global population centroid.

### 12.4 Explore embedding structure

Use unsupervised visualisations carefully:

- PCA;
- UMAP;
- nearest-neighbour graphs;
- lineage centroids;
- distance distributions.

UMAP and similar projections are exploratory only. All quantitative conclusions must use distances in the original embedding space.

### 12.5 Interpretability analysis

Investigate which proteins contribute to similarity or separation where the model permits.

Potential approaches:

- leave-one-protein-out analysis;
- module ablation;
- protein masking;
- attention inspection, if scientifically interpretable;
- comparison of contextualised protein embeddings;
- gradient- or perturbation-based contribution measures.

## Outputs

```text
results/pst/genome_embeddings.tsv
results/pst/contextual_protein_embeddings/
results/pst/lineage_centroids.tsv
results/pst/nearest_neighbours.tsv
results/pst/protein_contribution_analysis.tsv
results/pst/figures/
reports/08_protein_set_transformer.md
```

## Decision gate 8

Continue to large-scale PST searches only if:

- embeddings are reproducible;
- known phage genomes cluster meaningfully;
- the model is not driven solely by genome length or protein count;
- performance is compared with simpler baselines;
- computational costs are documented.

---

# 13. Phase 9: Fragmentation benchmark

## Objective

Measure how robust each method is to incomplete metagenomic contigs.

## Experimental design

For every complete or near-complete phage genome:

### 13.1 Proportional fragments

Generate fragments retaining:

```text
100%
75%
50%
25%
10%
```

Use multiple breakpoints and replicates.

### 13.2 Fixed-length fragments

Generate, where possible:

```text
5 kb
10 kb
15 kb
20 kb
30 kb
```

### 13.3 Module-aware fragments

Generate fragments containing:

- structural module only;
- replication module only;
- lysogeny region only;
- host-recognition region only;
- combinations of modules.

### 13.4 Re-call genes

For nucleotide fragments, rerun gene calling rather than merely selecting proteins from the complete genome. This captures realistic partial-gene effects.

### 13.5 Methods compared

Evaluate:

- nucleotide alignment;
- protein homology;
- gene-content similarity;
- pooled protein embeddings;
- PST embeddings;
- integrated evidence score.

### 13.6 Metrics

For each fragment record:

```text
source_genome
lineage
fragment_start
fragment_end
fragment_length
retained_fraction
number_complete_proteins
number_partial_proteins
modules_retained
method
similarity_score
nearest_neighbour
nearest_neighbour_lineage
rank_of_true_family
classification
```

## Outputs

```text
results/benchmark/fragment_manifest.tsv
results/benchmark/fragmentation_results.tsv
results/benchmark/fragmentation_summary.tsv
results/benchmark/fragmentation_figures/
reports/09_fragmentation_benchmark.md
```

## Required figures

1. recall versus retained genome fraction;
2. recall versus fragment length;
3. recall versus number of proteins;
4. method comparison by retained module;
5. false-positive rate for one-module fragments;
6. nearest-neighbour rank distributions.

---

# 14. Phase 10: Divergence and difficult-negative benchmark

## Objective

Determine where AI-based protein representations outperform conventional sequence methods.

## Positive sets

Use:

- held-out phage genomes;
- held-out lineages;
- external validated relatives;
- biologically plausible divergent relatives discovered through protein and synteny searches.

Avoid treating simple random nucleotide mutation as the main divergence benchmark.

## Difficult negatives

Include:

- phages of similar genome size;
- phages sharing common morphogenesis genes;
- phages sharing replication modules;
- phages infecting the same predicted host;
- embedding-space nearest neighbours outside the family;
- plasmids or bacterial elements with phage-like modules;
- prophage fragments;
- chimeric or contaminated contigs where available.

## Leakage prevention

Partition by genome cluster or lineage before:

- threshold selection;
- model fitting;
- feature weighting;
- performance evaluation.

Fragments from one genome must remain in the same partition.

## Metrics

Prioritise:

- precision–recall AUC;
- recall at fixed precision;
- top-k retrieval recall;
- family rank;
- calibration;
- false positives by negative category;
- performance versus aligned nucleotide fraction;
- performance versus shared protein fraction.

## Integrated score

Begin with an interpretable score or shallow model using features such as:

```text
nucleotide_aligned_fraction
nucleotide_identity
shared_protein_fraction
module_count
gene_content_similarity
pooled_embedding_distance
pst_distance
candidate_length
protein_count
viral_confidence
```

Only use a more complex model if it improves held-out performance materially.

## Outputs

```text
results/benchmark/positive_manifest.tsv
results/benchmark/negative_manifest.tsv
results/benchmark/divergence_results.tsv
results/benchmark/difficult_negative_results.tsv
results/benchmark/model_comparison.tsv
results/benchmark/calibration.tsv
results/benchmark/runtime_and_memory.tsv
results/benchmark/figures/
reports/10_remote_family_benchmark.md
```

## Decision gate 10

Define operational family-search thresholds only after reviewing:

- held-out precision;
- failure modes;
- fragment sensitivity;
- module-sharing false positives;
- runtime.

A threshold that performs well only on easy random negatives is unacceptable.

---

# 15. Phase 11: External search

## Objective

Determine whether phage-like phages occur outside the original CF cohort.

## Search order

### Stage A: local and existing assemblies

Search:

- existing CF assemblies;
- bronchiectasis assemblies;
- laboratory viral-contig collections;
- related respiratory datasets already available locally.

### Stage B: curated viral catalogues

Search suitable public or locally mirrored viral catalogues.

### Stage C: selected metagenome assemblies

Prioritise:

- respiratory metagenomes;
- oral metagenomes;
- gut metagenomes;
- wastewater;
- built environment;
- environmental reservoirs related to the predicted host.

Do not begin with indiscriminate raw-read downloads.

## Tiered search

### Tier 1: nucleotide search

Search for close strains and known lineages.

### Tier 2: protein-family screen

Require evidence from multiple independent modules.

### Tier 3: embedding retrieval

Query against:

- individual phage genome embeddings;
- lineage centroids;
- the population centroid.

### Tier 4: independent validation

For each candidate:

- confirm viral nature;
- calculate nucleotide relationship;
- calculate protein-family relationship;
- inspect synteny;
- inspect genome completeness;
- map source reads if available;
- assess contamination or chimerism;
- classify retained modules.

## Candidate evidence table

Create one row per candidate with:

```text
candidate_id
source_database
source_sample
biome
geography
host_metadata
contig_length
viral_confidence
estimated_completeness
nucleotide_identity
nucleotide_aligned_fraction
shared_protein_fraction
shared_modules
gene_content_similarity
pooled_embedding_distance
pst_distance
best_vijiphage_lineage
source_read_support
candidate_class
manual_review_status
notes
```

## Outputs

```text
results/external_search/candidates.tsv
results/external_search/validated_candidates.fasta
results/external_search/rejected_candidates.tsv
results/external_search/candidate_proteins.faa
results/external_search/source_metadata.tsv
results/external_search/figures/
reports/11_external_distribution.md
```

---

# 16. Phase 12: Optional host and biological-state analysis

This phase is scientifically valuable but should not delay the core population atlas.

## Potential analyses

### 16.1 ONT junction search

Identify reads containing:

```text
bacterial sequence — phage sequence
```

Validate:

- split alignment;
- breakpoint consistency;
- independent reads;
- host assignment;
- integration boundaries.

### 16.2 CRISPR spacer search

Search bacterial assemblies or MAGs for spacers matching phage.

### 16.3 Host prediction

Use a multi-evidence approach including:

- direct junctions;
- CRISPR;
- host-prediction tools;
- composition;
- cohort-specific co-occurrence.

### 16.4 Biological state

Explore whether samples support:

```text
integrated
induced
free_phage
mixed
uncertain
```

Use:

- phage-to-host depth ratio;
- junction reads;
- circular or terminal evidence;
- free phage coverage;
- host population carrying the prophage.

Do not infer biological state from phage depth alone.

## Outputs

```text
results/host/junction_reads.tsv
results/host/integration_sites.tsv
results/host/crispr_matches.tsv
results/host/host_predictions.tsv
results/host/biological_state.tsv
reports/12_host_and_state.md
```

---

# 17. Statistical analysis principles

## Repeated measures

Samples from the same patient are not independent.

Use:

- mixed-effects models;
- patient-blocked permutation;
- grouped cross-validation;
- patient-level bootstrap;
- other appropriate repeated-measures methods.

## Platform effects

Do not compare raw mapped reads or depth between MGI and ONT as direct abundance measures without normalisation.

Where total sequence yield is available, calculate measures such as:

```text
mapped phage bases per million non-human sequenced bases
```

Keep platform-specific analyses visible even after normalisation.

## Multiple testing

For gene-, variant-, or metadata-wide tests:

- report the number of tests;
- apply false-discovery-rate control;
- report effect sizes and confidence intervals;
- avoid focusing only on adjusted P values.

## Train/test separation

For machine-learning evaluation, split by:

- patient for cohort prediction;
- genome or lineage cluster for family discovery.

Never split random fragments from the same underlying genome across training and test sets.

---

# 18. Workflow and software requirements

## Preferred workflow properties

The workflow should be:

- resumable;
- cluster-aware;
- deterministic where practical;
- modular;
- configurable;
- explicit about resource use;
- safe with symbolic links;
- able to process one sample or the full cohort.

Snakemake, Nextflow, or a clear SLURM-oriented workflow is acceptable. Use the simplest system that remains maintainable.

## Configuration

Keep thresholds and paths in configuration, including:

```text
reference_path
sample_manifest
minimum_mapping_quality
minimum_base_quality
detection_thresholds
reconstruction_thresholds
protein_clustering_thresholds
fragment_sizes
embedding_model
pst_checkpoint
external_database_paths
cpu_threads
memory
gpu_resources
```

## Logging

Each task should record:

- command;
- start and end time;
- exit status;
- software versions;
- input checksums;
- output paths;
- resource use where available.

## Tests

Add tests for:

- sample-name parsing;
- manifest generation;
- paired-read matching;
- malformed summary tables;
- metric calculations;
- coordinate rotation;
- protein-to-genome mapping;
- fragmentation;
- train/test leakage;
- candidate-score calculations.

Scientific workflows should also include small integration tests with miniature FASTA/FASTQ examples.

---

# 19. Reporting structure

Each phase report should contain:

1. objective;
2. inputs;
3. methods;
4. quality-control results;
5. observations;
6. interpretations;
7. limitations;
8. unresolved questions;
9. decision made;
10. next recommended step.

Reports should link to tables and figures using relative paths.

## Master report

Maintain:

```text
reports/VIIJIPHAGE_POPULATION_ATLAS.md
```

Correct the filename to:

```text
reports/VIJIPHAGE_POPULATION_ATLAS.md
```

The accidental doubled `I` above is intentionally called out so it is not propagated.

The master report should be updated after each phase and should synthesise results rather than duplicate all technical details.

---

# 20. Expected final figures

The final atlas should aim to include:

1. cohort prevalence and coverage evidence;
2. representative per-base coverage profiles;
3. sample-specific genome reconstruction summary;
4. whole-genome similarity heat map;
5. phage lineage network;
6. patient-by-time lineage trajectories;
7. variant density and recurrent mutations;
8. comparative genome maps;
9. gene-presence/absence matrix;
10. conserved and variable genome modules;
11. protein-embedding atlas;
12. fragmentation benchmark;
13. method-performance comparison;
14. external candidate distribution by biome;
15. validated remote-family genome comparisons.

All figure-generating code must be reproducible and included in scripts or notebooks.

---

# 21. Initial execution sprint

Codex should begin with the following concrete sequence.

## Sprint 1A: audit

1. Read `SCIENTIFIC_BRIEF.md` and this file.
2. Inspect the repository and linked directories.
3. Produce `reports/00_data_audit.md`.
4. Produce the canonical sample manifest.
5. Validate `phage.fasta`.
6. Validate the existing mapping summary and a subset of BAMs.

## Sprint 1B: mapping evidence

1. Recalculate missing or suspect mapping metrics.
2. Generate per-base coverage profiles.
3. Rank samples for reconstruction.
4. Generate `reports/01_mapping_and_sample_evidence.md`.
5. Identify the pilot reconstruction set.

## Sprint 1C: reconstruction pilot

1. Implement read extraction with unit and integration tests.
2. Reconstruct the pilot samples using at least two methods.
3. Remap and compare candidate genomes.
4. Produce `reports/02_reconstruction_pilot.md`.
5. Recommend the production reconstruction strategy.

Do not begin PST implementation before Sprint 1C is complete unless a minimal environment test is needed to assess model feasibility.

---

# 22. Stop conditions and escalation

Codex should stop and report rather than silently continue when:

- sample identifiers cannot be matched reliably;
- input files are missing or duplicated;
- reference coordinates are inconsistent;
- mapping results suggest widespread contamination;
- reconstruction methods produce incompatible genome structures;
- the presumed single phage is clearly several unrelated phages;
- the pretrained PST cannot be obtained or its requirements are incompatible;
- GPU memory is insufficient for the selected embedding model;
- benchmark leakage cannot be prevented;
- external database licences or access conditions prevent use;
- scientific interpretation depends on unavailable metadata.

The report should state:

- what was attempted;
- what evidence revealed the problem;
- which outputs remain valid;
- the smallest decision or information needed to continue.

---

# 23. Definition of completion

The analysis is complete when it has produced a defensible answer to the following:

1. What constitutes the phage population in this cohort?
2. How many lineages are supported?
3. How do lineages vary among and within patients?
4. Which genome regions and proteins are conserved or variable?
5. What evidence supports persistence, replacement, or mixed populations?
6. How well do nucleotide, protein-homology, pooled-embedding, and PST methods recover the family?
7. Where does PST add measurable value?
8. How fragmented can a genome be before family retrieval becomes unreliable?
9. Which external sequences are credible phage-family members?
10. What ecological distribution is supported by validated candidates?
11. Which conclusions are strong, provisional, or unresolved?
12. What is the most important next experiment or dataset?

The final deliverables should include:

```text
reports/VIJIPHAGE_POPULATION_ATLAS.md
reports/METHODS.md
results/genomes/
results/atlas/
results/pangenome/
results/pst/
results/benchmark/
results/external_search/
notebooks/
figures/
```

---

# Final instruction to Codex

Execute the analysis incrementally. At each decision gate, use the observed data to determine the next step.

Do not optimise for the number of completed pipeline stages. Optimise for the quality, traceability, and biological significance of the conclusions.
