# First refinement of plan_workflow.md

First: Read and understand the current pipeline, scripts and output files. 

## Details to change/add

## Change #1
In config.yaml: 
```text
# The supplied files are tab-delimited despite their .csv extension.
projects:
  Warinner2014:
    samplesheet: input/metagenome_assembly-Warinner2014.csv
    library_type: single
```
- Read both .csv and .tsv files (remove # The supplied files are tab-delimited despite their .csv extension.)
- Library type should be defined for a specific sample in the spreadsheet, without an extra parameter here (library_type: single). Column R0 is filled for single-end libraries, and columns R1 and R2 for paired-end libraries.

## Change #2
MD tags are not present in the current panmap/*.bam files, limiting downstream analyses. Add them: 

```
samtools calmd -@ 8 -b Warinner2014/panmap/SRS473742.bam Warinner2014/panmap/SRS473742.ref.fa > Warinner2014/panmap/SRS473742.tmp.bam && \
mv SRS473742.tmp.bam SRS473742.bam && \
samtools index -@ 8 SRS473742.bam
```
Ideally, do not create a separate rule but incorporate it into the panmap rule if possible. If it's not a good solution, create a new rule in snakemake.

## Change #3
Replace MapDamage2 with pyDamage (https://pydamage.readthedocs.io/en/latest/), which is faster. 

- Remove mapdamage2 from rules and incorporate pyDamage, running a command: 
```
pydamage --outdir pydamage analyze Warinner2014/panmap/SRS473742.bam --plot
```
Comment: The BAM file must be corrected to have MD tags.
