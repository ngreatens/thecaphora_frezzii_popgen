## preprocessing data

* fastp on everything

```
#!/bin/bash



forward_reads=$1
reverse_reads=$2
id=$3

eval "$(conda shell.bash hook)" #intialize shell for conda environments
conda activate /project/fdwsru_fungal/Nick/conda/envs/fastp
#fastp 0.23.4

fastp \
        --in1 $forward_reads \
        --in2 $reverse_reads \
        --out1 ${id}_fastp_1.fastq \
        --out2 ${id}_fastp_2.fastq \
        --html $id.html \
        --json $id.json
conda deactivate
```

* also concat fastp cleaned reads for mat screens
