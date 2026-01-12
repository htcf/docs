# Workflow Managers

## Overview

Workflow managers help automate multi-step analyses with dependency tracking, parallelization, and reproducibility. They handle job submission and manage complex pipelines.

## Snakemake

[Snakemake](https://snakemake.readthedocs.io/) is a Python-based workflow manager popular in bioinformatics.

### Installation

```bash
# Install via Spack
spack install py-snakemake

# Or via pip in a virtual environment
python -m venv ~/snakemake-env
source ~/snakemake-env/bin/activate
pip install snakemake
```

### Basic Usage with HTCF

Create a Snakemake profile for HTCF (`~/.config/snakemake/htcf/config.yaml`):

```yaml
executor: slurm
default-resources:
  mem_mb: 4000
  runtime: 60
  cpus_per_task: 1
  partition: general
```

Run your workflow:

```bash
# From login node
snakemake --profile htcf --jobs 100
```

### Resources

- [Snakemake Documentation](https://snakemake.readthedocs.io/)
- [Snakemake Tutorial](https://snakemake.readthedocs.io/en/stable/tutorial/tutorial.html)
- [Snakemake SLURM Executor](https://snakemake.github.io/snakemake-plugin-catalog/plugins/executor/slurm.html)

## Nextflow

[Nextflow](https://www.nextflow.io/) is a workflow manager using a domain-specific language, popular for genomics pipelines.

### Installation

```bash
# Install Nextflow
curl -s https://get.nextflow.io | bash
mv nextflow ~/bin/
```

### Basic Usage with HTCF

Create a Nextflow config (`nextflow.config`):

```groovy
process {
    executor = 'slurm'
    queue = 'general'
    memory = '4 GB'
    time = '1h'
    cpus = 1
}
```

Run your workflow:

```bash
nextflow run my_pipeline.nf
```

### nf-core Pipelines

[nf-core](https://nf-co.re/) provides production-ready Nextflow pipelines:

```bash
# Run an nf-core pipeline
nextflow run nf-core/rnaseq -profile singularity --input samplesheet.csv
```

### Resources

- [Nextflow Documentation](https://www.nextflow.io/docs/latest/)
- [Nextflow Training](https://training.nextflow.io/)
- [nf-core Pipelines](https://nf-co.re/pipelines)

## Comparison

| Feature | Snakemake | Nextflow |
|---------|-----------|----------|
| Language | Python | Groovy DSL |
| Learning curve | Lower (if you know Python) | Moderate |
| Pre-built pipelines | Snakemake Workflows | nf-core |
| Container support | Good | Excellent |
| SLURM integration | Via plugin | Built-in |

## General Tips

1. **Run from login node** - Workflow managers submit jobs, so run them from login node
2. **Use containers** - Combine with Singularity for reproducibility
3. **Start small** - Test with a few samples before running full datasets
4. **Check logs** - Workflow managers create detailed logs for debugging
5. **Resume on failure** - Both support resuming failed runs
