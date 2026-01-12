# Before You Begin

Before using the HTCF, familiarity with the following concepts and technologies will be helpful for effective use of the cluster.

## Required Skills

### Linux/Unix Command Line

A firm grasp of the Bash shell is expected for all HTCF users.

**Essential commands:**

- Navigation: `cd`, `ls`, `pwd`
- File operations: `cp`, `mv`, `rm`, `mkdir`, `touch`
- Viewing files: `cat`, `less`, `head`, `tail`, `grep`
- Permissions: `chmod`, `chown`, `ls -l`
- Disk usage: `df`, `du`
- Process management: `ps`, `top`

**Resources:**

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/html_node/index.html)
- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)

### Environment Variables

Understanding how to set and use environment variables is crucial for configuring software on the cluster.

**Key concepts:**

- `export VAR=value` - Set an environment variable
- `echo $VAR` - Display a variable's value
- `$PATH` - Where the shell looks for executables
- `$HOME` - The home directory path
- `.bashrc` - Shell configuration file

**Resources:**

- [Bash Environment Variables](https://linuxhint.com/bash-environment-variables/)
- [Environment Variables Guide](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-linux)

### SSH (Secure Shell)

SSH is used to connect to the HTCF cluster.

**Key concepts:**

- Basic SSH connection: `ssh username@hostname`
- SSH config file for easier connections
- Port forwarding for Jupyter/RStudio

**Resources:**

- [SSH Tutorial](https://www.ssh.com/academy/ssh/command)

## Recommended Skills

### Text Editors

Editing scripts and configuration files on the command line requires a text editor. Options include:

**Official Documentation:**

- [Vim Official Documentation](https://www.vim.org/docs.php) - Complete Vim reference
  - [OpenVim Tutorial](https://www.openvim.com/) - Interactive vim tutorial
  - [Vim Cheat Sheet](https://vim.rtorr.com/) - Quick reference
- [GNU Nano Manual](https://www.nano-editor.org/docs.php) - Beginner-friendly editor
- [GNU Emacs Manual](https://www.gnu.org/software/emacs/manual/) - Feature-rich editor

### File Transfer

Transferring files between local machines and the cluster can be done with various tools.

**Official Documentation:**

- [rsync Manual](https://linux.die.net/man/1/rsync) - Handles large files and resumable transfers
  - [rsync Tutorial](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories) - Complete guide
- [scp Manual](https://linux.die.net/man/1/scp) - Simple secure file copying
- [Globus Documentation](https://docs.globus.org/) - Option for very large datasets
  - See [Storage documentation](storage/index.md#sharing-files-publicly) for HTCF-specific setup

### Version Control (Git)

For version control of code and scripts, Git is one option.

**Official Documentation:**

- [Git Official Documentation](https://git-scm.com/doc) - Complete reference and tutorials
- [Pro Git Book (free)](https://git-scm.com/book/en/v2) - Comprehensive guide from basics to advanced
- [GitHub Docs](https://docs.github.com/en/get-started) - GitHub-specific features and workflows
- [Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/) - Quick reference

## Programming & Scripting

### Bash Scripting

Write scripts to automate job submission and data processing.

**Official Documentation:**

- [Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html) - Complete bash documentation
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/) - In-depth tutorial with examples
- [Bash Scripting Tutorial](https://linuxconfig.org/bash-scripting-tutorial) - Beginner-friendly guide
- [ShellCheck](https://www.shellcheck.net/) - Online tool to check your bash scripts for errors

### Python

Manage Python packages without affecting the system using virtual environments.

**Official Documentation:**

- [Python Official Documentation](https://docs.python.org/3/) - Complete Python reference
- [Python venv Tutorial](https://docs.python.org/3/tutorial/venv.html) - Virtual environment guide
- [Python Package Installation Guide](https://packaging.python.org/en/latest/tutorials/installing-packages/) - pip and package management
- [Python for Scientific Computing](https://scipy-lectures.org/) - NumPy, SciPy, Matplotlib tutorials

### R

For bioinformatics and statistical computing users.

**Official Documentation:**

- [R Official Documentation](https://www.r-project.org/) - R project homepage and manuals
- [R Manuals](https://cran.r-project.org/manuals.html) - Official R manuals and guides
- [CRAN Task Views](https://cran.r-project.org/web/views/) - Curated lists of packages by domain
- [Bioconductor](https://www.bioconductor.org/) - Bioinformatics packages for R
- [R for Data Science (free book)](https://r4ds.had.co.nz/) - Modern R programming guide

See [R Guide](software/r/index.md) for HTCF-specific details on using R on the cluster.

## HPC Concepts

### Job Schedulers (Slurm)

The HTCF uses Slurm to manage job submission and resource allocation.

**Key concepts:**

- **Batch jobs**: Submit scripts to run without interaction. You write a shell script containing your commands and resource requirements, then submit it with `sbatch`. The job runs when resources become available, and output is written to files you can review later. This is the primary way to run computational work on the cluster.

- **Interactive jobs**: Request resources for interactive work. Use `srun` to get a shell on a compute node where you can run commands directly, useful for testing, debugging, or exploratory analysis. Interactive sessions have time limits, so they're best for short-term work rather than long computations.

- **Job arrays**: Run the same script with different inputs. When you need to process many files or run the same analysis with varying parameters, job arrays let you submit hundreds of similar jobs with a single command. Each array task gets a unique index (`$SLURM_ARRAY_TASK_ID`) that you use to select different input files or parameters.

- **Resources**: CPU cores, memory, time limits, GPUs. Every job must specify what resources it needs. Request only what you'll actually use—overestimating wastes resources and may increase your wait time in the queue. Underestimating can cause your job to be killed if it exceeds memory limits or runs past its time allocation.

- **Partitions**: Different queues (default, interactive, gpu). Partitions group nodes with similar characteristics or purposes. The `default` partition handles most batch work, `interactive` is optimized for quick-turnaround interactive sessions, and `gpu` provides access to GPU-equipped nodes. Choose the partition that matches your job's requirements.

**Official Documentation:**

- [Slurm Official Documentation](https://slurm.schedmd.com/) - Complete Slurm reference
- [Slurm Quick Start Guide](https://slurm.schedmd.com/quickstart.html) - Introduction to Slurm
- [sbatch Documentation](https://slurm.schedmd.com/sbatch.html) - Batch job submission
- [srun Documentation](https://slurm.schedmd.com/srun.html) - Interactive jobs and parallel execution
- [sacct Documentation](https://slurm.schedmd.com/sacct.html) - Job accounting information

See [Getting Started](getting-started.md) for HTCF-specific Slurm usage and examples.

### Parallel Computing

Understanding parallelization helps with efficient use of cluster resources.

**Types of parallelism:**

- **Embarrassingly parallel**: Independent tasks (array jobs)
- **Shared memory**: Multi-threaded (OpenMP)
- **Distributed memory**: Multi-node (MPI)
- **GPU**: Graphics processing units

**Official Documentation:**

- [OpenMP Official Site](https://www.openmp.org/) - Shared memory parallel programming
- [MPI Forum](https://www.mpi-forum.org/) - Message Passing Interface standard
- [OpenMPI Documentation](https://www.open-mpi.org/doc/) - Popular MPI implementation
- [CUDA Documentation](https://docs.nvidia.com/cuda/) - NVIDIA GPU programming

### Storage Hierarchy

Know where to put your data:

- **HDS** (`/home`): Small, backed up - scripts and config (20GB)
- **LTS** (`/lts`): Large, backed up - raw and finished data
- **Scratch** (`/scratch`): Fast, temporary - active processing (2TB/user)
- **REF** (`/ref`): Software and reference databases

See [Storage Overview](storage/index.md) for details.

## Getting Help

### Documentation Resources

- [HTCF Getting Started](getting-started.md)
- [Quick Start Tutorials](tutorials/index.md)
- [Troubleshooting](troubleshooting/index.md)

### Support

For assistance:

1. Check the [FAQ](troubleshooting/faq.md)
2. Review relevant troubleshooting guides
3. Post in **#general** on Slack with:
   - Description of the task being attempted
   - Command that was run
   - Error messages (complete output)
   - Job ID (if applicable)

## Self-Assessment

Before proceeding, ensure you can:

- [ ] Connect to a remote server via SSH
- [ ] Navigate directories and list files
- [ ] Create, edit, and save text files
- [ ] Copy files between your computer and a remote server
- [ ] Understand file permissions (read/write/execute)
- [ ] Set and use environment variables
- [ ] Write basic shell scripts

If you're comfortable with these tasks, you're ready to start using the HTCF!

## Next Steps

Ready to begin? Start with:

1. [Getting Started Guide](getting-started.md) - Connect and run your first job
2. [Your First Job Tutorial](tutorials/first-job.md) - Step-by-step walkthrough
3. [Software Installation](software/index.md) - Install the tools you need
