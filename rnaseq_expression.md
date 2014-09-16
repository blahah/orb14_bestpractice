---
layout: page
title: RNASeq expression
tagline:
---
{% include JB/setup %}

## Overview

This page gives guidelines and tutorials for different components of an RNA-seq transcript expression analysis. To get straight into the analysis, follow one of the compatible workflows, or to learn more about the best practices for each step, have a look at the guides below.

<img style="display: block; margin-left: auto; margin-right: auto; width: 300px;" src="assets/images/rnaseq-flowchart.svg"></img>

#### Workflows meeting this standard

- [Fry lab RNASeq pipeline v1.0](http://blahah.net/orb14_bestpractice/workflows/rnaseq_expression/frylab_v1.0.html)
- [TGAC RNASeq pipeline v1.0]()
- [NGLess v1.2](workflows/rnaseq_expression/ngless_1.2.html)

### Steps

0. [Data](#data)
1. [QC](#qc)
2. [Preprocessing](#preprocessing)
3. [Alignment](#alignment)
4. [Quantification](#quantification)
5. [Differential expression](#differential-expression)

<div class="alert-message warning block-message">
  <a class="close" href="#">×</a>
  <p><strong>Note:</strong> this document follows <a href="https://www.ietf.org/rfc/rfc2119.txt">RFC 2119</a> definitions of *must*, *must not*, *should* and *should not*.</p>
</div>

## Data

<div class="alert-message block-message info">
  <a class="close" href="#">×</a>
  <p>It is crucial to document the way data was generated, and to report this when archiving your data and in publication.</p>
</div>

### Steps

How to collect the data. Why each thing is important - what downstream steps does it contribute to, or why is it crucial for reproducibility.

You **must**:

- Submit your raw reads to *either* the [NCBI Sequence Read Archive](http://www.ncbi.nlm.nih.gov/books/NBK47529/) *or* the [EBI European Nucleotide Archive](http://www.ebi.ac.uk/ena/submit).

### Reporting
=======

You **must** record:

- Experiment metadata as described at the [SRA/ENA](http://www.ebi.ac.uk/ena/submit/read-submission)
- RNA extraction protocol
- Fragmentation protocol
- Sequencing chemistry and protocol
- Adapter and primer sequences
- Whether the reads are paired-end
- Read size
- Insert size distribution parameters:
  - mean
  - standard deviation

See: [Sample metadata file template]()

## QC

<div class="alert-message block-message info">
  <a class="close" href="#">×</a>
  <p>All current sequencing technology is inherently error-prone, and mistakes or problems in the sequencing process can lead to problems in downstream analysis if they are not detected. It is crucial to examine the reads using quality control software to identify potential problems.</p>
</div>

### Steps

You *must*:

- **Either** remove adapter sequences from your reads **or** remove reads that contain adapter sequences
- **Either** remove reads with low quality **or** trim low quality bases from reads
- Remove reads with a high proportion of ambiguous bases **or** remove ambiguous bases from the ends of reads

### Reporting

You *must*:

- Follow the software reporting guidelines for any software you use
- Report the number of reads present in each sample before and after trimming

## Preprocessing

## Alignment

### Reporting

You *must*:

- Follow the software reporting guidelines for any software you use
- Report whether you aligned to genome or transcriptome reference
- If your reference was from a publicly available assembly, state its name and version
- If your reference was from a *de-novo* assembly, fully report the assembly protocol
- State the number and proportion of reads aligning for each sample

## Quantification

### Reporting

You *must*:

- Follow the software reporting guidelines for any software you use
- Report the raw counts of reads aligning to each feature (gene or transcript). The counts may optionally be effective counts.
- Report normalised expression (RPKM if displaying a single sample, TPM if comparing multiple samples).

## Normalization


- [NGS best practice overview for beginners](http://biorxiv.org/content/early/2014/06/19/006403)


## Differential expression
- [Differential expression analysis with DEseq](http://bioconductor.org/packages/release/bioc/vignettes/DESeq/inst/doc/DESeq.pdf)
