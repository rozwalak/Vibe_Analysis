all_summary.tsv should also contain information from {project}_summary.tsv is now incomplete, probably because of incomplete mapping statistics for at least one sample. Check aDNA_panmap_snakemake to see how it should looks like. 

Remove results/errors.

Errors should be reported at global level as txt file with project and sample names that was corrupted at some point. and details should be inside specific folders. 

if project have no errors, the folder "errors" shouldn't be created. 

pydamage log is two times, report only pydamage.log.pydamage

"status" folder should be a subfolder of "logs"

csv to_rerun should be only at project level. remove something like this "PacificCalculus_ERSX0004058_to_rerun.csv"
