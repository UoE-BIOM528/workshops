---
layout: page
title: Setup
order: 1
---
# Workshop setup

There are two ways of completing these workshops:

1. Use a virtual machine on OpenStack
  * You will need to create an instance on OpenStack
  * All software and data needed will be pre-installed
  * An internet connection will be required (+ a VPN for off-campus access)
2. Use your own machine
  * A mac or linux machine will be needed - if you use Windows please use option 1.
  * Software will need to be installed on your local machine
  * Data will need to be downloaded

## 1. Openstack

### Creating an instance on Openstack

A video tutorial to connect to [OpenStack](https://openstack.exeter.ac.uk/) and launching an instance is [here](https://youtu.be/JIoFJBWTtlg). You can also find a step by step walkthrough [here](https://biomedicalhub.github.io/openstack/05-create-instance.html).  

To successfully create an instance you will need to select the following options:

#### Log on details
* Domain: default
* User name: hubtraining_student
* Password: yTpEMUJsfD7z


#### Openstack instance
Please name your instance something that you will remember - including your name is a good idea. Then follow the instructions using the following options:

* Boot source: _Instance Snapshot_
* Name: BIOM528-RA
* Flavour: _u1.xxlarge_

Once the instance has started remember to take note of the IP address (use Edit-Copy).

### Log onto your instance

A video tutorial to connect to your virtual machine using Terminal (Mac) or MobaXterm (windows) is [here](https://youtu.be/qPPBjTSppvE).

The log in details are:
* User name: ubuntu
* Password: zymoseptoria

If you're on Linux or Mac you can complete the workshops simply using SSH `ssh ubuntu@<ip-address>` and enter the password when prompted (you may need to accept permissions when prompted). If you want to access the instance using a graphical interface then use MobaXterm.

Once you have successfully logged on, its time to begin the workshop!

## 2. Using your own machine

It will also be possible to use your own machine to complete the workshop. A Linux machine or Mac would be best suited for these workshops as most of the available software is only available on these systems and access to the command-line is essential.

To successfully use your own machine you will need to:
* Install all required software
* Download the workshop data

### Required software

In these workshops we will be using:
* SOAPdeNOVO2 (genome assembly)
* Augustus (gene prediction)
* BLAST and MCL algorithm (gene family identification)
* RAxML (gene tree inference)
* PAML (Estimation of accelerated evolution)

More detailed information on these software, including installation instructions can be found below.

### Downloading workshop data

There are a few files to download and some organisation required if you are to complete these workshops on your own machine.

First we will download the raw sequence data in 4 parts - use right click and 'Save as':
* [SRR4239880 part 1](data/SRR4239880_subset_part1.fastq)
* [SRR4239880 part 2](data/SRR4239880_subset_part2.fastq)
* [SRR4239881 part 1](data/SRR4239881_subset_part1.fastq)
* [SRR4239880 part 2](data/SRR4239881_subset_part2.fastq)

These files have had to be split into small parts so that they can be stored on GitHub. We will now need to combine them for the workshops. On the command line (Linux and Mac) navigate to the directory storing these files and use the commands:

```
cat SRR4239880_subset_part1.fastq SRR4239880_subset_part2.fastq > SRR4239880_subset.fastq
```

then

```
cat SRR4239881_subset_part1.fastq SRR4239881_subset_part2.fastq > SRR4239881_subset.fastq
```

Next, we need to download two chromosome sequences that can be found [here](data/Z.tritici.chr10.fasta) and [here](data/Z.tritici.chr11.fasta). Again right click the links and chose 'Save as'.

Now, we can download 4 files containing protein sequences that will be used to identify protein families. The files for 4 species of *Zymoseptoria* can be accessed by right clicking and choosing 'Save As' on the species links below:

* [*Z. tritici*](data/Z.tritici.proteins.fasta)
* [*Z. ardabiliae*](data/Z.ardabiliae.proteins.fasta)
* [*Z. brevis*](data/Z.brevis.proteins.fasta)
* [*Z. pseudotritici*](data/Z.pseudotritici.proteins.fasta)

Next, we can download the files used to conduct our evolutionary analysis. We a [file](data/selection_families.txt) that contains a selection of protein families and a set of transcript files:

* [*Z. tritici*](data/Z.tritici.transcripts.fasta)
* [*Z. ardabiliae*](data/Z.ardabiliae.transcripts.fasta)
* [*Z. brevis*](data/Z.brevis.transcripts.fasta)
* [*Z. pseudotritici*](data/Z.pseudotritici.transcripts.fasta)

Finally, we need some organisation of these data files to match the references that will be used throughout the workshop. On the virtual machine, the `Data` directory is located on the `Desktop`. Below is a schematic of the directory structure we will be using:

* Data/
  * fastq/
    * SRR4239880_subset.fastq
    * SRR4239881_subset.fastq
  * Z.tritici/
    * Z.tritici.chr10.fasta
    * Z.tritici.chr11.fasta
  * clusters/
    * Z.tritici.proteins.fasta
    * Z.brevis.proteins.fasta
    * Z.ardabiliae.proteins.fasta
    * Z.pseudotritici.proteins.fasta
  * families/
    * selection_families.txt
    * species/
      * Z.tritici.transcripts.fasta
      * Z.tritici.proteins.fasta
      * Z.brevis.transcripts.fasta
      * Z.brevis.proteins.fasta
      * Z.ardabiliae.transcripts.fasta
      * Z.ardabiliae.proteins.fasta
      * Z.pseudotritici.transcripts.fasta
      * Z.pseudotritici.proteins.fasta

#### SOAPdenovo2 Genome Assembler

To infer a genome sequence from raw DNA sequencing reads we will be using SOAPdenovo2 a *de novo* genome assembler.

The [SOAPdenovo2 GitHub page](https://github.com/aquaskyline/SOAPdenovo2) contains information on the program, its installation and usage. If you're running a linux machine you can download and install the latest version from here.

If you're using a Mac or Linux machine and want to use an older version with a precompiled binary (no installation required) you can download version r240 (compiled for Mac and Linux) from this [sourceforge page](https://sourceforge.net/projects/soapdenovo2/files/SOAPdenovo2/). The binary files are located in the bin directory.

When you have downloaded and installed the program check it runs by executing the command:

```
<path-to-install-dir>/SOAPdenovo-63mer
```

where `<path-to-install-dir>` is the path to where the SOAPdenovo2 executable is installed. If successful you should see something like:

```
Version 2.04: released on July 13th, 2012
Compile Dec 23 2013	10:45:55

Usage: SOAPdenovo <command> [option]
    pregraph        construct kmer-graph
    sparse_pregraph construct sparse kmer-graph
    contig          eliminate errors and output contigs
    map             map reads to contigs
    scaff           construct scaffolds
    all             do pregraph-contig-map-scaff in turn
```

#### Augustus Gene Predictor

In order to predict genes from a genome sequence we will be using [Augustus](https://academic.oup.com/bioinformatics/article/19/suppl_2/ii215/180603)

The Augustus program, complete with tutorials, examples and documentation can be found on [GitHub](https://github.com/Gaius-Augustus/Augustus).

Augustus is primarily available for linux but there is an older version (3.2.1) that has been cloned for Mac OS X. See this [GitHub](https://github.com/nextgenusfs/augustus) page for code and instructions to download and install Augustus 3.2.1 for Mac OS X. Even with this version there can be issues with the auxiliary programs when installing the software. As these extra programs are not required for these workshops you can simply not compile them when installing Augustus.

To not install the auxiliary programs you will need to slightly alter the `MakeFile` in the Augustus directory. You will have to change the line:

```
cd auxprogs && ${MAKE} CC=$(BAMTOOLS_CC) CXX=$(BAMTOOLS_CXX) BAMTOOLS=$(BAMTOOLS)
```

to

```
#cd auxprogs && ${MAKE} CC=$(BAMTOOLS_CC) CXX=$(BAMTOOLS_CXX) BAMTOOLS=$(BAMTOOLS)
```

Adding the `#` at the start of the line makes your compiler interpret this line as a comment so it is never executed and the compiler doesn't try to install the unnecessary auxiliary programs.

When you have downloaded and installed the program check it runs by executing the command:

```
<path-to-install-dir>/augustus
```

where `<path-to-install-dir>` is the path to where the Augustus executable is installed. If successful you should see some outout that starts:

```
AUGUSTUS (3.2.1) is a gene prediction tool
written by M. Stanke, O. Keller, S. König, L. Gerischer and L. Romoth.

usage:
augustus [parameters] --species=SPECIES queryfilename

'queryfilename' is the filename (including relative path) to the file containing the query sequence(s)
in fasta format.

SPECIES is an identifier for the species. Use --species=help to see a list.
```

#### BLAST and MCL algorithm

To identify gene family from predicted protein sequences were are going to use both BLAST and the MCL algorithm.

1. BLAST can be installed by following the instructions [here](https://www.ncbi.nlm.nih.gov/books/NBK279671/). It's available for linux, MacOSX and Windows machines all with pre-compiled binaries or installation packages. Once you have installed BLAST you can check you the required programs are working correctly with:

```
makeblastdb -h
```

where you  should see something along the lines of:

```
USAGE
  makeblastdb [-h] [-help] [-in input_file] [-input_type type]
    -dbtype molecule_type [-title database_title] [-parse_seqids]
    [-hash_index] [-mask_data mask_data_files] [-mask_id mask_algo_ids]
    [-mask_desc mask_algo_descriptions] [-gi_mask]
    [-gi_mask_name gi_based_mask_names] [-out database_name]
    [-max_file_sz number_of_bytes] [-logfile File_Name] [-taxid TaxID]
    [-taxid_map TaxIDMapFile] [-version]

DESCRIPTION
   Application to create BLAST databases, version 2.7.1+

Use '-help' to print detailed descriptions of command line arguments
```

and also:

```
blastp -h
```

which should  give:

```
USAGE
  blastp [-h] [-help] [-import_search_strategy filename]
    [-export_search_strategy filename] [-task task_name] [-db database_name]
    [-dbsize num_letters] [-gilist filename] [-seqidlist filename]
    [-negative_gilist filename] [-negative_seqidlist filename]
    [-entrez_query entrez_query] [-db_soft_mask filtering_algorithm]
    [-db_hard_mask filtering_algorithm] [-subject subject_input_file]
    [-subject_loc range] [-query input_file] [-out output_file]
    [-evalue evalue] [-word_size int_value] [-gapopen open_penalty]
    [-gapextend extend_penalty] [-qcov_hsp_perc float_value]
    [-max_hsps int_value] [-xdrop_ungap float_value] [-xdrop_gap float_value]
    [-xdrop_gap_final float_value] [-searchsp int_value]
    [-sum_stats bool_value] [-seg SEG_options] [-soft_masking soft_masking]
    [-matrix matrix_name] [-threshold float_value] [-culling_limit int_value]
    [-best_hit_overhang float_value] [-best_hit_score_edge float_value]
    [-window_size int_value] [-lcase_masking] [-query_loc range]
    [-parse_deflines] [-outfmt format] [-show_gis]
    [-num_descriptions int_value] [-num_alignments int_value]
    [-line_length line_length] [-html] [-max_target_seqs num_sequences]
    [-num_threads int_value] [-ungapped] [-remote] [-comp_based_stats compo]
    [-use_sw_tback] [-version]

DESCRIPTION
   Protein-Protein BLAST 2.7.1+

Use '-help' to print detailed descriptions of command line arguments
```

2. The [MCL algorithm](https://micans.org/mcl/) can be [downloaded](https://micans.org/mcl/src/) and installed by following the instructions from this [page](https://micans.org/mcl/index.html?sec_license). To follow these instructions you will need to be using a Linux or Mac system.

When you have downloaded and installed the MCL algorithm check it runs by executing the command:

```
<path-to-install-dir>/mcl
```
where `<path-to-install-dir>` is the path to where the MCL executable is installed. You may not need the path here if you have followed the installation instructions correctly. If successful you should see some outout that starts:

```
[mcl] usage: mcl <-|file name> [options], do 'mcl -h' or 'man mcl' for help
```

#### Muscle

In order to align sequences we will need to use software for Multiple Sequence Alignment. For this we will use [Muscle](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-5-113), a fast aligner for multiple sequences. A download page for precompiled Muscle executables can be found [here](https://www.drive5.com/muscle/downloads.htm). Once you have downloaded the software and put it somewhere it can be accessed (either in your HOME directory or on your PATH) try:

```
muscle
```

**OR**

```
muscle3.8.31_i86darwin64
```

You should then see the following output:

```
MUSCLE v3.8.31 by Robert C. Edgar

http://www.drive5.com/muscle
This software is donated to the public domain.
Please cite: Edgar, R.C. Nucleic Acids Res 32(5), 1792-97.


Basic usage

    muscle -in <inputfile> -out <outputfile>

Common options (for a complete list please see the User Guide):

    -in <inputfile>    Input file in FASTA format (default stdin)
    -out <outputfile>  Output alignment in FASTA format (default stdout)
    -diags             Find diagonals (faster for similar sequences)
    -maxiters <n>      Maximum number of iterations (integer, default 16)
    -maxhours <h>      Maximum time to iterate in hours (default no limit)
    -html              Write output in HTML format (default FASTA)
    -msf               Write output in GCG MSF format (default FASTA)
    -clw               Write output in CLUSTALW format (default FASTA)
    -clwstrict         As -clw, with 'CLUSTAL W (1.81)' header
    -log[a] <logfile>  Log to file (append if -loga, overwrite if -log)
    -quiet             Do not write progress messages to stderr
    -version           Display version information and exit
```

#### PAL2NAL

We will also have to reformat alignments and sequences as part of our analysis pipeline. To accomplish this task we will use [PAL2NAL](http://www.bork.embl.de/pal2nal/), a Perl script that converts a protein alignment and associated coding sequences into a into codon alignment. You can download the latest version [here](http://www.bork.embl.de/pal2nal/#Download). Place the Perl script on your path or in the current directory and run with:

```
pal2nal.pl
```

Then you should see the following output:

```
pal2nal.pl  (v14)

Usage:  pal2nal.pl  pep.aln  nuc.fasta  [nuc.fasta...]  [options]


    pep.aln:    protein alignment either in CLUSTAL or FASTA format

    nuc.fasta:  DNA sequences (single multi-fasta or separated files)

    Options:  -h            Show help

              -output (clustal|paml|fasta|codon)
                            Output format; default = clustal

              -blockonly    Show only user specified blocks
                            '#' under CLUSTAL alignment (see example)

              -nogap        remove columns with gaps and inframe stop codons

              -nomismatch   remove mismatched codons (mismatch between
                            pep and cDNA) from the output

              -codontable  N
                    1  Universal code (default)
                    2  Vertebrate mitochondrial code
                    3  Yeast mitochondrial code
                    4  Mold, Protozoan, and Coelenterate Mitochondrial code
                       and Mycoplasma/Spiroplasma code
                    5  Invertebrate mitochondrial
                    6  Ciliate, Dasycladacean and Hexamita nuclear code
                    9  Echinoderm and Flatworm mitochondrial code
                   10  Euplotid nuclear code
                   11  Bacterial, archaeal and plant plastid code
                   12  Alternative yeast nuclear code
                   13  Ascidian mitochondrial code
                   14  Alternative flatworm mitochondrial code
                   15  Blepharisma nuclear code
                   16  Chlorophycean mitochondrial code
                   21  Trematode mitochondrial code
                   22  Scenedesmus obliquus mitochondrial code
                   23  Thraustochytrium mitochondrial code


              -html         HTML output (only for the web server)

              -nostderr     No STDERR messages (only for the web server)


    - sequence order in pep.aln and nuc.fasta should be the same.

    - IDs in pep.aln are used in the output.
```

#### RAxML

For some of our analyses we will need to infer some phylogenetic trees. To do this we will use [RAxML](https://cme.h-its.org/exelixis/web/software/raxml/). The homepage contains useful links to the [GitHub repository](https://github.com/stamatak/standard-RAxML), which includes Windows executables, the [manual](https://cme.h-its.org/exelixis/resource/download/NewManual.pdf) and guides for installation on [Mac OSX](http://www.sfu.ca/biology2/staff/dc/raxml/).

Once installed you can run:

```
raxmlHPC-AVX -v
```

and you should see some output like:

```
This is RAxML version 8.2.12 released by Alexandros Stamatakis on May 2018.

With greatly appreciated code contributions by:
Andre Aberer      (HITS)
Simon Berger      (HITS)
Alexey Kozlov     (HITS)
Kassian Kobert    (HITS)
David Dao         (KIT and HITS)
Sarah Lutteropp   (KIT and HITS)
Nick Pattengale   (Sandia)
Wayne Pfeiffer    (SDSC)
Akifumi S. Tanabe (NRIFS)
Charlie Taylor    (UF)
```

#### PAML

We will be using [PAML](http://abacus.gene.ucl.ac.uk/software/paml.html), which has extensive documentation and installation instructions. Please follow the instructions to install PAML on your machine. Instructions are included for Windows, Mac OSX (including precompiled binaries) and Linux.

Once you have the data and all software installed you're ready to being the workshop.
