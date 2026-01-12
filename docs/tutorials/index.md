# Quick Start Tutorials

These step-by-step tutorials will help you get started with common HTCF tasks. Each tutorial is designed to be completed in 15-30 minutes and includes working examples you can run immediately.

## Beginner Tutorials

### [Your First Batch Job](first-job.md)
Complete walkthrough from login to results. Learn to create, submit, and monitor your first batch job on the HTCF.

**Time:** 15 minutes | **Level:** Beginner
**You'll learn:** Job scripts, sbatch, squeue, output files, sacct

### [Your First GPU Job](first-gpu-job.md)
Introduction to GPU computing on the HTCF. Request a GPU, verify access, and run a simple GPU program.

**Time:** 20 minutes | **Level:** Beginner
**Prerequisites:** Complete "Your First Batch Job" first
**You'll learn:** GPU requests, CUDA setup, nvidia-smi, GPU verification

### [Complete Data Workflow](data-workflow.md)
Learn the standard data lifecycle on HTCF: LTS → Scratch → Processing → LTS. Essential for all HTCF users.

**Time:** 25 minutes | **Level:** Beginner
**You'll learn:** Storage systems, rsync, scratch usage, data management

## Intermediate Tutorials

### [Running Array Jobs](array-jobs.md)
Process multiple samples in parallel using array jobs. Dramatically reduce total runtime for batch processing.

**Time:** 25 minutes | **Level:** Intermediate
**Prerequisites:** Complete "Your First Batch Job" first
**You'll learn:** Array job syntax, SLURM_ARRAY_TASK_ID, parallel processing, lookup files

## What You'll Need

Before starting these tutorials, ensure you have:

- **HTCF account** and login credentials
- **SSH access** to login.htcf.wustl.edu
- **Basic Linux knowledge** (cd, ls, mkdir, editing files)
- **Text editor** (nano, vim, or similar)

## Tutorial Format

Each tutorial follows a consistent format:

1. **Learning objectives** - What you'll accomplish
2. **Prerequisites** - What you need to know
3. **Step-by-step instructions** - Detailed walkthrough
4. **Complete examples** - Copy-paste ready code
5. **Expected output** - What success looks like
6. **Troubleshooting** - Common issues and solutions
7. **Next steps** - Where to go from here

## Getting Help

If you get stuck:

1. Check the **Troubleshooting** section in each tutorial
2. Review the [FAQ](../troubleshooting/faq.md)
3. See the [Troubleshooting Guide](../troubleshooting/index.md)
4. Post in **#general** on Slack with your job ID and error messages

## After the Tutorials

Once you've completed these tutorials, explore:

- [Batch Jobs Guide](../using/batch.md) - Comprehensive batch job documentation
- [GPU Computing](../gpu/index.md) - In-depth GPU usage
- [Best Practices](../best-practices/index.md) - Optimize your workflows
- [Advanced Workflows](../workflows/index.md) - Domain-specific examples

--8<-- "includes/getting-help.md"
