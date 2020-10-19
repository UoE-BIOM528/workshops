---
layout: page
title: Introduction
order: 2
---

## Instructor

Ryan Ames <mailto:r.ames@exeter.ac.uk>

## Objectives

The main objectives of these workshops will be to:
* Understand some of the tools and methods used to perform comparative genomics analysis of fungal pathogens
  * Download current genomics data
  * *De novo* assembly of raw sequencing reads to build a genome assembly
  * *Ab initio* prediction of gene sequences from a genome assembly
  * Organisation of predicted genes into gene families
  * Analysis of gene families to identify signs of positive selection
* Interpret the results to make inferences about the evolution of plant pathogens


## Introduction

Genomic data provides a wealth of information about an organism including gene content, chromosome organisation, regulatory sequences etc. Comparative genomics, the comparison of genome and gene sequences between evolutionarily related organisms, is a powerful approach that can reveal insights into, among other things, organism evolution and gene function.

With the advance of next-generation sequencing technologies that can generate vast amounts of genomic data ever more quickly and cheaply, we can now compare more genomes than ever before. However, to do this we need to make use of bioinformatics and computational biology tools in order to store, process, analyse and visualise these data. To successfully run these tools a biologist must have a working understanding of web services, a unix environment and common biological data file formats. 

## In this workshop

![RNA-Seq Overview](/images/overview.png)

For more background checkout [Wang *et al*., 2009](https://www.nature.com/articles/nrg2484) and [Ozsolak & Milos, 2011](https://www.nature.com/articles/nrg2934).

In this workshop we will look at the analysis of an RNA-seq dataset. Specifically: a differential gene expression experiment, where we attempt to find a set of genes that are expressed at different levels in two strains of *Arabidopsis Thaliana*, when challenged with a fungal pathogen *Sclerotinia sclerotiorum*.  Following on from DGE analysis we will identify enriched GO terms in the set of differentially expressed genes. This will allow us to assign specific functions to these genes.

Here is the workflow that we will use in this workshop

![Workflow](/images/workflow.png)
