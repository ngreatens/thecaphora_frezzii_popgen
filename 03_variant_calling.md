# map all reads

* using duplicate marked and read group added bam files, use freebayes to call variants in parallel

```
#!/bin/bash


ref=$1
bamlist=$2
out=$3


module load freebayes


freebayes-parallel \
        <(fasta_generate_regions.py ${ref}.fai 100000) 144 \
        --fasta-reference ${ref} \
        --bam-list $bamlist > $out
```
