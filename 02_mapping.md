#!/bin/bash

* index genome tfrez.scaffolds.fa

```
#!/bin/bash

ml bwa_mem2/2.2.1_7aa5ff6

bwa-mem2 index $1
```

* map all reads to genome and mark dups
```
#!/bin/bash

ml bwa_mem2/2.2.1_7aa5ff6

bwa-mem2 index $1

(base) [nicholas.greatens@ceres24-compute-11 02_mapping]$ cat ../scripts/bwa_mem2_map.sh
#!/bin/bash

index=$1
fwd_reads=$2
rev_reads=$3
out_prefix=$4

ml bwa_mem2
ml samtools

bwa-mem2 mem -t 16 $index $fwd_reads $rev_reads |
samtools sort -@8 -n - |
samtools fixmate -@8  -m - ${out_prefix}_namesorted.bam
samtools sort -@8  ${out_prefix}_namesorted.bam |
samtools markdup -@8 - ${out_prefix}_markdup.bam
samtools index ${out_prefix}_markdup.bam

rm ${out_prefix}_namesorted.bam
```
* and add RGS

```
#!/bin/bash

module load \
        java/17 \
        picard/3.0.0


input=$1
id=$2
output=${1%.*}_new.bam

#library
RGLB=${id}_1

#platform
RGPL=ILLUMINA

#sample_name
RGSM=$input

#ID
RGID=${id}

#platform unit e.g. run barcode
RGPU=${RGID}_001

java -Xmx16G -jar /software/el9/apps/picard/3.0.0/picard.jar \
\
AddOrReplaceReadGroups \
        --INPUT $input\
        --OUTPUT $output \
        --RGLB $RGLB \
        --RGPL $RGPL \
        --RGSM $RGSM \
        --RGID $RGID \
        --RGPU $RGPU


ml samtools

samtools index ${1%.*}_new.bam
```

* get coverage of everything and get average mapping

```
mosdepth -n -x 
```

