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
MD tags are missing from the current panmap/*.bam files, limiting downstream analyses. Add them: 

```
samtools calmd -@ 8 -b Warinner2014/panmap/SRS473742.bam Warinner2014/panmap/SRS473742.ref.fa > Warinner2014/panmap/SRS473742.tmp.bam && \
mv SRS473742.tmp.bam SRS473742.bam && \
samtools index -@ 8 SRS473742.bam
```
Ideally, do not create a separate rule but incorporate it into existing panmap rule if possible. If it's not a good solution, create a new rule in snakemake.

## Change #3
Replace MapDamage2 with pyDamage (https://pydamage.readthedocs.io/en/latest/), which is faster. 

Remove mapdamage2 from rules and incorporate pyDamage, for running a command: 
```
pydamage --outdir Warinner2014/pydamage analyze Warinner2014/panmap/SRS473742.bam --plot
```
Comment: The BAM file must be corrected to include MD tags.

## Change #4
Create comprehensive statistics of mapped reads (in bam file), similar to the analysis run by bam-filter (https://github.com/genomewalker/bam-filter). However, due to problems with calculating some values, I would like to re-implement this here instead of running bam-filter.  
```text
samtools view -F 2308 Warinner2014/panmap/SRS473742.bam | \
awk '{
    cigar=$6
    nm=-1

    for(i=12;i<=NF;i++)
        if($i ~ /^NM:i:/) {
            split($i,a,":")
            nm=a[3]
        }

    # Calculate reference/query aligned length from CIGAR
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
}' > Warinner2014/panmap/SRS473742_read_stats.tsv
```
This is just an example. Adapt it to produce expected results, described below: 


