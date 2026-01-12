# GPU Computing on HTCF

## Overview

The HTCF provides GPU resources for computationally intensive tasks that benefit from GPU acceleration, including deep learning, machine learning, molecular dynamics simulations, and other parallel computing workloads.

### Available GPU Resources

The HTCF currently has the following GPUs available:

- **6x NVIDIA A100 80GB** - Latest generation datacenter GPUs with 80GB HBM2e memory
- **2x NVIDIA V100 32GB** - Previous generation datacenter GPUs with 32GB HBM2 memory

### GPU Specifications

| Model | Quantity | Memory | CUDA Cores | Tensor Cores | Common Use Cases |
|-------|----------|--------|------------|--------------|------------------|
| A100 80GB | 6 | 80 GB HBM2e | 6912 | 432 (Gen 3) | Large deep learning models, multi-GPU training |
| V100 32GB | 2 | 32 GB HBM2 | 5120 | 640 (Gen 1) | General GPU computing, smaller models |

## When to Use GPUs

### Ideal GPU Workloads

GPUs excel at:

- **Deep learning training** - Neural network training with frameworks like PyTorch, TensorFlow, JAX
- **Machine learning inference** - Batch prediction and model serving
- **Molecular dynamics** - GROMACS, LAMMPS, AMBER with GPU acceleration
- **Image/video processing** - Batch processing with GPU libraries
- **Scientific simulations** - CUDA-accelerated physics, chemistry, and engineering codes
- **Bioinformatics** - GPU-accelerated alignment, assembly, and variant calling

### When NOT to Use GPUs

Don't use GPUs for:

- Tasks that aren't GPU-accelerated (use CPU batch jobs instead)
- Simple scripts or data processing
- Software that doesn't support GPUs
- Interactive sessions for editing/testing (unless testing GPU code)

!!! tip "Check if Software Supports GPUs"
    Not all software can use GPUs. Check the software documentation for GPU support before requesting GPU resources.

## GPU Partition and Policies

### GPU Partition

All GPU jobs must use the `gpu` partition:

```bash
#SBATCH -p gpu
```

### Fair Use

GPUs are a shared, limited resource. Please:

- Use GPUs only when the code can benefit from them
- Request only the number of GPUs you can effectively use
- Test code on CPU or with small datasets before running large GPU jobs
- Monitor GPU utilization and adjust if GPUs are underutilized

## Requesting GPUs

### Basic GPU Request

In the batch script, add these SBATCH directives:

```bash
#!/bin/bash

#SBATCH -p gpu          # Use GPU partition
#SBATCH --gpus=1        # Request 1 GPU

# The GPU code here
```

Submit with:

```bash
$ sbatch my_gpu_job.sh
```

### Requesting Multiple GPUs

For multi-GPU jobs:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=2        # Request 2 GPUs
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G

# Multi-GPU code here
```

!!! warning "Multi-GPU Programming"
    Simply requesting multiple GPUs doesn't automatically parallelize your code. Your software must support multi-GPU execution (e.g., PyTorch DistributedDataParallel, TensorFlow MirroredStrategy).

### Requesting Specific GPU Types

To request a specific GPU model:

```bash
# Request A100 GPU
#SBATCH --constraint=A100

# Request V100 GPU
#SBATCH --constraint=V100
```

If you don't specify a constraint, Slurm will assign any available GPU.

### Complete GPU Job Example

```bash
#!/bin/bash

#SBATCH --job-name=gpu_test
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=4:00:00
#SBATCH --output=gpu_job_%j.out

# Load CUDA
eval $(spack load --sh cuda@12.0)

# Verify GPU is available
nvidia-smi

# Run the GPU program
python train_model.py
```

## Verifying GPU Access

### Check GPU Allocation

After the job starts, verify the GPU is accessible within the job:

```bash
$ srun --ntasks-per-node=1 --jobid=<job-id> nvidia-smi
```

Example output:

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.60.13    Driver Version: 525.60.13    CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA A100 80G...  Off  | 00000000:17:00.0 Off |                    0 |
| N/A   39C    P0    62W / 300W |      0MiB / 81920MiB |     23%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0                12043      C   guppy_basecaller                 4753MiB |
+-----------------------------------------------------------------------------+
```

Key information:
- **GPU Name**: Which GPU model was assigned
- **Memory-Usage**: Current GPU memory usage out of total
- **GPU-Util**: Current GPU utilization percentage
- **Processes**: The process should appear here when using the GPU

### Within Job Scripts

Add GPU verification to the job script:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Verify GPU availability
echo "Checking GPU availability..."
nvidia-smi

# Check CUDA version
nvcc --version || echo "nvcc not found - load CUDA module"

# The GPU code
python my_gpu_script.py
```

## GPU Resource Combinations

### Typical Resource Requests

Choose resources based on the workload:

**Single GPU, Light Compute:**
```bash
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
```

**Single GPU, Heavy Compute:**
```bash
#SBATCH --gpus=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
```

**Multi-GPU Training:**
```bash
#SBATCH --gpus=4
#SBATCH --cpus-per-task=16
#SBATCH --mem=128G
```

**Large Model (A100 Required):**
```bash
#SBATCH --gpus=1
#SBATCH --constraint=A100
#SBATCH --mem=128G
```

!!! tip "CPU and Memory Balance"
    - Request 4-8 CPUs per GPU for data preprocessing
    - Request sufficient memory for your dataset (typically 2-4x the data size)
    - For A100 80GB: consider requesting more memory if loading large datasets

## Interactive GPU Sessions

For testing and development:

```bash
$ srun -p gpu --gpus=1 --mem=16G -J interactive --pty /bin/bash -l
```

Once on the GPU node:

```bash
$ nvidia-smi
$ eval $(spack load --sh cuda py-pytorch)
$ python
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.cuda.get_device_name(0)
'NVIDIA A100 80GB PCIe'
```

!!! reminder "Interactive GPU Policy"
    Interactive GPU sessions should be used for development and testing only. Production runs should use batch jobs.

## Common GPU Frameworks

### PyTorch

```bash
# In job script
eval $(spack load --sh cuda py-pytorch)

# Verify GPU
python -c "import torch; print(torch.cuda.is_available())"
```

### TensorFlow

```bash
# In job script
eval $(spack load --sh cuda py-tensorflow)

# Verify GPU
python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```

### JAX

```bash
# In job script
eval $(spack load --sh cuda py-jax)

# Verify GPU
python -c "import jax; print(jax.devices())"
```

See [CUDA Setup](cuda-setup.md) for detailed environment configuration.

## Quick Start Checklist

Before submitting your first GPU job:

- [ ] Verify your software supports GPUs
- [ ] Test on a small dataset first
- [ ] Use `-p gpu` and `--gpus=N` in your script
- [ ] Load CUDA module if needed
- [ ] Include `nvidia-smi` to verify GPU access
- [ ] Monitor GPU utilization (see [Monitoring](monitoring.md))
- [ ] Request appropriate time limit if needed

## Next Steps

- [CUDA Setup](cuda-setup.md) - Configure CUDA environment and GPU frameworks
- [GPU Examples](examples.md) - Simple examples to get started
- [GPU Monitoring](monitoring.md) - Monitor and debug GPU usage
- [First GPU Job Tutorial](../tutorials/first-gpu-job.md) - Step-by-step first GPU job
- [Machine Learning Workflows](../workflows/machine-learning.md) - End-to-end ML examples

## Getting Help with GPUs

--8<-- "includes/getting-help.md"
