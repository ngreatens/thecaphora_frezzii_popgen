# Use SNPeff to get variants in coding regions, near predicted effectors, etc.


## configure config file and directories with proper data

* Make config file

snpEff.config: 
```
#Thecaphora frezzii genome, version 1.0
thecaphora.frezzii.genome : Thecaphora frezzii
```

* Make data directory. The files in data/ must have these names.
```
data/genomes:
thecaphora.frezzii.fa

data/thecaphora.frezzii:
cds.fa  genes.gtf  protein.fa 
```

* Use AGAT to covert gff3 to gtf 2.2
```
ml apptainer

apptainer exec /project/fdwsru_fungal/Nick/sifs/AGAT.sif agat_convert_sp_gff2gtf.pl -gff $gff --gtf_version 2.2 -o genes.gtf

```


## Build database

```
ml java snpeff

java -jar /software/el9/apps/snpeff/5.2f/snpEff.jar build -c snpEff.config  -gtf22 thecaphora.frezzii
```

## Use SNPeff to annotate vcf

e.g.
```
java -jar /software/el9/apps/snpeff/5.2f/snpEff.jar thecaphora.frezzii ../03_variant_calling/out.vcf > test.vcf
```

* explore annotations in html file output and filter for location of mutation: in protein coding gene, etc.



