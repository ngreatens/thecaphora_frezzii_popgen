* 12/16/25

## first get phylogenetic tree from SNPS

```
python /project/fdwsru_fungal/Nick/git_repos/vcf2phylip/vcf2phylip.py -i biallelic_SNPS.vcf --output-folder phylogeny
cd phylogeny
iqtree2 -s biallelic_SNPS.min4.phy --seq-type DNA -m GTR
```

```
(SanGuillermo_markdup.bam:0.0046670928,(((((((((((((((((((((((((Poale_markdup.bam:0.0032378201,135885_markdup.bam:0.0015523033):0.0011671009,33_5_markdup.bam:0.0013153348):0.0009960985,(30_5_markdup.bam:0.0008006143,36_1_markdup.bam:0.0042797978):0.0004126663):0.0006513902,136066_markdup.bam:0.0006536207):0.0043296905,((AgnaDulce_markdup.bam:0.0003793340,34_1_markdup.bam:0.0046314088):0.0010954446,(56_1_markdup.bam:0.0018860026,Cerroni_markdup.bam:0.0003385140):0.0002098948):0.0020273991):0.0022211846,(Ponte_markdup.bam:0.0001783692,(141696_markdup.bam:0.0014472733,45_2_markdup.bam:0.0017158740):0.0008892359):0.0011130102):0.0011362241,18_10_markdup.bam:0.0019056377):0.0016955041,((((12_2_markdup.bam:0.0022538822,135674_markdup.bam:0.0002993933):0.0004795940,((135909_markdup.bam:0.0046317816,136044_markdup.bam:0.0013266934):0.0044237672,17_10_markdup.bam:0.0028594956):0.0013594684):0.0054149638,55_11_markdup.bam:0.0026674936):0.0050377011,22_1_markdup.bam:0.0035643340):0.0036408881):0.0031373997,48_2_markdup.bam:0.0011751039):0.0018239372,((136133_markdup.bam:0.0032544509,((136101_markdup.bam:0.0044204171,32_2_markdup.bam:0.0035242835):0.0010139234,23_1_markdup.bam:0.0037373601):0.0007349144):0.0116078416,29_2_markdup.bam:0.0004202375):1.7872136735):0.0016779528,54_5_markdup.bam:0.0017793582):0.0054687327,(((141782_markdup.bam:0.0003157044,(50_9_markdup.bam:0.0077504236,3_1_markdup.bam:0.0018132666):0.0009527650):0.0019584960,((138938_markdup.bam:0.0001503587,136570_markdup.bam:0.0006377901):0.0003961772,19_5_markdup.bam:0.0117399368):0.0019433820):0.0014952436,((141736_markdup.bam:0.0003768754,135825_markdup.bam:0.0016133237):0.0005767225,(7_3_markdup.bam:0.0020880330,43_2_markdup.bam:0.0037445607):0.0012473259):0.0019621236):0.0078612154):0.0046106464,6_2_markdup.bam:0.0045607519):0.0038470073,(26_4_markdup.bam:0.0032716536,8_2_markdup.bam:0.0038058983):0.0023172246):0.0051282918,(15_3_markdup.bam:0.0019085032,(49_2_markdup.bam:0.0035855918,27_10_markdup.bam:0.0014288044):0.0009623090):0.0014596447):0.0015620752,136308_markdup.bam:0.0006546757):0.0018372242,136060_markdup.bam:0.0000239687):0.0008234955,138770_markdup.bam:0.0003005209):0.0012092158,135985_markdup.bam:0.0002862462):0.0017678515,138816_markdup.bam:0.0002348864):0.0027212862,(136023_markdup.bam:0.0002637824,Comet_markdup.bam:0.0031431910):0.0007085405):0.0072712039,((((Nelio_markdup.bam:0.0251782739,((136429_markdup.bam:0.0013646588,Eschomi_markdup.bam:0.0015415110):0.0031045113,38_1_markdup.bam:0.0038224622):0.0043465201):0.0106433828,1_2_markdup.bam:0.0015648082):0.0052315919,136013_markdup.bam:0.0046280104):0.0090242013,51_1_markdup.bam:0.0084237695):0.0041965017):0.0042527899,Galliano_markdup.bam:0.0000744968):0.0006480489,137527_markdup.bam:0.0003965108):0.0025887798,53_3_markdup.bam:0.0029733699):0.0028201817,(136127_markdup.bam:0.0031928953,135899_markdup.bam:0.0002430360):0.0004812989);
```

## Structure analysis

* Using filtered biallelic SNPS 

* Convert vcf to plink format. use newest version of plink2 and rename chromosomes scaffold_01 > 1, e.g
* rename chromosomes using new plink2 function. use rename key rename.txt and set the number of chromosomes. remove variants mapped to unplaced contigs (28, 29)

```
# get bedtools file from faidx
GENOME=tfrez.scaffolds.fa
samtools faidx $GENOME
awk 'BEGIN {OFS = "\t"} {print $1, "1", $2}' ${GENOME}.fai | sort -V | head -27 > chromosomes.bed
```
* extract SNPs on chromosomes (or large scaffolds, > 50kb) with bedtools using biallellic SNPS file

```
VCF=biallelic_SNPS.vcf
bgzip $VCF; bcftools index ${VCF}.gz
bcftools view ${VCF}.gz -R chromosomes.bed > ${VCF%.*}_chroms.vcf
```

Now format SNPS for plink2

* rename scaffolds (remove 'scaffold_' and make key with old names in first column and new names in second
```
cut -f1 ${GENOME}.fai | sort -V | head -27 > tmp
cut -f2 -d '_' tmp > tmp2
paste tmp tmp2 > rename.txt
rm tmp tmp2

```
plink2 --vcf ${VCF%.*}_chroms.vcf --make-pgen --out sorted_pgen --sort-vars --rename-chrs rename.txt  --chr-set 27
plink2 --pfile sorted_pgen --make-bed --out sorted_pgen
```

*get estimates of structure from admixture

```
for i in {2..6}; do admixture --cv sorted_pgen.bed ${i} > log${i}.out; done
awk '/CV/ {print $3,$4}' *out | cut -c 4,7-20
```

returns:

2 seems right, as it is consistent with phylogeny. One outlier possibly due to contamination based on odd result from mat gene screening suggesting likley contamination.

Subset two pops.

```
mkdir lineage1 lineage2

# subset vcf based on inclusion of samples, and filter out sites without an alternate allele (it's in the other pop)
_
bcftools view ${VCF%.*}_chroms.vcf -S ^novel_genotypes_plus.txt | vcffilter -f "AC > 0" > lineage1/${VCF%.*}_chroms_lineage1.vcf
bcftools view ${VCF%.*}_chroms.vcf -S novel_genotypes.txt | vcffilter -f "AC > 0" > lineage2/${VCF%.*}_chroms_lineage2.vcf
```






#plot output in R



# pca 

```
plink2 --vcf out.recode.vcf --make-pgen --out sorted_pgen_fileset --sort-vars --rename-chrs rename.txt  --chr-set 38 --set-missing-var-ids @:# --mind .5 --geno 0.1 --vcf-half-call missing
plink2 -pfile sorted_pgen_fileset --pca
```



## Linkage disequlibrium

First convert to format suitable for plink2

Using rename file from above, and SNPS on chromosomes 
```
plink2 \
        --vcf ${VCF%.*}_chroms.vcf \
        --make-pgen \
        --out sorted_pgen \
        --sort-vars \
        --rename-chrs rename.txt \
        --chr-set 27 \
        --set-missing-var-ids @:# \
        --mind .5 \
        --geno 0.1 \
        --vcf-half-call missing

plink2 \
        -pfile sorted_pgen \
        --ld-window 100 \
        --ld-window-kb 1000 \
        --r2-unphased \
        --ld-window-r2 0
```

Now in R, graph linkage disequlibrium from output "plink2.vcor" file

``` {r}
DIR <- "/90daydata/fdwsru_fungal/Nick/peanut_smut_popgen/08_popgen/LD"
setwd(DIR)

df <- read.table('plink2.vcor', header = FALSE)
BINSIZE=100
df$dist <- df$V5 - df$V2
df$bin <- round(df$dist/BINSIZE, 0)

library(plyr)
df2 <- ddply(df, .(bin), summarise,
      meanr2 = mean(V7))

#get plot
png(filename = "LD_all_samples.png", width = 800, height = 600, units = "px")
plot(df2$bin*BINSIZE/1000, df2$meanr2, xlab="Physical distance (kp)", ylab="R2", main="LD decay rate")
dev.off()
```
* with all samples, the graph looks noisy, with some odd patterns, while still showing decay, consistent with recombination. Noise is likely due to population structure.

![linkage drag plot for all samples](https://github.com/ngreatens/thecaphora_frezzii_popgen/blob/main/LD_all_samples(2).png)

### subset populations and reassess linkage drag

* using inital data from phylogenetic tree and population structure analysis, subset samples based on clear patterns. Also remove 'Nelio', a haploid included mistakenly and misnamed in first round and one odd sample, apparently contamination based on dome kmer inclusion of both all three alleles of the MAT genes

```
mkdir lineage1 lineage2

# subset vcf based on inclusion of samples, and filter out sites without an alternate allele (it's in the other pop)

bcftools view ${VCF%.*}_chroms.vcf -S ^novel_genotypes_plus.txt | vcffilter -f "AC > 0" > lineage1/${VCF%.*}_chroms_lineage1.vcf
bcftools view ${VCF%.*}_chroms.vcf -S novel_genotypes.txt | vcffilter -f "AC > 0" > lineage2/${VCF%.*}_chroms_lineage2.vcf
```

And rerun plink and R script as above for each lineage

![LD in Lineage 1](https://github.com/ngreatens/thecaphora_frezzii_popgen/blob/main/LD_lineage1.png)
![LD in Lineage 2](https://github.com/ngreatens/thecaphora_frezzii_popgen/blob/main/LD_lineage2.png)


Now low linkage disequilibrium with low and rapidly decreasing R2 values is readily apparent in lineage 1. In lineage 2 (5 samples), a low number of samples, shared ancestry, possibly recent founder effects, etc. affect the plot more significantly, but low and generally diminishing R2 values with distance similarly suggest recombination.

Removing alleles with low MAF will likely clear up graph for final analysis







