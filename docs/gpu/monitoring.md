# GPU Monitoring and Debugging

## Overview

Monitoring GPU usage is crucial for understanding if your code is effectively using GPU resources, identifying bottlenecks, and troubleshooting performance issues. This guide covers tools and techniques for monitoring GPUs on the HTCF.

## nvidia-smi - NVIDIA System Management Interface

### Basic Usage

The primary tool for monitoring NVIDIA GPUs is `nvidia-smi`:

```bash
$ nvidia-smi
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
| N/A   45C    P0    120W / 300W |  12543MiB / 81920MiB |     87%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A     12043      C   python                           12531MiB|
+-----------------------------------------------------------------------------+
```

### Understanding nvidia-smi Output

**Header Section:**
- **Driver Version**: NVIDIA driver version
- **CUDA Version**: Maximum CUDA version supported

**GPU Section:**
- **GPU**: GPU index (0, 1, 2, etc.)
- **Name**: GPU model
- **Temp**: Temperature in Celsius
- **Perf**: Performance state (P0 = maximum, P12 = minimum)
- **Pwr:Usage/Cap**: Current power usage / maximum power
- **Memory-Usage**: Used memory / total memory
- **GPU-Util**: GPU utilization percentage (0-100%)

**Processes Section:**
- **PID**: Process ID
- **Type**: C (Compute) or G (Graphics)
- **Process name**: Name of the process using GPU
- **GPU Memory Usage**: Memory used by this process

### Monitoring Your Running Job

To check GPU usage for your running job:

```bash
# Get your job ID
$ squeue -u $USER

# Run nvidia-smi on your job's node
$ srun --ntasks-per-node=1 --jobid=<JOBID> nvidia-smi
```

### Continuous Monitoring

Watch GPU status updating every second:

```bash
$ watch -n 1 nvidia-smi
```

Exit with `Ctrl+C`.

### Query Specific Information

Get specific GPU metrics:

```bash
# GPU utilization only
$ nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader

# Memory usage
$ nvidia-smi --query-gpu=memory.used,memory.total --format=csv

# Temperature
$ nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader

# Power usage
$ nvidia-smi --query-gpu=power.draw --format=csv,noheader
```

### All Available Queries

See all available query options:

```bash
$ nvidia-smi --help-query-gpu
```

## Advanced nvidia-smi Options

### Custom Output Format

Create custom output with specific fields:

```bash
$ nvidia-smi --query-gpu=index,name,temperature.gpu,utilization.gpu,utilization.memory,memory.used,memory.total --format=csv
```

### Process Monitoring

Monitor specific processes:

```bash
$ nvidia-smi pmon

# Output:
# gpu   pid  type    sm   mem   enc   dec   command
#   0 12043     C    87    45     0     0   python
```

Metrics:
- **sm**: Streaming Multiprocessor utilization (%)
- **mem**: Memory utilization (%)
- **enc**: Encoder utilization (%)
- **dec**: Decoder utilization (%)

### Detailed Monitoring

Monitor GPU stats continuously:

```bash
$ nvidia-smi dmon -s pucvmet

# p: power
# u: utilization
# c: clocks
# v: violations
# m: memory
# e: ecc errors
# t: temperature
```

### Logging GPU Metrics

Log GPU metrics to a file:

```bash
$ nvidia-smi --query-gpu=timestamp,name,utilization.gpu,memory.used --format=csv --loop=1 > gpu_log.csv
```

This logs metrics every second. Stop with `Ctrl+C`.

## Monitoring GPU Usage in Job Scripts

### Add Monitoring to Your Job

Include GPU monitoring in your job script:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --job-name=gpu_monitor

# Load software
eval $(spack load --sh cuda py-pytorch)

# Start logging GPU usage in background
nvidia-smi --query-gpu=timestamp,utilization.gpu,utilization.memory,memory.used --format=csv --loop=1 > gpu_usage_${SLURM_JOB_ID}.csv &
NVIDIA_SMI_PID=$!

# Run your GPU program
python train.py

# Stop logging
kill $NVIDIA_SMI_PID

# Create summary
echo "GPU Usage Summary:"
nvidia-smi --query-gpu=name,memory.used,memory.total,utilization.gpu --format=csv
```

### Monitor Multiple GPUs

For multi-GPU jobs:

```bash
# Monitor all GPUs
nvidia-smi --query-gpu=index,name,utilization.gpu,memory.used --format=csv

# Monitor specific GPU
nvidia-smi -i 0  # GPU 0 only
nvidia-smi -i 1  # GPU 1 only
```

## Interpreting GPU Utilization

### What is Good Utilization?

**GPU Utilization (GPU-Util):**

- **80-100%**: Excellent - GPU is well utilized
- **50-80%**: Good - Some room for optimization
- **20-50%**: Fair - Likely bottlenecks elsewhere
- **0-20%**: Poor - GPU is mostly idle

**Memory Utilization:**

- High memory usage alone doesn't indicate efficient GPU use
- You want both high GPU-Util AND appropriate memory usage

### Common Patterns

**Pattern 1: High GPU-Util, High Memory**
```
GPU-Util: 95%
Memory:   75000/81920 MB
```
✓ Excellent - GPU working hard with good memory usage

**Pattern 2: Low GPU-Util, High Memory**
```
GPU-Util: 15%
Memory:   60000/81920 MB
```
⚠ Problem - Data loaded but GPU not computing. Likely CPU bottleneck in data loading.

**Pattern 3: Low GPU-Util, Low Memory**
```
GPU-Util: 10%
Memory:   2000/81920 MB
```
⚠ Problem - GPU barely used. Check if code is actually using GPU.

**Pattern 4: Variable GPU-Util**
```
GPU-Util oscillates: 0% → 100% → 0% → 100%
```
⚠ Problem - Bursty computation. Likely I/O or data loading bottleneck.

## Python-Based GPU Monitoring

### Using PyTorch

Monitor GPU from within Python:

```python
import torch

if torch.cuda.is_available():
    # Current GPU memory
    allocated = torch.cuda.memory_allocated(0) / 1e9
    reserved = torch.cuda.memory_reserved(0) / 1e9

    print(f"Allocated: {allocated:.2f} GB")
    print(f"Reserved: {reserved:.2f} GB")

    # GPU properties
    props = torch.cuda.get_device_properties(0)
    print(f"Total memory: {props.total_memory / 1e9:.2f} GB")
    print(f"GPU: {torch.cuda.get_device_name(0)}")

    # Memory summary
    print(torch.cuda.memory_summary())
```

### Periodic Monitoring in Training

```python
import torch
import time

def monitor_gpu(interval=10):
    """Monitor GPU every `interval` seconds"""
    while True:
        if torch.cuda.is_available():
            allocated = torch.cuda.memory_allocated(0) / 1e9
            max_allocated = torch.cuda.max_memory_allocated(0) / 1e9
            print(f"GPU Memory: {allocated:.2f} GB (max: {max_allocated:.2f} GB)")
        time.sleep(interval)

# In your training script:
import threading

# Start monitoring in background
monitor_thread = threading.Thread(target=monitor_gpu, daemon=True)
monitor_thread.start()

# Your training loop
for epoch in range(num_epochs):
    train(...)
```

### Using TensorFlow

```python
import tensorflow as tf

# List GPUs
gpus = tf.config.list_physical_devices('GPU')
print(f"GPUs: {gpus}")

# Memory growth (prevent full allocation)
for gpu in gpus:
    tf.config.experimental.set_memory_growth(gpu, True)

# Get memory info
if gpus:
    gpu_info = tf.config.experimental.get_memory_info('GPU:0')
    print(f"Current memory: {gpu_info['current'] / 1e9:.2f} GB")
    print(f"Peak memory: {gpu_info['peak'] / 1e9:.2f} GB")
```

## Troubleshooting GPU Issues

### GPU Not Being Used

**Symptoms:**
- GPU-Util stays at 0%
- Memory usage is minimal
- Training is slow

**Debugging:**

```python
import torch

print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version: {torch.version.cuda}")
print(f"Number of GPUs: {torch.cuda.device_count()}")

# Check if model is on GPU
print(f"Model device: {next(model.parameters()).device}")

# Check if data is on GPU
print(f"Data device: {batch_data.device}")
```

**Common Causes:**

1. Model not moved to GPU:
   ```python
   model = model.cuda()  # or model.to('cuda')
   ```

2. Data not moved to GPU:
   ```python
   data = data.cuda()    # or data.to('cuda')
   ```

3. No GPU requested in job script:
   ```bash
   #SBATCH --gpus=1  # Add this!
   ```

### Low GPU Utilization

**Symptoms:**
- GPU-Util fluctuates or stays low (< 50%)
- Training slower than expected

**Common Causes:**

1. **CPU Bottleneck** - Data loading too slow
   ```python
   # Increase data loader workers
   DataLoader(dataset, batch_size=64, num_workers=4)  # Increase num_workers
   ```

2. **Small Batch Size** - Not enough work for GPU
   ```python
   # Increase batch size
   batch_size = 128  # Try larger values
   ```

3. **I/O Bottleneck** - Reading data from slow storage
   ```bash
   # Copy data to scratch first
   rsync -av /lts/data/ /scratch/mylab/$USER/data/
   ```

### Out of Memory Errors

**Symptoms:**
```
RuntimeError: CUDA out of memory. Tried to allocate 2.00 GiB
```

**Solutions:**

1. Reduce batch size:
   ```python
   batch_size = 32  # Reduce from 64
   ```

2. Use gradient accumulation:
   ```python
   accumulation_steps = 4
   for i, (data, target) in enumerate(train_loader):
       output = model(data)
       loss = criterion(output, target) / accumulation_steps
       loss.backward()

       if (i + 1) % accumulation_steps == 0:
           optimizer.step()
           optimizer.zero_grad()
   ```

3. Clear cache periodically:
   ```python
   import gc
   gc.collect()
   torch.cuda.empty_cache()
   ```

4. Use mixed precision training:
   ```python
   from torch.cuda.amp import autocast, GradScaler

   scaler = GradScaler()

   with autocast():
       output = model(data)
       loss = criterion(output, target)

   scaler.scale(loss).backward()
   scaler.step(optimizer)
   scaler.update()
   ```

5. Request A100 with more memory:
   ```bash
   #SBATCH --constraint=A100  # 80GB vs V100 32GB
   ```

### Performance Degradation

**Symptoms:**
- Training starts fast but slows down
- GPU utilization drops over time

**Debugging:**

```python
import torch

# Check for memory leaks
for epoch in range(num_epochs):
    print(f"Epoch {epoch}:")
    print(f"  Allocated: {torch.cuda.memory_allocated(0) / 1e9:.2f} GB")
    print(f"  Cached: {torch.cuda.memory_reserved(0) / 1e9:.2f} GB")

    train_one_epoch()

    # Clear cache if growing
    torch.cuda.empty_cache()
```

## Best Practices

### Monitor During Development

Always monitor GPU usage when developing:

1. Start with small dataset
2. Run for a few iterations
3. Check nvidia-smi
4. Verify high GPU utilization
5. Scale up to full dataset

### Include Monitoring in Production

For production runs:

```bash
#!/bin/bash

#SBATCH -p gpu --gpus=1

# Continuous logging
nvidia-smi --query-gpu=timestamp,utilization.gpu,memory.used --format=csv --loop=10 > gpu_log.csv &
MONITOR_PID=$!

# Run training
python train.py

# Stop logging
kill $MONITOR_PID

# Analyze
python << EOF
import pandas as pd
df = pd.read_csv('gpu_log.csv')
print(f"Mean GPU utilization: {df['utilization.gpu [%]'].mean():.1f}%")
print(f"Max memory used: {df[' memory.used [MiB]'].max()} MiB")
EOF
```

### Set Up Alerts

Monitor for problems:

```bash
# Alert if GPU utilization is low
nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader | awk '{if ($1 < 20) print "WARNING: Low GPU utilization: " $1"%"}'
```

## Analyzing GPU Logs

### Parse GPU Log

```python
import pandas as pd
import matplotlib.pyplot as plt

# Read log
df = pd.read_csv('gpu_log.csv')
df['timestamp'] = pd.to_datetime(df['timestamp'])

# Plot utilization over time
plt.figure(figsize=(12, 6))
plt.plot(df['timestamp'], df['utilization.gpu [%]'])
plt.xlabel('Time')
plt.ylabel('GPU Utilization (%)')
plt.title('GPU Utilization Over Time')
plt.grid(True)
plt.savefig('gpu_utilization.png')

# Summary statistics
print(f"Mean GPU utilization: {df['utilization.gpu [%]'].mean():.1f}%")
print(f"Median GPU utilization: {df['utilization.gpu [%]'].median():.1f}%")
print(f"Max memory used: {df[' memory.used [MiB]'].max()} MiB")
```

## Quick Reference

### Essential Commands

```bash
# Current GPU status
nvidia-smi

# Monitor continuously
watch -n 1 nvidia-smi

# GPU utilization only
nvidia-smi --query-gpu=utilization.gpu --format=csv

# Memory usage
nvidia-smi --query-gpu=memory.used,memory.total --format=csv

# Process monitoring
nvidia-smi pmon

# Log to file
nvidia-smi --query-gpu=timestamp,utilization.gpu,memory.used --format=csv --loop=1 > gpu.log
```

### In Python

```python
# PyTorch
torch.cuda.is_available()
torch.cuda.memory_allocated(0) / 1e9
torch.cuda.memory_summary()

# TensorFlow
tf.config.list_physical_devices('GPU')
tf.config.experimental.get_memory_info('GPU:0')
```

## Next Steps

- [GPU Examples](examples.md) - Working code examples
- [CUDA Setup](cuda-setup.md) - Configure your environment
- [Machine Learning Workflows](../workflows/machine-learning.md) - Complete pipelines
- [Job Optimization](../best-practices/job-optimization.md) - Optimize performance

--8<-- "includes/getting-help.md"
