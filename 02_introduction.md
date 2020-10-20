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

Our work will focus on genome sequences of an important fungal pathogen *Zynoseptoria tritici*, which causes blotch disease of wheat. Given the importance of the pathogen to the production of wheat, *Z. tritici* is the focus of much scientific research. As a result, there is lots of data available making it possible to conduct comparative genomics studies.

For background reading and an understanding of the data we will be using, see:
* [Stukenbrock *et al*., 2011](https://genome.cshlp.org/content/21/12/2157.short)
* [Grandaubert *et al*., 2015](https://www.g3journal.org/content/5/7/1323).

In these workshops we will work through a set of analyses to take raw reads produced by a next-generation sequencing project (DNA sequencing of our fungal pathogen) to assemble a genome, predict the genes and then analyse the sequence of these genes in multiple species to make inferences about pathogen evolution. We will use some light scripting/programming and a variety of bioinformatics software to complete our tasks. The workflow of our workshops will be as follows:

![Workflow](/images/workflow.png)
