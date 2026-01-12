# Bioinformatics Pipelines

## Overview

Optimized workflows for common bioinformatics tasks on HTCF.

**Official Documentation:**

- [STAR Manual](https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf) - RNA-seq aligner
- [FastQC Documentation](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) - Quality control
- [samtools Documentation](http://www.htslib.org/doc/) - SAM/BAM manipulation
- [GATK Best Practices](https://gatk.broadinstitute.org/hc/en-us/sections/360007226651-Best-Practices-Workflows) - Variant calling
- [DESeq2 Vignette](https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html) - Differential expression
- [Bioconductor Workflows](https://www.bioconductor.org/packages/release/BiocViews.html#___Workflow) - Complete analysis pipelines

## RNA-seq Pipeline

### Quality Control
```bash
#!/bin/bash

#SBATCH --job-name=fastqc
#SBATCH --array=1-20%5       # 20 samples, 5 concurrent
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G
#SBATCH --time=1:00:00

eval $(spack load --sh fastqc)

SAMPLE=$(sed -n ${SLURM_ARRAY_TASK_ID}p samples.txt)

fastqc --threads 4 \
       --outdir qc_results/ \
       /lts/lab/raw_data/${SAMPLE}.fastq.gz
```

### Alignment
```bash
#!/bin/bash

#SBATCH --job-name=alignment
#SBATCH --array=1-20%5
#SBATCH --cpus-per-task=16
#SBATCH --mem=64G
#SBATCH --time=4:00:00

eval $(spack load --sh star)

SAMPLE=$(sed -n ${SLURM_ARRAY_TASK_ID}p samples.txt)

# Set up scratch directory
WORK_DIR=/scratch/mylab/$USER/align_${SLURM_JOB_ID}_${SLURM_ARRAY_TASK_ID}
mkdir -p $WORK_DIR
cd $WORK_DIR

# Align (read directly from LTS)
STAR --runThreadN 16 \
     --genomeDir /ref/genome/star_index \
     --readFilesIn /lts/lab/raw_data/${SAMPLE}.fastq.gz \
     --readFilesCommand zcat \
     --outFileNamePrefix ${SAMPLE}_

echo "Results in $WORK_DIR"
```

!!! note "Copy Results After Array Job Completes"
    After all array tasks finish, copy results from login node:
    ```bash
    rsync -av /scratch/mylab/$USER/align_*/*.bam /lts/lab/aligned/
    rm -rf /scratch/mylab/$USER/align_*
    ```

### Count Features
```bash
#!/bin/bash

#SBATCH --job-name=featurecounts
#SBATCH --array=1-20
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=1:00:00

eval $(spack load --sh subread)

SAMPLE=$(sed -n ${SLURM_ARRAY_TASK_ID}p samples.txt)

featureCounts -T 4 \
              -a /ref/genome/genes.gtf \
              -o ${SAMPLE}_counts.txt \
              /lts/lab/aligned/${SAMPLE}_Aligned.out.bam
```

## Variant Calling Pipeline

### Preprocessing
```bash
#!/bin/bash

#SBATCH --cpus-per-task=8
#SBATCH --mem=32G

eval $(spack load --sh bwa samtools)

# Align
bwa mem -t 8 ref.fa sample.fastq.gz | \
    samtools sort -@ 8 -o sample.bam

# Mark duplicates
samtools markdup -@ 8 sample.bam sample.dedup.bam
```

### Variant Calling
```bash
#!/bin/bash

#SBATCH --array=1-22        # Per chromosome
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G

eval $(spack load --sh bcftools)

CHR=$SLURM_ARRAY_TASK_ID

bcftools mpileup -Ou -f ref.fa \
                 --regions chr${CHR} \
                 sample.dedup.bam | \
    bcftools call -mv -Oz \
                  -o sample_chr${CHR}.vcf.gz
```

## Assembly Pipeline

```bash
#!/bin/bash

#SBATCH --cpus-per-task=32
#SBATCH --mem=250G
#SBATCH --time=24:00:00

eval $(spack load --sh spades)

# Set up scratch directory
WORK_DIR=/scratch/mylab/$USER/assembly_${SLURM_JOB_ID}
mkdir -p $WORK_DIR
cd $WORK_DIR

# Assemble (read directly from LTS)
spades.py --threads 32 \
          --memory 240 \
          --pe1-1 /lts/lab/reads/R1.fastq.gz \
          --pe1-2 /lts/lab/reads/R2.fastq.gz \
          -o assembly_output

echo "Results in $WORK_DIR/assembly_output"
```

After job completes, copy from login node:
```bash
rsync -av /scratch/mylab/$USER/assembly_*/assembly_output/ /lts/lab/assembly_results/
```

## Best Practices

### Use Scratch for I/O
- Jobs can read directly from LTS (read-only)
- All job output should write to scratch
- Copy results to LTS from login node after job completes
- Clean up scratch when done

### Array Jobs for Samples
Process multiple samples in parallel:
```bash
#SBATCH --array=1-100%20    # 100 samples, 20 concurrent
```

### Resource Estimation
Test with one sample first:
```bash
$ sacct -j <JOBID>
```

Then scale appropriately for all samples.

### Reference Data
Keep reference genomes on fast storage or use local node storage when available.

--8<-- "includes/getting-help.md"
