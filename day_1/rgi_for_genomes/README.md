# Command Line RGI for genomes & genome assemblies: resistome annotation and visualization

A training session on how to analyze genomes, genome assemblies, or metagenomic assemblies using RGI at the command line, plus an introduction to publication grade heat map generation.

CARD Members: Amos Raphenya, Brian Alcock, Andrew McArthur

Requires RGI installed on your laptop or remote server or a RGI server account. Requires familiarity with the command line.

## Introduction

This module gives an introduction to prediction of antimicrobial resistome and phenotype based on comparison of genomic DNA sequencing data to reference sequence information. While there is a large diversity of reference databases and software, this tutorial is focused on the Comprehensive Antibiotic Resistance Database ([CARD](http://card.mcmaster.ca)) for genomic AMR prediction.

The relationship between AMR genotype and AMR phenotype is complicated and no tools for complete prediction of phenotype from genotype exist. Instead, analyses focus on prediction or catalog of the AMR resistome – the collection of AMR genes and mutants in the sequenced sample. While BLAST and other sequence similarity tools can be used to catalog the resistance determinants in a sample via comparison to a reference sequence database, interpretation and phenotypic prediction are often the largest challenge. To start the tutorial, we will use the Comprehensive Antibiotic Resistance Database ([CARD](http://card.mcmaster.ca)) to examine the diversity of resistance mechanisms, how they influence bioinformatics analysis approaches, and how CARD’s Antibiotic Resistance Ontology (ARO) can provide an organizing principle for interpretation of bioinformatics results.

CARD’s website provides the ability to: 

* Browse the Antibiotic Resistance Ontology (ARO) and associated knowledgebase.
* Browse the underlying AMR detection models, reference sequences, and SNP matrices.
* Download the ARO, reference sequence data, and indices in a number of formats for custom analyses.
* Perform integrated genome analysis using the Resistance Gene Identifier (RGI).

## Demonstration Data

> If you are using your laptop instead of a CARD RGI account on UPPSALA, your instructor will provide you with a secure download link for the demonstration data. While this is anonymized data, it cannot be shared outside of the IIDR. Start this download now.

## CARD Detection Models

In this part of the tutorial, your instructor will walk you through the following use of the CARD website to familiarize yourself with its resources:

* Examine the mechanisms of resistance as described by the Antibiotic Resistance Ontology.
* Examine the NDM-1 beta-lactamase protein, it’s mechanism of action, conferred antibiotic resistance, it’s prevalence, and it’s detection model. (BLASTP of NDM-1 against CARD)
* Examine the aac(6')-Iaa aminoglycoside acetyltransferase, it’s mechanism of action, conferred antibiotic resistance, it’s prevalence, and it’s detection model. (BLASTP of aac(6')-Iaa against CARD)
* Examine the recently reported colistin resistance MCR-1 protein, it’s mechanism of action, conferred antibiotic resistance, it’s prevalence, and it’s detection model. (BLASTP of MCR-1 against CARD)
* Examine the fluoroquinolone resistant gyrB for *M. tuberculosis*, it’s mechanism of action, conferred antibiotic resistance, and it’s detection model. (Why would BLASTP be inappropriate for this resistance determinant?)
* Examine the glycopeptide resistance gene cluster VanA, it’s mechanism of action, conferred antibiotic resistance, and it’s detection model(s). (Why would BLASTP be inappropriate for this resistance determinant?)
* Examine the MexAB-OprM efflux complex, it’s mechanism of action, conferred antibiotic resistance, it’s prevalence, and it’s detection model(s). (Why would BLASTP be inappropriate for this resistance determinant?)

## RGI for Genome Analysis

As illustrated by the exercise above, the diversity of antimicrobial resistance mechanisms requires a diversity of detection algorithms and a diversity of detection limits. CARD’s Resistance Gene Identifier (RGI) currently integrates four CARD detection models: **Protein Homolog Model**, **Protein Variant Model**, **rRNA Variant Model**, and **Protein Overexpression Model**. Unlike naïve analyses, CARD detection models use curated cut-offs, currently based on BLAST/DIAMOND bitscore cut-offs. Many other available tools are based on BLASTN or BLASTP without defined cut-offs and avoid resistance by mutation entirely. 

In this part of the tutorial, your instructor will walk you through the following use of CARD’s Resistome Gene Identifier with “Perfect and Strict hits only”:

* Resistome prediction for the multidrug resistant *Acinetobacter baumannii* MDR-TJ, complete genome (NC_017847)
* Resistome prediction for the plasmid isolated from *Escherichia coli* strain MRSN388634 (KX276657)
* Explain the difference in triclosan resistance between two clinical strains of *Pseudomonas aeruginosa* that appear clonal based on identical MLST (*Pseudomonas1.fasta*, *Pseudomonas2.fasta*; these are SPAdes assemblies)
 
## RGI at the Command Line

RGI is a command line tool as well, so we’ll do a demo analysis of 112 clinical multi-drug resistant *E. coli* from Hamilton area hospitals, sequenced on MiSeq and assembled using SPAdes. We’ll additionally try RGI’s heat map tool.

Login into your command line and make a rgigenomes directory:

```bash
mkdir rgigenomes
cd rgigenomes
```

Take a peak at the list of *E. coli* samples and the options for the RGI software:

> Paths will be different if using your own laptop.

```bash
ls /home/agmcarthur/ecoli_112_fasta
rgi -h
```

First we need to acquire the latest AMR reference data from CARD. CARD data can be installed at the system level, but that requires a SysAdmin with root privileges. This is not supported on UPPSALA.

```bash
rgi load -h
wget https://card.mcmaster.ca/latest/data
tar -xvf data ./card.json
less card.json
rgi load -i card.json --local
ls
```

We don’t have time to analyze all 112 samples, so let’s analyze 1 as an example using a number of RGI options (the GitHub repo contains an EXCEL version of the *Ecoli_37_d.txt* file):

```bash
rgi main –h
rgi main -i /home/agmcarthur/ecoli_112_fasta/Ecoli_37.fasta -o Ecoli_37_a -t contig -a BLAST -n 8 --local --clean
rgi main -i /home/agmcarthur/ecoli_112_fasta/Ecoli_37.fasta -o Ecoli_37_b -t contig -a DIAMOND -n 8 --local --clean
rgi main -i /home/agmcarthur/ecoli_112_fasta/Ecoli_37.fasta -o Ecoli_37_c -t contig -a DIAMOND -n 8 --local --clean --include_loose
rgi main -i /home/agmcarthur/ecoli_112_fasta/Ecoli_37.fasta -o Ecoli_37_d -t contig -a DIAMOND -n 8 --local --clean --include_loose --split_prodigal_jobs
ls
less Ecoli_37_d.json
less Ecoli_37_d.txt
```

I have pre-compiled Perfect and Strict results for all 112 samples, so let’s try RGI’s heat map tool (pre-compiled images can be downloaded from the GitHub repo, or viewed [here](https://github.com/arpcard/state-of-the-card-2019/tree/master/day_3/rgi_for_genomes/heatmaps)):

```bash
ls /home/agmcarthur/ecoli_112_json
rgi heatmap –h
rgi heatmap -i /home/agmcarthur/ecoli_112_json -cat gene_family -o genefamily_samples -clus samples
rgi heatmap -i /home/agmcarthur/ecoli_112_json -cat drug_class -o drugclass_samples -clus samples
rgi heatmap -i /home/agmcarthur/ecoli_112_json -o cluster_both -clus both
rgi heatmap -i /home/agmcarthur/ecoli_112_json -o cluster_both_frequency -f -clus both
ls
```

Lastly, let's predict pathogen-of-origin for the Perfect and Strict RGI hits for *Ecoli_37_a.json*. First, we need to create the k-mer reference data:

```
rgi kmer_build -h
rgi card_annotation -i card.json > annotation.log 2>&1
wget -O wildcard_data.tar.bz2 https://card.mcmaster.ca/latest/variants
mkdir -p wildcard
tar -xvf wildcard_data.tar.bz2 -C wildcard
ls
```

Note that the next command depends on the version of CARD data and may need to be updated (**this will take a lot of time, precompiled reference data available from your instructor**):

```
rgi kmer_build -i wildcard/ -c card_database_v3.0.1.fasta -k 61 > kmer_build.61.log 2>&1
rgi load --kmer_database 61_kmer_db.json --amr_kmers all_amr_61mers.txt --kmer_size 61 --local --debug > kmer_load.61.log 2>&1
ls
```

Now analyze the RGI output, which will create text output with k-mer results appended to RGI results (the GitHub repo contains an EXCEL version of the *Ecoli_37_a.61kmer_61mer_analysis_rgi_summary.txt* file):

```
rgi kmer_query -h
rgi kmer_query -n 8 -i Ecoli_37_a.json --rgi -k 61 --minimum 10 -o Ecoli_37_a.61kmer --local > Ecoli_37_a.61kmer.log 2>&1
```

## RGI for Merged Metagenomics Reads (or short contigs)

The standard RGI tool can be used to analyze metagenomics data, but only for merged reads with Prodigal calling of partial open reading frames (ORFs). This is a computationally expensive approach, since each merged read set may contain a partial ORF, requiring RGI to perform massive amounts of BLAST/DIAMOND analyses. While computationally intensive (and thus generally not recommended), this does allow analysis of metagenomic sequences in protein space, overcoming issues of high-stringency read mapping relative to nucleotide reference databases. Assembled metagenomic data or short contig / low quality data in general could alternatively be used instead of merged reads.

Lanza et al. ([Microbiome 2018, 15:11](https://www.ncbi.nlm.nih.gov/pubmed/29335005)) used bait capture to sample human gut microbiomes for AMR genes. Use RGI under “Low quality / coverage” and “Perfect, Strict and Loose hits” settings, analyze the first 500 merged metagenomic reads from their analysis (file *ResCap_first_500.fasta*). Take a close look at the predicted “sul2” and “sul4” hits. How good is the evidence for these AMR genes? The GitHub repo contains an EXCEL version of the *ResCap_first_500.txt* file.

```
rgi main -i /home/agmcarthur/fasta/ResCap_first_500.fasta -o ResCap_first_500 -t contig -a DIAMOND -n 8 --local --clean --include_loose --split_prodigal_jobs --low_quality
less ResCap_first_500.txt
head -n 1 ResCap_first_500.txt | cut -f 6,9,10,21,24
grep sul ResCap_first_500.txt | cut -f 6,9,10,21,24 | sort
```
