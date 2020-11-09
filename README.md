## Ancestral Genome Composition Classifier 

For this project the objective is to train a classification model that can correctly classify an individual into a specfic ancestral subpopulation using portions of thier genomic data as input. 

For this project I knew I wanted to work with large scale human genomic data. In recent years companies like 23andMe and Ancestry have been able to offer the public relatively cheap insights into the secrets that an individual's DNA holds. We have seen applications extend far beyond what we thought imaginable just a decade ago, which can be attributed to the improvement in both laboratory sequencing techniques and computational analysis strategies. The steps I will go over in this project are as follows: 
- Downloading Genomic Data in the form of variant call files 
- Conducting exploratory data analysis and quality control measures to ensure variant call quality
- Cleaning and scaling features, which involved identifying and reducing multicollinearlity between variants that arose due to biological processes
- Reducing dimensionality of our input features, and indentify the most significant biomarkers
- Training an SVM algorithm on the resulting features with the objective of classifying samples based on their respective geographic subpopulation 


__Data Collection__

Genomic data can exist in many different forms. However, with the goal of finding a way to classify individuals based upon their ancestral lineage, it should make sense that we are going to be looking for the differences in nucleotide bases between and among groups of people. This fact makes dealing with our data a lot easier, while our whole genome sequence is a little over 6 billion basepairs, anywhere from 90%-95% of nucleotide basepairs are conserved throughout all humans. This means that the majority of an individuals' genomic information, at least for our purposes, can be ignored. In this way, we are using our understanding of biology and what we know our species genome to reduce the dimensionality of our data before we even begin. 

The *1000s Genome Project* website has made public, genomic data of 2508 non-related individuals belonging to 25 different ancestral subpopulations. Genomic data can be found in a few file formats depending on the type of sequencing data and where along the sequencing pipeline you look. Nevertheless, a common file format for genomic data is going to be a Variant Call File (VCF), which includes only the locations along our the samples population's genome that differ from a specific reference genome, *Human Reference Genome CH38* for this dataset. These files are organized with each column representing a sample and each row representing a locus where at least one sampled individual possessed an alternate allel. At any single point along the genome there could be as many as 5016 alternate alleles. Fortunately for us, the scientists who took part in this sequencing project used laboratory techniques to run quality checks on each read. Provided neatly in a column named "FILTER QUALITY", we are able to select only the variant call locations that have a high probability of being accurate. Even so, it is still important to run our own evaluations after data collection to ensure that our data is reliable. The LOCATION: an integer, REFERENCE: a nucleotide, and ALTERNATE: a different nucleotide, features will be essential; however, using visualizations to examine the distribution of our read depth and allele count features should prove informative as well. 

Even though the 1000s Genome Project collected variants from each chromosome, creating a model using all 24 chromosomes (no reads done on ChrY) would require a significant amount of time and computing power, and complexity does not always guarantee accuracy. It is important to understand that each of our chromosomes have different lengths and structure, and of course they code for different genes. After some research I decided to train my model using only the call file for Chr21. Not only has this chromosome been known to vary considerably between populations, making it a stronger predictor of ancestral lineage, but it is conveniently one of the smallest chromosomes and therefor a more manageable amount of data to manipulate. 

__Preprocessing Data__

The first step that we must do is isolate each population using the sample codes that were webscraped off of the International Genome Project website page. Using the population specific samples codes (see Webscraping.ipynb) we were able to assemble genotype arrays for 10 randomly selected subpopulations, where each column was an individual and each row was a basepair location. The data was formatted where a 0 represented the reference allele and a 1 represented a variant at each location. With 10 different population arrays of different size I concatenated our data into one large array containing each individual from every subpopulation. It was important to record where each population started and ended so that after dimensionality reduction we could separate the populations for classification purposes. I decided on Principal Component Analysis (PCA) in order to reduce dimensionality, and because our input data is not continuous, and instead is binary values, PCA needed to be performed using Singular Value Decomposiiton.

__Exploratory Data Analysis__ 

The next step is to ensure the quality of our data, which I did by using my understanding of biology and NGS data to create a few informative visualizations and calculations. In the file labeled *Genomics_Project_File* I was able to examine three important features of our Variant Call File (.vcf), checking each result against the expected biological norm. 

> (1)Variant Loci 
> (2)Variant Depth Coverage
> (3)Ti:Tv 

The first feature I visualized were the positions, also known as loci, along chromosome 21 that contained single nucleotide variants(SNVs). What this showed was that there were essentially no mutations appearing within the first 5e<sup>6</sup> basepair positions on this chromosome for all individuals across every subpopulation. Meaning all individuals in our file, belonging to 25 different subpopulations, had the same nucleotide sequence as each other AND as our reference genome (Hrf.g38). While this might seem uncommon, a stretch of the genome that is conserved amongst all humans occurs every so often in the genome. It can be interpreted as a region that is structurally protected against mutation via molecular processes like methylation, or a region that codes for functions so essential for development and/or survival that any mutation in this region will fail to persist through multiple generations. After this convserved region of the genome the distribution graph shows a spike of variants occuring arround 15 million basepairs along the chromosome, these mutations are seen evenly distributed with a frequency of approximately .03 bp<sup>-1</sup> for the remaining windows within our 48 million nucleotide sequence. The initial conserved region seen on this graph could also be a result of the acrocentric structure of the chromsome. Studies have proven that the region that codes for the centromere has always been structurally constricted and evolutionarily conserved within eukaryotic cells. The data that is provided by this visualization at the very least does not suggest anything to be amiss with our data. With a similar thought process as the members of the HGP, Chromosome 21's high heterogenity and relatively small number of basepairs make it an ideal candidate to choose for primary evaluation efforts. 

The next exploratory step I conducted also constructs a histogram attribute of the variant calls. Instead of observing the positions [POS] of the variants, we take a look at the Read Depth [RD] of each variant call. Read depth can be thought of as the number of times that our single nucleotide variant was cataloged during the next generation sequencing process. The more times that a variant call is "read" at a location the more confident we can be that the variant was accurately identified, and not a false read. This confidence can in part be deteremined by the read depth [RD] and is an important feature to consider in any variant callset. While our sequencing data has already been filtered for quality using Phred quality scores as part of the *1000s Genome Project* it can't hurt to examine the distribution of read depths for each of our variant calls. The distribution that we see is a long tailed distribution with a majotiry of our variants being read less than 10,000 times. This distribution is presumably a result of the preprocessing steps, where researches will set a cutoff that eliminates the variant calls that were not read enough due to the increased likelihood of those variants being *false reads*. So this explororatory analysis and data visualization validates another feature of our population genomic callset. 

Finally, I used two features present in our data table to examine the types of mutations that were present in our dataset. The complicated process of DNA replication leaves room for a number of different types of mutations to manifest. These include multiple copies of a sequence being produced, nucleotide insertions/deletions, and most relevant to this study: single nucleotide variantions (SNV). This last form of mutation occurs when one of our correct nucleotide bases "A,C,G or T" is substituted for an incorrect nucleotide, and can either be a result of a transversion or translation depending on which base pair is subsituted for which. What scientists have discovered during their time studying the human genome is that, for a number of biochemical processes and molecular structure constraints, the ratio of transitions to transversions throughout most of the human genome is between 2.00 ~ 3.00. In order to ensure that the variants in my dataset arose from congruent biological processes, I wrote a function to calculate this genomic attribute. Using the REFERENCE and ALTERNATE columns, which contains the corresponding nucleotides for that individual in a string format. I examined only mutations classified as SNPs and iterated through the DataFrame counting  up the occurance of transversions and transitions. Then with these totals I simply divided the amount of transitions/tranversions giving us a ratio of 2.32 Ti/Tv. Considering the experimental value for (Ti/Tv) in the entire human genome is approximately 2.0-2.1 and as high as 3.0 in protein coding regions of the genome, our observed value of 2.3 is on par with what we would expect to see from properly sequenced human genetic data. 

__Feature Engineering__

Once the data had been validated the next step was determining how we were going to assemble a set of features that can provide predictive value for the individuals within each subpopulation. Our current dataset had biallelic variant calls at approximately 1 million different locations along the genome, meaning for the entire population sampled there was at least one variant call at 1 million different loci. Now that does not mean that each individual had 1 million different alleles from the human reference genome, in fact of our ~2500 individuals most of the single nucleotide variants called at a specific locus were only present in a small portion of the samples. This resulted in a sparse matrix of features for each sample, with over 1 million rows attached to each sample, but an overwhelming majority of them containing the expected reference allele at each locus.

