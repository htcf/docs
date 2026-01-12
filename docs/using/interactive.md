# Interactive Jobs

## Overview

Interactive sessions allow you to work directly on compute nodes with access to more resources than the login nodes provide. They're designed for tasks that require user interaction, such as development, testing, visualization, and software compilation.

!!! note "HTCF is Primarily a Batch System"
    The HTCF is primarily a batch queuing system.

    Interactive jobs are meant to function as **daily workspaces**, not for long-running production computations. Because interactive jobs are by their nature inefficient, they are not meant to be running continuously for more than 1 day.

    When using interactive tools such as RStudio or Jupyter, please make sure the jobs are using the "interactive" queue (using sbatch/srun parameters `-J interactive -p interactive`).

    **Jobs using interactive tools that are not in the interactive queue will be subject to cancellation in order to free up resources for batch jobs.**

    Thanks for helping to ensure fairness for all folks on the HTCF.

## When to Use Interactive Jobs

### Appropriate Uses

Interactive jobs are ideal for:

- **Development and debugging** - Testing code and scripts
- **Interactive visualization** - Plotting and data exploration
- **Software installation** - Installing and configuring software
- **Parameter testing** - Testing different parameters before batch runs
- **Interactive applications** - RStudio, Jupyter Lab, and similar tools
- **Software compilation** - Building software from source

### When to Use Batch Jobs Instead

Convert to batch jobs for:

- **Production runs** - Any finalized computational workflow
- **Long-running computations** - Jobs that take more than a few hours
- **Multiple sample processing** - Use array jobs for parallel processing
- **Automated workflows** - Anything that doesn't need interaction

Tools such as Rscript can be used to run R programs in a batch fashion. Jupyter notebooks can also be run in a [batch fashion](http://tritemio.github.io/smbits/2016/01/02/execute-notebooks/).

## Starting Interactive Sessions

### Basic Interactive Session

Start an interactive session with the `srun` command:

```bash
$ srun -J interactive -p interactive --pty /bin/bash -l
```

This starts an interactive bash session with default resources:

- 1 CPU core
- 1 GB RAM
- 16 hour time limit
- Interactive partition

Once your session starts, you'll have a command prompt on a compute node where you can run commands interactively.

### Requesting Custom Resources

#### Requesting More Memory

```bash
$ srun --mem-per-cpu=8G -J interactive -p interactive --pty /bin/bash -l
```

Or specify total memory:

```bash
$ srun --mem=32G -J interactive -p interactive --pty /bin/bash -l
```

#### Requesting Multiple CPUs

```bash
$ srun --cpus-per-task=8 -J interactive -p interactive --pty /bin/bash -l
```

#### Specifying Time Limit

```bash
$ srun --time=4:00:00 -J interactive -p interactive --pty /bin/bash -l
```

!!! note "Interactive Partition Limits"
    - Maximum time limit: 16 hours
    - Maximum memory per node: 250GB
    - Partition: interactive (always use `-p interactive`)

#### Combining Resource Requests

```bash
$ srun --cpus-per-task=4 --mem=16G --time=6:00:00 -J interactive -p interactive --pty /bin/bash -l
```

### GPU Interactive Sessions

For interactive GPU work:

```bash
$ srun -p gpu --gpus=1 -J interactive --pty /bin/bash -l
```

!!! tip "GPU Resources"
    Interactive GPU sessions should be used sparingly as GPU resources are limited.

See the [GPU Computing documentation](../gpu/index.md) for more details.

## Interactive Session Examples

### Development and Testing

Start a session, load software, and test your script:

```bash
# Start interactive session
$ srun --mem=8G --cpus-per-task=2 -J interactive -p interactive --pty /bin/bash -l

# On compute node - load software
$ eval $(spack load --sh python py-numpy)

# Test your script
$ python test_script.py

# Exit when done
$ exit
```

### Data Exploration

```bash
# Start session with more memory
$ srun --mem=32G -J interactive -p interactive --pty /bin/bash -l

# Load R
$ eval $(spack load --sh r)

# Start R
$ R

# In R, load and explore data
> data <- read.csv("/scratch/user/data.csv")
> summary(data)
> plot(data$x, data$y)

# Exit R and session
> quit()
$ exit
```

### Software Compilation

```bash
# Start session with multiple cores for parallel compilation
$ srun --cpus-per-task=8 --mem=16G -J interactive -p interactive --pty /bin/bash -l

# Navigate to source directory
$ cd /scratch/mylab/$USER/software/package-1.0/

# Configure and build
$ ./configure --prefix=$HOME/software/package-1.0
$ make -j 8

# Install
$ make install

# Exit session
$ exit
```

## Interactive Tools

### Jupyter Lab

Jupyter Lab provides an interactive web-based environment for data science and scientific computing.

**Installation and Usage:**

See the complete [Jupyter Lab Guide](../software/jupyter/index.md) for detailed instructions on:

- Installing Jupyter with Spack
- Starting Jupyter Lab in an interactive job
- Accessing the web interface via SSH tunneling
- Managing kernels and extensions

**Quick Start:**

```bash
# Start interactive session
$ srun --mem=8G --cpus-per-task=4 -J interactive -p interactive --pty /bin/bash -l

# Load and start Jupyter
$ eval $(spack load --sh py-jupyterlab)
$ jupyter lab --no-browser --ip=0.0.0.0
```

Then set up SSH tunneling from your local machine to access the web interface.

### RStudio Server

RStudio Server provides a familiar IDE for R development.

**Installation and Usage:**

See the complete [RStudio Guide](../software/rstudio/index.md) for detailed instructions on:

- Compiling RStudio Server from source
- Starting RStudio in an interactive job
- Accessing the web interface
- Managing R packages and libraries

### Command Line Tools

#### Text Editors

Interactive sessions support all standard text editors:

- **nano** - Simple, beginner-friendly
- **vim** - Powerful, modal editor
- **emacs** - Extensible editor

```bash
$ nano script.py
$ vim analysis.R
$ emacs workflow.sh
```

#### Screen and Tmux

For persistent sessions that survive disconnections:

```bash
# Start screen session
$ screen -S mywork

# Detach with Ctrl+A then D
# Reattach later
$ screen -r mywork
```

!!! warning "Session Persistence"
    Screen and tmux only persist while your interactive job is running. If the job time limit is reached, the session and all processes will be terminated.

### Interactive Python/R/MATLAB Sessions

Load and use interactive interpreters:

```bash
# Python
$ eval $(spack load --sh python)
$ python
>>> import numpy as np
>>> exit()

# R
$ eval $(spack load --sh r)
$ R
> library(ggplot2)
> quit()

# IPython for interactive Python with better features
$ eval $(spack load --sh py-ipython)
$ ipython
```

## Monitoring Interactive Sessions

### Check Resource Usage

While in an interactive session, check resource usage:

```bash
# CPU and memory usage
$ top

# Memory usage
$ free -h

# Your processes
$ ps aux | grep $USER

# GPU usage (if on GPU node)
$ nvidia-smi
```

### Check Time Remaining

```bash
# View your running jobs
$ squeue -u $USER
```

Look at the TIME column to see how long your job has been running. Subtract from your requested time limit to see remaining time.

## Ending Interactive Sessions

### Graceful Exit

Always exit interactive sessions when you're done:

```bash
$ exit
```

This frees up resources for other users.

### Automatic Termination

Interactive sessions will automatically terminate when:

- Time limit is reached (default: 16 hours)
- You logout/disconnect and don't use screen/tmux
- The node needs to be rebooted for maintenance

!!! reminder "Save Your Work"
    Save your work frequently and before exiting. Interactive sessions can be terminated unexpectedly due to node failures or maintenance.

## Best Practices

### Resource Management

!!! tip "Request What You Need"
    - Start with minimal resources and increase if needed
    - Don't request more CPUs than your task can use
    - Close sessions when not actively using them

### Login Node vs Interactive Jobs

| Task | Login Node | Interactive Job |
|------|------------|-----------------|
| File management (ls, cp, mv) | ✓ | ✓ |
| Text editing | ✓ | ✓ |
| Job submission | ✓ | ✓ |
| Quick file viewing | ✓ | ✓ |
| Software compilation (< 5 min) | ✓ | Preferred |
| Software compilation (> 5 min) | ✗ | ✓ |
| Data analysis | ✗ | ✓ |
| Running scripts | ✗ | ✓ |
| Testing code | ✗ | ✓ |

!!! warning "Login Node Policy"
    Computational work on login nodes is not permitted. See [HTCF Policies](../policies.md#login-node-policy) for details.

### Development Workflow

Follow this efficient development workflow:

1. **Edit** code on login node or local machine
2. **Test** in an interactive session
3. **Submit** as batch job for production runs
4. **Analyze** results, adjust parameters
5. **Repeat** steps 2-4 as needed

### Interactive Job Checklist

Before starting an interactive job:

- [ ] Do I actually need interaction, or could this be a batch job?
- [ ] Have I estimated the resources I need (CPUs, memory)?
- [ ] Is my time estimate reasonable (< 16 hours)?
- [ ] Am I using the interactive partition (`-p interactive`)?
- [ ] Am I setting the job name correctly (`-J interactive`)?

## Common Issues

### Session Disconnected

If your SSH connection drops:

- Your interactive job continues running (for now)
- Use `screen` or `tmux` to maintain sessions across disconnects
- Start a new SSH connection and reattach with `screen -r`

### Can't Start Interactive Session (Pending)

If `srun` doesn't start immediately:

```bash
$ squeue -u $USER
```

Common reasons for pending:

- **Resources unavailable** - Try requesting fewer resources
- **Partition full** - Wait for other jobs to complete
- **High priority jobs** - Batch jobs may have priority

Consider working on the login node for non-computational tasks while waiting.

### Out of Memory

If your interactive session runs out of memory:

- The process or session may be killed
- Restart with more memory: `--mem=<larger_size>`
- Check actual usage with `top` or `free -h`

### Session Killed

If your session is terminated unexpectedly:

- Check if you exceeded the time limit
- Verify you're using the interactive partition
- Check if you exceeded memory limits
- Look for system maintenance notifications

## Examples by Use Case

### Quick Script Testing (Minimal Resources)

```bash
$ srun -J interactive -p interactive --pty /bin/bash -l
$ python myscript.py
$ exit
```

### Data Analysis (More Memory)

```bash
$ srun --mem=32G --cpus-per-task=4 -J interactive -p interactive --pty /bin/bash -l
$ eval $(spack load --sh r)
$ R
# ... do analysis ...
$ exit
```

### Parallel Code Development

```bash
$ srun --cpus-per-task=16 --mem=64G -J interactive -p interactive --pty /bin/bash -l
$ eval $(spack load --sh parallel-tool)
$ # Test parallel code
$ exit
```

### GPU Development

```bash
$ srun -p gpu --gpus=1 --mem=16G -J interactive --pty /bin/bash -l
$ nvidia-smi
$ eval $(spack load --sh cuda py-pytorch)
$ python
>>> import torch
>>> torch.cuda.is_available()
$ exit
```

## Transitioning to Batch Jobs

Once your code is working in an interactive session, convert it to a batch job:

**Interactive commands:**
```bash
$ eval $(spack load --sh program)
$ program input.txt output.txt
```

**Batch script (myjob.sh):**
```bash
#!/bin/bash

#SBATCH --job-name=production
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --output=job_%j.out

eval $(spack load --sh program)

cd /scratch/mylab/$USER/project/
program input.txt output.txt
```

**Submit:**
```bash
$ sbatch myjob.sh
```

## Next Steps

- [Batch Jobs](batch.md) - Submit production jobs
- [Job Accounting](accounting.md) - Analyze resource usage
- [Jupyter Lab Guide](../software/jupyter/index.md) - Interactive notebooks
- [RStudio Guide](../software/rstudio/index.md) - Interactive R development
- [GPU Computing](../gpu/index.md) - Using GPUs interactively

--8<-- "includes/getting-help.md"
