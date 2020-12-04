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

### Navigate to the data directory

First we will navigate to our data directory:

```
cd ~/Desktop/Data/Z.tritici/
```

If you use `ls` to list the files in the directory you will see a range of *Z. tritici* files.

### Getting help

In order get some usage information for Augustus and a brief description of some of the parameters we can use the command:

```
augustus --help
```

To see the full list of parameters we can use:

```
augustus --paramlist
```

We can also view the online [readme](https://github.com/Gaius-Augustus/Augustus/blob/master/docs/RUNNING-AUGUSTUS.md) to see a more detailed explanation of the usage parameters and some tips and hints for running Augustus. This information is essential for any type of advanced analysis with Augustus.

Next, we two types of input data.

### Genome assembly

In the last workshop we looked at how to assemble a genome. For this task we will use a single chromosome from an already complete and assembled genome for *Z. tritici*. If you're working on your own machine you should have already downloaded the data. For this exercise we'll be using the sequence for chromosome 10 - `Z.tritici.chr10.fasta`.  We're only going to infer the genes on a single chromosome first as the gene prediction algorithm can take minutes to hours.

### Reference species

Now, we need to choose our reference species. To see a list of all species options we can use the command:

```
augustus --species=help
```

This shows us the full list of species we can use as a reference for inferring genes in our genome sequence. However, you will notice that *Z. tritici* (or its synonyms *Septoria tritici, Mycosphaerella graminicola*). So now we have two options:

1. [Retrain Augustus](https://github.com/Gaius-Augustus/Augustus/blob/master/docs/RUNNING-AUGUSTUS.md#retraining-augustus)  
2. Use a closely related species

Here, we will use the latter as retraining Augustus is an in-depth process and beyond the scope of this workshop. We therefore need to find a closely related species (mostly likely to have similar gene structure) to our fungal pathogen. The work by [Chang *et al*., 2016](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1005904) used a concatenated sequence alignment of 46 orthologous single-copy to infer a phylogeny fungal plant pathogens. We can see from this phylogeny that the closest related species available in Augustus is *Aspergillus nidulans*, even though this is an outgroup in the phylogeny.

![phylogeny](/images/fungal_phylogeny.png)

We will use *A. nidulans* (`aspergillus_nidulans`) in our initial analysis.

## Running our initial gene prediction

Now we have our input parameters we can run Augustus. Let's start with the basic command:

```
augustus --species=aspergillus_nidulans Z.tritici.chr10.fasta
```

The output of our Augustus run is by default printed to the screen. The format is the General Feature Format (GFF) where the columns contain:

```
seqname   source     feature    start   end   score   strand   frame    transcript and gene name
```

This is a useful format for representing features such as genes in large genome sequences. You should also notice that the sequence of the predicted proteins identified is also displayed.

We can tell by the gene IDs (line such as `# start gene g577`) that Augustus predicted 577 genes on chromosome 10. As *Z. tritici* is already annotated we can use the Ensembl fungi browser to view [chromosome 10](http://fungi.ensembl.org/Zymoseptoria_tritici/Location/Chromosome?r=10) and we can see that there are an annotated 516 genes (plus 16 non coding genes).

## Exercises

1. Run the analysis again on chromosome 10 but this time choose a different fungal pathogen as your reference.
  * Are there any differences?
  * How might you check these predictions against one another or with the *Z.tritici* reference?
2. Try running Augustus on using the chromosome 11 file: `Z.tritici.chr11.fasta`.
  * Use the Ensembl fungi browser to check the prediction
  * Can you think of any reasons why there might be a discrepancy?
3. Augustus by default only predicts the protein sequences of identified genes. Can you also predict coding sequences?

## A final gene prediction

When we last ran Augustus the output was printed to the screen, which is of no use if we want to use the results in downstream analysis. So to save our results we can use:

```
augustus --species=aspergillus_nidulans Z.tritici.chr10.fasta > Z.tritici.chr10.gff
```

The `>` redirects the output to a new file called `Z.tritici.chr10.gff`, which will store our results for future analysis.

## Extraction of protein sequences

Even though we have saved our results they are only in GFF file format. Our next analysis will need the extracted protein sequences in fasta format. Luckily Augustus provides a set of [utility scripts](https://github.com/Gaius-Augustus/Augustus/tree/master/scripts) that can perform a variety of tasks. One such script, `getAnnoFasta.pl` will extract the protein sequences from a GFF file. Lets try the command:

```
perl ~/software/augustus/scripts/getAnnoFasta.pl Z.tritici.chr10.gff
```

NB - if you're running this on your own machine the path to augustus and the perl script may be different. If you run the above command we can see the file `Z.tritici.chr10.aa` has been generated. This is a fasta formatted file that will look something like:

```
>g1.t1
MPPVLLYLAVSNIVPGATLTNAFENQPPVLLYLAVSKSAPGATLTNAFENQPDRQTDDGVGIARVDYPILATGFALPRRLKYRPRGDPDQRLREHWVGVG
RGCWRRYLHGRWSDRQTDDGVGIARVGYPTLAIGFVLPRRLKFRPRGDPDQRLYENRVGVGRGCWRRYLYGRWSDRQTDDGVGIARVGYPTLAIGFVLPR
RLKFRPRGDPDQRLYENRPPVLLYLAVSKSAPGATLTNAFENEPDRQTDDGVGIARVDYPILATGFALPRRLKYRPRGDPDQRLREHRVGVGRGCWRRYL
HGRWSDRQTDDGVGIARVDYPILATGFALPRLK
>g2.t1
MRYNAQLASFFDSLYVDPRPYRYRYLLRSCIRIKKPSPGRLLAKRSFPYIVETIQEEVAAGKLPKLDIDPIPFGYESDFNPPKRRRKEYASPSPRAVALD
IPDPPSPALRDFFGLVPLTPPALCLRVDKDREQDEQHLSFSRAESAPLCEGYDLPIHQTRKILRRLSELSNSFLIVKYRRDFKEKQERAMGVIRDRCGYN
AKQLILDEKLSPFEALKILREHFSKGGAGEFARVSTLWNLITMDGAKGIGHYGSEIRRLWNRFAEIDSSLKIPEPHAVQKFIAGLPESFEPFVTSFNQNN
KLIPKTENNRTVTNAVSLGEAMYAMLGTRPDIAYSVSVVSRFCANPTEAH
```

Here the `>` denotes the header line and identifier for the protein sequence immediately below. The protein IDs are names by gene ID i.e. `g1` and transcript ID i.e. `t1` seperated by `.`. You can use:

```
head Z.tritici.chr10.aa
```

to view the start of the protein sequence file you have created.

## Summary

We have now used a gene prediction program, Augustus, to predict gene positions and associated protein sequences in a genome assembly. Specifically you will have learned to:

* Run Agusutus to predict genes with a known reference
* Extract the associated protein sequences

We will next use protein sequences, that can be predicted with the methods discussed here, to identify protein families.
