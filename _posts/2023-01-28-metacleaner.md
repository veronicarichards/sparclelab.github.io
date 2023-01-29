---
tag: markup
author_profile: false
title: Metacleaner
subtitle: Automated curation of barcode sequence databases for metabarcoding and metagenomics
header:
    image: /assets/images/Pollen-Baskets.jpg
    teaser: /assets/images/Pollen-Baskets.jpg
excerpt: Automated curation of barcode sequence databases for metabarcoding and metagenomics
description: Automated curation of barcode sequence databases for metabarcoding and metagenomics
comments: true
---

## DNA metabarcoding

Say you're managing honey bee colonies for the purpose of honey production, and you want to know what their product is made of. What sources of nectar are your bees foraging on? You could spend weeks following your bees around, taking note of which flowers they frequent. You might notice some common genera represented - like clover (*Trifolium*) or locust (*Robinia*). But, knowing that *Apis mellifera* is a generalist forager and you've got tens of thousands of bees per colony, you could never possibly collect as much data as they already are. So, how can you leverage that data? (Or, if you're totally uninterested in the composition of honey, say you're interested in what species of bacteria are colonizing your GI tract. Insert your own "I have an environmental sample and I want to know what's in it" problem here.)

[DNA metabarcoding](https://en.wikipedia.org/wiki/Metabarcoding) is a method for assessing taxonomic composition within a sample by "barcoding" of DNA: matching up the sequences of DNA molecules in a sample with a reference sequence database. The process is straightforward: sample homogenization, DNA extraction, PCR amplification, sequencing, and data analysis.

## Assumptions and problems

Like all scientific methods, the basis of metabarcoding rests on a few fundamental assumptions:
- 1) You've not contaminated your sample while handling it, and
- 2) The loci that you're amplifying (and by extension, the sequences in your reference database) are sufficiently different between taxa to assign short sequencing reads (usually \< 400 nucleotides long after merging paired-end reads) to taxa.

In this post, I want to address some problems that emerge from assumption \#2, and propose some solutions. Throughout, I will use data generated through metabarcoding analysis of European honey bee *Apis mellifera* honey samples collected from Pennsylvania beekeepers, using barcode sequences extracted from the [ITS1/ITS2 region for plant DNA barcoding](https://mbmg.pensoft.net/article/68155/). The reference sequence databases used here were constructed by retrieving all publicly available ITS1/ITS2 region sequences identified in plants within Magnoliopsida (the class of flowering plants that pollinator insects like the honey bee are mutualists with) which were accessible on NCBI as of 01-28-22, and using open-source software ([MetaCurator](https://github.com/RTRichar/MetaCurator)) to automatically identify and extract the barcode marker region of interest from those sequences. [See here for database construction and curation methods](https://github.com/sbresnahan/metacleaner/blob/main/ITS1_ITS2_databases.md). In total, the databases are composed of 61,717 ITS1 barcodes representing 5,687 genera, with an average of 10.86 barcodes per genera (SD = 37.56), and 58,482 ITS2 barcodes representing 5,489 genera, with an average of 10.65 barcodes per genera (SD = 35.28).

### Problem \#1: mislabeled sequences
If your reference database is composed of sequences retrieved from open-access repositories like [NCBI GenBank](https://www.ncbi.nlm.nih.gov/nucleotide/), you will unfortunately encounter many sequences that have been mislabeled. In some cases, these are sequences which GenBank staff could not verify as being of the taxa that the uploader identified them as being. For example, sequence accession [FJ860092.1](https://www.ncbi.nlm.nih.gov/nuccore/FJ860092.1) is labeled as being from a sample of Chinese yam, *Dioscorea polystachya* - however, this sequence is 100% identical to sequence accession [KX826865.1](https://www.ncbi.nlm.nih.gov/nuccore/KX826865.1) from an "uncultured fungus" sample, which could also not be verified. In other cases, they have passed the verification process and are thus "valid" sequences. For example, sequence accession [AJ251663.1](https://www.ncbi.nlm.nih.gov/nuccore/AJ251663.1) is labeled as being from a sample of an Italian alder, *Alnus cordata* - however, this sequence is 100% identical to sequence accession [MW215901.1](https://www.ncbi.nlm.nih.gov/nuccore/MW215901.1) again from an unculture fungus sample, possibly containing *Malassezia restricta* (which are [commonly found on the skin surfaces of many animals, including humans](https://en.wikipedia.org/wiki/Malassezia)). This is especially problematic for honey sample metabarcoding, as [many genera of fungi](https://www.beeculture.com/honey-bees-fungi/#:~:text=Most%20beekeepers%20are%20familiar%20with,colonies%20all%20around%20the%20world.) inhabit flowers, honey bees, honey bee colonies, [and thus, honey](https://imafungus.biomedcentral.com/articles/10.1186/s43008-019-0021-7).

This problem and some solutions have already been discussed in detail:
- [Phylogeny-aware identification and correction of taxonomically mislabeled sequences](https://academic.oup.com/nar/article/44/11/5022/2468320?login=true)
- [Using taxonomic consistency with semi-automated data pre-processing for high quality DNA barcodes](https://besjournals.onlinelibrary.wiley.com/doi/10.1111/2041-210X.12824)

These methods use algorithms to try and identify mislabeled sequences and place them within their correct lineages. But, what if I don't understand what an Evolutionary Placement Algorithm is? I have tens of thousands of sequences, how can I possibly check through them all to be sure they have truly been correctly relabeled?

Most importantly, how pervasive is this problem, really? Using [a popular metabarcoding bioinformatics pipeline](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13314) for reference, where sequencing reads are assigned to a genera if 98% of the read's sequence is shared with a barcode's sequence (in terms of sequence identity and query coverage), I checked to see how many of the barcodes in my ITS1/ITS1 database match non-plant (not within Embryophyta) sequences at these thresholds. I downloaded all publically-accessible non-plant ITS1/ITS2 region sequences from NCBI and used [local BLASTn searches](https://www.ncbi.nlm.nih.gov/books/NBK279690/) to identify matching barcodes. I excluded sequences from "uncultured eukaryotic clone" samples, as these are often sequences from dust samples which contain plant pollen (e.g., sequence accession [KF800566.1](https://www.ncbi.nlm.nih.gov/nuccore/KF800566.1) for which 98.33% of its sequence is 100% identical with sequences in several species of Willow (*Salix*). Given "step 1" is all Magnoliopsida barcodes, without any filtering, and "step 2" is after filtering barcodes with matches to non-plant sequences, I found this problem affects roughly 1% of ITS1 barcodes and 6.1% of ITS2 barcodes. Why so many more for ITS2? Leave a comment below if you've got any insight. 

DATA

What about valid plant sequences that are mislabeled as belonging to - or match 100% with - *different plants*?

### Problem \#2: sequence conservation
Metabarcoding is limited by sequence evolution; the extent of this limitation increases as the length of the barcode region decreases. In extracting a specific barcode region of interest, "irrelevant" sequence is trimmed out, which reduces the total space for any given sequencing read to align. This decreases our runtime, but also increases the likelihood that two barcode sequences will be identical. Thus, sequencing reads may be randomly assigned, or inflate the composition of your sample for some taxa. This is further amplified if our alignment software does not handle multi-mapping reads in a meaningful way. 

Okay, but, for what percentage of barcodes in my database is this really an issue? What is the potential false-positive rate in my analysis due to this problem?

DATA HERE

Well, for what percentage of the sequencing reads from my samples is this really an issue?

DATA HERE

## Solution: metacleaner - software for 'cleaning' reference sequence databases for metabarcoding and metagenomics

Hopefully it is obvious that these are really the same problem - the fundamental problem - *are the sequences in my database sufficiently different between taxa?* A simple solution is to automate the process of searching for sequence conservation between our barcodes, and filtering non-unique barcodes in a taxonomically-aware way. 