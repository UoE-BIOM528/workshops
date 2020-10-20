---
layout: page
title: Gene prediction
order: 4
---

## Gene prediction

Now that we have assembled a genome into scaffolds, long chromosome like sequences, we are ready to identify the coding regions or genes within these sequences. It is important to identify our gene sequences for a variety of reasons:

* Genes are the functional unit of a genome. An organisms genes can give us clues about their evolution and lifestyle.
* Sequences that do not code for genes (intergenic regions) make up most of the genome and can be useful for some but not all studies.
* Comparison of gene sequences is important for evolutionary and functional analyses.

## Augustus

There are many computational methods available to identify gene sequences from genomic data. Each can use different algorithms and different input data sources (gene models, DNA sequencing, RNA sequencing) to identify genes. These tools all have their own strengths and weaknesses. For example, some tools require lots of input data or are good at identifying genes in particular species, while others may require known gene models or high-performance computing availability. It is often a good idea to look for papers that benchmark tools on different data sets and report their performance. A recent study by [Scalzitti *et al*., 2020](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-020-6707-9) benchmarked 5 *ab initio* (from the beginning) gene prediction programs on gene sequences from 147 species and quantified their prediction accuracy.

In this workshop we will use [Augustus](https://academic.oup.com/bioinformatics/article/19/suppl_2/ii215/180603), an *ab initio* gene prediction program, that in it's simplest form requires only an input set of sequences i.e. a genome assembly and a species name with known gene models. The software is based on a Hidden Markov Model that takes into account a number of methods and models that capture intron lengths, reading-frame and GC content for a range of species. Importantly, Augustus has been trained on a variety of organisms (including several fungi) so can accurately predict genes from many different genome sequences.

## Input data

### Genome assembly

In the last workshop we looked at how to assemble a genome. For this task we will use a single chromosome from an already complete and assembled genome for *Z. tritici*. To download the sequence for chromosome 10 right click and save choose save as on this [link](data/Z.tritici.chr10.fasta).
