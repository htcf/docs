# Containers (Singularity)

## Overview

Singularity containers allow you to run software in isolated, reproducible environments. Unlike Docker, Singularity is designed for HPC systems and runs without root privileges.

**When to use containers:**

- Software with complex dependencies
- Reproducing published analyses
- Using Docker images from public repositories
- Sharing exact software environments

## Running Containers

### Basic Usage

```bash
# Run a command in a container
singularity exec container.sif <command>

# Interactive shell
singularity shell container.sif

# Run the container's default command
singularity run container.sif
```

### Example: Running a Tool from Docker Hub

Many bioinformatics tools are available as Docker images. Singularity can pull and convert them:

```bash
# Pull from Docker Hub (do this once, on login node)
singularity pull docker://biocontainers/samtools:v1.9-4-deb_cv1

# Run in a job
singularity exec samtools_v1.9-4-deb_cv1.sif samtools view input.bam
```

### Job Script Example

```bash
#!/bin/bash

#SBATCH --job-name=container_job
#SBATCH --cpus-per-task=4
#SBATCH --mem=8G
#SBATCH --time=2:00:00

cd /scratch/mylab/$USER/project/

# Run analysis using container
singularity exec /ref/mylab/software/containers/tool.sif \
    my-tool --threads 4 input.txt output.txt
```

## Building Containers

Build containers in an interactive session (not on login node):

```bash
$ srun --mem=8G --cpus-per-task=4 -J interactive -p interactive --pty /bin/bash -l
```

### From Docker Hub

```bash
# Pull and convert Docker image
singularity pull docker://python:3.11-slim
```

### From Definition File

Create a definition file (`mycontainer.def`):

```
Bootstrap: docker
From: ubuntu:22.04

%post
    apt-get update && apt-get install -y \
        python3 \
        python3-pip
    pip3 install numpy scipy

%runscript
    python3 "$@"
```

Build the container:

```bash
singularity build mycontainer.sif mycontainer.def
```

!!! note "Build Location"
    Build containers in scratch, then move to `/ref/<lab>/software/containers/` for storage.

## Binding Directories

By default, Singularity binds your home directory and current working directory. To access other paths:

```bash
# Bind additional directories
singularity exec --bind /lts/mylab:/data container.sif command

# Multiple binds
singularity exec --bind /lts/mylab:/input,/scratch/mylab/$USER:/output container.sif command
```

## GPU Containers

For GPU-enabled containers:

```bash
singularity exec --nv container.sif python train.py
```

The `--nv` flag provides GPU access inside the container.

### Example: PyTorch Container

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --mem=16G
#SBATCH --time=4:00:00

singularity exec --nv /ref/mylab/software/containers/pytorch.sif \
    python train_model.py
```

## Container Storage

Store built containers in reference storage:

```
/ref/<lab>/software/containers/
├── samtools_1.9.sif
├── star_2.7.sif
└── pytorch_2.0.sif
```

## Common Registries

| Registry | Pull Command |
|----------|--------------|
| Docker Hub | `singularity pull docker://image:tag` |
| BioContainers | `singularity pull docker://biocontainers/tool:version` |
| NVIDIA NGC | `singularity pull docker://nvcr.io/nvidia/pytorch:tag` |
| Quay.io | `singularity pull docker://quay.io/org/image:tag` |

## Best Practices

1. **Pull once, run many** - Pull containers to `/ref` once, use in all jobs
2. **Version your containers** - Include version tags in filenames
3. **Document your containers** - Record how containers were built
4. **Use --bind for data** - Don't copy data into containers

## Troubleshooting

**Container not found:**
```bash
# Check path
ls -la /ref/mylab/software/containers/
```

**Permission denied inside container:**
```bash
# Bind a writable directory
singularity exec --bind /scratch/mylab/$USER:/tmp container.sif command
```

**GPU not available:**
```bash
# Ensure you requested GPU and used --nv flag
#SBATCH --gpus=1
singularity exec --nv container.sif nvidia-smi
```

## Resources

- [Singularity Documentation](https://docs.sylabs.io/guides/latest/user-guide/)
- [BioContainers Registry](https://biocontainers.pro/)
- [NVIDIA NGC Catalog](https://catalog.ngc.nvidia.com/)
- [Docker Hub](https://hub.docker.com/)
