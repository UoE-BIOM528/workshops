---
layout: page
title: Inferring positive selection
order: 6
---

## Positive selection

We now have assembled a genome, learned how to identify the genes and proteins, and grouped proteins into families. Although it's possible to learn a lot from the analyses we have done already, we can gain a lot of information by comparing sequences using phylogenetic evolutionary analysis to identify signatures of positive selection.

Positive selection is the process by which new advantageous genetic variants sweep a population. Signs of positive selection can be used as evidence for adaptive evolution and the gain of new functions that may benefit a population. However, positive selection is hard to detect and one common method is to calculate the ratio of rates of nonsynonymous mutations to synonymous mutations, known as *dN/dS*. If this rate is >1 it indicates an abundance of mutations that will change the protein sequence and may suggest selection for new functions. Any protein families exhibiting a high rate of nonsynonymous sequence would be interesting candidates to look at in further evolutionary studies.

## Input data

We will need three sources or information for this analysis:

* The coding sequences for all 4 *Zymoseptoria* species
* The protein sequences for all 4 *Zymoseptoria* species
* The protein family definitions inferred by the MCL algorithm

## Steps of our analysis

There are several steps to this analysis:

1. Extract protein and coding sequences for a gene family
2. Align the protein sequences
3. Combine the aligned protein and unaligned coding sequences to create a codon alignment
4. Infer a gene phylogeny
5. Calculate the *dN/dS* ratio

### Extracting protein and coding sequences


To start we will navigate to the correct data directory:

```
cd ~/Desktop/Data/families/
```

If we use `ls` to list the files in this directory you will see there is a file named `selection_families.txt` and a directory named  `species`, which contains the transcript and protein sequences for each of our *Zymoseptoria* species.

First we need to extract the coding and protein sequences for each protein family. For this analysis we will use only 3 of our identified protein families. These can be found in the file `selection_families.txt`. Next, create a new file, `fasta_creator.py`, with your favourite text editor and paste in the following code:

``` { python }
import sys

def read_fasta(file, sequences):
    '''
    Read a fasta file and record the sequences
    Returns a dictionary storing the header->sequence
    '''
    header = ''
    seq = ''
    fh = open(file, 'r')
    for line in fh.readlines():
        line = line.rstrip()
        if line.startswith('>'):
            if header != '':
                sequences[header] = seq
                header = ''
                seq = ''
            header = line.replace('>', '')
        else:
            seq = seq + line
    sequences[header] = seq
    fh.close()
    return sequences


# Get the user defined parameters
family_file = sys.argv[1]
fasta_files = sys.argv[2:]

# Read in all the sequences
transcripts = {}
proteins = {}
for transcript_file in fasta_files:
    transcripts = read_fasta(transcript_file, transcripts)
    protein_file = transcript_file.replace('transcripts', 'proteins')
    proteins = read_fasta(protein_file, proteins)

# Create a protein and coding fasta for each gene family
fam_id = 1
fh = open(family_file, 'r')
for line in fh.readlines():
    line = line.rstrip()
    ids = line.split('\t')

    trans = open('Fam{:04d}_coding.fa'.format(fam_id), 'w')
    prot = open('Fam{:04d}_protein.fa'.format(fam_id), 'w')

    for id in ids:
        if id in transcripts and id in proteins:
            trans.write(">{0}\n{1}\n".format(id, transcripts[id]))
            prot.write(">{0}\n{1}\n".format(id, proteins[id]))

    fam_id = fam_id + 1
    trans.close()
    prot.close()
fh.close()
```

Briefly, this Python script will read in protein and transcript files for our 4 species, then iterate round the protein families and extract the relevant protein and coding sequences. Finally the script creates a protein and coding sequence fasta file for each protein family. To run the script we will use the command:

```
python fasta_creator.py selection_families.txt species/Z.tritici.transcripts.fasta species/Z.ardabiliae.transcripts.fasta species/Z.brevis.transcripts.fasta species/Z.pseudotritici.transcripts.fasta
```

In this command we run `python` and pass the script we want to run `fasta_creator.py`. Next, we pass the file containing our protein families `selection_families.txt` and finally pass a set of transcript fasta files stored in the species directory. You will notice the script produces 6 files; a coding and protein fasta file for each of the three families (labelled 0001, 0002 and 0003) in our analysis. These files each contain only the sequences for these families and will be used in the next stage of our analysis.

### Aligning protein sequences

In order to be able to calculate rates of change in the proteins and infer selection we need to first align the protein sequences. Sequence alignment aims to identify substitutions, insertions and deletions in a set of related sequences. These changes can be used to infer evolutionary relations (evolutionary distances, functional changes) between the sequences. To align our protein sequences we will use [Muscle](https://pubmed.ncbi.nlm.nih.gov/15034147/). Let's start with Fam0001:

```
muscle -in Fam0001_protein.fa -out Fam0001_protein.afa
```

Here we call `muscle` and pass an `-in` file, which is a fasta file of our unaligned protein sequences and an `-out` file, which will store the resulting alignment. If we view the first sequence with:

```
head Fam0001_protein.afa
```

Our output will show the first aligned sequence. You can see that Muscle has inserted gap `-` characters during the alignment.

```
>Zymar1|772961
------------------------------------------------------------
---------------MATLAFDYTPGRPENIDQILNGLDRYNPET---TTIFQDYVSSQC
ENQTYDCYANLALLK------------LYQFNPHL----------------------TRD
ETTTNILVKSLTVFPSPD------------------------------------------
------------------------------------------------------------
------------------------------------------------------------
-----------------------FSLCLSLLPPYVLQDSKATG-----------------
------------------------------------------------------GASSGT
LAEAV--------------------QYLDVL---HNQLNNAQYTDFWSTLDSSDLYADL-
```

#### Exercise

1. Perform protein alignments for Fam0002 and Fam0003.


### Convert protein alignments and transcript sequences to codon alignments

In order to calculate *dN/dS* we need what's known as a codon alignment. This is nucleotide sequences alignment by codon rather than by base. To produce a codon alignment we first take a protein alignment to note the positions of matches and gaps in the alignment. Then in the transcript sequences we print 3 nucleotides (codon) for each match in the protein alignment and three gap, `-`, characters for each gap in the protein alignment. If you have an interest in programming this is a relatively simple and interesting small project and the code produced will be useful. However, for this exercise we will use [Pal2Nal](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1538804/), a tool developed specifically for this purpose:

```
~/Bin/pal2nal.v14/pal2nal.pl Fam0001_protein.afa Fam0001_coding.fa -output paml -nogap -nostderr > Fam0001_codon.fa
```

Here we call the pal2nal Perl script and pass (in a specific order) the aligned protein file, the unaligned transcript file, specify the output format `-output paml`, tell the program to remove gaps `-nogap` (these are uninformative) and do not print lots of progress information to the screen `nostderr`. Finally, we redirect `>` the output to a new file called `Fam0001_codon.fa`. You can see the first few lines of the file with:

```
head Fam0001_codon.fa
```

which produces:

```
15    249
Zymar1|772961
ACATTCAGCGACTCGCCACGTACATCTCCGCTCTCTGCCTCAAGAAGACGGCCGTCGCTA
AGATCTGGTGGAGAGGACACGCCCAATGCCGCGCTCAAGTTCATATACATAACCGTCACA
AAAGACAGCGCCACGCCATGGCCTCATAAGCCTGAGGTCGTCAATACAGTGGAGAACAAG
GGAAGCTCGGAGGTCACAGCGACAGATGAAGTCGTGGGAATTCGCCCGATTCAAGAAGGA
GGAGATATC
Zymps1|798113
ACTGCTGCTTGCAAAGCTGGAATGGGAGAGCGTTCGATCGGACGATCCGTGAATCTTGAG
GACTTCGTGTATATGAAGTCACACACACCGGCTGTCGATCCGAAAGATGAGGATGGCGGC
```

You can see here that firstly the format is different. The file is now in PAML format, as we will be using this program later. We also have no gaps in our alignment any more as we removed these with pal2nal. Gaps are uninformative for some forms of evolutionary analysis and can be discarded. Now that we have a codon alignment we only have one more stage before we can do our evolutionary analysis. Next we will infer a phylogenetic tree.

#### Exercise

1. Compute codon alignments for families Fam0002 Fam0003

### Inferring a phylogenetic tree

The final input needed for our evolutionary analysis is a phylogenetic tree of all the protein sequences in our alignment, known as a gene tree. This phylogenetic information will tell PAML the inferred evolutionary relationships between our sequences. To infer the tree we are going to use a Maximum Likelihood method known as [RAxML](). We can infer the tree using the command:

```
raxmlHPC-AVX -f a -x 12345 -p 12345 -# 100 -m PROTGAMMAGTR -s Fam0001_protein.afa -n Fam0001
```

Here we call RAxML, with the specific command `raxmlHPC-AVX` and pass a variety of options:

* `-f a` instructs RAxML to run a bootstrap analysis with a final Maximum Likelihood analysis to find the best single tree
* `-x 12345` specifies a random seed, which ensures our analysis is repeatable
* `-p 12345` specifies a second random seed for the initial parsimony analysis peformed by RAxML
* `-# 100` tells RAxML to construct 100 bootstrapped trees
* `-m PROTGAMMAGTR` gets RAxML to use the GTR model of sequence evolution for proteins with a gamma distribution. See the RAxML manual for a detailed description of the models available and their uses.
* `-s` is our input sequence alignment
* `-n` is the name used in our output analysis.

The program will take a few minutes to run and produce a series of output files. There is lots of information in these files but at the moment we are only interested in a single file, `RAxML_bestree.FAM0001`. This file contains our tree in newick string format, a text representation of a phylogenetic tree. We can see the contents of our best tree file with:

```
head RAxML_bestree.Fam0001
```

to see our tree:

```
(Zymps1|798113:0.00000100000050002909,(((((Zymbr1|9759:0.09014626614446710762,Zymps1|801345:0.00000100000050002909):0.53791360951592093187,((Zymar1|770182:1.22293334018824739751,Zymps1|799202:0.62571431279416289684):0.76576247449122913924,Mycgr3|74837:1.52236307255735758837):0.28436436496399358775):0.07624492770551322129,(Zymbr1|6287:1.31308392694966546976,Zymbr1|7118:1.17242533882395338907):0.51684360936579276657):0.39962943127324834780,((Zymar1|771545:0.00000100000050002909,(Zymps1|804201:0.00000100000050002909,Zymbr1|8201:0.00608665423711026582):0.01215868354045264617):1.19177149286231420788,(Zymbr1|6575:1.70306974830461599346,Zymar1|770837:1.21242619752115676768):0.39219739726890623377):0.00000100000050002909):0.56060977562631053583,Zymar1|773289:1.69881765817092511561):1.25300201617919038100,Zymar1|772961:0.00000100000050002909):0.0;
```

In this format you can see the name of our genes/proteins with an associated branch lengths separated by a `:` (i.e. `Zymbr1|9759:0.09`). The brackets group the genes/proteins into clades, which represent the evolutionary relationships between the genes. This format is great for computer programs but no very readable for us, it is possible to visualise this tree with [Icytree](https://icytree.org/). Visit the site and use File -> Load from file ... to import the tree.

![GeneTree](/images/gene_tree.png)

You can now see the evolutionary relationships between proteins. We can easily tell which sequences are more evolutionarily similar and which are distant. We could also make some inferences about the evolution of the protein family from this tree. We are now ready to use our alignments and tree to calculate the *dN/dS* ratio or our protein families.

#### Exercise

1. Use RAxML to produce phylogenetic trees for Fam0002 and Fam0003

### Calculating *dN/dS* with PAML

To calculate our evolutionary rates we are going to use the phylogenetic analysis package [PAML](http://abacus.gene.ucl.ac.uk/software/paml.html). PAML is a package of programs for phylogenetic analyses of DNA or protein sequences using maximum likelihood. It can apply a variety of models to calculate different evolutionary metrics. The full documentation for all the PAML programs can be found [here](http://abacus.gene.ucl.ac.uk/software/pamlDOC.pdf).

To run PAML we will first need a control file. A control file stores all the parameters we use to run a specific program. These are particularly useful when a program has lots of parameters (which would make the command line look messy and hard to follow) and also serves to store the exact run conditions of a program, which helps reproducibility. Create a new text file using your favourite text editor and enter the following text:

```
seqfile = Fam0001_codon.fa          * sequence data filename
treefile = RAxML_bestTree.Fam0001   * tree structure file name
outfile = Fam0001.txt               * main result file name

noisy = 9       * 0,1,2,3,9: how much rubbish on the screen
verbose = 1     * 0: concise; 1: detailed, 2: too much
runmode = 0     * 0: user tree;  1: semi-automatic;  2: automatic 3: StepwiseAddition; (4,5):PerturbationNNI; -2: pairwise

seqtype = 1     * 1:codons; 2:AAs; 3:codons-->AAs
CodonFreq = 2   * 0:1/61 each, 1:F1X4, 2:F3X4, 3:codon table

clock = 0                   * 0:no clock, 1:clock; 2:local clock; 3:CombinedAnalysis
aaDist = 0                  * 0:equal, +:geometric; -:linear, 1-6:G1974,Miyata,c,p,v,a
aaRatefile = dat/jones.dat  * only used for aa seqs with model=empirical(_F)

model = 0    * 0:one, 1:b, 2:2 or more dN/dS ratios for branches

NSsites = 0  * 0:one w;1:neutral;2:selection; 3:discrete;4:freqs;
             * 5:gamma;6:2gamma;7:beta;8:beta&w;9:beta&gamma;
             * 10:beta&gamma+1; 11:beta&normal>1; 12:0&2normal>1;
             * 13:3normal>0

icode = 0    * 0:universal code; 1:mammalian mt; 2-10:see below
Mgene = 0    * codon: 0:rates, 1:separate; 2:diff pi, 3:diff kapa, 4:all diff

fix_kappa = 0   * 1: kappa fixed, 0: kappa to be estimated
kappa = 2       * initial or fixed kappa
fix_omega = 0   * 1: omega or omega_1 fixed, 0: estimate
omega = .4      * initial or fixed omega, for codons or codon-based AAs

fix_alpha = 1    * 0: estimate gamma shape parameter; 1: fix it at alpha
alpha = 0.      * initial or fixed alpha, 0:infinity (constant rate)
Malpha = 0      * different alphas for genes
ncatG = 8       * # of categories in dG of NSsites models

getSE = 0           * 0: don't want them, 1: want S.E.s of estimates
RateAncestor = 1    * (0,1,2): rates (alpha>0) or ancestral states (1 or 2)

Small_Diff = .5e-6
cleandata = 1   * remove sites with ambiguity data (1:yes, 0:no)?
method = 0      * Optimization method 0: simultaneous; 1: one branch a time
```

Save the file as `codeml.ctl`. In this file parameters are defined by name (i.e. `seqfile`) and assigned with the value after the `=`. All the text after a `*` is treated as a comment and not read by PAML but is useful for users to know what is happening.  There are only a handful of parameters we need to worry about and the rest are left as default values:

* `seqfile` the name of our codon alignment file
* `treefile` the name of the tree file
* `outfile` the main file name that will store our results
* `seqtype` we specify `1` for codon data
* `model` we specify `0` to calculate just one value of *dN/dS* for the protein family. Note that we could also specify `1` to calculate different rates of *dN/dS* for different branches or `2` to calculate different rates of *dN/dS* for different clades (this requires labelling of the tree).

Now lets run our evolutionary analysis with:

```
codeml codeml.ctl
```
You will see a lot of text printed to the screen as the program runs and outputs some running information. After a few minutes the program will complete and you will see a range of new files produced. We are interested in our output file `Fam0001.txt`. There's lots of information in this file including our input alignments, codon usage frequencies, transversion/transistion rates and our estimates of *dN/dS*. The parts we're interested in follow the lines:

```
Nei & Gojobori 1986. dN/dS (dN, dS)
```

Following this there is a matrix (triangle shaped) showing some pairwise *dN/dS* estimates. Then some newick format trees that characterise the nucleotide substitution rates and finally our *dN/dS* estimate:

```
omega (dN/dS) =  0.45429
```

Note that below this is a *dN/dS* value for each branch but as we specified `model = 0`, which only infers one value of *dN/dS* all the values in this table are the same. If we changed our control file to `model = 1`, *dN/dS* would be estimated for each branch and these values would change.

#### Exercise

1. Modify the `codeml.ctl` to run the PAML analysis for families Fam0002 and Fam0003.

### Interpretation of results

Now that we have our results files for our three families of interest we can view them all and compare them using the `grep` command:

```
grep 'omega' *.txt
```

this searches all our `.txt` files in the  current directory (i.e. the results files from PAML) for a text specified with quotes (`omega`) and produces:

```
Fam0001.txt:omega (dN/dS) =  0.45429
Fam0002.txt:omega (dN/dS) =  2.26800
Fam0003.txt:omega (dN/dS) =  1.61059
```

We can see that we have a range of inferred *dN/dS* values for our three families. For family `Fam0001` our *dN/dS* value is 0.45, which is <1 and indicates a higher rate of synonymous substitutions compared to non-synonymous substitutions. Generally a higher rate of synonymous substitutions is interpreted to represent purifying selection and a maintenance of the amino acid sequence and hence function of the proteins. Family `Fam0003` shows a higher rate of *dN/dS* of 1.61. A value > 1 signifies a higher rate of non-synonymous substitutions and may be a sign of positive selection. Here a high rate of non-synonymous substitutions will result in a changed amino acid sequence and potentially a changed function that may be beneficial to the organism. We see a very similar pattern with family `Fam0003`, which shows a *dN/dS* of 2.26. However, this value might be considered a very high rate of non-synonymous mutations where positions in the alignment have been predicted to have undergone more than a single mutation. Predicting multiple substitutions at a single position in an alignment is notoriously difficult and results of large *dN/dS* ratios need to be treated with some scrutiny.

Another caveat to point out is that a high rate of *dN/dS* does not necessarily signify positive selection. There are alternative hypotheses for why we might observe a high rate of change in protein sequences. The main alternative is that of relaxed purifying selection, where a high rate of change may be observed when there is no selective constraint to maintain a protein's function. A scenario that may lead to this observation is pseudogenisation, where a protein, which is no longer needed and hence no longer under selection, is undergoing degenerative mutations to lose function. In this case we might also see a high level of non-synonymous change.

At this point we are finished with our workshop but there are still potential avenues of research we could conduct in a research project. We have identified a range of protein families that may be experiencing positive selection. An example next step may be look at the potential functions of these proteins using resources such as the [Gene Ontology](http://geneontology.org/) or other annotation resources. We can also look at the scientific literature to see whether any of our proteins have been associated with disease. We can see that one protein, Mycgr3|97500, found in family `Fam0002` has been predicted to form part of the *Z. tritici* secretome by [this study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3517617/). The protein is also short (<200 amino acids) and cysteine rich, which provides some evidence that the protein may act as an effector, a secreted protein that acts to interfere with the hosts plants immune system and facilitate infection. Evidence gained from these supplementary analyses may be used to generate new hypotheses and direct future lab experiments.

### Summary

We have now used a variety of programs to calculate the rate of non-synonymous to synonymous substitutions in protein alignments to detect signs of positive selection. In particular we have:

* Used a python script to extract protein and transcript sequences for protein families
* Used Muscle to align protein sequences
* Used RAxML to infer a phylogenetic tree
* Used PAL2NAL to create a codon alignment
* Used PAML to calculate the *dN/dS* ratio

Overall, we have completed our full workflow and you have now gained experience in:

* *De novo* genome assembly from raw sequencing reads
* Gene prediction from genomic sequence
* Protein family identification
* Evolutionary analysis of protein families
