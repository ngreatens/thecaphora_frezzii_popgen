# filter variants by pop and quality
```
#!/bin/bash

vcf=$1

ml vcftools

# separate indels
vcftools --vcf $vcf --keep-only-indels --recode --recode-INFO-all --out output_indels-only.vcf
# separate SNPs
vcftools --vcf $vcf --remove-indels --recode --recode-INFO-all --out output_snps-only.vcf

eval "$(conda shell.bash hook)" #intialize shell for conda environments
conda activate vcflib

vcffilter -f "QUAL > 20 & QUAL / AO > 10 & SAF > 0 & SAR > 0 & RPR > 1 & RPL > 1 & AC > 0" output_snps-only.vcf.recode.vcf > SNPs_filtered.vcf

ml vcftools

vcftools \
        --vcf SNPs_filtered.vcf \
        --out merged \
        --min-alleles 2 \
        --max-alleles 2 \
        --max-missing .9 \
        --minDP 5 \
        --maf .05 \
        --recode


## SNPS per sample

Get SNPs per sample after filtering for quality

```
conda activate rtg-tools
rtg vcfstats merged.recode.vcf | awk '/Sample/ {printf "%s\t", $0; getline; print $0}'| sed 's/Sample Name: //g' | sed 's/ //g' | sed 's/SNPs://g'
```














