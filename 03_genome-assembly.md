---
layout: page
title: Genome assembly
order: 3
---

## Genome assembly

The start of our analysis pipeline begins by assembling a genome from some DNA reads. The reads are produced by whole genome sequencing

### Raw sequencing data

Our raw sequencing data comes from [Goodwin *et al.* 2011](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1002070), which published the first completed genome of *Z. tritici* (synonym *Mycosphaerella graminicola*) and characterised the structure and gene content of the genome. The raw reads, as well as an assembled genome and predicted genes can be found at the NCBI under the project ID [PRJNA19047](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA19047).

![NCBI](/images/NCBI.png)

The screenshot shows the NCBI entry for the genome data. You can see information about the data type, organism and where the data is submitted. At the bottom of the page there is information on how to access the raw data through the NCBI. These links include the whole genome sequence (WGS), protein sequences and SRA experiments. The SRA stands for Sequence Read Archive (SRA) and this is where the raw genome reads are stored. If we follow this link there are 16 files that can be accessed, each storing reads from the genome sequencing project. We can download these files either through the NCBI's webpages or using command line tools such as the [sra-toolkit](https://www.ncbi.nlm.nih.gov/books/NBK158900/), which can download files and convert them between different useful formats.

Raw sequencing data are stored in a file format named fastq. A fastq file has the format:

```
@SRR4239881.1 1 length=39
AGTTACCTAGGGAGAAGTCCTAGCTACAATCTAGAGGNN
+SRR4239881.1 1 length=39
I'IIIIIIIIII%I<9%I<;I'=04+2)+447'*&;.$$
```

Each read in a fastq file consists of 4 lines. The first line is an ID, in this case `SRR4239881.1` and contains information about the lengths of the read. The second line contains the DNA sequence of the read, which will be sequenced region of the genome or mRNA. The third line is a repeat of the ID line. Finally, the fourth line is an ASCII code that represents the quality of the above DNA sequence, where each character is a Phred quality score and gives us information on how reliable each base call was in the sequenced region of DNA. Although the general format of a fastq file remains the same some of the information and encoding can be different between sequencing technologies and storage databases. For this workshop we will be using *Z. tritici* data downloaded from the NCBI's SRA. You can find specific information on the NCBI's fastq format [here](https://en.wikipedia.org/wiki/FASTQ_format#NCBI_Sequence_Read_Archive) and on the quality scores used by different technologies [here](https://en.wikipedia.org/wiki/FASTQ_format#Encoding).

Fastq files are large, with sequencing experiments being able to generate millions to billions of reads. This means that the file sizes are also large and often in the GB range. In fact, each of the 16 fastq files available are around 1GB in size and contain ~30 million reads. Using the whole data set to assemble a genome will be slow, in both downloading the data and running the assembly software. Therefore, in this workshop we will use a cutdown version of the sequencing data. Our results won't be correct but running the software will serve to teach the basics of genome assembly.

In this workshop we will use the first million reads in just 2 of the fastq files (SRR4239880, and SRR4239881) from project PRJNA19047. If you were to download these data yourself and wanted to extract the first 1 million reads, this can be achieved using some simple Linux commands. Navigate to the directory containing the fastq files and use:

```
head -n 4000000 SRR4239880.fastq > SRR4239880_subset.fastq
```

This command uses `head`, which is a program to view the initial contents of a file. By default `head` only views the first 10 lines of the file but we can use the `-n` option to specify the number of lines to output. Here, we use `head` to output the first 4 million lines (corresponds to 1 million reads) of the the input file `SRR4239880.fastq`. Finally, we use a redirect `>` to redirect the output of the command from the screen to a new file called `SRR4239880_subset.fastq`. The resulting file contains the first 1 million reads and will be much more tractable to analyse during the workshop. This step has already been done for you.

You will find the data by moving to the data directory that stores the fastq files using:

```
cd ~/Desktop/Data/fastq/
```

If you then use the command `ls` to list files in the directory you will see two subsetted fastq files. You can view the format of these files with the Linux command `less`.

```
less SRR4239880_subset.fastq
```

You can scroll through the first few lines to see what our actual data looks like. Once you are finished press `q` to quit the program.

## Genome assembly with SOAPdenovo2

### Introduction to SOAPdenovo2

In this workshop we will use [SOAPdenovo2](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3626529/), a software that uses *de Bruijn* graphs to assemble DNA reads into contigs (longer sequences of DNA) and then construct scaffolds (chromosome like sequences). SOAPdenovo2 is an update to the previous software where the major improvements are: 1) enhancing the error correction algorithm, 2) providing a reduction in memory consumption, 3) resolving longer repeat regions in contig assembly, 4) increasing assembly length and coverage in scaffolding and 5) improving gap closure.

The software [GitHub](https://github.com/aquaskyline/SOAPdenovo2) page allows us to download the software (see the prerequisities page for installation instructions for this workshop) and gives us information about running the tool. The first thing to note is that SOAPdenovo2 requires a configuration file to run. This is common for bioinformatics software that has lots of input options, which would make the command line difficult to read and understand if the options needed to be provided that way. The configuration file should have the following information:

```
1) avg_ins
   This value indicates the average insert size of this library or the peak value position in the insert size distribution figure.
2) reverse_seq
   This option takes value 0 or 1. It tells the assembler if the read sequences need to be complementarily reversed.
Illumima GA produces two types of paired-end libraries: a) forward-reverse, generated from fragmented DNA ends with typical insert size less than 500 bp; b) reverse-forward, generated from circularizing libraries with typical insert size greater than 2 Kb. The parameter "reverse_seq" should be set to indicate this: 0, forward-reverse; 1, reverse-forward.
3) asm_flags
   This indicator decides in which part(s) the reads are used. It takes value 1(only contig assembly), 2 (only scaffold assembly), 3(both contig and scaffold assembly), or 4 (only gap closure).
4) rd_len_cutof
   The assembler will cut the reads from the current library to this length.
5) rank
   It takes integer values and decides in which order the reads are used for scaffold assembly. Libraries with the same "rank" are used at the same time during scaffold assembly.
6) pair_num_cutoff
   This parameter is the cutoff value of pair number for a reliable connection between two contigs or pre-scaffolds. The minimum number for paired-end reads and mate-pair reads is 3 and 5 respectively.
7) map_len
   This takes effect in the "map" step and is the minimun alignment length between a read and a contig required for a reliable read location. The minimum length for paired-end reads and mate-pair reads is 32 and 35 respectively.
```

### The config file

Let's create our own config file called `config.txt`, based off the template available on the GitHub page, that we will use to run our own analysis. Use your favourite text editor (vi, vim, nano) to edit a new file ad input the following information:

```
#maximal read length
max_rd_len=39
[LIB]
#average insert size
avg_ins=2000
#if sequence needs to be reversed
reverse_seq=0
#in which part(s) the reads are used
asm_flags=3
#use only first 100 bps of each read
rd_len_cutoff=100
#in which order the reads are used while scaffolding
rank=1
# cutoff of pair number for a reliable connection
pair_num_cutoff=3
#minimum aligned length to contigs for a reliable read location
map_len=32
# A fastq file
q=./SRR4239881_subset.fastq
q=./SRR4239880_subset.fastq
```

Save the file as `config.txt`.

Our config file contains lots of information so lets go through exactly what options we have used.
* `max_rd_len` specifies the maximum read length in our input data. A quick look at the fastq files has the information `length=39`.
* `[LIB]` next we specify a library - some sequencing projects can use multiple libraries, each of which may need different parameters but our toy example only uses 1 library. All parameters specified below this tag will be used for this library.
* `avg_ins` is our average insert size during library preparation for sequencing. We can see in the original manuscript by [Goodwin *et al.* 2011](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1002070) that they used three libraries with insert sizes ranging from 2kb to 40kb. For our example we're assuming a single library with the smallest insert size of 2kb.
* `reverse_seq` some paired-end sequencing technologies produce reads from both DNA strands, meaning that some sequences will need to be reverse complemented during assembly. This is not the case for this example so we can leave this to 0 (False).
* `asm_flags` signifies an option for using reads for both contig and scaffold assembly.
* `rd_len_cutoff` tells SOAPdenovo2 to use only the first 100 bases of reads. This can help trim reads if the ends are of poor quality, but for our analysis does nothing as the read lengths are less than this cutoff.
* `rank` ranks the different libraries to determine which order they are used for scaffold assembly. In our case this parameter won't make any difference as we have a single library.
* `pair_num_cutoff` this dictates a a reliable connection between contigs for merging them into scaffolds. We have left this parameter as the default.
* `map_len` a minimum alignment length to map a read to a contig. Given our read length is 39 a map length of 32 is quite stringent as the majority of the read will have to map to be included.

Finally, we have the `q` parameter, which specifies our input fastq files. As this library has two files we specify two `q` parameters, but we could specify one or many more depending on our sequencing experiment. It is also worth noting that SOAPdenovo2 can handle different input file types that includes fastq (`q`), fasta (`f`) and BAM (`b`). Sequencing experiments can also produce paired data, where a DNA fragment is sequenced from both ends, which produces two read files (often denoted R1 and R2) that store the paired reads. You could specify paired end fastq files in SOAPdenovo2 with `q1` and `q2` parameters.

### Running the Software

It's now time to run SOAPdenovo2 and build our genome assembly. In our working directory we have our 2 fastq files and our created config file. To actually run SOAPdenovo2 we use:

```
SOAPdenovo>SOAPdenovo-63mer all -s config.txt -K 13 -p 4 -o z.tritici 2>assembly.log
```

Let's break this command down. We start by specifying the program to run, `SOAPdenovo-63mer`, which is a specific version of SOAPdenovo2. Then we specify `all`, which runs the full SOAPdenovo2 pipeline programs (including pregraph, contig, map, scaff). We could run each of these steps individually but the `all` command makes things simple. We then tell SOAPdenovo, Kmer size to use with the `-K` flag, this is the size of the sequence used to align reads and build contigs/scaffolds. We use a small value her as our reads are short. Next is `-p 4` to tell SOAPdenovo2 to use 4 CPUs to run calculations in parallel and speed up the processing. Next is the `-o` flag where we can specify a `z.tritici` prefix for the output files. Finally, we use a redirect `2>assembly.log` to redict the logging information to a file called  `assembly.log`. We do this so that we can save the running information about which parameters have been used and for debugging in case the program crashes or was unable to finish.

SOAPdenovo2 should run for about 3 minutes before finishing. Use `ls` to list the files in our working directory. You should see a bunch of new files beginning with the prefix of `z.tritici`. To see a comprehensive list of output files see [Appendix B](https://github.com/aquaskyline/SOAPdenovo2). You can see that each program in the pipeline produces multiple output files, however we are only really interested in 2 of these.

### SOAPdenovo2 outputs

The first is the contig file that appears as `z.tritici.contig`. We can use `less` to see the contents of this file:

```
less z.tritici.contig
```

Should provide something that looks like:

```
>1 length 14 cvg_0.0_tip_0
AAAAAAAAAAAAAT
>3 length 14 cvg_0.0_tip_0
AAAAAAAAAAAAAG
>5 length 14 cvg_0.0_tip_0
AAAAAAAAAAAAAA
>7 length 14 cvg_0.0_tip_0
AAAAAAAAAAAAAC
```

That shows a range of contigs, short genomic sequences assembled from the raw reads, in fasta format. We can use some simple Linux command to see how many contigs we have in our assembly:

```
grep '>' z.tritici.contig | wc -l
```

This command, which uses `grep` to extract all the headers (lines that start with `>`) from the contigs file and then pipes `|` the output into word count (`wc`) to  count all the headers tells us that there are 743,371 contigs in our assembly. For a normal genome assembly this is very high and the contig sequences are very short but that is likely down to our subsetted input data.

The next file we are most interested in is the `z.tritici.scafSeq` file that provides our scaffolds, which resemble the chromosome sequences for the sequenced organism. As before we can use `less` to take a look at the contents of the file:

```
less z.tritici.scafSeq
```

Which produces some longer scaffold sequences:

```
>C1485088  7.0
CTTTAACATTAACGTCACTACTTAGTCATACATTAATTCTATCTGAGTTACCCTTTATTACAATTTGTCCGTACCCTTAATATTGGAGTTTAGCAAGAAC
>C1485090  1.0
TAAATATTGTATTCGACAAGGCCCTTCAGCCGGACTCCTTTAAGATTAAGGTACTCCTAGATAGAACTATAGAGCGCAAGGATGCGCATACGGATAGCTA
>C1485092  1.0
GCCCCGTGCCATTGCACGTGCTGCTTCGTTAACCGGACTACTAAGGTCTAGATTCGCCCTAGAGAAGTACTCTAGATAGGCCCTATACTAACGTGCGATG
```

Here, we have a scaffold ID on the header line `>` with the sequence following below. We can see that the scaffolds are longer genomic sequences that could be used for further analysis. Using our counting command from earlier we can identify the number of scaffolds in the file with:

```
grep '>' z.tritici.scafSeq | wc -l
```

Which tells us we have 18 scaffold sequences. Compared to the contig sequences there are much fewer sequences and they are longer. The sequences also look much more like potential genomic sequences in their composition. However, as with the contig file, these sequences are still short, likely because of our subsetted input data. If we were to run SOAPdenovo2 on the full data set of ~480,000,000 reads rather than this cut down input data set of 4,000,000 reads, we would see much longer scaffold sequences that better resemble the underlying genome. If we look at the original [paper](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1002070) the authors assembled the full data set into 128 scaffolds containing 41.2 Million bases.

## Summary

We have now gained experience of using a genome assembly program, SOAPdenovo2, to take the raw reads of a sequencing experiment and assemble them into a set of chromosome-like sequences called scaffolds. Specifically, you will have:

* Learned about the fastq file format that stores raw sequencing reads.
* Run SOAPdenovo2 with a configuration file.
* Understood the output files that store scaffold sequences

Next we will learn how to extract gene regions from our scaffolds.
