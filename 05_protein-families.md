---
layout: page
title: Protein family identification
order: 5
---

## Protein families

We've now seen how we can use gene prediction programs to identify potential coding sequences and proteins within a genome assembly. Now we will see how we can take protein sequences from 4 different genomes and group them into protein families. This is useful because we can compare the sequences within protein families to learn about their function and their evolution.

## Identifying protein families

As with many types of bioinformatics analysis there are multiple tools that can be used to identify protein families. In this workshop we will use the [Markov Cluster](https://micans.org/mcl/) (MCL) algorithm. MCL is a fast and scalable unsupervised cluster algorithm for graphs (also known as networks) based on simulation of (stochastic) flow in graphs. Although there are many uses of MCL a specific workflow is the clustering of proteins based on their sequence similarities, which can be used to identify protein families.

## Input data

To run this analysis we will use the proteome (all protein sequences encoded in a genome) of 4 species of *Zymoseptoria*, which can be downloaded from the [JGI](https://jgi.doe.gov/). Our species will be *Z. tritici*, *Z. pseudotritici*, *Z. ardabiliae* and *Z. brevis*. All sequences (including genome assemblies, coding sequences and protein sequences) are available from the JGI's [Genome Portal](https://genome.jgi.doe.gov/portal/), simply search the species name in the search box and find the appropriate files.

![GenomePortal](/images/GenomePortal.png)

If you want to do this yourself you will need to create an account with your University email. Otherwise the required data is available from ...

## Steps of our analysis

To infer our protein families we will first have to complete multiple preparation steps. The workflow for this part of our analysis will be:

1. Reformat our protein sequences
2. Run BLAST to calculate protein similarity
3. Run MCL to identify protein families

### Data reformatting

Our initial data is in fasta format. We can use the `head` command to view the first 10 lines of one of our fasta files. Navigate to the directory containing the fasta files and use the command:

```
head Z.tritici.fasta
```

You will see an output that looks like:

```
>jgi|Mycgr3|90311|fgenesh1_pg.C_chr_2000001
MTELESRICRGDPKQQSSTHTPPRAASKKCGTKAAFRTGTVPRYGVEELPITATDLTRPVAIMASNTQFT
ASLPTLRLPLDNNSDSSGRTQLYDVLSQPREILDAVDQWLELDSTDGFAMRGSTERG*
>jgi|Mycgr3|107801|estExt_fgenesh1_pg.C_chr_20002
MARSQALHTNMGVDYVIVYRFPPTEKAKAAASFEQLCQKLSSVGLATESRNGEGSSVLLFVKVANEEHMF
AEVYRSRVKDWIHGVRSGEPARDTRESLESQPLTAAERLRIIYQLITNPENEGGAGIRPKSEEYPYVENI
FSLHDHTFNKHWIKKWATNYKIEVDDLDEIRDRFGEKVAFYFAFEQTYFNFLLFPAAFGTLAWLFLGGFS
TIYAMAISLWSVVFVEWWKHRERDLAIRWGVRGVGAIDTKRTGFVPTTEVEDPATGEIQKIFPATERVKR
QLLQLPFAILAIVVLGSLIFACFAIEIFIGEIYDGPGKSILAFTPTVILTTCLPLLTGALSNMAKQLNEY
ENYETDSSYDRSYTGKLFVLDFITSYTGIILTAFVYVPFGSVLKPYLNIFARLFGSHQSNLKTDSVNSHA
```

This is the traditional fasta format for storing sequences. Each sequence contains a header denoted by `>` followed by the ID of the sequence. In this case the ID contains 4 fields each separated by `|`. The first field is the sequence source `jgi`, the second field denotes the genome/organism ID `Mycgr3`, in this case this abbreviation stands for *Mycosphaerella graminicola* a synonym for *Z. tritici*. The next field contains the gene or protein ID and the final field contains the scaffold or chromosome for the gene.

The first thing we want to do is combine all our sequences into a single file so that we can use blast to calculate protein similarity.

```
cat Z.tritici.proteins.fasta Z.pseudotritici.proteins.fasta Z.ardabiliae.proteins.fasta Z.brevis.proteins.fasta > all_seqs.fa
```

Here we use the `cat` command to concatenate a list of files (the `proteins.fasta` file) and redirect `>` the output into a new file called `all_seqs.fa`. We can use our common header counting command to see how many sequences we have:

```
grep '>' all_seqs.fa | wc -l
```

Which tells us that we have 43,331 sequences in our file.

However, when we identify our protein families, the programs will use the whole header lines as the IDs. Normally, this wouldn't be a problem but the `jgi` tag and scaffold information is not useful and will only serve to clutter the output files. Luckily, Linux is incredibly powerful and makes quickly  making minor changes to very large files easy.

Let's begin by removing the scaffold information from the header lines with the command:

```
sed 's/\(.*\)|\(.*\)/\1/' < all_seqs.fa > all_seqs_new.fa
```

This command looks confusing so lets break it down. We start with calling the program `sed`, which is a stream editor used to make basic text transformations on a stream of text. Next we specify a substitution regex `s/\(.*\)|\(.*\)/\1/`, which substitutes `s` everything after the first `/` with what's after the second `/`. We start by searching the input for `\(.*\)|\(.*\)`, where `\(.*\)` just means any text `.` of any length `*` either side of a `|`. Here's the tricky bit, there are many `|` in the input header lines but `sed` is what's known as greedy when it comes to matching text so it actually matches the last instance of `|` in the header line. So if we take an example header line:

```
>jgi|Mycgr3|90311|fgenesh1_pg.C_chr_2000001
```

Our `sed` code matches the text before the last `|`, `>jgi|Mycgr3|90311` and everything after the last `|`, `fgenesh1_pg.C_chr_2000001`. We then substitute the matched text with `\1`, which is everything within the first set of `()` or `>jgi|Mycgr3|90311`. Finally we need to tell `sed`, which text to manipulate so we input `<` our fasta file `all_seqs.fa` and redirect `>` the output to a new file `all_seqs_new.fa` to save the results.

We can take a look at what we have done with:

```
head all_seqs_new.fa
```

which should produce:

```
>jgi|Mycgr3|90311
MTELESRICRGDPKQQSSTHTPPRAASKKCGTKAAFRTGTVPRYGVEELPITATDLTRPVAIMASNTQFT
ASLPTLRLPLDNNSDSSGRTQLYDVLSQPREILDAVDQWLELDSTDGFAMRGSTERG*
>jgi|Mycgr3|107801
MARSQALHTNMGVDYVIVYRFPPTEKAKAAASFEQLCQKLSSVGLATESRNGEGSSVLLFVKVANEEHMF
AEVYRSRVKDWIHGVRSGEPARDTRESLESQPLTAAERLRIIYQLITNPENEGGAGIRPKSEEYPYVENI
FSLHDHTFNKHWIKKWATNYKIEVDDLDEIRDRFGEKVAFYFAFEQTYFNFLLFPAAFGTLAWLFLGGFS
TIYAMAISLWSVVFVEWWKHRERDLAIRWGVRGVGAIDTKRTGFVPTTEVEDPATGEIQKIFPATERVKR
QLLQLPFAILAIVVLGSLIFACFAIEIFIGEIYDGPGKSILAFTPTVILTTCLPLLTGALSNMAKQLNEY
ENYETDSSYDRSYTGKLFVLDFITSYTGIILTAFVYVPFGSVLKPYLNIFARLFGSHQSNLKTDSVNSHA
```

Here, we can see that we have succeeded in removing the scaffold information from the header lines. Interestingly, none of the sequences are changed and that is because no sequence lines contain `|`, which has to be found in order to do the substitution. Next, we want to remove the `jgi` tag and we can do this in a very similar way:

```
sed 's/jgi|//' < all_seqs_new.fa > all_seqs.fa
```

As before we used `sed` to substitute `s` some input input text. This time we simply substitute the JGI tag `jgi|` with nothing (there is no text after the second `/`). We use `all_seq_new.fa` as our input `<` and redirect `>` the output to overwrite our exiting file `all_seqs.fa`. We can check everything has worked:

```
head all_seqs.fa
```

to give:

```
>Mycgr3|90311
MTELESRICRGDPKQQSSTHTPPRAASKKCGTKAAFRTGTVPRYGVEELPITATDLTRPVAIMASNTQFT
ASLPTLRLPLDNNSDSSGRTQLYDVLSQPREILDAVDQWLELDSTDGFAMRGSTERG*
>Mycgr3|107801
MARSQALHTNMGVDYVIVYRFPPTEKAKAAASFEQLCQKLSSVGLATESRNGEGSSVLLFVKVANEEHMF
AEVYRSRVKDWIHGVRSGEPARDTRESLESQPLTAAERLRIIYQLITNPENEGGAGIRPKSEEYPYVENI
FSLHDHTFNKHWIKKWATNYKIEVDDLDEIRDRFGEKVAFYFAFEQTYFNFLLFPAAFGTLAWLFLGGFS
TIYAMAISLWSVVFVEWWKHRERDLAIRWGVRGVGAIDTKRTGFVPTTEVEDPATGEIQKIFPATERVKR
QLLQLPFAILAIVVLGSLIFACFAIEIFIGEIYDGPGKSILAFTPTVILTTCLPLLTGALSNMAKQLNEY
ENYETDSSYDRSYTGKLFVLDFITSYTGIILTAFVYVPFGSVLKPYLNIFARLFGSHQSNLKTDSVNSHA
```

Which now shows that our header lines only contain the organism ID and the gene ID. Everything we will need for the remainder of our analysis.

Finally we can clean up our temporary file `all_seqs_new.fa` with:

```
rm -f all_seqs_new.fa
```

### Running BLAST

Now that we have our input sequences nicely formatted we can move on to calculating sequence similarities. To do this we are going to use BLAST. BLAST,which stands for Basic Local Alignment Search Tool is probably the most popular similarity search tool. Sequence similarity searching is one of the more important bioinformatics activities and often provides the first evidence for the function of a newly sequenced gene or piece of sequence. Some information on running BLAST for the type of analysis we're doing in this workshop can be found [here](https://www.ncbi.nlm.nih.gov/books/NBK279680/). The full user manual including installation instructions, detailed reference manuals and cookbooks can be found [here](https://www.ncbi.nlm.nih.gov/books/NBK279690/).

Before we start, we're going to make a small change to our sequences file. Currently, we have 43,331 sequences and if we were to calculate the sequence similarity of every sequence against all others (this is what we want to do!), we will be performing 43,331 x 43,331 calculations. This is a lot, and even though BLAST is extremely quick it will still take hours for this section of the workshop to run. Instead, we will use a subset of our data. We can randomly select 1,000 sequences using the following short script:

```
cat all_seqs.fa |\
awk '/^>/ { if(i>0) printf("\n"); i++; printf("%s\t",$0); next;} {printf("%s",$0);} END { printf("\n");}' |\
shuf |\
head -n 1000 |\
awk '{printf("%s\n%s\n",$1,$2)}' > all_seqs_subset.fa
```

The script has 4 lines each separated by `\`, which just allows multiple lines on the terminal to be executed at once. Note that each line also ends with a pipe `|` meaning that all these commands are chained together staring with the first command. `cat` reads in the `all_seqs.fa` and pipes the fasta text into a complex `awk` command that, briefly, puts each header and associated sequence on the same line. Next, a call to `shuf` shuffles these sequences, effectively so we can take a random sample. With our now randomly sorted sequences we next use `head` with `-n` to select the first 1,000 lines (remember that each header and sequence is on one line). Finally, we use `awk` again to split the header ID and sequences and use a redirect `>` to print the output to a new file called `all_seqs_subset.fa`.

We can view the first few lines of the random file with:

```
head all_seqs_subset.fa
```

which will show:

```
>Zymbr1|3367
MPPRLQLLRRACVALPTQQFYQPLAPFLYPFFTQQRSASILSSLSDTKGAYNKKIRRGRGPASGKGKTSGRGMNGQKQHGKVPAGFNGGQTPDWVVSGSRGFENHFSVDMSKVNLNRIQYWINQGRLDPSKPITLRELNKSRCTHGIKDGIKLLARGKEELTTPINIVVSRASADAIAAVEKLGGTVTTRYYTNFAISKILKGEMDPINSLQSRASTAAATGVVEEDAAQVLEQDKKVYQYRLPDPTSRKHLEYYRDSAHRGYLSHLVEAGQGPSLFFKTPGSGKKDGKKTPGKATKTQQRAENRLW*
>Mycgr3|84938
MASSPPMKEIRPIPFLHPTQAIYSGLPQAESAFTCPADKKPLPGHVQYKWTSRDNRKGRHTLVVPRASPHNVATPPPTTTFKATLHGLWRMLTYYPYWDISYLVAVFFTLGSLVWVLNSFFVFLPLANPRSNFPGEVLYGGGITAFIGVIIFEIGGALAVLEAVNENRERCFGWAVARLYAGRKSVDGENDDDDDAVKILPLLDPKPMHTYSWHPVIPSRRRRKRWVWWPSPEALRMHHSRSIGFLASLILFVSATIFFISGFTSLPGIINTMNRTQTCILYWVPQLIGGVGFVVSGWMFMIETQQHWWKPAVDVLGWHVGFWNFVGGIGFLLCPCFGFDSQPWAQYQACLATLWGSWAFLIGSVVQWYECLEKNPVVVARR*
```

We can see that the order of sequences has changed (as well as sequences now appear all on one line) with the first sequence belonging to *Z. brevis* and the second *Z. tritici*. We will continue the next few stages of our analysis using the subsetted data.

We start by creating a database from our fasta file of protein sequences. To do this we will use the BLAST accessory program `makeblastdb`. To create the database we use the program in its most simple form:

```
makeblastdb -in all_seqs_subset.fa -dbtype prot
```

All we have to do here is specify the input file `-in all_seqs.fa` and specify the database type, which can be either `prot` or `nucl`. The output printed to the screen gives a brief summary of the running of the program.

```
Building a new DB, current time: 11/06/2020 12:16:45
New DB name:   /Users/Ryan/Data/BIOM528/clusters/all_seqs_subset.fa
New DB title:  all_seqs_subset.fa
Sequence type: Protein
Keep MBits: T
Maximum file size: 1000000000B
Adding sequences from FASTA; added 1000 sequences in 0.0236759 seconds.
```

Which tells us that all 1,000 sequences were added to the database in less than a 0.025 seconds. If we use the `ls` command to list the files in the directory we can see that three new files with the suffixes `phr`, `pin` and `psq` have been created. These are index files used by BLAST to speed up searching of the dataabse when we run the database query command.

To search the database we will be using the program `blastp`. We can perform the sequence similarity search with:

```
blastp -query all_seqs_subset.fa -db all_seqs_subset.fa -out all_seqs_subset.results -outfmt 7
```

This command calls `blastp` the protein sequence search version of BLAST. We pass a `-query` file as input and give it the name of our database using `-db`, the database is the input file we used with `makeblastdb`. Note, here that our query and database are the same as we are trying to search each sequence against all others. Finally, we specify and output file name with `-out` to store our results and an output format with `-outfmt`, which will prodce a tabular format we will need for the MCL algorithm. It will take a minute or two to run while it compares each sequence against all others. We can take a look at the produced output file with:

```
head all_seqs_subset.results
```

which shows:

```
# BLASTP 2.7.1+
# Query: Zymbr1|3367
# Database: all_seqs_subset.fa
# Fields: query acc.ver, subject acc.ver, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score
# 6 hits found
Zymbr1|3367	Zymbr1|3367	100.000	308	0	0	1	308	308	0.0	636
Zymbr1|3367	Zymps1|800588	35.294	51	25	2	221	266	977	1024	0.28	28.5
Zymbr1|3367	Mycgr3|90232	27.451	51	35	1	60	110	202	250	5.5	24.3
Zymbr1|3367	Zymbr1|9039	26.582	79	49	4	125	200	156	228	6.7	23.9
Zymbr1|3367	Zymar1|768610	31.818	44	27	1	49	89	382	425	7.9	23.5
```

In this file lines that start with `#` are comment lines and give some information about the current search results. We can see the BLAST version, query sequence, database, description of the fields and number of hits. The following lines show our results. On the first result line we can see that sequence `Zymbr1|3367` matches `Zymbr1|3367` (itself) with 100% identity. The next best hit is sequence `Zymps1|800588`, which is from *Z. pseudotritici* and matches with a much lower identity of 35.2%. For a full description of the BLAST summary statistics see the [BLAST tutorial](https://www.ncbi.nlm.nih.gov/BLAST/tutorial/Altschul-1.html).

### Running the MCL algorithm

We are now ready to run MCL and we will be following the protocol outlined [here](https://micans.org/mcl/man/clmprotocols.html#blast). Before we start we need to remove our comment lines from our BLAST results files. We could have specified an output format for the `blastp` command that didn't include comment lines but they make the file more readable and can be useful if you want to write your own scripts to parse the results. We can remove comment lines by using:

```
grep -v '#' all_seqs_subset.results > all_seqs_subset.cblast
```

`grep` is a command that searches through a file to find a specific text, in this case we look for lines with `#`, comments. However, rather than output all the lines with comments, which wouldn't be useful, we use the `-v` flag to output all the lines that *don't* match our text. The redirect `>` then puts all the unmatched lines into a new file.

Now we need to extract the columns of interest:

```
 cut -f 1,2,11 all_seqs_subset.cblast > all_seqs_subset.abc
```

This extracts the first, second and eleventh columns from our results file and put them in a new file `all_seqs_subset.abc`. Our new file contains the query sequence ID, the match sequence ID and the e-value. This format presents the BLAST hits like a network where the query and match are connected by an link (the e-value), which tells us about the strength of that link. This information can be used by the MCL algorithm to identify protein families.

We next use an MCL ancillary program, `mcxload` to create a network file and a dictionary.

```
mcxload -abc all_seqs_subset.abc --stream-mirror -stream-tf 'ceil(200)' -o all_seqs_subset.mci -write-tab all_seqs_subset.tab
```

Now that we have created a dictionary `all_seqs_subset.mci` and network file `all_seqs_subset.tab`, we can use them to cluster the network and infer protein families with the command:

```
mcl all_seqs_subset.mci -I 1.4  -use-tab all_seqs_subset.tab
```

Here we use `mcl` and our input files to generate the protein families. We specify the `-I` flag with a value of 1.4, this flag controls the granularity of the clusters identified (i.e. effects number and size). It is often useful to run the clustering with multiple values of `-I`, as referenced in the [MCL protocols](https://micans.org/mcl/man/clmprotocols.html#blast), to see the effect of this parameter. We have now created a file, `out.all_seqs_subset.mci.I14`, which contains all the identified families.

Take a look at the file. Each line contains a different gene family. We can use:

```
wc -l out.all_seqs_subset.mci.I14
```

to count the number of lines and see that there are 43 clusters identified. On each line are all the genes or proteins contained within that family separated by a tab. A quick scan of the file reveals that each family can contain multiple sequences from one organism as well as sequences from few or all organisms in our analysis.

## Exercises

1. Run the clustering with different values of `-I`
  * Are a different number of clusters identified?
  * Can you tell whether the size, or average size of clusters has changed? (This will require some coding)
  * What do you think this would mean for any real analysis of protein families?

## Summary

We have now used the MCL algorithm to identify gene families and you will have learned to:

* Use common Linux programs to view and reformat standard data files
* Use BLAST to calculate sequence similarity
* Use the MCL algorithm to cluster a similarity network and identify protein families

We will next use these protein families to identify evidence of accelerated evolution and positive selection.
