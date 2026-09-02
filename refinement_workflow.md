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

Add to the current panmap commands: 
```text
--trim-start 7 --trim-end 7 
```
```text
--min-depth 3
```
It should improve placement to the reference and quality of the consensus sequence.

## Change #3
MD tags are missing from the current panmap/*.bam files, limiting downstream analyses. Add them: 

```
samtools calmd -@ 8 -b Warinner2014/panmap/SRS473742.bam Warinner2014/panmap/SRS473742.ref.fa > Warinner2014/panmap/SRS473742.tmp.bam && \
mv SRS473742.tmp.bam SRS473742.bam && \
samtools index -@ 8 SRS473742.bam
```
Ideally, don't create a separate rule; incorporate it into the existing panmap rule if possible. If it's not a good solution, create a new rule in snakemake.

## Change #4
Replace MapDamage2 with pyDamage (https://pydamage.readthedocs.io/en/latest/), which is faster. 

Remove mapdamage2 from rules and incorporate pyDamage, by running a command: 
```
pydamage --outdir results/Warinner2014/pydamage analyze results/Warinner2014/panmap/SRS473742.bam --plot
```
Comment: The BAM file must be corrected to include MD tags.

## Change #5
Create comprehensive statistics of mapped reads (in bam file), similar to the analysis run by bam-filter (https://github.com/genomewalker/bam-filter). However, due to problems with calculating some values, I would like to re-implement this here instead of running bam-filter.  

In the current version, this analysis is partially run in the coverage rule. The coverage rule should be expanded and renamed to "mapping_statistics". Output folder should be "mapping_statistics". However, the coverage plot and covered_positions.tsv should be generated as before, although below I will ask for some changes to the coverage plot. 

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
This is just an example. Adapt it to produce the expected results described below: 

Columns in the output SRS473742_stats.tsv file: 
- reference [ID of the reference genome]
- reference_length [length of the reference genome]
- n_reads [aligned reads in the bam file]
- read_length_mean [mean of all read lengths]
- read_length_std [read length standard deviation]
- read_length_min [minimum observed read length]
- read_length_max [maximum observed read length]
- read_length_median [median observed read length]
- mapping_quality [example code: samtools view -F 4 input.bam | awk '{sum+=$5; n++} END {if(n>0) print "Mean MAPQ:", sum/n}']
- edit_distances [mean value of edit distances; however, in "raw" table, the edit distance for each read should be present to calculate the histograms]
- read_ani_mean [mean of average nucleotide identities]
- read_ani_std [standard deviation of average nucleotide identities]
- read_ani_median [median of average nucleotide identities]
- coverage_mean [mean coverage value for mapped reads]
- bases_covered [number of positions in the reference genomes covered by a minimum of 1 read]
- bases_covered_depth3 [number of positions in the reference genomes covered by a minimum of 3 reads, something like that was previously calculated in the workflow to make sumary statistics]
- bases_covered_depth5 [number of positions in the reference genomes covered by a minimum of 5 reads, similar to above]
- bases_covered_depth10 [number of positions in the reference genomes covered by a minimum of 10 reads, similar to above]
- breadth [% of positions in the reference genome covered by a minimum of 1 read]
- breadth3 [% of positions in the reference genome covered by a minimum of 3 reads]
- breadth5 [% of positions in the reference genome covered by a minimum of 5 reads]
- breadth10 [% of positions in the reference genome covered by a minimum of 10 reads]

## Change #6
Here, I ask for changes in the visualisation of mapping_statistics. In the first version, only horizontal coverage (breadth) was visualised. Now I want to combine multiple plots in a single figure. 

In the first row, three plots: 

- three histograms. The first one with read length distribution, the second with edit distances and the third with read ANI. For all of them, plot median and mean values within each plot, but don't make dashed lines. 

In the second row, one plot presenting horizontal coverage, similar to the one from the previous version, but in a new version add a slightly transparent histogram with 1000bp bins presenting number of reads mapped in a specific genomic region and each bin should be colored by the mean ANI of reads mapped to the corresponding bins. I need a visual representation if some specific genomic regions are more distantly related to the reference than others. 

... here about adding genomic visualisation with functional annotation.  

For visualising that, use data generated in mapping_statistics.
## Change #7
Change Warinner2014_summary.tsv

In a new version, it should be a combination of outputs from pyDamage (results/Warinner2014/SRS473742/pydamage/pydamage_results.csv) and mapping_statistics (/results/Warinner2014/SRS473742/mapping_statistics/SRS473742_stats.tsv). 

The first column should be "sample" so e.g. SRS473742, then all columns from SRS473742_stats.tsv and from pydamage_results only "CtoT-0","CtoT-1","CtoT-2","CtoT-3","CtoT-4","CtoT-5","CtoT-6","CtoT-7","CtoT-8","CtoT-9","predicted_accuracy","pvalue".

## Change #8
If many projects are analysed and multiple output folders exist in /results/ e.g. /results/Warinner2014 and /results/Ottoni2021, then as a last step concatenate all summary files e.g. Warinner2014_summary.tsv and Ottoni2021_summary.tsv into one all_summary.tsv saved to /results

Some ideas: 
To the coverage plot, add an annotated genome below the plot. To the plot add a slightly transparent histogram filled with mean read ANI in the specific window; it allows you to estimate whether poorly mapped regions also have low read ANI values. 


