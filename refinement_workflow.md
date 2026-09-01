# Refinement of plan_panmap_pipeline.md

First: Overview of the pipeline, scripts and output files. 

## Details to change/add
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
