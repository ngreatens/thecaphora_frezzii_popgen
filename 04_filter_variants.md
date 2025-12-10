# filter variants by pop and quality

* first separate samples by populations identified by mating type analysis and Rachel's variant calling and preliminary popgen analysis

filter following samples with file novel_genotype.txt: 

```
136101_markdup.bam
136133_markdup.bam
23_1_markdup.bam
29_2_markdup.bam
32_2_markdup.bam
```

```
ml bcftools

bcftools view -S ^novel_genotype.txt out.vcf > main_lineage.vcf
bcftools view -S novel_genotype.txt out.vcf > novel_lineage.vcf
```

* filter for quality

```
conda activate vcflib

vcffilter -f "QUAL > 20 " main_lineage.vcf > main_lineage_qual20.vcf
vcffilter -f "QUAL > 20 " novel_lineage.vcf > novel_lineage_qual20.vcf
```

* filter for MAF > 05 to remove rare variants and variants present in other pops. and separate SNPs

```
vcftools --vcf main_lineage_qual20.vcf --maf .05 --recode --recode-INFO-all --out main_lineage_qual20
vcftools --vcf main_lineage_qual20.recode.vcf --recode --recode-INFO-all --out main_lineage_qual20_SNPs  --remove-indels
vcftools --vcf novel_lineage_qual20.vcf --maf .05 --recode --recode-INFO-all --out novel_lineage_qual20
vcftools --vcf novel_lineage_qual20.recode.vcf --recode --recode-INFO-all  --out novel_lineage_qual20_SNPs  --remove-indels
```

* intersect to find number of SNPS shared between populations. approx 8500

```
bedtools intersect -a main_lineage_qual20_SNPs.recode.vcf -b novel_lineage_qual20_SNPs.recode.vcf | wc -l 

```

* isec and merge to get merged dataset of SNPS.
```
bgzip main_lineage_qual20_SNPs.recode.vcf; bcftools index main_lineage_qual20_SNPs.recode.vcf.gz
bgzip novel_lineage_qual20_SNPs.recode.vcf; bcftools index novel_lineage_qual20_SNPs.recode.vcf.gz
bcftools isec -p 2pop main_lineage_qual20_SNPs.recode.vcf.gz novel_lineage_qual20_SNPs.recode.vcf.gz
cd 2pop/
bcftools merge 0002.vcf 0003.vcf > merged.vcf.gz
bcftools view merged.vcf.gz > 2pops_SNPS_shared.vcf
```




## SNPS per sample

Get SNPs per sample after filtering for quality

```
conda activate rtg-tools
rtg vcfstats $FILE | awk '/Sample/ {printf "%s\t", $0; getline; print $0}'| sed 's/Sample Name: //g' | sed 's/ //g' | sed 's/SNPs://g'
```

## Also get biallelic SNPS for whole dataset

* also get rid of "Nelio" sample, actually arias et al. data

```
vcftools --vcf out.vcf --remove-indels --recode --recode-INFO-all --out all_SNPs
vcftools \
        --vcf all_SNPs.recode.vcf \
        --min-alleles 2 \
        --max-alleles 2 \
        --max-missing .9 \
        --maf .05 \
        --recode \
        --recode-INFO-all \
        --stdout > tmp.vcf 

bcftools view -s ^Nelio_markdup.bam tmp.vcf > all_biallelic_SNPs.vcf

```











