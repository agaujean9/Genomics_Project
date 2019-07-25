# Genomics_Project

For this project the objective is to train a classification model that can correctly classify an individual into a specfic ancestral subpopulation using portions of thier genomic data as input. 

For this project I knew I wanted to work with large scale human genomic data. In recent years companies like 23andMe and Ancestry have been able to offer the public relatively cheap insights into the secrets that an individual's DNA holds. We have seen applications extend far beyond what we thought imaginable just a decade ago, which can be attributed to the improvement in both, laboratory sequencing techniques and computational analysis strategies. 


__Data Collection__

Genomic data can exist in many different forms. However, with the goal of finding a way to classify individuals based upon their ancestral lineage, it should make sense that we are going to be looking for the differences in the genomic sequence between each group of people. This makes dealing with our data a lot easier because while our whole genome sequence is a little over 6 billion basepairs anywhere from 90%-95% of those nucleotides are conserved throughout all humans. This means that the majority of an individuals' genomic information, at least for our purposes, can be ignored. In this way, we are using our understanding of biology and what we know our species genome to reduce the dimensionality of our model before we even begin. 

The *1000s Genome Project* website has made public, genomic data of 2548 non-related individuals belonging to 25 different ancestral subpopulations. The data is organized in a Variant Call File (VCF), which includes only the locations along our the samples population's genome that differ from the most recent *Human Reference Genome CH38*. These files are organized with each row representing the basepair location where at least one sampled individual possessed an alternate allel. At any single point along the genome there could be as many as 5016 alternate allels (two for each individual). Fortunately for us, the scientists who took part in this sequencing project, used laboratory techniques to run quality checks on each read. Provided, neatly in a column named "FILTER QUALITY" we are able to select only the variant call locations that have a high probability of being accurate. Even so, it is still important to run our own evaluations after data collection, to ensure that our data is reliable. The LOCATION: an integer, REFERENCE: a nucleotide, and ALTERNATE: a different nucleotide, features will be essential; however, using visualizations to examine the distribution of our read depth and allele count features proved informative as well. 

Even though the 1000s Genome Project collected variants for from each chromosome, creating a model using all 24 chromosomes (no reads done on ChrY) would require significantly more time and computing power, and complexity does not guarantee accuracy. It is important to understand that each of our chromosomes have different lengths and structure, and of course they posses sequences for different genes. So after a some research I decided to train my model using only the call file for Chr21. Not only has this chromosome been known to vary considerably between populations, making it a stronger predictor of ancestral lineage, but it is conveniently one of the smallest chromosomes. 

__Preprocessing Data__

The first step that we must do is isolate each population using the sample codes that were webscraped off of the International Genome Project website page. Using the population specific samples codes (see Webscraping.ipynb) we were able to assemble genotype arrays for each of our 10 subpopulations, where each column was an individual and each row was a basepair location. The data was formatted where a 0 represented the reference allele and a 1 represented a variant at each location. With 10 different population arrays of different size I concatenated our data into one large array containing each individual from every subpopulation. It was important to record where each population started and ended so that after dimensionality reduction we could separate the populations for classification purposes. I decided on Principal Component Analysis (PCA) in order to reduce dimensionality, and because our input data is not continuous, and instead is binary values, PCA needed to be performed using Singular Value Decomposiiton.

__Exploratory Data Analysis__ 

The next step is to ensure the quality of our data, which I did by using my understanding of biology and NGS data to create a few informative visualizations and calculations. In the file labeled *Genomics_Project_File* I was able to examine three important features of our Variant Call File (.vcf), checking each result against the expected biological norm. 

(1)Variant loci 
(2)Variant Depth 
(3)Ti:Tv 

The first feature I visualized were the positions, also known as loci, along chromosome 21 that contained single nucleotide variants(SNVs). What this showed was that there were no variants within the first 5 million basepair positions of the chromosome, meaning all individuals in our file, belonging to 25 different subpopulations, had the same nucleotide sequence as each other and as our reference genome. While this might seem uncommon, a stretch of the genome that is conserved amongst all humans occurs every so often in the genome. It can be interpreted as a region that is structurally protected against mutation via methylation, or a region that codes for functions so essential for development and/or survival that any mutation in this region will fail to produce a viable offspring. After this conserved region of the genome there is a spike in the mutation rate occuring around... 



