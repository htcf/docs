# Tutorial: Your First GPU Job

## Learning Objectives

By the end of this tutorial, you will:

- Request a GPU in a batch job
- Verify GPU access with nvidia-smi
- Run a simple GPU program
- Monitor GPU utilization

**Time Required:** 20 minutes
**Prerequisites:** Complete [Your First Batch Job](first-job.md) tutorial
**Level:** Beginner

## Step 1: Set Up Your Working Directory

```bash
$ cd /scratch/mylab/$USER
$ mkdir -p first-gpu-tutorial
$ cd first-gpu-tutorial
```

## Step 2: Create a Simple GPU Test Script

Create `check_gpu.py`:

```bash
$ nano check_gpu.py
```

Add this content:

```python
#!/usr/bin/env python3
import torch

print("=" * 50)
print("GPU Availability Check")
print("=" * 50)

cuda_available = torch.cuda.is_available()
print(f"CUDA available: {cuda_available}")

if cuda_available:
    gpu_count = torch.cuda.device_count()
    print(f"Number of GPUs: {gpu_count}")

    for i in range(gpu_count):
        print(f"\nGPU {i}:")
        print(f"  Name: {torch.cuda.get_device_name(i)}")
        props = torch.cuda.get_device_properties(i)
        print(f"  Memory: {props.total_memory / 1e9:.2f} GB")

    # Test tensor creation on GPU
    print("\nCreating test tensor on GPU...")
    x = torch.randn(1000, 1000, device='cuda')
    print(f"✓ Success! Tensor created on {x.device}")
else:
    print("ERROR: No GPU found!")
```

## Step 3: Create GPU Job Script

Create `run_gpu_test.sh`:

```bash
$ nano run_gpu_test.sh
```

Add this content:

```bash
#!/bin/bash

#SBATCH --job-name=gpu_test
#SBATCH -p gpu              # GPU partition
#SBATCH --gpus=1            # Request 1 GPU
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --time=10:00        # 10 minutes
#SBATCH --output=gpu_test_%j.out

echo "Job started at: $(date)"
echo "Job ID: $SLURM_JOB_ID"
echo "Running on: $(hostname)"
echo ""

# Load CUDA and PyTorch
echo "Loading CUDA and PyTorch..."
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)
echo ""

# Check GPU with nvidia-smi
echo "GPU Information (nvidia-smi):"
nvidia-smi
echo ""

# Run Python GPU test
echo "Running Python GPU test..."
python check_gpu.py
echo ""

echo "Job completed at: $(date)"
```

## Step 4: Submit the GPU Job

```bash
$ sbatch run_gpu_test.sh
Submitted batch job 123457
```

## Step 5: Monitor the Job

```bash
$ squeue -u $USER
```

GPU jobs may take a bit longer to start since GPU resources are limited.

## Step 6: Check Results

Once the job completes:

```bash
$ cat gpu_test_123457.out
```

Expected output includes:

```
Job started at: Tue Jan  7 14:30:00 CST 2026
Job ID: 123457
Running on: gpu01

Loading CUDA and PyTorch...

GPU Information (nvidia-smi):
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.60.13    Driver Version: 525.60.13    CUDA Version: 12.0   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
|   0  NVIDIA A100 80G...  Off  | 00000000:17:00.0 Off |                    0 |
+-----------------------------------------------------------------------------+

Running Python GPU test...
==================================================
GPU Availability Check
==================================================
CUDA available: True
Number of GPUs: 1

GPU 0:
  Name: NVIDIA A100-PCIE-80GB
  Memory: 80.00 GB

Creating test tensor on GPU...
✓ Success! Tensor created on cuda:0

Job completed at: Tue Jan  7 14:30:15 CST 2026
```

## Step 7: Analyze GPU Job Statistics

```bash
$ sacct -j 123457
```

## Congratulations!

You've successfully:
- ✓ Requested a GPU with `--gpus=1`
- ✓ Loaded CUDA and GPU-enabled software
- ✓ Verified GPU access with nvidia-smi
- ✓ Ran a GPU program with PyTorch

## Common Issues and Solutions

### GPU Not Detected

**Problem:** `CUDA available: False`

**Solutions:**
1. Check you requested GPU: `#SBATCH --gpus=1`
2. Verify partition: `#SBATCH -p gpu`
3. Load CUDA module: `eval $(spack load --sh cuda)`

### Job Stays Pending

GPU resources are limited. Check pending reason:

```bash
$ squeue -u $USER -o "%.18i %.9P %.8j %.8u %.2t %.10M %.6D %.20R"
```

If REASON is "Resources", wait for GPUs to become available.

### nvidia-smi Not Found

Make sure you're running on a GPU node and CUDA is loaded:

```bash
eval $(spack load --sh cuda)
nvidia-smi
```

## Next Steps

### Try a Real GPU Computation

Modify `check_gpu.py` to do actual GPU computation:

```python
import torch
import time

# Matrix multiplication on GPU
size = 5000
x = torch.randn(size, size, device='cuda')
y = torch.randn(size, size, device='cuda')

start = time.time()
z = torch.mm(x, y)
torch.cuda.synchronize()
gpu_time = time.time() - start

print(f"GPU matrix multiplication ({size}x{size}): {gpu_time:.4f} seconds")
```

### Learn More

- [GPU Computing Overview](../gpu/index.md) - Complete GPU documentation
- [CUDA Setup](../gpu/cuda-setup.md) - Configure your environment
- [GPU Examples](../gpu/examples.md) - More GPU code examples
- [GPU Monitoring](../gpu/monitoring.md) - Monitor GPU usage
- [Machine Learning Workflows](../workflows/machine-learning.md) - Train models on GPUs

--8<-- "includes/getting-help.md"
