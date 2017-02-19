# Characterizing microbial communities using PathoScope

![banner](https://raw.githubusercontent.com/ecastron/PS_demo/master/img/banner.png)

## Welcome! 👋🏼


In this tutorial, we will learn how to obtain taxonomic profiles from metagenomic Next-Generation Sequencing data. If you haven't already, .

**_PathoScope2_** is a command line tool that is written in python. We run PathoScope by issuing commands in the Terminal and commandThe program is command line based so we need to use the Terminal in Unix-based systems such as Linux and Mac. On Windows machines, the syntax is the same, however, you have to make sure that you have PathoScope's dependencies installed and on your Environmental variable. Finally, you can check out a more in-depth tutorial in the **PathoScope** repo [here](https://github.com/PathoScope/PathoScope/raw/master/pathoscope2.0_v0.02_tutorial.pdf).


For this part of the tutorial, we running programs in the PathoScope package through our docker image. To make this easier, we can set up an alias for calling PathoScope:

```bash
alias pathosuite='docker run -ti -u rstudio -v $(pwd):/hostwd -w /hostwd --rm mlbendall/pathosuite pathoscope'
```

The alias will disappear if you exit out of the terminal, Now, using your alias, check that everything is working:

```bash
pathosuite --help
```

### What PathoScope can do for you

PathoScope is a modular piece of software that will allow you to go all the way from a fastq file to a text file (typically tab-delimited) with columns representing genomes, their proportions, etc.  
There are 6 **PathoScope modules**, however, for this demo we will focus on the three most important ones:  
- **_PathoLib_** - Allows users to automatically generate custom reference genome libraries for specific scenarios or datasets  
- **_PathoMap_** - Aligns reads to target reference genome library and removes sequences that align to the filter and host libraries  
- **_PathoID_** - Reassigns ambiguous reads, identifies microbial strains present in the sample, and estimates proportions of reads from each genome  

Once you run your samples through **PathoScope**, you can easily import the output files into R for downstream exploratory data analysis and statistical inferences.  

## Reference Databases

One of the advantages of using **_PathoScope_** is the ability to easily customize reference databases. Typically, each experiment needs two different reference databases:

+ The **target** database is a collection of genomes that are closely related to the species or strains in your sample.
+ The **filter** database contains the sequences of possible contaminants, such as internal controls or host genome.

The sequences in either library can be complete genomes, draft genomes (contigs), or other types of nucleotide sequences. For our production pipelines, we typically use either NCBI's RefSeq Genomes or (recently) NCBI Reference and Representative genomes. You are free to use any sequences you like, but the key is to have a unique ID for each OTU, and all sequences belonging to each OTU should be labeled with this ID. (Later on, in PathoID, we use these identifiers in our reassignment model). We find that the NCBI Taxonomy ID is a good identifier, and we can use this in subsequent analysis to get the full lineage for an OTU. 



> Originally:  
\>gi|40555938|ref|NC_005309.1| Canarypox virus, complete genome  

> but PathoScope likes:  
\>ti|44088|gi|40555938|ref|NC_005309.1| Canarypox virus, complete genome  


customizing the output format of `blastdbcmd` using `-outfmt`:

```
blastdbcmd -db nt.00 -entry all -out - -outfmt ">ti|%T|gi|%g|ref|%a| %t##X##%s"
```


We also provide the NCBI nucleotide database already formatted [here] (ftp://pathoscope.bumc.bu.edu/data/nt_ti.fa.gz) (25.4 GB as of Feb. 2016). You could also use **PathoLib** to subsample this big file (50 GB uncompressed) and select only the taxa that you want. For instance, obtaining all the virus entries in nt_ti.fa (virus taxonomy ID = 10239)

The **_PathoLib_** module is
The purpose of the PathoLib module is to obtain reference sequences and to index these sequences for downstream analysis. 


Alternatively, we provide the entire NCBI nucleotide database already formatted [here] (ftp://pathoscope.bumc.bu.edu/data/nt_ti.fa.gz) (10 GB file). You could also use **PathoLib** to subsample this big file (50 GB uncompressed) and select only the taxa that you want. For instance, obtaining all the virus entries in nt_ti.fa (virus taxonomy ID = 10239)

		python pathoscope2.py -LIB python pathoscope.py LIB -genomeFile nt_ti.fa -taxonIds 10239 --subTax -outPrefix virus

Or in order to create a filter library, say all human sequences:
		
		python  pathoscope2.py -LIB python pathoscope.py LIB -genomeFile nt_ti.fa -taxonIds 9606 --subTax -outPrefix human

However, I'm providing a target and filter library already formatted that you can download [here](https://www.dropbox.com/s/7z2c8c2walg92yv/HMP.zip?dl=1), and [here for human](https://www.dropbox.com/s/nljz9cjoc5z43k8/human.zip?dl=1) and the [internal control](https://www.dropbox.com/s/sjy94bnxw6h4a6m/phix.zip?dl=1). The target library is a collection of genomes from the reference library of the Human Microbiome Project (description [here](http://hmpdacc.org/HMREFG/)), and the filter library is simply the human genome (hg19) plus the PhiX174 phage genome that Illumina uses as internal control, which may or may not be added to the sequencing experiment.  

```bash
wget ftp://ftp.ncbi.nih.gov/blast/db/16SMicrobial.tar.gz

```


## PathoMap


### Getting data and reference genomes
We are going to use data from a study exploring microbiome diversity in oropharingeal swabs from schizophrenia patients and healthy controls. The SRA accession number is `SRR1519057`. 

![SRA](https://github.com/ecastron/PS_demo/raw/master/img/img01.png)

This file is probably too big for a demo so I randomly subsampled the reads down to a more manageable size (~40 M to 40 K reads)  
* Go ahead and download the data [here](https://www.dropbox.com/s/dkfy5hcxfi7kvwq/ES_211.fastq?dl=1). If you are interested, sequences are part of [this study](https://peerj.com/articles/1140/)  
* Now you need at least two files, one to be used as target library (where your reads are going to be mapped) and another one to be used as filter library (internal controls, host genome, contaminants, etc. that you want to remove)

As target library, you can use any multi fasta file containing full or draft genomes, or even nucleotide entries from NCBI, and combinations of both. The only condition is that the fasta entries start with the taxonomy ID from NCBI as follows:

> Originally:  
\>gi|40555938|ref|NC_005309.1| Canarypox virus, complete genome  

> but PathoScope likes:  
\>ti|44088|gi|40555938|ref|NC_005309.1| Canarypox virus, complete genome  

You could do this very easily in **PathoLib**:

		python pathoscope2.py LIB -genomeFile my_file.fasta -outPrefix target_library



### Let's map the reads
Once you have your data and target and filter libraries, we are ready to go ahead with the mapping step. For this, we use bowtie2 so we will need to tell **PathoMap** where the bowtie2 indices are. If you don't have bowtie2 indices, not a problem, **PathoMap** will create them for you. And if your fasta files are larger than 4.6 GB (Bowtie2 limit), not a problem either, **PathoMap** will split your fasta files and create indices for each one of the resulting files.

If you have fasta files and not bowtie2 indices:

		python pathoscope2.py MAP -U ES_211.fastq -targetRefFiles HMP_ref_ti_0.fa,HMP_ref_ti_1.fa -filterRefFiles human.fa,phix174.fa  -outDir . -outAlign ES_211.sam  -expTag Bangladesh

But if you already have Bowtie2 indices (our case), you can issue the following command:

		python pathoscope2.py MAP -U ES_211.fastq -indexDir /Users/ecastron/Dropbox/14_workshops/Bangladesh/HMP  -targetIndexPrefixes HMP_ref_ti_0,HMP_ref_ti_1 -filterIndexPrefixes genome,phix174  -outDir . -outAlign ES_211.sam  -expTag Bangladesh

Let's give it a try...

![ps2](https://github.com/gwcbi/phylobang/raw/master/img/ps2.png)

So that should have taken ~3 minutes to run. Now you have a number of things that were printed to the screen as well as files that were created. The summary of the STDOUT is:

| Reads Mapped  | Library  | 
|:------------- | ---------------:|
| 1053      | HMP\_ref\_ti\_0 |
| 1132      | HMP\_ref\_ti\_1 |
| 916 | genome |
| 0 | phix174 |

And you should have one .sam file per library, plus another file containing the reads mapped to all target libraries, a fastq file of the reads mapping to all targets, and the file you most care about: ES_211.sam

![mapout](https://github.com/gwcbi/phylobang/raw/master/img/mapout.png)

### Let's get a taxonomic profile from our .sam file
The last step in our demo is to obtain a taxonomic profile from ES_211.sam using the read reassignment model implemented in **PathoID**

		python pathoscope2.py ID -alignFile ES_211.sam -fileType sam -outDir . -expTag DAV -thetaPrior 1000000

After running the command line above, you should get a tab-delimited file with **PathoScope's** output, and an updated .sam file representing an alignment after **PathoScope's** reassignment model was applied. The main output is the .tsv file. You can open this fle in a text editor or in Microsoft Excel.   

#### For your reference, the output generated by PathoScope in this example is [here](https://github.com/gwcbi/phylobang/tree/master/PS2_output)  

### Output TSV file format

At the top of the file in the first row, there are two fields called "Total Number of Aligned Reads" and "Total Number of Mapped Genomes". They represent the total number of reads that are aligned and the total number of genomes to which those reads align to in the given alignment file.

Columns in the TSV file:

1. **Genome:**  
This is the name of the genome found in the alignment file.
2. **Final Guess:**  
This represent the percentage of reads that are mapped to the genome in Column 1 (reads aligning to multiple genomes are assigned proportionally) after pathoscope reassignment is performed.
3. **Final Best Hit:**  
This represent the percentage of reads that are mapped to the genome in Column 1 after assigning each read uniquely to the genome with the highest score and after pathoscope reassignment is performed.
4. **Final Best Hit Read Numbers:**  
This represent the number of best hit reads that are mapped to the genome in Column 1 (may include a fraction when a read is aligned to multiple top hit genomes with the same highest score) and after pathoscope reassignment is performed.
5. **Final high confidence hits:**  
This represent the percentage of reads that are mapped to the genome in Column 1 with an alignment hit score of 50%-100% to this genome and after pathoscope reassignment is performed.
6. **Final low confidence hits:**  
This represent the percentage of reads that are mapped to the genome in Column 1 with an alignment hit score of 1%-50% to this genome and after pathoscope reassignment is performed.
7. **Initial Guess:**  
This represent the percentage of reads that are mapped to the genome in Column 1 (reads aligning to multiple genomes are assigned proportionally) before pathoscope reassignment is performed.
8. **Initial Best Hit:**  
This represent the percentage of reads that are mapped to the genome in Column 1 after assigning each read uniquely to the genome with the highest score and before pathoscope reassignment is performed.
9. **Initial Best Hit Read Numbers:**  
This represent the number of best hit reads that are mapped to the genome in Column 1 (may include a fraction when a read is aligned to multiple top hit genomes with the same highest score) and before pathoscope reassignment is performed.
10. **Initial high confidence hits:**  
This represent the percentage of reads that are mapped to the genome in Column 1 with an alignment hit score of 50%-100% to this genome and before pathoscope reassignment is performed.
11. **Initial low confidence hits:**  
This represent the percentage of reads that are mapped to the genome in Column 1 with an alignment hit score of 1%-50% to this genome and before pathoscope reassignment is performed.
