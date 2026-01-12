## GPU-Specific SLURM Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `-p gpu` | Use the GPU partition (required) | `#SBATCH -p gpu` |
| `--gpus` | Number of GPUs to request (preferred) | `#SBATCH --gpus=2` |
| `--constraint` | Specify GPU type (A100 or V100) | `#SBATCH --constraint=A100` |
| `--gres` | Generic resource syntax (older, also works) | `#SBATCH --gres=gpu:2` |

!!! tip "GPU Request Syntax"
    Use `--gpus=N` (modern, clear syntax). The older `--gres=gpu:N` syntax also works. To request a specific model, combine with `--constraint=A100` or `--constraint=V100`.

!!! note "GPU Time Limits"
    The default time limit for GPU jobs is 8 hours. Use `--time` to specify longer durations if needed.

!!! tip "Available GPUs"
    - 6x NVIDIA A100 80GB GPUs
    - 2x NVIDIA V100 32GB GPUs
