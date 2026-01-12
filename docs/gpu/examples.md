# GPU Examples

## Overview

This page provides simple, working examples to help you get started with GPU computing on the HTCF. All examples include complete job scripts that you can adapt for your own work.

## Basic GPU Examples

### Example 1: GPU Availability Check

Simple Python script to verify GPU access:

**File: `check_gpu.py`**

```python
#!/usr/bin/env python3
"""
Simple script to check GPU availability
"""
import torch

print("=" * 50)
print("GPU Availability Check")
print("=" * 50)

# Check if CUDA is available
cuda_available = torch.cuda.is_available()
print(f"CUDA available: {cuda_available}")

if cuda_available:
    # Get number of GPUs
    gpu_count = torch.cuda.device_count()
    print(f"Number of GPUs: {gpu_count}")

    # Print info for each GPU
    for i in range(gpu_count):
        print(f"\nGPU {i}:")
        print(f"  Name: {torch.cuda.get_device_name(i)}")
        print(f"  Memory: {torch.cuda.get_device_properties(i).total_memory / 1e9:.2f} GB")
        print(f"  Compute Capability: {torch.cuda.get_device_properties(i).major}.{torch.cuda.get_device_properties(i).minor}")

    # Test tensor on GPU
    print("\nTesting tensor creation on GPU...")
    x = torch.randn(1000, 1000, device='cuda')
    print(f"Tensor device: {x.device}")
    print(f"Tensor shape: {x.shape}")
    print("✓ GPU tensor creation successful!")
else:
    print("No CUDA-capable GPUs found.")
    print("Make sure you:")
    print("  1. Requested GPU in your job script (#SBATCH --gpus=1)")
    print("  2. Loaded CUDA module (eval $(spack load --sh cuda))")
    print("  3. Are running on a GPU node")
```

**Job Script: `check_gpu.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=check_gpu
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=8G
#SBATCH --time=10:00
#SBATCH --output=check_gpu_%j.out

# Load CUDA and PyTorch
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

# Run GPU check
python check_gpu.py

# Also show nvidia-smi output
echo ""
echo "=" * 50
echo "nvidia-smi output:"
echo "=" * 50
nvidia-smi
```

**Submit:**

```bash
$ sbatch check_gpu.sh
```

### Example 2: Simple Matrix Multiplication

Compare CPU vs GPU performance for matrix multiplication:

**File: `matrix_multiply.py`**

```python
#!/usr/bin/env python3
"""
Compare CPU vs GPU matrix multiplication performance
"""
import torch
import time

# Matrix size
n = 5000

print(f"Matrix multiplication: {n}x{n} matrices\n")

# CPU benchmark
print("CPU:")
x_cpu = torch.randn(n, n)
y_cpu = torch.randn(n, n)

start = time.time()
z_cpu = torch.mm(x_cpu, y_cpu)
cpu_time = time.time() - start

print(f"  Time: {cpu_time:.4f} seconds")

# GPU benchmark
if torch.cuda.is_available():
    print("\nGPU:")
    x_gpu = torch.randn(n, n, device='cuda')
    y_gpu = torch.randn(n, n, device='cuda')

    # Warm up
    _ = torch.mm(x_gpu, y_gpu)
    torch.cuda.synchronize()

    start = time.time()
    z_gpu = torch.mm(x_gpu, y_gpu)
    torch.cuda.synchronize()
    gpu_time = time.time() - start

    print(f"  Time: {gpu_time:.4f} seconds")
    print(f"\nSpeedup: {cpu_time / gpu_time:.2f}x")
else:
    print("\nGPU not available")
```

**Job Script: `matrix_multiply.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=matrix_mult
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=15:00
#SBATCH --output=matrix_mult_%j.out

# Load software
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

# Run benchmark
python matrix_multiply.py
```

### Example 3: GPU Memory Management

Test GPU memory allocation and usage:

**File: `gpu_memory.py`**

```python
#!/usr/bin/env python3
"""
Demonstrate GPU memory management
"""
import torch
import gc

def get_gpu_memory():
    """Get current GPU memory usage in GB"""
    if torch.cuda.is_available():
        allocated = torch.cuda.memory_allocated(0) / 1e9
        reserved = torch.cuda.memory_reserved(0) / 1e9
        return allocated, reserved
    return 0, 0

print("GPU Memory Management Example\n")

# Check initial memory
alloc, reserved = get_gpu_memory()
print(f"Initial - Allocated: {alloc:.2f} GB, Reserved: {reserved:.2f} GB\n")

# Allocate tensors of increasing size
sizes = [1000, 5000, 10000]
tensors = []

for size in sizes:
    print(f"Allocating {size}x{size} tensor...")
    t = torch.randn(size, size, device='cuda')
    tensors.append(t)

    alloc, reserved = get_gpu_memory()
    print(f"  Allocated: {alloc:.2f} GB, Reserved: {reserved:.2f} GB")

# Clear tensors
print("\nClearing tensors...")
tensors.clear()
gc.collect()
torch.cuda.empty_cache()

alloc, reserved = get_gpu_memory()
print(f"After cleanup - Allocated: {alloc:.2f} GB, Reserved: {reserved:.2f} GB")

# Show total GPU memory
if torch.cuda.is_available():
    props = torch.cuda.get_device_properties(0)
    total_memory = props.total_memory / 1e9
    print(f"\nTotal GPU memory: {total_memory:.2f} GB")
    print(f"GPU: {torch.cuda.get_device_name(0)}")
```

**Job Script: `gpu_memory.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=gpu_memory
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --mem=8G
#SBATCH --time=10:00
#SBATCH --output=gpu_memory_%j.out

eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

python gpu_memory.py
```

## Deep Learning Examples

### Example 4: Simple Neural Network Training

Train a simple neural network on GPU:

**File: `simple_nn.py`**

```python
#!/usr/bin/env python3
"""
Simple neural network training on GPU
"""
import torch
import torch.nn as nn
import torch.optim as optim
import time

# Set device
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")
if torch.cuda.is_available():
    print(f"GPU: {torch.cuda.get_device_name(0)}\n")

# Simple neural network
class SimpleNN(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(SimpleNN, self).__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, hidden_size)
        self.fc3 = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        x = self.fc3(x)
        return x

# Parameters
input_size = 784
hidden_size = 512
output_size = 10
batch_size = 128
num_epochs = 10

# Create model and move to device
model = SimpleNN(input_size, hidden_size, output_size).to(device)
print(f"Model parameters: {sum(p.numel() for p in model.parameters()):,}")

# Loss and optimizer
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Training loop
print(f"\nTraining for {num_epochs} epochs...")
start_time = time.time()

for epoch in range(num_epochs):
    epoch_start = time.time()

    # Simulate batch (replace with real data loader)
    for batch in range(100):
        # Generate random data
        inputs = torch.randn(batch_size, input_size).to(device)
        labels = torch.randint(0, output_size, (batch_size,)).to(device)

        # Forward pass
        outputs = model(inputs)
        loss = criterion(outputs, labels)

        # Backward and optimize
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

    epoch_time = time.time() - epoch_start
    print(f"Epoch {epoch+1}/{num_epochs}, Loss: {loss.item():.4f}, Time: {epoch_time:.2f}s")

total_time = time.time() - start_time
print(f"\nTotal training time: {total_time:.2f} seconds")
print(f"Average time per epoch: {total_time/num_epochs:.2f} seconds")
```

**Job Script: `simple_nn.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=simple_nn
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --time=30:00
#SBATCH --output=simple_nn_%j.out

eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

python simple_nn.py

# Monitor GPU usage
echo -e "\nGPU Usage Summary:"
nvidia-smi --query-gpu=name,memory.used,memory.total,utilization.gpu --format=csv
```

## Multi-GPU Examples

### Example 5: Data Parallel Training

Use multiple GPUs with DataParallel:

**File: `multi_gpu.py`**

```python
#!/usr/bin/env python3
"""
Multi-GPU training with DataParallel
"""
import torch
import torch.nn as nn

# Check GPUs
if not torch.cuda.is_available():
    print("No CUDA GPUs available!")
    exit(1)

num_gpus = torch.cuda.device_count()
print(f"Number of GPUs available: {num_gpus}")
for i in range(num_gpus):
    print(f"  GPU {i}: {torch.cuda.get_device_name(i)}")

# Simple model
class SimpleModel(nn.Module):
    def __init__(self):
        super(SimpleModel, self).__init__()
        self.fc1 = nn.Linear(1000, 512)
        self.fc2 = nn.Linear(512, 10)

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        x = self.fc2(x)
        return x

# Create model
model = SimpleModel()

# Use DataParallel if multiple GPUs
if num_gpus > 1:
    print(f"\nUsing DataParallel across {num_gpus} GPUs")
    model = nn.DataParallel(model)

model = model.cuda()

# Test forward pass
batch_size = 64
x = torch.randn(batch_size, 1000).cuda()
output = model(x)

print(f"\nInput shape: {x.shape}")
print(f"Output shape: {output.shape}")
print("✓ Multi-GPU model working!")

# Show memory usage on each GPU
print("\nGPU Memory Usage:")
for i in range(num_gpus):
    allocated = torch.cuda.memory_allocated(i) / 1e9
    print(f"  GPU {i}: {allocated:.2f} GB")
```

**Job Script: `multi_gpu.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=multi_gpu
#SBATCH -p gpu
#SBATCH --gpus=2              # Request 2 GPUs
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=15:00
#SBATCH --output=multi_gpu_%j.out

eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

python multi_gpu.py

# Show GPU utilization
nvidia-smi
```

## CUDA C++ Examples

### Example 6: Vector Addition in CUDA

Simple CUDA C++ program:

**File: `vector_add.cu`**

```cuda
#include <stdio.h>
#include <cuda_runtime.h>

// CUDA kernel for vector addition
__global__ void vectorAdd(const float *A, const float *B, float *C, int n) {
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    if (i < n) {
        C[i] = A[i] + B[i];
    }
}

int main() {
    int n = 1000000;
    size_t size = n * sizeof(float);

    // Allocate host memory
    float *h_A = (float *)malloc(size);
    float *h_B = (float *)malloc(size);
    float *h_C = (float *)malloc(size);

    // Initialize host arrays
    for (int i = 0; i < n; i++) {
        h_A[i] = i;
        h_B[i] = i * 2;
    }

    // Allocate device memory
    float *d_A, *d_B, *d_C;
    cudaMalloc(&d_A, size);
    cudaMalloc(&d_B, size);
    cudaMalloc(&d_C, size);

    // Copy data to device
    cudaMemcpy(d_A, h_A, size, cudaMemcpyHostToDevice);
    cudaMemcpy(d_B, h_B, size, cudaMemcpyHostToDevice);

    // Launch kernel
    int threadsPerBlock = 256;
    int blocksPerGrid = (n + threadsPerBlock - 1) / threadsPerBlock;
    vectorAdd<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, n);

    // Copy result back to host
    cudaMemcpy(h_C, d_C, size, cudaMemcpyDeviceToHost);

    // Verify result
    for (int i = 0; i < 10; i++) {
        printf("%.0f + %.0f = %.0f\n", h_A[i], h_B[i], h_C[i]);
    }

    // Cleanup
    cudaFree(d_A);
    cudaFree(d_B);
    cudaFree(d_C);
    free(h_A);
    free(h_B);
    free(h_C);

    printf("\n✓ Vector addition completed successfully!\n");
    return 0;
}
```

**Job Script: `vector_add.sh`**

```bash
#!/bin/bash

#SBATCH --job-name=vector_add
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --mem=4G
#SBATCH --time=10:00
#SBATCH --output=vector_add_%j.out

# Load CUDA
eval $(spack load --sh cuda@12.0)

# Compile CUDA code
echo "Compiling CUDA code..."
nvcc -o vector_add vector_add.cu

# Run
echo "Running vector addition..."
./vector_add

# Show GPU info
echo ""
nvidia-smi --query-gpu=name,memory.used --format=csv,noheader
```

## Performance Tips

### Batch Size Optimization

Test different batch sizes to maximize GPU utilization:

```python
import torch
import time

def benchmark_batch_size(model, input_size, batch_sizes, device='cuda'):
    """Benchmark different batch sizes"""
    results = {}

    for batch_size in batch_sizes:
        # Create dummy data
        data = torch.randn(batch_size, *input_size).to(device)

        # Warm up
        for _ in range(10):
            _ = model(data)

        # Benchmark
        torch.cuda.synchronize()
        start = time.time()

        for _ in range(100):
            _ = model(data)

        torch.cuda.synchronize()
        elapsed = time.time() - start

        throughput = (100 * batch_size) / elapsed
        results[batch_size] = throughput

        print(f"Batch size {batch_size:4d}: {throughput:8.2f} samples/sec")

    return results

# Usage
# results = benchmark_batch_size(model, (3, 224, 224), [16, 32, 64, 128, 256])
```

## Next Steps

- [GPU Monitoring](monitoring.md) - Monitor GPU usage during jobs
- [CUDA Setup](cuda-setup.md) - Configure CUDA environment
- [Machine Learning Workflows](../workflows/machine-learning.md) - Complete ML pipelines
- [First GPU Job Tutorial](../tutorials/first-gpu-job.md) - Step-by-step tutorial

## Getting Help

--8<-- "includes/getting-help.md"
