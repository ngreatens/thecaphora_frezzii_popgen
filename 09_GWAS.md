# GWAS with occurence on resistance vc susceptible phenotypes



## Gemma

subset vcf to exclude samples without phenotyping data (occurrence on susceptible peanut).
* exclude comet and poale


```
plink2 --vcf phenotyped.vcf --make-pgen         --out sorted_pgen         --sort-vars         --rename-chrs rename.txt         --chr-set 27         --set-missing-var-ids @:#
plink2 --pfile sorted_pgen --make-bed --out for_gemma
```


Change sixth column of .fam file to be all '1' to indicate a phenotype is available for the samples
```
awk '{print $1, $2, $3, $4, $5, 1}' for_gemma.fam > tmp; mv tmp for_gemma.fam
```
Make phenotype file with data from spreadsheet in file phenotypes.txt

```
cut -f2 for_gemma.fam | sed 's/_markdup.bam//g' > samples
while read a; do grep ^${a} phenotype.txt; done < samples | cut -f2 > phenotypes_for_gemma.txt
```

Get matrix to account for population structure
```
gemma -bfile for_gemma -gk 1 -o tmp
```

Run gemma with phenotypes 
```
gemma -bfile for_gemma -p phenotypes_for_gemma.txt -lm 4 -o infection -r2 1 -hwe 0 -maf 0.05 -k output/tmp.cXX.txt
```

Now in R, plot output as Manhattan plot

## Mahattan plot
```{r}

DIR <- "/90daydata/fdwsru_fungal/Nick/peanut_smut_popgen/08_popgen/GWAS_resistance/output"
setwd(DIR) 

library("qqman")

res <- read.table("infection.assoc.txt", header = TRUE)
manhattan(x = res, chr = "chr", bp = "ps", p = "p_lrt", snp = "rs", col = c("blue4", "orange3"), logp = TRUE)


png(filename = "Manhattan plot.png", width = 400, height = 400, units = "px")
manhattan(x = res, chr = "chr", bp = "ps", p = "p_lrt", snp = "rs", col = c("blue4", "orange3"), logp = TRUE)
dev.off()
```

![Manhattan plot for all samples](https://github.com/ngreatens/thecaphora_frezzii_popgen/blob/main/Manhattan%20plot.png)

Ideally, a hit is above P ≤ 5 x 10⁻⁸, so there are no solid hits here. Some small towers on scaffolds 5 and 24, with the best hit on 16. All are in non-coding regions, possibly hitting on regulatory elements, etc. 


Downstream of the SNP is a gene rich region, while upstream is mostly TEs
* The hit on 2 is in a CACTA TIR in a gene rich region. No obvious candidates

* The his on 5 is  in a gene rich space, but at first glance, but nothing stands out at a first glance.

* Hit on 10 in a gene poor region full of TEs, but 100kb upstream of a couple of predicted effectors

* Hi on 12 is in a repeat rich, gene poor region near the end of a chromosome. Nearby genes are mostly annotated within repeats, possibly erroneous annotation

* The hit on 16 is about 2 kb upstream of ACQY0O_003241 (ENOG503Q535), a "GTPase-activator protein for Rho-like GTPases" per EGGNOG, likley regulating GYP8 downstream. This gene has been demonstrated to be necessary for pathogenicity in other systems(https://www.sciencedirect.com/science/article/pii/S2095311923000758). Downstream of the SNP is a gene rich region, while upstream is mostly TEs

* The hit on 24 is in a repeat rich, gene sparse region that is possibly not well assembled 





