## Making a Phylogenetic Tree ##

So you've assembled and binned your metagenome, but whats the next step? Building a phylogenetic tree of course! 
This is a breif tutorial with associated scripts to show you how to build a phylogenetic tree using the 16 ribosomal proteins that tend to be syntenic and colocated. These 16 ribosomal proteins were used in [Hug et al. 2016](10.1038/nmicrobiol.2016.48).
 
This tutorial will go through the following:

1. How to generate gene predictions using prodigal for genomes of interest

2. How to identify ribosomal proteins using an hmm model

3. How to align, trim, and concatenate proteins for a phylogenetic tree

4. building a phylogenetic tree

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




