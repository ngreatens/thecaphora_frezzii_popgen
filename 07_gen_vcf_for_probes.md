```
# using filtered dataset "SNPs_maf05_filtered.vcf"

conda activate vcflib
ml bcftools bedtools

# also exclude haploid "Nelio", mistakenly named arias data, a haploid
bcftools view -S ^novel_genotypes_plus.txt SNPs_maf05_filtered.vcf > tmp.vcf
vcffilter -f "AC > 0" tmp.vcf | bgzip > main_lineage_SNPs_filtered_maf05.vcf.gz
bcftools index main_lineage_SNPs_filtered_maf05.vcf.gz

# only novel genotypes
bcftools view -S novel_genotypes.txt SNPs_maf05_filtered.vcf > tmp.vcf
vcffilter -f "AC > 0" tmp.vcf | bgzip > novel_lineage_SNPs_filtered_maf05.vcf.gz
bcftools index novel_lineage_SNPs_filtered_maf05.vcf.gz


# intersect two files

bcftools isec -p shared main_lineage_SNPs_filtered_maf05.vcf.gz novel_lineage_SNPs_filtered_maf05.vcf.gz
for file in shared/*vcf; do bgzip $file; bcftools index ${file}.gz; done
bcftools merge shared/0002.vcf.gz shared/0003.vcf.gz | bgzip > shared.vcf.gz; bcftools index shared.vcf.gz


# get main lineage SP SNPs

# first prepare bed file from secreted genes list

while read a; do grep ${a} tfrez.gff3 | grep gene; done < SPS_list.txt | cut -f1,4-5 > genes.bed

# get main lineage SP SNPs
# edit AD field due to vcf preprocessing issue to make it compatible with bcftools merge
bcftools view novel_lineage_SNPs_filtered_maf05.vcf.gz |  grep '#'  > header.txt
bedtools intersect -a novel_lineage_SNPs_filtered_maf05.vcf.gz -b genes.bed > tmp.vcf
cat header.txt tmp.vcf | sed 's/##FORMAT=<ID=AD,Number=R/##FORMAT=<ID=AD,Number=./1' | bgzip > novel_effectors.vcf.gz; bcftools index novel_effectors.vcf.gz


# get novel SP SNPS
# edit AD field due to vcf preprocessing issue to make it compatible with bcftools merge
bcftools view main_lineage_SNPs_filtered_maf05.vcf.gz |  grep '#'  > header.txt
bedtools intersect -a main_lineage_SNPs_filtered_maf05.vcf.gz -b genes.bed > tmp.vcf
cat header.txt tmp.vcf | sed 's/##FORMAT=<ID=AD,Number=R/##FORMAT=<ID=AD,Number=./1' | bgzip > main_effectors.vcf.gz; bcftools index main_effectors.vcf.gz

# merge sets
bcftools merge main_effectors.vcf.gz novel_effectors.vcf.gz | bgzip > merged_effectors.vcf.gz; bcftools index merged_effectors.vcf.gz

# concat two files
bcftools concat -a -D merged_effectors.vcf.gz shared.vcf.gz> final_set.vcf
```
