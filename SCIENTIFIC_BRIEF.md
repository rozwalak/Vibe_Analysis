# SCIENTIFIC BRIEF: Bacteriophage codiversification with humans

## Project summary

We have identified a bacteriophage family, provisionally named **calcyviruses**, that is broadly detectable across a collection of human dental calculus metagenomes. Our initial analysis with vContact2 identified this family as very distinct from other Caudoviricetes bacteriophages, and iPhoP predictions indicate the host is the Desulfobulbus oralis pathobiont. Preliminary search across millions of phage genomes from MetaVR and collection of proteins from 34 mammalian species suggests a direct switch from environmental sources to the human oral cavity, as we did not find any signal from these phage sequences in mammals other than Neanderthals and Anatomically Modern Humans. 

The scientific objective is now to move beyond genomic description of the new family and conduct a **phylogenomic analysis of ancient bacteriophages from the last 100,000 years**. Specifically, we want to use a pangenome of complete calcyvirus genomes constructed in panMAN (https://turakhia.ucsd.edu/panman/index.html) and align reads to the pangenome using panmap (https://amkram.github.io/panmap/), which allows genotyping and assembly of consensus genome sequences to build a time-calibrated bayesian phylogenetic tree. 

The primary objective is the **bioinformatic pipeline**. Code should be reproducible, modular, testable, and documented (snakemake) but implementation decisions should be driven by their ability to answer the biological questions.

## Available data

The project directory contains or links to the following resources:

```text
input/bam_cov_filter.sh
input/plot_cov.py
input/calcy_complete_reoriented.panman 
input/calcy_complete_reoriented.panman.idx
input/metagenome_assembly-Eisenhofer2020.tsv
input/metagenome_assembly-Ottoni2021.tsv
```

Expected meanings:

- `input/bam_cov_filter.sh`: bash script to use samtools to extract information about coverage depth of a specific position in the reference genome.
- `input/plot_cov.py`: Python script for visualising horizontal coverage for the specific sample.
- `input/calcy_complete_reoriented.panman`: Pangenome generated before using panMAN.
- `input/calcy_complete_reoriented.panman.idx`: Panmap index to reuse.
- `input/metagenome_assembly-Eisenhofer2020.tsv`: Table with paths to paired-end data processed with nfcore/eager. 
- `input/metagenome_assembly-Ottoni2021.tsv`: Table with paths to single-end data processed with nfcore/eager. 

Expected outputs: 

```text
results/Ottoni2021/ERS6256686/panmap/ERS6256686.bam
results/Ottoni2021/ERS6256686/panmap/ERS6256686.bam.bai
results/Ottoni2021/ERS6256686/panmap/ERS6256686.consensus.fa
results/Ottoni2021/ERS6256686/panmap/ERS6256686.mpileup
results/Ottoni2021/ERS6256686/panmap/ERS6256686.placement.tsv
results/Ottoni2021/ERS6256686/panmap/ERS6256686.ref.fa
results/Ottoni2021/ERS6256686/panmap/ERS6256686.ref.fa.fai
results/Ottoni2021/ERS6256686/panmap/ERS6256686.vcf
results/Ottoni2021/ERS6256686/panmap/ERS6256686.vcf.gz
results/Ottoni2021/ERS6256686/panmap/ERS6256686.vcf.gz.tbi

results/Ottoni2021/ERS6256686/coverage/ERS6256686_covered_positions.tsv
results/Ottoni2021/ERS6256686/coverage/ERS6256686_cov_plot.png 
 
results/Ottoni2021/ERS6256686/mapdamage2/5pCtoT_freq.txt

results/Ottoni2021/Ottoni2021_summary.tsv
```
Expected meanings:

- `results/Ottoni2021/ERS6256686/panmap/ERS6256686.bam`: Mapping output.
- `results/Ottoni2021/ERS6256686/panmap/ERS6256686.consensus.fa`: Genome with consensus sequence after genotyping. This is the most important output, which will be used for downstream processing. 
- `results/Ottoni2021/ERS6256686/panmap/ERS6256686.placement.tsv`: File with information about the reference genome selected for mapping, plot_cov.py uses information from this file.
- `results/Ottoni2021/ERS6256686/coverage/ERS6256686_covered_positions.tsv`: Output from bam_cov_filter.sh, plot_cov.py uses information from this file.
- `results/Ottoni2021/ERS6256686/mapdamage2/5pCtoT_freq.txt`: Output from MapDamage2, it contains information about damage of reads mapped to the reference genome. 



Do not assume that every optional output exists. Discover inputs explicitly and fail clearly when required data are absent.

## Conda environments
Each rule in the snakemake should create its own conda environment.

---

## Goal

Building a snakemake pipeline to achieve expected results. Each step should be explained and consulted with me. 

## Specific steps

# Step 1
Inspect existing input files and read documentation of software that will be implemented (snakemake, panmap, samtools etc.).

# Step 2 


Do not merely execute the proposed plan mechanically. Inspect the data, compatibility of conda environments, test assumptions, identify unexpected patterns, and update the pipeline to make it simple, reproducible in different systems (PC vs server and cluster), efficient, and self-explanatory at each stage.
