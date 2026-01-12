# CUDA Setup and Environment

## Overview

CUDA (Compute Unified Device Architecture) is NVIDIA's parallel computing platform and programming model. To use GPUs on the HTCF, you need to set up the CUDA environment and load GPU-enabled software.

**Official Documentation:**

- [CUDA Documentation](https://docs.nvidia.com/cuda/) - NVIDIA CUDA complete reference
- [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/) - GPU programming guide
- [PyTorch Documentation](https://pytorch.org/docs/stable/index.html) - PyTorch with CUDA
- [TensorFlow GPU Guide](https://www.tensorflow.org/install/gpu) - TensorFlow GPU setup
- [JAX GPU Documentation](https://jax.readthedocs.io/en/latest/gpu_performance_tips.html) - JAX on GPUs

## CUDA Availability

### Checking Available CUDA Versions

The HTCF provides CUDA through Spack. To see available versions:

```bash
$ spack find cuda
```

Or search for specific versions:

```bash
$ spack find -v cuda
```

### Current CUDA Version

The HTCF GPU nodes are currently running with **CUDA 12.0** driver support:

```bash
$ nvidia-smi
```

Look for the "CUDA Version" in the top right of the output.

## Loading CUDA

### Using Spack

Load CUDA with Spack in your job script:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Load CUDA 12.0
eval $(spack load --sh cuda@12.0)

# Verify
nvcc --version

# Your GPU code
./my_cuda_program
```

### Loading Specific Versions

```bash
# Load latest CUDA
eval $(spack load --sh cuda)

# Load specific version
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh cuda@11.8)
```

!!! tip "CUDA Version Compatibility"
    Make sure the CUDA version you load is compatible with:

    - Your GPU hardware (A100, V100)
    - Your deep learning framework version
    - Your compiled CUDA code

## Environment Variables

### Required CUDA Environment Variables

When you load CUDA via Spack, these variables are automatically set:

- **`CUDA_HOME`** or **`CUDA_PATH`** - Path to CUDA installation
- **`PATH`** - Includes CUDA bin directory for `nvcc`, `nvidia-smi`, etc.
- **`LD_LIBRARY_PATH`** - Includes CUDA library directories

### Verify Environment

```bash
# Check CUDA home
echo $CUDA_HOME

# Check nvcc is in PATH
which nvcc

# Check CUDA version
nvcc --version
```

### Manual Environment Setup (If Needed)

In rare cases where you need to manually set up CUDA:

```bash
export CUDA_HOME=/path/to/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH
```

## CUDA Toolkit Components

### CUDA Compiler (nvcc)

The NVIDIA CUDA Compiler (nvcc) compiles CUDA C/C++ code:

```bash
$ nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Mon_Apr_3_17:16:06_PDT_2023
Cuda compilation tools, release 12.0, V12.0.140
```

Compile CUDA code:

```bash
$ nvcc my_program.cu -o my_program
$ ./my_program
```

### CUDA Libraries

Common CUDA libraries available:

- **cuBLAS** - Basic Linear Algebra Subprograms
- **cuDNN** - Deep Neural Network library
- **cuFFT** - Fast Fourier Transform library
- **cuSPARSE** - Sparse matrix operations
- **cuRAND** - Random number generation

These are typically included with the CUDA installation or available as separate Spack packages.

### CUDA Samples

CUDA provides sample programs for learning and testing:

```bash
# Samples are typically in
$CUDA_HOME/samples/
```

## Python GPU Frameworks

### PyTorch with CUDA

Install and use PyTorch with GPU support:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Load CUDA and PyTorch
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

# Verify PyTorch sees CUDA
python << EOF
import torch
print(f"PyTorch version: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA version: {torch.version.cuda}")
print(f"Number of GPUs: {torch.cuda.device_count()}")
if torch.cuda.is_available():
    print(f"GPU name: {torch.cuda.get_device_name(0)}")
EOF
```

Expected output:

```
PyTorch version: 2.0.1
CUDA available: True
CUDA version: 12.0
Number of GPUs: 1
GPU name: NVIDIA A100 80GB PCIe
```

#### PyTorch with Multiple GPUs

```python
import torch

# Check all available GPUs
for i in range(torch.cuda.device_count()):
    print(f"GPU {i}: {torch.cuda.get_device_name(i)}")

# Use specific GPU
device = torch.device('cuda:0')
model = model.to(device)

# Data parallel across GPUs
model = torch.nn.DataParallel(model)
```

### TensorFlow with GPU

Install and use TensorFlow with GPU support:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Load CUDA and TensorFlow
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-tensorflow+cuda)

# Verify TensorFlow sees GPUs
python << EOF
import tensorflow as tf
print(f"TensorFlow version: {tf.__version__}")
print(f"GPUs available: {tf.config.list_physical_devices('GPU')}")
EOF
```

Expected output:

```
TensorFlow version: 2.12.0
GPUs available: [PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

#### TensorFlow GPU Configuration

```python
import tensorflow as tf

# List physical devices
gpus = tf.config.list_physical_devices('GPU')
print(f"GPUs available: {len(gpus)}")

# Limit GPU memory growth (prevent allocation of all memory)
for gpu in gpus:
    tf.config.experimental.set_memory_growth(gpu, True)

# Or set specific memory limit
tf.config.set_logical_device_configuration(
    gpus[0],
    [tf.config.LogicalDeviceConfiguration(memory_limit=40960)]  # 40GB
)
```

### JAX with CUDA

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Load CUDA and JAX
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-jax)

# Verify JAX sees GPUs
python << EOF
import jax
print(f"JAX devices: {jax.devices()}")
print(f"Default backend: {jax.default_backend()}")
EOF
```

## CUDA-Enabled Software Packages

### Installing with Spack

Many scientific packages have CUDA variants available through Spack:

```bash
# Search for CUDA variants
$ spack info package-name

# Install with CUDA support
$ spack install package-name+cuda

# Example: GROMACS with CUDA
$ spack install gromacs+cuda
```

### Common CUDA-Enabled Packages

| Package | Spack Command | Use Case |
|---------|---------------|----------|
| PyTorch | `spack install py-pytorch+cuda` | Deep learning |
| TensorFlow | `spack install py-tensorflow+cuda` | Deep learning |
| JAX | `spack install py-jax+cuda` | ML research |
| GROMACS | `spack install gromacs+cuda` | Molecular dynamics |
| LAMMPS | `spack install lammps+cuda` | Molecular dynamics |
| NAMD | `spack install namd+cuda` | Molecular dynamics |

## Creating a GPU-Enabled Environment

### Complete Deep Learning Environment

```bash
#!/bin/bash
# setup_dl_env.sh - Create complete deep learning environment

# Load CUDA
eval $(spack load --sh cuda@12.0)

# Load PyTorch
eval $(spack load --sh py-pytorch+cuda)

# Load common packages
eval $(spack load --sh py-numpy)
eval $(spack load --sh py-scipy)
eval $(spack load --sh py-matplotlib)
eval $(spack load --sh py-pandas)

# Verify
python << EOF
import torch
import numpy as np
import matplotlib
print("Environment ready!")
print(f"CUDA available: {torch.cuda.is_available()}")
EOF
```

Make it executable and source in your job scripts:

```bash
$ chmod +x setup_dl_env.sh
```

In your job script:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

source setup_dl_env.sh

python train.py
```

### Using Python Virtual Environments

For pip packages with GPU support:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

# Load CUDA first
eval $(spack load --sh cuda@12.0)

# Create/activate virtual environment
python -m venv ~/myenv
source ~/myenv/bin/activate

# Install GPU-enabled packages
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121

# Verify
python -c "import torch; print(torch.cuda.is_available())"
```

!!! warning "CUDA Compatibility"
    When using pip to install packages like PyTorch, make sure the wheel version matches your CUDA version (e.g., cu121 for CUDA 12.1).

## Verifying CUDA Setup

### Complete Verification Script

```bash
#!/bin/bash
# verify_cuda.sh - Verify complete CUDA setup

echo "=== CUDA Environment ==="
echo "CUDA_HOME: $CUDA_HOME"
echo ""

echo "=== NVIDIA Driver ==="
nvidia-smi --query-gpu=name,driver_version,memory.total --format=csv
echo ""

echo "=== CUDA Version ==="
nvcc --version
echo ""

echo "=== PyTorch CUDA ==="
python << EOF
import torch
print(f"PyTorch: {torch.__version__}")
print(f"CUDA available: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"CUDA version: {torch.version.cuda}")
    print(f"cuDNN version: {torch.backends.cudnn.version()}")
    print(f"Number of GPUs: {torch.cuda.device_count()}")
    for i in range(torch.cuda.device_count()):
        print(f"  GPU {i}: {torch.cuda.get_device_name(i)}")
EOF
echo ""

echo "=== TensorFlow GPU ==="
python << EOF
try:
    import tensorflow as tf
    print(f"TensorFlow: {tf.__version__}")
    gpus = tf.config.list_physical_devices('GPU')
    print(f"GPUs available: {len(gpus)}")
    for gpu in gpus:
        print(f"  {gpu}")
except ImportError:
    print("TensorFlow not installed")
EOF
```

Use in a job:

```bash
#!/bin/bash

#SBATCH -p gpu
#SBATCH --gpus=1

eval $(spack load --sh cuda py-pytorch py-tensorflow)

bash verify_cuda.sh > cuda_verification.txt
cat cuda_verification.txt
```

## Troubleshooting CUDA Setup

### CUDA Not Found

**Problem:** `nvcc: command not found` or CUDA libraries not found

**Solution:**

```bash
# Make sure CUDA is loaded
eval $(spack load --sh cuda)

# Verify
which nvcc
echo $CUDA_HOME
```

### Version Mismatches

**Problem:** PyTorch/TensorFlow built for different CUDA version

**Solution:**

```bash
# Check driver CUDA version
nvidia-smi | grep "CUDA Version"

# Load matching CUDA version
eval $(spack load --sh cuda@<matching-version>)

# Or install package with matching CUDA version
spack install py-pytorch+cuda ^cuda@12.0
```

### Library Loading Errors

**Problem:** `libcudart.so.12.0: cannot open shared object file`

**Solution:**

```bash
# Ensure LD_LIBRARY_PATH is set
echo $LD_LIBRARY_PATH

# Should include CUDA lib64
# If not, reload CUDA
eval $(spack load --sh cuda)

# Verify
ldd $(which python) | grep cuda
```

### GPU Not Visible to Framework

**Problem:** PyTorch/TensorFlow can't see GPU

**Checklist:**

1. Job is running on GPU node: `squeue -j $SLURM_JOB_ID`
2. GPU requested in sbatch: `#SBATCH --gpus=1`
3. GPU visible: `nvidia-smi`
4. CUDA loaded: `which nvcc`
5. Framework GPU-enabled: `spack info py-pytorch | grep cuda`

## Best Practices

### Load Order

Always load CUDA before GPU-enabled packages:

```bash
# Correct order
eval $(spack load --sh cuda)
eval $(spack load --sh py-pytorch)

# May cause issues if reversed
```

### Version Pinning

For reproducibility, pin specific versions:

```bash
# Pin specific versions
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch@2.0.1+cuda)
```

### Reusable Environment Scripts

Create a reusable environment script:

```bash
# ~/.spack_gpu_env
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)
eval $(spack load --sh py-numpy)
```

Source in job scripts:

```bash
source ~/.spack_gpu_env
```

## Next Steps

- [GPU Examples](examples.md) - Try simple GPU programs
- [GPU Monitoring](monitoring.md) - Monitor GPU usage
- [Machine Learning Workflows](../workflows/machine-learning.md) - Complete ML examples

--8<-- "includes/getting-help.md"
