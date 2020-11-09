---
layout: page
title: Setup
order: 1
---
# Workshop setup

There are two ways of completing these wokshops:

1. Use a virtual machine on OpenStack
  * You will need to create an instance on OpenStack
  * All software and data needed will be pre-installed
  * An internet connection will be required (+ a VPN for off-campus access)
2. Use your own machine
  * A mac or linux machine will be best
  * Software will need to be installed on your local machine
  * Data will need to be downloaded

## 1. Openstack

### Creating an instance on Openstack

A video tutorial to connect to OpenStack and launching an instance is [here](https://youtu.be/JIoFJBWTtlg)

To successfully create an instance you will need to select the following options:

#### Log on details
* Domain:
* User name:
* Password:


#### Openstack instance
Please name your instance something that you will remember - including your name is a good idea. Then follow the instructions using the following options:

* Boot source: _Instance Snapshot_
* Name:
* Flavour: _u1.xxlarge_

Once the instance has started remember to take note of the IP address (use Edit-Copy).

### Log onto your instance

A video tutorial to connect to your virtual machine using Terminal (mac) or MobaXterm (windows) is [here](https://youtu.be/qPPBjTSppvE)

The log in details are:
* User name:
* Password:

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
* MCL algorithm (gene family identification)
* RAxML (gene tree inference)
* PAML (Estimation of accelerated evolution)

More detailed information on these software, including installation instructions can be found below.

### Downloading workshop data

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
