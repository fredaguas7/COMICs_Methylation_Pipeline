# COMICs_Methylation_Pipeline

Pipeline used in the COMICs Lab for analysis of methylation BS-Seq data.

This pipeline utilizes Trim-Galore and Bismark to analyze methylation data. 

## Tools 

The tools used are all available in conda. 

````
conda install bioconda::bismark
conda install bioconda::trim-galore
````

## Step 1 - Trim the reads 

Trim Galore trims the adapter and helps with QC. Specific trimming for certain kits should be used as stated in: https://github.com/FelixKrueger/Bismark/blob/master/docs/bismark/library_types.md

```
trim_galore --paired R_1.fastq.gz R_2.fastq.gz
```

## Step 2 - Allign with Bismark (via bowtie2)

Don't forget to prepare the methylated genome. 

```
bismark_genome_preparation Genomes/YourGenomeFolder
```

Allign.

```
bismark --bowtie2 Genomes/YourGenomeFolder -1 R_1_val_1.fq.gz -2 R_2_val_2.fq.gz
```

## Step 3 - Deduplication and filtering

For most kits, a deduplication step is recommended in bismark. Again, check the documentation: https://github.com/FelixKrueger/Bismark/blob/master/docs/bismark/library_types.md

```
deduplicate_bismark --bam bam_pe.bam
```

We can filter using samtools to keep only reads with high mapping quality (q > 10).

```
samtools view -b -q 10 bam_pe.deduplicated.bam > filtered_deduplicated_bam_pe.bam
```

## Step 4 - Methylation extraction 

Finally, extract the results.

```
bismark_methylation_extractor -p --comprehensive filtered_deduplicated_bam_pe.bam
```

Proceed with the outputs as needed. For example, using bedtools to cross methylated citosines with gene promoters.

## Notes 

Low mapping effiency should be handled as in https://github.com/FelixKrueger/Bismark/blob/master/docs/faq/low_mapping.md

For allignments to poorly described genomes, lowering the stringency or using --local is recommended.

When in doubt don't forget to read the original documentation of the tools! 

