## Making a Phylogenetic Tree ##

So you've assembled and binned your metagenome, but whats the next step? Building a phylogenetic tree of course! 
This is a breif tutorial with associated scripts to show you how to build a phylogenetic tree using the 16 ribosomal proteins that tend to be syntenic and colocated. These 16 ribosomal proteins were used in [Hug et al. 2016](10.1038/nmicrobiol.2016.48).
 
This tutorial will go through the following:

1. How to generate gene predictions using prodigal for genomes of interest

2. How to identify ribosomal proteins using an hmm model

3. How to align, trim, and concatenate proteins for a phylogenetic tree

4. building a phylogenetic tree

**This procedure assumes that you have binned your assembled metagenomes and have Metagenome Assembled Genomes (MAGs) ready for analysis.**

Another version of this protocol with more detailed information on what should be in your directory after each command is found on ProtocolsIO: dx.doi.org/10.17504/protocols.io.mp5c5q6

To cite this workflow reference this paper:
 Graham, E. D., J. F. Heidelberg, and B. J. Tully. 'Potential for primary productivity in a globally-distributed bacterial phototroph.' The ISME journal (2018)
 
## Required Dependencies ##

To run this protocol you will need the following programs:

[Python2.7](https://www.python.org/download/releases/2.7/)

[BioPython](http://biopython.org/)

[HMMER](http://hmmer.org/download.html)

[Prodigal](https://github.com/hyattpd/prodigal/wiki/Installation)

[BinSanity](https://github.com/edgraham/BinSanity/wiki/Installation)

[Muscle](https://www.drive5.com/muscle/manual/install.html)

[TrimAL](http://trimal.cgenomics.org/)

[FastTree](http://www.microbesonline.org/fasttree/)


HMMER and BinSanity can be installed as follows:

>**Install [HMMER](http://hmmer.org/)**
   * HMMER can be downloaded like this:
```
$ wget http://eddylab.org/software/hmmer3/3.1b2/hmmer-3.1b2.tar.gz
$ tar -zxvf hmmer-3.1b2.tar.gz
$ cd hmmer-3.1b2
$ ./configure && make && sudo make install
$ cd easel && make check && sudo make install
```
>**Install [BinSanity](https://github.com/edgraham/BinSanity/wiki/Installation)**

`sudo pip install BinSanity`

The remaining dependencies can be installed as executable binaries.


### Step 1: Download Github Repo ###

First you need to get access to all of the files and scripts needed to run this workflow. Run the following command to install.

```
$ git clone https://github.com/edgraham/PhylogenomicsWorkflow.git
$ cd PhylogenomicsWorkflow/Example
```

### Step 2: Run IdentifyHMM to extract marker genes ###
The primary script in this workflow in `identifyHMM`.
This program does the following:

1. Runs prodigal on MAGs
2. Runs `hmmsearch` on these gene calls
3. Parses out genes matching one of the 16 Ribosomal protiens

The script relies on the user providing the location of a file containing Hidden Markov Models (HMMs) for their genes of interest. Here we have provided you with a file called `hug_ribosomalmarkers.hmm`. This contains HMM models for 16 Ribosomal proteins: RpL14, RpL15, RpL16 ,RpL18, RpL22, RpL24 ,RpL2, RpL3, RpL4, RpL5, RpL6, RpS10, RpS17, RpS19, RpS3, RpS8. 

The help message for `identifyHMM` is given below:

```
usage: identifyHMM [-h] [--markerdb MARKERDB] [--performProdigal] [--cut_tc]
                   [--outPrefix OUTPREFIX] [--Num NUM] [-E E]
                   Input

Identify marker genes in in protein sequences of genomes.

positional arguments:
  Input                 Target file(s). Provide unifying text of desired
                        genome(s). Ext must be 'fna' or 'faa'.

optional arguments:
  -h, --help            show this help message and exit
  --markerdb MARKERDB   Provide HMM file of markers. Markers should have a
                        descriptive ID name.
  --performProdigal     Run Prodigal on input genome nucleotide FASTA file
  --cut_tc              use hmm profiles TC trusted cutoffs to set all
                        thresholding
  --outPrefix OUTPREFIX
                        Provide prefix of names for marker output files.
  -E E                  Set E-Value to be used in hmmsearch. Default: 1E-5

```
The best way to show how the script is used by using an example. What I have provided on the github is an folder called `Example` that contains two subfolders. One containing MAGs (`Genomes`) and one containing reference ribosomal proteins pulled from NCBI (`HugRef`).

**Note** When using `identifyHMM` on your own data you need to remember that you should be in the directory containing your MAGs when you run the program and ensure that the only genomes in this directory are the genomes of interest. The program will parse through every file with the suffix given as the [input]. So in this case our input is `.fna`. Second, if you want to run identifyHMM with your own gene calls you will want to exclude the `--performProdigal` flag and be sure that your gene calls end with the suffix `.faa` and are in the same directory as the genomes of interest.


Now you should enter the directory containing your genomes (in our case /PhylogenomicsWorkflow/Example/Genomes) 

```
$ cd Example/Genomes
```

Once in the `Genomes` directory run `identifyHMM` as Follows:

```
$ identifyHMM --markerdb ../../hug_ribosomalmarkers.hmm --performProdigal --outPrefix HUG .fna

```

The output of this will be 16 `.faa` files appended with the prefix spefied by `--outPrefix`, 5 `.faa` files corresponding to the prodigal gene calls, and 5 hmm reports (Stored in the `.tbl` files). So the files that look like `HUG_RpL14.faa` are ribosomal proteins pulled from your genomes of interest.
 
 
 ## Step 3: Combine Reference Proteins with proteins from your MAGs ##
 
 Now that we have extracted marker genes from our genomes of interest we can merge together the Ribosomal Protein calls we just made on our genomes of interest with those in the folder '/PhylogenomicsWorkflow/Example/HugRef' accordingly.

 To do this we will use a quick bash trick that uses the text file `hug_marker_list.txt` which contains a list of the marker proteins we are searching for. Use the following Bash command:
 
```
 while read p; do cat ../HugRef/ExampleRefSet."$p".faa HUG_"$p".faa > Dataset1_"$p".faa; done < ../../hug_marker_list.txt
``` 
This bash script is iterating through the list given in `hug_marker_list.txt` and concatenate appropriate reference and experimental protein sets. So for example it would concatenate `ExampleRefSet.RpL6.faa` and `Hug_RpL6.faa` into `Dataset1_RpL6.faa`. 


## Step 4: Align your Proteins ##

Now that we have 16 files containing Ribosomal proteins from our 5 unknown genomes and 100 references we can move on to aligning our proteins.
To align our proteins we can use muscle (You could also alternatively use [MAFFT](https://mafft.cbrc.jp/alignment/software/algorithms/algorithms.html) here):

```
while read p; do muscle -maxiters 16 -in Dataset1_"$p".faa -out Dataset1_"$p".aln; done < ../../hug_marker_list.txt
```
We will use the same trick as above where we feed the list of ribosomal markers in `hug_marker_list.txt` into a bash loop thar runs muscle on head of the protein fasta files we generated. This will end in 16 alignment (`.aln`) files.


## Step 5: Trim your alignments ##

Now that we have our alignments we need to trim those alignments. There are many programs that do this and I implore you to try a couple and see how different ones work, or even try manual trimming. Here we will use Trimal because its automated and works quickly when running alignments with lots of sequences.

You can run the following to trim your alignments:

```
while read p;do trimal -automated1 -in Dataset1_"$p".aln -out Dataset1_"$p".trimmed.aln; done < ../../hug_marker_list.txt
```

Once your sequences are trimmed I advise manually inspecting the file to ensure everything looks right.

## Step 6: Concatenate proteins ##

Now that you have the trimmed and aligned sequences you can concatenate these 16 files. To do this we will use the `concat` script packaged with BinSanity. 

Usage is shown below:
```
usage: concat -f directory -e Alignment Extension --Prefix file linker -o output

    *****************************************************************************
    *********************************BinSanity***********************************
    **     The `concat` script is used to concatenate multiple sequence        **
    **     alignments for conducting a phylogenomic analysis. Note that you    **
    **     receive an error if there are any duplicate sequence ids in an      **
    **     alignment.
    *****************************************************************************

optional arguments:
  -h, --help  show this help message and exit
  -f          Specify directory where alignments are
  -e          Specify the extension for your alignments (must be in Fasta format)
  --Prefix    Specify the prefix that links your alignments (ex: if you have two alignments TOBG_RpL10, TOBG_RpL24, the --Prefix would be TOBG
  -o          Specify output file
  -N          Specify the minimum number of sequences needed to be included in concatenation
  ```
  
  Run it as follows:
  
```
  concat -f . -e .trimmed.aln --Prefix Dataset1 -o Dataset1.HugRibosomal.trimmed.concat.aln -N 8

``` 
## Step 7: Build the Tree ##

You now have a concatenated alignment 'Dataset1.HugRibosomal.trimmed.concat.aln' which can be used to build a phylogenetic tree.

We will be using FastTree with the `-gamma` and `-lg` parameters. These parameters are optional and just indicate what models we would like to  use for branch length calculation and amino acid evolution respectively. Feel free to adjust these for your own purposes after looking at the FastTree Manual.

```
FastTree -gamma -lg Dataset1.HugRibosomal.trimmed.concat.aln > Dataset1.HugRibosomal.trimmed.concat.newick

```

## Step 8: View Tree ##

Now you have a newick file which can be viewed in a variety of tree views and edited. One of my favorite tools for making publication quality trees is [ITOL](https://itol.embl.de/)!
