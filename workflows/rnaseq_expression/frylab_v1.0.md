---
layout: page
title: Frylab pipeline
tagline:
---

{% include JB/setup %}


You can download the sample pipeline file here.


#Table of contents
0. [Pre-requirements](#Pre-requirements)
1. [Fetching data](#Fetching data)
2. [Custom sequence]()
3. [Data management]()
4. [Initial QC]()
5. [Read alignment]()

#Pre-requirements

In order to navigate this document, you will need to have basic familiarity with the Unix command line.
For an introduction, check out our Unix basics guide.

If you are running analysis on a remote server or cluster, please check out our computing infrastructure for beginners guide.


#Fetching data

##From GEO

You can fetch data using wget. You then need to uncompress SRA files. The SRA toolkit tool does this.

```bash
/sratoolkit/bin/fastq-dump --split-3 ./SRR445718.sra
```

Rather than being installed as a package, this is just a script that you execute.
The purpose of the "--split-3 parameter is to make the script check whether the files are single end or paired end.
When this parameter is included, the paired end files will be created as separate files.

##Custom sequence

If you get data straight from a sequencer or from other sources, the raw data files appropriate for starting the analysis are .fq / .fastq.
It is also fine to start with compressed files - e.g. .fq.gz. Most tools can deal with this and will uncompress them for you automatically for the purposes of the analysis.


#Data management

It is a good idea to keep a folder of raw files that are untouched, which can then be used to reproduce the analysis if required. If you give them informative names, this can make the downstream analysis easier to follow.

Storing metadata for future reference is also good practice. Here is a basic metadata template file.


#Initial QC

It is essential to check the data quality before proceeding. We illustrate the process here using the FastQC tool. Other QC tools include:
- RNAseQC
- RSeqC
- Fastx

Sample FastQC command:

```bash
> fastqc raw_data/*
```

This creates an html QC page for each raw data file.


For help with interpreting the QC results, go to our QC interpretation help page.


#Read alignment

Tophat is used to illustrate this step. For RNA-seq, it is essential to use a "transcript-aware" aligner. For example, Bowtie will align reads to the genome, but does not consider splice junctions, so will fail to align a considerable portion of the reads that do not align to the genome exactly. The way Tophat works is that it initially runs Bowtie, but later considers gene annotations for mapping reads to splice junctions.

Other aligners appropriate for RNAseq include: ...

If you are running Tophat for the first time, you will need to prepare the genome indices
[genome index instructions]()

Warning: the Tophat step is long! It's a good idea to run it on a server or cluster, and make sure that you run the command so that hanging up the connection doesn't interrupt it, e.g. by using nohup.

Sample Tophat command:

```bash
 > tophat -p 4 -o sample1_thout -G ~/genomes/Homo_sapiens.GRCh38.76.withchr.gtf -m 2 /home/ja313/genomes/Homo_sapiens/Ensembl/GRCh38/Sequence/Bowtie2Index/hg38_genome sample1.fq.gz
```

-p specifies the number of processors to use. If you are running your analysis on a multiprocessor machine, using this setting to run things in parallel will speed up your analysis
-o specifies the output directory that will be created for the sample in question
-G gives the relevant set of gene annotations
-m specifies the default number of sequence mismatches permitted
the genome index and sample file go on the end of the command

Results

The output is a .bam file of aligned reads:
accepted_hits.bam

This file is a binary file. It can be converted to .sam using
```bash
samtools view -h sample1_thout/accepted_hits.bam > sam_files/sample1_accepted.sam
```

The original file of accepted hits will contain a range of results including multiple alignments. To filter for unique reads only, you can use the flags in the sam file to select the right lines:
```bash
egrep '(NH:i:1)|(^@)' sam_files/sample1_accepted.sam > sam_files/sample1_unique.sam
```

If the downstream applications require bam files, you can then convert this back to bam again using samtools:
```bash
samtools view -S -b sam_files/sample1_unique.sam > bam_files/sample1_unique.bam
```

#Visualising data in a genome browser

It is always a good idea to have a look at what your data looks like, as this will flag some more obvious errors.

##In IGV

Bam files can be opened directly in IGV, as long as an index file (.bai) is included in the same folder.

Index files can be created using samtools:
```bash
samtools index sample1_unique.bam bam_files/sample1_unique.bai
```

##On UCSC

For uploading bam files to UCSC, it is usual to create a bigwig file for upload. This is done in multiple steps:

Creating a bedgraph file:
```bash
genomeCoverageBed -bg -split -ibam bam_files/sample1_unique.bam -g ~/software/UCSC/hg38_genome_UCSC.table > bedgraph/sample1_unique.bedgraph
```

Replace MT with M to match UCSC chromosome names. If it isn't included already, you may also need to add "chr" to the beginning of each chromosome name.

```bash
sed -e "s/chrMT/chrM/ig" bedgraph/sample1_unique.bedgraph > /tmp/tempfile.tmp
     mv /tmp/tempfile.tmp bedgraph/sample1_unique.bedgraph
```

To add "chr", you can use:
```bash
awk '{print "chr"$0}' bedgraph/sample1_unique.bedgraph > bedgraph/sample1_unique_chr.bedgraph
```

Convert bedgraph to bigwig:
```bash
./bedGraphToBigWig bedgraph/sample1_unique.bedgraph hg38_genome_UCSC.table bigwig/sample1_unique.bw
This is done using UCSC scripts
You will need to have a table of UCSC chromosome lengths. If you don't have this, you can fetch it using this script.
```

#Generating summary counts

In preparation for a differential expression analysis, it is usual to acquire a file of summary counts per feature of interest, for example per gene or per exon. The htseq tool can perform this:

```bash
htseq-count -i gene_id -m union sam_files/sample1_unique.sam ~/genomes/Homo_sapiens.GRCh38.76.withchr.gtf > sample1_gene_counts.txt
```

#Reading data into R

The following steps are performed in R. We use the DEseq package to illustrate some of the QC steps. Other popular packages include:
- edgeR

Reading in required libraries:
```bash
library("DESeq")
```

Reading in counts data:
```bash
data <- read.table("htseq_gene_trans_counts_table.txt", header=T,
                         row.names=1, sep="\t")
```

DEseq requires a metadata table. Here is an example with multiple conditions:
```bash
ribosomalDesign = data.frame(
  rownames =colnames(counts),
  condition = conds,
  libType = c(rep("single-end", 16)),
  family= c(rep("unrelated", 4), rep("family", 12)),
  gender =c(rep("female", 12), rep("male", 4)))
```

#Normalization


For normalization, DEseq estimates scaling factors. These are numbers that the raw data counts are divided with in order to make the data across different replicates and conditions directly comparable with each other.

To calculate size factors:
```
cds <- newCountDataSet( counts, conds )
cds <- estimateSizeFactors( cds )
```

To print out size factors for future use:
```
factors <- sizeFactors( cds )
write.table(factors, file="results/size_factors.txt", sep="\t")
```

To print out the normalized counts:
```
nCounts <- counts(cds, normalized=TRUE)
write.table(nCounts, file="results/normalized_counts.txt", sep="\t")
```

#RNA-seq technical QC

Plotting dispersion estimates:
```
pdf(file="results/Dispersion.pdf");
plotDispEsts(cds)
dev.off();
```

Gene scatterplot:
```
geneScatterplot <- function( x, y, xlab, ylab, col ) {
  plot( NULL, xlim=c( -.1, 6.2 ), ylim=c( -1, 6.2 ),
        xaxt="n", yaxt="n", xaxs="i", yaxs="i", asp=1,
        xlab=xlab, ylab=ylab )
  abline( a=-1, b=1, col = "lightgray", lwd=2 )
  abline( a=0, b=1, col = "lightgray", lwd=2 )
  abline( a=1, b=1, col = "lightgray", lwd=2 )
  abline( h=c(0,2,4,6), v=c(0,2,4,6), col = "lightgray", lwd=2 )
  points(
    ifelse( x > 0, log10(x), -.7 ),
    ifelse( y > 0, log10(y), -.7 ),
    pch=19, cex=.2, col = col )
  axis( 1, c( -.7, 0:6 ),
        c( "0", "1", "10", "100", expression(10^3), expression(10^4),
           expression(10^5), expression(10^6) ) )
  axis( 2, c( -.7, 0:6 ),
        c( "0", "1", "10", "100", expression(10^3), expression(10^4),
           expression(10^5), expression(10^6) ), las=2 )
  axis( 1, -.35, "//", tick=FALSE, line=-.7 )
  axis( 2, -.35, "\\\\", tick=FALSE, line=-.7 )
}

colSample <- rgb(78,185,75, maxColorValue=255)
colControl <- rgb(190,91,165, maxColorValue=255)
geneScatterplot( nCounts[,1], nCounts[,2],
                 "normalized read count, Sample1", "normalized read count, Sample2",
                 colSample )
geneScatterplot( nCounts[,1], nCounts[,3],
                 "normalized read count, Control1", "normalized read count, Control2",
                 colControl )
```


Differential expression analysis

```
results <- nbinomTest( cds, "sample", "control" )
```
