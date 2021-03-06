# Command Line RGI for metagenomics & pathogen-of-origin prediction

A training session on CARD’s latest software tools (beta testing) for analysis of metagenomic reads, including an overview of the Burrows Wheeler Transform, the importance of diverse reference sequences, and k-mer algorithms for prediction of pathogen-of-origin of AMR genes.

CARD Members: Amos Raphenya, Brian Alcock, Andrew McArthur

Requires RGI installed on your laptop or remote server or a RGI server account. Requires familiarity with the command line.

<a name="intro"></a>
## Introduction

This module gives an introduction to prediction of antimicrobial resistome based on comparison of metagenomic DNA sequencing data to reference sequence information. While there is a large diversity of reference databases and software, this tutorial is focused on the Comprehensive Antibiotic Resistance Database ([CARD](http://card.mcmaster.ca)) for genomic AMR prediction.

**It is highly recommended you familiarize yourself with CARD and the Resistance Gene Identifier (RGI) before completing this tutorial. In particular, see [Command Line RGI for genomes & genome assemblies: resistome annotation and visualization](https://github.com/arpcard/state-of-the-card-2021/tree/main/day_1/rgi_for_genomes)**.

The Resistance Gene Identifier for metagenomics (RGI*BWT for short) has several key features:

* Use of common read alignment tools such as KMA, BWA or Bowtie2 to align sequencing reads to CARD reference sequences.
* Reference sequences can either be the canonical CARD reference data (i.e. genes in peer reviewed published literature, with clear experimental evidence of elevated MIC) or the canonical CARD reference data *plus* the *in silico* predicted AMR allelic diversity available in [CARD Resistomes & Variants](https://card.mcmaster.ca/genomes). Inclusion of the latter is highly recommended.
* If [CARD Resistomes & Variants](https://card.mcmaster.ca/genomes) data are included, epidemiological interpretive data are provided from [CARD Prevalence](https://card.mcmaster.ca/prevalence).
* Custom k-mer classifiers for pathogen-of-origin & plasmid prediction for detected AMR sequences.
* Optional support for bait-capture AMR metagenomics (not yet ready for testing)

## Demonstration Data

We are going to use the data from Lanza et al paper.

## Build the Reference Files

Login into your command line and make a rgigenomes directory:

```bash
mkdir rgigenomes
cd rgigenomes
```

First we need to acquire the latest AMR reference data from CARD. CARD data can be installed at the system level, but that requires a SysAdmin with root privileges or locally. The `rgi auto_load` command will add the [CARD Resistomes & Variants](https://card.mcmaster.ca/genomes) and [CARD Prevalence](https://card.mcmaster.ca/prevalence) data, and precompiled k-mer reference data too. Please see demo on how to load CARD databases [here](https://github.com/arpcard/state-of-the-card-2021/blob/main/day_1/installing_rgi/linux.sh)

## Analyze Metagenomics Data

Lanza et al. ([Microbiome 2018, 15:11](https://www.ncbi.nlm.nih.gov/pubmed/29335005)) used bait capture to sample human gut microbiomes for AMR genes. We will analyze a subset of these data (1000 paired reads) using RGI*BWT, followed by pathogen-of-origin prediction using k-mers.

```
rgi bwt -h
rgi bwt --read_one gut_R1.fastq.gz --read_two gut_R2.fastq.gz --aligner kma --output_file AMRmeta --debug --local --threads 8 > AMRmeta.log 2>&1
rgi kmer_query -n 8 -i AMRmeta.sorted.length_100.bam --bwt -k 61 --minimum 10 -o AMRmeta.sorted.length_100.bam.61 --local --debug > AMRmeta.sorted.length_100.bam.61.query.log 2>&1
```

RGI for metagenomics provides several layers of output, all available in the GitHub repo as pre-compiled tab-delimited files:

| File | Description |
| -------- | -------- |
| AMRmeta.allele_mapping_data.txt | RGI*BWT read mapping results at AMR allele level |
| AMRmeta.gene_mapping_data.txt | RGI*BWT read mapping results at AMR gene level (i.e. summing allele level results by gene) |
| AMRmeta.artifacts_mapping_stats.txt | Statistics for read mapping artifacts |
| AMRmeta.overall_mapping_stats.txt | Statistics for overall read mapping results |
| AMRmeta.reference_mapping_stats.txt  | Statistics for reference matches |
| AMRmeta.sorted.length_100.bam.61_61mer_analysis.allele.txt | k-mer taxonomic results at AMR allele level |
| AMRmeta.sorted.length_100.bam.61_61mer_analysis.gene.txt | k-mer taxonomic results at AMR gene level (i.e. summing allele level results by gene) |
