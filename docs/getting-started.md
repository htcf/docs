# Getting Started

## Connecting

Connect to the HTCF login node using SSH with your WashU Key credentials:

```bash
ssh username@login.htcf.wustl.edu
```

Replace `username` with your WashU Key.

!!! note "Account Sharing Prohibited"
    As stated in the [HTCF Policies](policies.md#account-usage), accounts and passwords cannot be shared. All users must have their own account.

## After You Connect

When you log in, you'll be in your home directory (`/home/username`). This is your 20GB personal space for scripts and configuration files.

Check your storage usage across all systems:

```bash
storage-info
```

See what compute resources are available:

```bash
sinfo -N -p general -o '%n %c %m'
```

## Storage Overview

The HTCF has several storage locations for different purposes:

| Location | Purpose | Quota | Backed Up |
|----------|---------|-------|-----------|
| `/home/username` | Scripts, config files | 20GB | Yes |
| `/lts/labname` | Raw data, finished results | None | Yes |
| `/scratch/labname/username` | Active job I/O | 2TB | No |
| `/ref/labname` | Reference data, software | Shared | No |

*Replace `labname` and `username` with your actual lab and username.*

**Key points:**

- Jobs should read/write from `/scratch` for performance
- LTS is read-only from compute nodes
- Copy final results to LTS after jobs complete

See [Storage](storage/index.md) for details.

## Running Jobs

The HTCF uses Slurm to manage compute resources. Never run intensive computation on the login node.

### Interactive Sessions

For testing, debugging, or interactive tools:

```bash
srun -J interactive -p interactive --pty /bin/bash -l
```

Request more resources if needed:

```bash
srun --mem=4G --cpus-per-task=4 -J interactive -p interactive --pty /bin/bash -l
```

!!! warning "Interactive Queue Required"
    Interactive tools (RStudio, Jupyter) must use `-p interactive`. Jobs using interactive tools in other queues may be cancelled.

### Batch Jobs

For production workloads, submit batch scripts:

```bash
sbatch myjob.sh
```

A minimal job script:

```bash
#!/bin/bash
#SBATCH --mem=4G
#SBATCH --cpus-per-task=1
#SBATCH --time=1:00:00

# Your commands here
echo "Running on $(hostname)"
```

Check your jobs:

```bash
squeue -u $USER
```

See [Batch Jobs](using/batch.md) for the full reference.

### GPU Jobs

Request GPU resources with:

```bash
#SBATCH -p gpu
#SBATCH --gpus=1
```

See [GPU Computing](gpu/index.md) for setup and examples.

## Next Steps

- [Your First Batch Job](tutorials/first-job.md) - Step-by-step tutorial
- [Software Installation](software/index.md) - Installing packages with Spack
- [Storage Guide](storage/index.md) - Managing your data
- [Troubleshooting](troubleshooting/index.md) - Common issues and solutions

## Getting Help

Post in **#general** on Slack with:

- What you're trying to do
- The command you ran
- Complete error output
- Job ID (if applicable)
