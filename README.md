# Ev1_SelectionMetaAnalysis

There are lots of interesting patterns that you can extract from genetic marker data. This can include patterns of linkage, balancing selection, or even inbreeding signals. One of the most common ones is to try find sites on the genome that are under divergent selection. The following vignette will take you through the basics of genetic selection analysis. 

The project has been funded by the <a href="https://ausevo.com/ECR_grants_2022/">AES ERC Networking Grant Scheme</a>.

<h2>A <i>fairly</i> brief introduction to Genetic Outlier and Association Analysis</h2>

When we look through a genome to try find loci that are under divergent selection, we often conduct what is called outlier or association analyses. **Outlier analysis** requires just knowledge of the genetics of your samples (plus sample metadata, for example population groupings), and tries to find loci that behave very differently from the underlying patterns across the genome (with the assumption being that the rest of the genome represents patterns of neutral genetic diveristy)<sup>A</sup>. Meanwhile **association analysis** require some sort of covariate data, and tests whether there are any genetic variants statistically associated with this new data (you may have heard the term [GWAS](https://www.genome.gov/genetics-glossary/Genome-Wide-Association-Studies)). This data can come in the form of phenotype data (e.g. morphology, disease status, physiology measures), or could be spatial (e.g. environmental, climate). Association tests look for sites in the genome where the precense or absence of a variant is highly correlated with the values in the co-variate data, usually through some regression type analysis.

<sup>A</sup> it is very important then to account for any population substructure.

> :information_source: **An important note about additive genetic variance**
>
> It is important to bear in mind how the input genetic data for outlier or association models is being interreted by the model. When dealing with many of these models (and input genotype files) the assumption is that the SNP effects are [additive](https://link.springer.com/referenceworkentry/10.1007/978-3-319-47829-6_5-1). This can be seen from, for example, the way we encode homozygous reference allele, heterozygous, and homozygous alternate allele as "0", "1", and "2" respectively in a BayPass input genofile. For the diploid organism (with two variant copies for each allele) one copy of a variant (i.e. heterozygous) is assumed to have half the effect of having two copies. However, what if the locus in question has dominance effects? This would mean the heterozygous form behaves the same as the homozygous dominant form, and it would be more appropriate to label these instead as "0", "0", "1". But with thousands, if not millions of (most likely) completely unknown variants in a dataset, how can we possibly know? The answer is we cannot. And most models will assume additive effects, because this the simplest assumption. However, by not factoring in dominance effects we could possible be missing many important functional variants, as Reynolds et al. [2021](https://www.nature.com/articles/s41588-021-00872-5) demonstrates. Genomics is full of caveats and pitfalls, which while providing new directions to explore can be a bit overwhelming. Remember, you selection analysis doesn't have to be exhaustive, just make sure it is as fit for purpose within your study design. There is so much going on in just one genome, there is no way you can analyse everything in one go. 

Throughout this vignette I will refer to outlier and association analysis collectively as selection analysis. There are a lot of programs that exist currently. You can find a very long (though not exhaustive) list of them [here](https://bioinformaticshome.com/tools/gwas/gwas.html). 

This vignette still start by covering some very simple outlier analyses:
<ul>
<li><a href="https://bcm-uga.github.io/pcadapt/articles/pcadapt.html"><b>PCAdapt</b></a>, a program that can depect genetic marker outliers without having population<sup>B</sup> designations, using a Principle Component Analysis (PCA) approach.</li>
<li><a href="GBIF"><b>F<sub>ST</sub></b></a> outlier analysis, an approach that uses pairwise comparisons between two populations and the fixation index metric to assess each genetic marker.</li>
</ul>

Next we will conduct some more advanced outlier analysis:
<ul>
<li><a href="https://bcm-uga.github.io/pcadapt/articles/pcadapt.html"><b>Bayescan</b></a>, a program.</li>
<li><a href="GBIF"><b>Baypass</b></a>, a program that elaborates on the bayenv model (another popular association analysis program).</li>
</ul>

<sup>B</sup> I will say population for simplicity throughout this vignette. However, equally we can test for differences between sample sites, subpopulations, and other types of groupings. What counts as one 'group' of organisms will be dependent on your study system or study question.

We'll cover the pre-processing of program specific input files, how to run the programs, how to visualise the output and also in some cases we'll need to take extra steps to map the genetic markers of interest back to the SNP data.  

> :information_source: **An important note about reduced representation verses whole genome sequencing**
>
> Completing outlier analysis is possible and often done on reduced representation data. It is important to remember how your genome coverage (the number of genome variant sites / the genome length <sup>C</sup>) will affect your results and interpretation. Often with WGS data, you will see well resolved 'peaks' with a fairly smooth curve of points leading up to it either side. From this we often infer that the highest point is the genetic variant of interest, and the other sites either side of that exhibit signals of selection because they reside close to, and thus are linked, to the variant of interest. However, consider that even in WGS data, unless we have every single genetic variant represented (which may not be the case, depending on our variant calling and filtering parameters) it is possible that the genetic variant of interest that we have identified is not the main one, but is simply another neighbouring linked SNP to one that is not represented in the data. This problem becomes even more relevant with reduced representation sequencing (RRS), for which the genome coverage may be extremely patchy<sup>C</sup>. Thus with all outlier analysis, but especially so for those using RRS data, remember that your flagged outliers are not exhaustive, and may themselves only be liked to the variant that is truely under selection.

<sup>C</sup> You may also want to consider linkage blocks.


https://onlinelibrary.wiley.com/doi/10.1111/mec.14549

https://pubmed.ncbi.nlm.nih.gov/29486732/

 https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.9176

![ScreenShot](https://els-jbs-prod-cdn.jbs.elsevierhealth.com/cms/asset/4be56b5b-8593-4116-ab9a-ec7b9e3c9a05/gr1.jpg)

also include  an hour of group discussion on one of the days about research question framing + grant integration




## What data are we starting with

Metadata file, including individual and population names

covariates inc. environmental metadata.

The a genetic file. We will start with a VCF file.

Working with your own data


## Define you working directory for this project, and the VCF file location:

<pre class="r"><code>cd ~
mkdir outlier_analysis
DIR=~/outlier_analysis
cd $DIR
VCF=/location/of/vcf
</code></pre>

Our data tree will look like:

> :heavy_check_mark: outlier_analysis
> ss
> s
> analysis
>    ├── bayescan
> │   ├── baypass
> │   ├── pcadapt
> │   ├── summary
> │   └── vcftools_fst
> └── data

```
 analysis
    ├── bayescan
 │   ├── baypass
 │   ├── pcadapt
 │   ├── summary
 │   └── vcftools_fst
 └── data
```

So lets set up our directories to match this

<pre class="r"><code>cd $DIR
mkdir -p {analysis/{bayescan,baypass,pcadapt,summary,vcftools_fst},data}
</code></pre>

## PCAdapt

The PCAdapt manual is available [here](https://bcm-uga.github.io/pcadapt/articles/pcadapt.html).

Let's prepare what we need: first we convert the VCF to PLINK, and then to BED, as we need both file types at various point during this vignette

<pre class="r"><code>cd $DIR/data
module load vcftools/0.1.16
module load plink/1.90b6.7 
cp $VCF . #lets make sure we have a copy of our VCF in our project working directory
vcftools --vcf $VCF --out (basename $VCF .vcf).plink --plink
plink --file (basename $VCF .vcf).plink --make-bed --noweb --out (basename $VCF .vcf)
</code></pre>

Install PCAdapt, load in the data, and set your working directory.

<pre class="r"><code>module load R/3.5.3
R

install.packages("pcadapt")
library(pcadapt)

setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/pcadapt/")
</code></pre>

<pre class="r"><code>starling_bed <- "/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data/starling_3populations.bed"
starlings_pcadapt <- read.pcadapt(starling_bed, type = "bed")
</code></pre>

Produce K plot

starlings_pcadapt_kplot <- pcadapt(input = starlings_pcadapt, K = 20)

<pre class="r">
pdf("pcadapt_starlings_kplot.pdf")
plot(starlings_pcadapt_kplot, option = "screeplot")
dev.off()
</code></pre>

K value of 3 is most appropriate.

```
starlings_pcadapt_pca <- pcadapt(starlings_pcadapt, K = 3)
summary(starlings_pcadapt_pca)
```
> test
> 
> test
>                 Length Class  Mode
> scores            117  -none- numeric
> singular.values     3  -none- numeric
> loadings        15021  -none- numeric
> zscores         15021  -none- numeric
af               5007  -none- numeric
maf              5007  -none- numeric
chi2.stat        5007  -none- numeric
stat             5007  -none- numeric
gif                 1  -none- numeric
pvalues          5007  -none- numeric
pass             4610  -none- numeric

Investigate axis projections

poplist.names <- c(rep("Lemon", 13),rep("Warrnambool", 13),rep("Nowra", 13))
print(poplist.names)

pdf("pcadapt_starlings_projection1v2.pdf")
plot(starlings_pcadapt_kplot, option = "scores", i = 1, j = 2, pop = poplist.names)
dev.off()

pdf("pcadapt_starlings_projection5v7.pdf")
plot(starlings_pcadapt_kplot, option = "scores", i = 5, j = 7, pop = poplist.names)
dev.off()
Ignore the warning: Use of `df$Pop` is discouraged. Use `Pop` instead.




Investigate manhattan and qqplot

pdf("pcadapt_starlings_manhattan.pdf")
plot(starlings_pcadapt_pca, option = "manhattan")
dev.off()

pdf("pcadapt_starlings_qqplot.pdf")
plot(starlings_pcadapt_pca, option = "qqplot")
dev.off()



Plotting and correcting the pvalues

starling_pcadapt_pvalues <- as.data.frame(starlings_pcadapt_pca$pvalues)

library("ggplot2")

pdf("pcadapt_starlings_pvalues.pdf")
hist(starlings_pcadapt_pca$pvalues, xlab = "p-values", main = NULL, breaks = 50, col = "orange")
dev.off()

starlings_pcadapt_padj <- p.adjust(starlings_pcadapt_pca$pvalues,method="bonferroni")
alpha <- 0.1
outliers <- which(starlings_pcadapt_padj < alpha)
length(outliers)

write.table(outliers, file="starlings_pcadapt_outliers.txt") 



> length(outliers)

[1] 3

After this, we will be jumping out of R and back into the command line. 

q()

Mapping Outliers: PCAdapt

finding the SNP ID of the outlier variants

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/

#create list of SNPs in VCF, assign line numbers that can be used to find matching line numbers in outliers (SNP ID is lost in PCadapt & Bayescan, line numbers used as signifiers).
#we create this in the analysis folder because we will use it for more than just mapping the outlier SNPs for PCAdapt
grep -v "^#" ../../data/starling_3populations.recode.vcf | cut -f1-3 | awk '{print $0"\t"NR}' > starling_3populations_SNPs.txt

#grab column 2 of the outlier file, which contain the number of the outliers
awk '{print $2}' starlings_pcadapt_outliers.txt > starlings_pcadapt_outliers_numbers.txt

#list of outlier SNPS ID's
awk 'FNR==NR{a[$1];next} (($4) in a)' starlings_pcadapt_outliers_numbers.txt ../starling_3populations_SNPs.txt   | cut -f3 > snp_pcadapt_outliers_SNPs.txt






VCFtools windowed Fst:
https://vcftools.sourceforge.net/man_latest.html

We can work with out VCF file. But need to set up popfiles for pairwise comparisons

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data

grep "Lemon" 3pops.txt > 3pops_Lemon.txt
grep "War" 3pops.txt > 3pops_War.txt
grep "Nowra" 3pops.txt > 3pops_Nowra.txt
Now we can run the pairwise analysis. Let's focus on just Lemon v.s. War

module load vcftools/0.1.16

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/vcftools_fst

VCF=/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data/starling_3populations.recode.vcf

vcftools --vcf $VCF --weir-fst-pop ../../data/3pops_Lemon.txt --weir-fst-pop ../../data/3pops_War.txt --out lemon_war_fst

head lemon_war_fst.weir.fst

wc -l lemon_war_fst.weir.fst 
try stepped windows

vcftools --vcf $VCF --fst-window-size 50000 --fst-window-step 10000 --weir-fst-pop ../../data/3pops_Lemon.txt --weir-fst-pop ../../data/3pops_War.txt --out lemon_war_fst

head lemon_war_fst.windowed.weir.fst

wc -l lemon_war_fst.windowed.weir.fst 
plotting

awk '{print $0"\t"NR}' ./lemon_war_fst.windowed.weir.fst  > lemon_war_fst.windowed.weir.fst.edit

module load R/3.5.3
R

library("ggplot2")

setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/vcftools_fst")

windowed_fst <- read.table("lemon_war_fst.windowed.weir.fst.edit", sep="\t", header=TRUE)
str(windowed_fst)

quantile(windowed_fst$WEIGHTED_FST, probs = c(.95, .99, .999))

pdf("fst_starlings_windowed.pdf", width=10, height=5)
ggplot(windowed_fst, aes(x=X1, y=WEIGHTED_FST)) + 
geom_point() + 
theme_classic() +
geom_hline(yintercept=0.35, linetype="dashed", color = "red")
dev.off()


A much better example:



Grabbing the outliers

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/vcftools_fst
cat lemon_war_fst.windowed.weir.fst.edit | awk '$5>0.35' > lemon_war_fst.windowed.outliers
wc -l lemon_war_fst.windowed.outliers
#112 lemon_war_fst.windowed.outliers

cat lemon_war_fst.windowed.weir.fst.edit | awk '$5>0.35 ' | cut -f1-3 > lemon_war_fst.windowed.outliers.bed 
vcftools --vcf $VCF --bed lemon_war_fst.windowed.outliers.bed --out fst_outliers --recode
#After filtering, kept 63 out of a possible 5007 Sites






Bayescan:
https://github.com/mfoll/BayeScan

Run PGDSpider to convert file for bayescan: http://www.cmpg.unibe.ch/software/PGDSpider/

to PGD format:

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan

java -Xmx1024m -Xms512m -jar /srv/scratch/z5188231/KStuart.Starling-Aug18/programs/PGDSpider_2.1.1.5/PGDSpider2-cli.jar -inputfile /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data/starling_3populations.recode.vcf -inputformat VCF -outputfile starling_3populations.txt -outputformat  PGD -spid VCF_PGD.spid 
include snapshot of SPID:


# spid-file generated: Thu Aug 13 19:52:55 AEST 2020

# VCF Parser questions

PARSER_FORMAT=VCF

# Only output SNPs with a phred-scaled quality of at least:

VCF_PARSER_QUAL_QUESTION=

# Select population definition file:

VCF_PARSER_POP_FILE_QUESTION=/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data/3pops_pops.txt

# What is the ploidy of the data?

VCF_PARSER_PLOIDY_QUESTION=DIPLOID

# Do you want to include a file with population definitions?

VCF_PARSER_POP_QUESTION=true

# Output genotypes as missing if the phred-scale genotype quality is below:

VCF_PARSER_GTQUAL_QUESTION=

# Do you want to include non-polymorphic SNPs?

VCF_PARSER_MONOMORPHIC_QUESTION=false

# Only output following individuals (ind1, ind2, ind4, ...):

VCF_PARSER_IND_QUESTION=

# Only input following regions (refSeqName:start:end, multiple regions: whitespace separated):

VCF_PARSER_REGION_QUESTION=

# Output genotypes as missing if the read depth of a position for the sample is below:

VCF_PARSER_READ_QUESTION=

# Take most likely genotype if "PL" or "GL" is given in the genotype field?

VCF_PARSER_PL_QUESTION=false

# Do you want to exclude loci with only missing data?

VCF_PARSER_EXC_MISSING_LOCI_QUESTION=false



# PGD Writer questions

WRITER_FORMAT=PGD



au05_men        SOUTH

au06_men        SOUTH



then convert to bayescan

java -Xmx1024m -Xms512m -jar /srv/scratch/z5188231/KStuart.Starling-Aug18/programs/PGDSpider_2.1.1.5/PGDSpider2-cli.jar -inputfile starling_3populations.txt -inputformat PGD -outputfile starling_3populations.bs -outputformat GESTE_BAYE_SCAN
BAYESCAN RUNS

#!/bin/bash
#PBS -N 2021-11-21.bayescan_starling.pbs
#PBS -V
#PBS -l nodes=1:ppn=16
#PBS -l mem=40gb
#PBS -l walltime=12:00:00
#PBS -j oe
#PBS -M katarina.stuart@unsw.edu.au
#PBS -m ae

module load bayescan/2.1

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan

bayescan_2.1 ./starling_3populations.bs -od ./ -threads 16 -n 5000 -thin 10 -nbp 20 -pilot 5000 -burn 50000 -pr_odds 10

 

Identify outliers:

module load R/3.5.3
R
library(ggplot2)
setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan")
source("/apps/bayescan/2.1/R\ functions/plot_R.r")
outliers.bayescan=plot_bayescan("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan/starling_3population_fst.txt",FDR=0.05)
outliers.bayescan
write.table(outliers.bayescan, file="bayscan_outliers.txt")




Mapping Outliers

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan

#create list of SNPs in VCF, assign line numbers that can be used to find matching line numbers in outliers (SNP ID is lost in bayescan, line numbers used as signifiers).

grep -v "^#" ../../data/starling_3populations.recode.vcf  | cut -f1-3 | awk '{print $0"\t"NR}' > starling_3populations_SNPs.txt

awk '{print $2}' bayscan_outliers.txt > bayscan_outliers_numbers.txt

#list of outlier SNPS
awk 'FNR==NR{a[$1];next} (($4) in a)' bayscan_outliers_numbers.txt starling_3populations_SNPs.txt   | cut -f3 > bayscan_outliers_SNPs.txt

Bayescane Log Plot

module load R/3.5.3
R
library(ggplot2)
library(dplyr)
setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/bayescan")

bayescan.out<- read.table("starling_3population_fst.txt", header=TRUE)
bayescan.out <- bayescan.out %>% mutate(ID = row_number())
bayescan.outiers<- read.table("bayscan_outliers_numbers.txt", header=FALSE)
outliers.plot <- filter(bayescan.out, ID %in% bayescan.outiers[["V1"]])

png("bayescan_outliers.png", width=600, height=350)
ggplot(bayescan.out, aes(x=log10.PO., y=alpha))+
geom_point(size=5,alpha=1)+xlim(-1.3,3.5)+ theme_classic(base_size = 18) + geom_vline(xintercept = 0, linetype="dashed", color = "black", size=3)+
geom_point(aes(x=log10.PO., y=alpha), data=outliers.plot, col="red", fill="red",size=5,alpha=1) + theme(axis.text=element_text(size=18), axis.title=element_text(size=22,face="bold"))
dev.off()








BayPass:
http://www1.montpellier.inra.fr/CBGP/software/baypass/files/BayPass_manual_2.31.pdf



PLINK=/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/data/starling_3populations.plink.ped

cut -f 3- $PLINK > x.delete
paste 3pops_plink.txt x.delete > starling_3populations.plink.ped
rm x.delete 
cp ../../data/starling_3populations.plink.map .
cp ../../data/starling_3populations.plink.log .

#run the pop based allele frequency calculations
plink --file starling_3populations.plink --allow-extra-chr --freq counts --family --out starling_3populations
#manipulate file so it has baypass format, numbers set for plink output file and pop number for column count
tail -n +2 starling_3populations.frq.strat | awk '{ $9 = $8 - $7 } 1' | awk '{print $7,$9}' | tr "\n" " " | sed 's/ /\n/6; P; D' > starling_3populations_baypass.txt


Baypass runs:

#!/bin/bash
#PBS -N 2022-12-08.baypass_starling.pbs
#PBS -l nodes=1:ppn=16
#PBS -l mem=40gb
#PBS -l walltime=12:00:00
#PBS -j oe
#PBS -M katarina.stuart@unsw.edu.au
#PBS -m ae

module load baypass/2.1

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass

g_baypass -npop 3 -gfile ./starling_3populations_baypass.txt -outprefix starling_3populations_baypass -nthreads 16




Running in R to make the anapod data: 

module load R/3.6.3
R
setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass")
source("/apps/baypass/2.1/utils/baypass_utils.R")
library("ape")
library("corrplot")

omega=as.matrix(read.table("starling_3populations_baypass_mat_omega.out"))
pi.beta.coef=read.table("starling_3populations_baypass_summary_beta_params.out",h=T)$Mean
bta14.data<-geno2YN("starling_3populations_baypass.txt")
simu.bta<-simulate.baypass(omega.mat=omega, nsnp=5000, sample.size=bta14.data$NN, beta.pi=pi.beta.coef,pi.maf=0,suffix="btapods")


#!/bin/bash
#PBS -N 2022-12-08.baypass_starling2.pbs
#PBS -l nodes=1:ppn=16
#PBS -l mem=40gb
#PBS -l walltime=12:00:00
#PBS -j oe
#PBS -M katarina.stuart@unsw.edu.au
#PBS -m ae

module load baypass/2.1

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass

g_baypass -npop 2 -gfile G.btapods  -outprefix G.btapods -nthreads 16 



XtX calibration; get the pod XtX

module load R/3.6.3
R
setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass")
source("/apps/baypass/2.1/utils/baypass_utils.R")
library("ape")
library("corrplot")

pod.xtx=read.table("G.btapods_summary_pi_xtx.out",h=T)$M_XtX
Compute the 1% threshold

pod.thresh=quantile(pod.xtx,probs=0.99)
pod.thresh
Threshold is: 4.73302



Filter the data for the outlier snps:

#Find outliers that are above theanapod threshold
cat starling_3populations_baypass_summary_pi_xtx.out | awk '$6>4.73302 ' > baypass_outliers.txt

#create list of SNPs in VCF, assign line numbers that can be used to find matching line numbers in outliers (SNP ID is lost in bayescan, line numbers used as signifiers).
grep -v "^#" ../../data/starling_3populations.recode.vcf  | cut -f1-3 | awk '{print $0"\t"NR}' > starling_3populations_SNPs.txt

#list of outlier SNPS
awk 'FNR==NR{a[$1];next} (($4) in a)' baypass_outliers.txt starling_3populations_SNPs.txt | cut -f3 > baypass_outliers_SNPlist.txt
wc -l baypass_outliers_SNPlist.txt 
275



Running with covariate data

#!/bin/bash
#PBS -N 2022-12-08.baypass_starling3.pbs
#PBS -l nodes=1:ppn=16
#PBS -l mem=40gb
#PBS -l walltime=12:00:00
#PBS -j oe
#PBS -M katarina.stuart@unsw.edu.au
#PBS -m ae

module load baypass

cd /srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass

g_baypass -npop 3 -gfile starling_3populations_baypass.txt -efile baypass_environment_covariate.txt -scalecov -auxmodel -nthreads 16 -omegafile starling_3populations_baypass_mat_omega.out -outprefix starling_3populations_baypass_enviro



Plotting

module load R/3.6.3
R

library(ggplot2)

setwd("/srv/scratch/z5188231/KStuart.Starling-Aug18/Ev1_SelectionMetaAnalysis/analysis/baypass")

covaux.snp.res.mass=read.table("starling_3populations_baypass_enviro_summary_betai.out",h=T)
covaux.snp.xtx.mass=read.table("starling_3populations_baypass_summary_pi_xtx.out",h=T)$M_XtX

pdf("Baypass_plots.pdf")
layout(matrix(1:3,3,1))
plot(covaux.snp.res.mass$BF.dB.,xlab="Mass",ylab="BFmc (in dB)")
abline(h=20, col="red")
plot(covaux.snp.res.mass$M_Beta,xlab="SNP",ylab=expression(beta~"coefficient"))
plot(covaux.snp.xtx.mass, xlab="SNP",ylab="XtX corrected for SMS")
dev.off()




Filtering the data sets for SNPS above BFmc threshold and finding which are also divergent amongst populations

cat starling_3populations_baypass_enviro_summary_betai.out | awk '$6>20' > starling_3populations_baypass_enviro_BF20.txt
starling_3populations_baypass_enviro_BF20.txt
#40

awk 'FNR==NR{a[$2];next} (($4) in a)' starling_3populations_baypass_enviro_BF20.txt starling_3populations_SNPs.txt | cut -f3 > starling_3populations_baypass_enviro_BF20_SNPlist.txt

comm -12 <(sort starling_3populations_baypass_enviro_BF20_SNPlist.txt) <(sort baypass_outliers_SNPlist.txt) > output.txt
#38


## Funding 
<p align="center">

![ScreenShot](https://storage.corsizio.com/uploads/5cea29e798d9a757e03dba1c/events/6141a39502de2a7ff2f3f120/photo-_3UnQXVrb.jpg)

</p>

Thank you to the <a href="https://ausevo.com/ECR_grants_2022/">AES ERC Networking Grant Scheme</a> for funding this project.

## Project Contributors

https://www.dataschool.io/how-to-contribute-on-github/

