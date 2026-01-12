# Troubleshooting Guide

Quick reference for common HTCF issues and solutions.

## Quick Problem Finder

### Job Issues

| Problem | Quick Solution | Details |
|---------|---------------|---------|
| [Job won't start (pending)](job-errors.md#job-wont-start) | Check resources, partition | [Job Errors](job-errors.md) |
| [Job fails immediately](job-errors.md#job-fails-immediately) | Check script, paths | [Job Errors](job-errors.md) |
| [Out of memory](job-errors.md#out-of-memory) | Increase `--mem=` | [Job Errors](job-errors.md) |
| [Time limit exceeded](job-errors.md#time-limit-exceeded) | Increase `--time=` | [Job Errors](job-errors.md) |

### Storage Issues

| Problem | Quick Solution | Details |
|---------|---------------|---------|
| [Disk quota exceeded](storage-errors.md#disk-quota-exceeded) | Clean up scratch | [Storage Errors](storage-errors.md) |
| [Permission denied](storage-errors.md#permission-denied) | Check file permissions | [Storage Errors](storage-errors.md) |
| [Slow I/O](storage-errors.md#slow-io) | Use scratch, not LTS | [Storage Errors](storage-errors.md) |

### Software Issues

| Problem | Quick Solution | Details |
|---------|---------------|---------|
| [Module not found](software-errors.md#module-not-found) | Check Spack | [Software Errors](software-errors.md) |
| [Library errors](software-errors.md#library-errors) | Load dependencies | [Software Errors](software-errors.md) |
| [Command not found](software-errors.md#command-not-found) | Load module first | [Software Errors](software-errors.md) |

### GPU Issues

| Problem | Quick Solution | Details |
|---------|---------------|---------|
| GPU not detected | Check `--gpus=1`, `-p gpu` | [GPU Monitoring](../gpu/monitoring.md) |
| Low GPU utilization | Data loading bottleneck | [GPU Monitoring](../gpu/monitoring.md) |
| CUDA out of memory | Reduce batch size | [GPU Monitoring](../gpu/monitoring.md) |

## Common Error Messages

| Error Message | Meaning | Solution |
|---------------|---------|----------|
| `sbatch: error: Batch job submission failed: Invalid account` | Wrong partition/account | Use `-p general` or `-p gpu` |
| `srun: error: Unable to allocate resources: Invalid account` | Account issue | Check account with IT |
| `CANCELLED AT <time> DUE TO TIME LIMIT` | Job exceeded time | Increase `--time=` |
| `OUT_OF_MEMORY` | Job used more memory than requested | Increase `--mem=` |
| `Disk quota exceeded` | Storage quota full | Clean up files |
| `Permission denied` | File permission issue | Check `chmod` and ownership |
| `Command not found` | Software not loaded | Load with Spack |
| `libcuda.so: cannot open shared object file` | CUDA not loaded | `eval $(spack load --sh cuda)` |

## Troubleshooting Workflow

1. **Check job status**: `squeue -u $USER` or `sacct -j <JOBID>`
2. **View output files**: `cat slurm-<JOBID>.out`
3. **Check job details**: `scontrol show jobid -dd <JOBID>`
4. **Search this guide**: Use browser search (Ctrl+F)
5. **Check FAQ**: [Frequently Asked Questions](faq.md)
6. **Ask on Slack**: Post in **#general** with job ID and error messages

## Detailed Guides

- [Job Errors](job-errors.md) - Job submission, execution, and completion issues
- [Storage Errors](storage-errors.md) - Disk space, permissions, and I/O problems
- [Software Errors](software-errors.md) - Package loading, compilation, and dependencies
- [Performance Issues](performance.md) - Slow jobs, optimization, and bottlenecks
- [FAQ](faq.md) - Frequently asked questions

--8<-- "includes/getting-help.md"
