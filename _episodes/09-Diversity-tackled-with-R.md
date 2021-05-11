---
source: md
title: "Diversity tackled by R "
teaching: 40
exercises: 10
questions:
- "How can I obtain the abundance of the reads?"
- "How can I use R to explore diversity?"
objectives:
- "Comprehend which libraries are required for metagenomes diversity analysis."  
- "Grasp how a phyloseq object is made"
- "Undestand the needed commands to create a phyloseq object for analysis"
keypoints:
- "The library `phyloseq` manages metagenomics objects and computes alpha diversity."  
- "Transform your named matrixes into Phyloseq objects using `pyhloseq(TAX, OTU)`."
- "Use `help()` to discover the capabilities of libraries."
- "The library `ggplot2` allow publication-quality plotting in R."
---

Species diversity, is the number of species that are represented in a certain 
community. 
Once we know the taxonomic composition of our metagenomes, we can do diversity analyses. 
Here we will talk about the two most used diversity metrics, diversity α (within one metagenome) and β (between metagenomes).   

- α Diversity: Represents the richness (e.g. number of different species) and species' abundance. It can be measured by calculating richness, 
 and eveness, or by using a diversity index, such as Shannon's, Simpson's, Chao's, etc.  
 
- β Diversity: It is the difference (measured as distance) between two or more metagenomes. 
It can be measured with metrics like Bray-Curtis dissimilarity, Jaccard distance or UniFrac, to name a few.  

For this lesson we will use Phyloseq, an R package specialized in metagenomic analysis. We will use it along with Rstudio to analyze our data. 
[Rstudio cloud](https://rstudio.cloud/) and select "GET STARTED FOR FREE"

## α diversity  

|-------------------+-----------------------------------------------------------------------------------------------------------------|   
| Diversity Indices |                             Description                                                                         |   
|-------------------+-----------------------------------------------------------------------------------------------------------------|   
|      Shannon (H)  | Estimation of species richness and species evenness. More weigth on richness.                                   |   
|-------------------+-----------------------------------------------------------------------------------------------------------------|   
|    Simpson's (D)  |Estimation of species richness and species evenness. More weigth on evenness.                                    |                           
|-------------------+-----------------------------------------------------------------------------------------------------------------|   
|      ACE          | Abundance based coverage estimator of species richness.                                                         |   
|-------------------+-----------------------------------------------------------------------------------------------------------------|   
|     Chao1         | Abundance based on species represented by a single individual (singletons) and two individuals (doubletons).    |            
|-------------------+-----------------------------------------------------------------------------------------------------------------|   
 

- Shannon (H): 

| Variable             |  Definition   |     
:-------------------------:|:-------------------------:  
<img src="https://render.githubusercontent.com/render/math?math=H=-\sum_{i=1}^{S}p_i\:ln{p_i}"> | Definition
<img src="https://render.githubusercontent.com/render/math?math=S"> | Number of OTUs  
<img src="https://render.githubusercontent.com/render/math?math=p_i">|  The proportion of the community represented by OTU i   

- Simpson's (D) 

| Variable             |  Definition |   
:-------------------------:|:-------------------------:  
<img src="https://render.githubusercontent.com/render/math?math=D=\frac{1}{\sum_{i=1}^{S}p_i^2}">| Definition   
<img src="https://render.githubusercontent.com/render/math?math=S"> | Total number of the species in the community   
<img src="https://render.githubusercontent.com/render/math?math=p_i" align="middle"> | Proportion of community represented by OTU i    
  
- ACE  

| Variable             |  Definition |  
:-------------------------:|:-------------------------:  
<img src="https://render.githubusercontent.com/render/math?math=S_{ACE}=S_{abund}+\frac{S_{rare}}{C_{ACE}}+\frac{F_1}{C_{ACE}}+\gamma_{ACE}^2"> | Definition    
<img float="left" src="https://render.githubusercontent.com/render/math?math=S_{abund}"> | Number of abundant OTUs   
<img src="https://render.githubusercontent.com/render/math?math=S_{rare}">  | Number of rare OTUs   
<img src="https://render.githubusercontent.com/render/math?math=C_{ACE}">  | Sample abundance coverage estimator  
<img src="https://render.githubusercontent.com/render/math?math=F_1">   | Frequency of singletons  
<img src="https://render.githubusercontent.com/render/math?math=\gamma_{ACE}^2"> | Estimated coefficient  of variation in rare OTUs 

- Chao1  
  
| Variable             |  Desription |  
:-------------------------:|:-------------------------:  
 <img src="https://render.githubusercontent.com/render/math?math=S_{chao1}=S_{Obs}+\frac{F_1(F_1-1)}{2(F_2 + 1)}">  | Definition  
<img src="https://render.githubusercontent.com/render/math?math=F_1,F_2">|Count of singletons and doubletons respectively    
<img src="https://render.githubusercontent.com/render/math?math=S_{chao1}=S_{Obs}">| The number of observed species    

The rarefaction curves allow us to know if the sampling was exhaustive or not. 
In metagenomics this is equivalent to knowing if the sequencing depth was sufficient.

## β diversity  
Diversity β measures how different two or more metagenomes are, either in their composition (diversity)
or in the abundance of the organisms that compose it (abundance). 
- Bray-Curtis dissimilarity: Emphasis on abundance. Measures the differences 
from 0 (equal communities) to 1 (different communities)
- Jaccard distance: Based on presence / absence of species (diversity). 
It goes from 0 (same species in the community) to 1 (no species in common)
- UniFrac: Measures the phylogenetic distance; how alike the trees in each community are. 
There are two types, without weights (diversity) and with weights (diversity and abundance)  

It is easy to visualize using PCA, PCoA or NMDS analysis.

## Creating lineage and rank tables  
Packages like Quiime2, MEGAN, Vegan or Phyloseq in R allows to obtain these diversity indexes.  
We will use Phyloseq, in order to do so, we need to generate an abundance matrix from the Kraken output.  

~~~
$ cd ~/dc_workshop/taxonomy
$ head JP4D.kraken   
~~~
{: .bash}

~~~
U       k141_55805      0       371     0:337
U       k141_0  0       462     0:428
U       k141_55806      0       353     0:319
U       k141_55807      0       296     0:262
C       k141_1  953     711     0:54 1224:2 0:152 28211:2 0:15 953:3 0:449
U       k141_2  0       480     0:446
C       k141_3  28384   428     0:6 1286:2 0:8 28384:14 0:11 1:3 0:350
U       k141_4  0       302     0:268
U       k141_5  0       714     0:680
U       k141_6  0       662     0:628
  
~~~
{: .output}



|------------------------------+------------------------------------------------------------------------------|  
| column                       |                              Description                                     |  
|------------------------------+------------------------------------------------------------------------------|  
|   C                          |  Classified or unclassified                                                  |  
|------------------------------+------------------------------------------------------------------------------|  
|    k141_0                    |FASTA header of the read(contig)                                              |                
|------------------------------+------------------------------------------------------------------------------|  
|  1365647                     | Tax id                                                                       |  
|------------------------------+------------------------------------------------------------------------------|  
|    416                       |Read length                                                                   |           
|------------------------------+------------------------------------------------------------------------------|  
|  0:6 1286:2 0:8 28384:14 0:11|kmers hit to a taxonomic id E.g. 0:1 root 6 hit, 1286 has 2 hits, etc.        |           
|-------------------+-----------------------------------------------------------------------------------------|  







### Exploring metagenome data with the terminal  
  
In this lesson we will use R studio to analize two microbiome samples from CCB, you don't have to install anything, 
you already have an instance on the cloud ready to be used.   

The terminal is a program that executes programs, and is better to deal with long data sets than a visual interface.  
First to visualize the content of our directory you can use the `ls` command.  

~~~
$ ls
~~~~

Now you can also known in which directory you are standing in the reminal,by using `pwd`. 

Let's explore the content of some of our data files. So we have to move to the corresponding folder where our taxonomic-data files are: 

~~~
$ cd /home/dcuser/dc_workshop/results
~~~~
{: .bash}

Files `.kraken` and `kraken.report`are the output of the Kraken program, we can see a few lines of the file `.kraken` using the command `head`.   

~~~
$ head JC1A.kraken 
~~~
{: .bash}

How would you see the last lines of the file report?  

~~~
$ tail JC1A.report 
~~~
{: .bash}
  


##  Manipulating lineage and rank tables in phyloseq  

  
### Load required packages  

Phyloseq is a library with tools to analyze and plot your metagenomics tables. Let's install [phyloseq](https://joey711.github.io/phyloseq/) (This instruction might not work on certain versions of R) and other libraries required for its execution:  

~~~
$ if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

$ BiocManager::install("phyloseq") # Install phyloseq

$ install.packages(c("ggplot2", "readr", "patchwork")) #install ggplot2 and patchwork to   chart publication-quality plots. readr to read rectangular datasets.
~~~
{: .language-r}  

Once the libraries are installed, we must make them available for this R session. Now load the libraries (a process needed every time we began a new work 
in R and we are going to use phyloseq):

~~~
$ library("phyloseq")
$ library("ggplot2")
$ library("readr")
$ library("patchwork")
~~~
{: .language-r}

  
### Load data with the number of reads per OTU and taxonomic labels for each OTU.  

Next, we have to load the taxonomic assignation data into objets in R:

~~~
$ OTUS <- read_delim("JC1A.kraken_ranked-wc","\t", escape_double = FALSE, trim_ws = TRUE)
$ TAXAS <- read_delim("JC1A.lineage_table-wc", "\t", escape_double = FALSE, 
                    col_types = cols(subspecies = col_character(),  
                    subspecies_2 = col_character()), trim_ws = TRUE)

~~~
{: .language-r}


### Build Phyloseq object  

Phyloseq objects are a collection of information regarding a single or a group of metagenomes, 
these objects can be manually constructed using the basic data structures available in R or can
be created by importing the output of other programs, such as QUIIME and kraken-biom.

Since we imported our data to basic R data types, we will build our Phyloseq object manually
by extracting the OTU's names and abundances.

~~~
# Get OTU IDs from both lists
$ names1 <- OTUS$OTU
$ names2 <- TAXAS$OTU

# Remove OTU IDs from both lists
$ OTUS$OTU <- NULL
$ TAXAS$OTU <- NULL
~~~
{: .language-r}

~~~
# Convert both lists to matrix
$ abundances <- as.matrix(OTUS)
$ lineages <- as.matrix(TAXAS)

# Assign the OTU IDs as the names of the rows of the matrixes you just built
$ row.names(abundances) <- names1
$ row.names(lineages) <- names2
~~~
{: .language-r}

All that we've done so far is transforming the taxonomic lineage table 
and the OTU abundance table into a format that can be read by phyloseq.
Now we will make the phyloseq data types out of our matrices.

~~~
$ OTU <- otu_table(abundances, taxa_are_rows = TRUE)
$ TAX <- tax_table(lineages)
~~~
{: .language-r}

We will now construct a Phyloseq-object using Phyloseq data types we have created: 

~~~
$ metagenome_JC1A <- phyloseq(OTU, TAX)
~~~
{: .language-r}

> ## `.callout`
>
>If you look at our Phyloseq object, you will see that there are more data types 
>that we can use to build our object(?phyloseq), as a phylogenetic tree and metadata 
>concerning our samples. These are optional, so we will use our basic
>phyloseq object for now composed of the abundances of specific OTUs and the 
>names of those OTUs.  
{: .callout}


### Plot diversity estimates at desired taxonomic resolution

We want to know how is the bacterial diversity, so, we will prune all of the 
non-bacterial organisms in our metagenome. To do this we will make a subset 
of all bacterial groups and save them.
~~~
$ metagenome_JC1A <- subset_taxa(metagenome_JC1A, superkingdom == "Bacteria")
~~~
{: .language-r}

Now let's look at some statistics of our metagenomes:

~~~
$ metagenome_JC1A
$ sample_sums(metagenome_JC1A)
$ summary(metagenome_JC1A@otu_table@.Data)
~~~
{: .language-r}

By the output of the sample_sums command we can see how many reads they are
in the library. Also, the Max, Min and Mean outout on summary can give us an
idea of the eveness. Nevertheless, to have a more visual representation of the
diversity inside the sample (i.e. α diversity) we can now look at a ggplot2
graph created using Phyloseq:

~~~
$ p = plot_richness(metagenome_JC1A, measures = c("Observed", "Chao1", "Shannon")) 
$ p + geom_point(size=5, alpha=0.7)  
~~~
{: .language-r}

![image](https://user-images.githubusercontent.com/67386612/112221050-95ff2080-8bec-11eb-9fd0-b602d6f153ae.png)
Figure 2. Alpha diversity indexes for JC1A sample

Each of these metrics can give insight of the distribution of the OTUs inside 
our samples. For example. Chao1 diversity index gives more weight to singletons
and doubletons observed in our samples. While Shannon is a entrophy index 
remarking the impossiblity of taking two reads out of the metagenome "bag" 
and that these two will belong to the same OTU.

> ## Exercise 1: build a Phyloseq object by yourself
> 
> Import the `JP4D metagenome` and plot its α diversity with the mentioned metrics.
> 
>> ## Solution
>> 
>> Repeat the previous instructions replacing JC1A for JP4D whenever it's appropiate:
>>
>>  At the end, you should have the Phyloseq object 'metagenome_JP4D'. 
>> 
> {: .solution}
{: .challenge}  



> ## Exercise 2
> 
> Use the help from plot_richness to discover other ways to plot diversity estimates using Phyloseq
> and use another index to show the α diversity in our samples.
> 
>> ## Solution
>> 
>> `?plot_richness` or `help("plot_richness")`
>> 
>>One of the widely α diversity indexes used is Simpson diversity index, as an example
>>of solution, here it is the plot with an extra metric, which is Simpson α index:  
>> `p = plot_richness(metagenome_JC1A, measures = c("Observed", "Chao1", "Shannon", "Simpson"))`
>> ![image](https://user-images.githubusercontent.com/67386612/112221137-b62edf80-8bec-11eb-85aa-dd5be3e8ca16.png)
>>
>> 
> {: .solution}
{: .challenge}  
  
  

  
> ## `.discussion`
>
> How much did the α diversity can be changed by elimination of singletons
> and doubletons?
{: .discussion}
  
  
                             
{% include links.md %}
