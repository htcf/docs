# Frequently Asked Questions

## Account and Access

**Q: How do I get an HTCF account?**
A: Contact your lab PI or post in **#general** on Slack with your WashU Key username and department ID.

**Q: I can't log in to the cluster**
A: Check: (1) WashU Key password correct, (2) Connected to WashU Research Network (WURN) if off-campus, (3) SSH to `login.htcf.wustl.edu`

**Q: Can I share my account with lab members?**
A: No. Each person must have their own account per [HTCF policies](../policies.md).

## Jobs and Queuing

**Q: Why is my job pending (PD)?**
A: Check reason with: `squeue -u $USER -o "%.18i %.9P %.8j %.8u %.2t %.10M %.6D %.20R"`

Common reasons:
- **Resources**: No nodes available with the requested resources
- **Priority**: Other jobs have higher priority

**Q: How long will my job wait in the queue?**
A: Depends on resources requested and cluster usage. Jobs with smaller resource requests typically start faster.

**Q: Can I increase my job priority?**
A: No. Priority is determined by fair-share algorithm based on recent usage.

**Q: My job was killed. Why?**
A: Check `sacct -j <JOBID>`. Common reasons:
- Time limit exceeded (`TIMEOUT`)
- Out of memory (`OUT_OF_MEMORY`)
- Node failure (`NODE_FAIL`)
- User cancelled (`CANCELLED`)

**Q: How do I cancel a job?**
A: `scancel <JOBID>` or `scancel -u $USER` for all your jobs

## Storage

**Q: How much storage do I have?**
A: Scratch: 2TB default. LTS: Lab-specific quota. Check with `storage-info`

**Q: Where should I put my data?**
A: Raw data and results → LTS. Active processing → Scratch. See [Data Workflow](../tutorials/data-workflow.md).

**Q: My home directory is full**
A: Home directories have small quotas. Use `/scratch/<lab>/$USER` or `/lts/<lab>/` for data.

**Q: How do I increase my scratch quota?**
A: Post a request in **#general** on Slack. Default is 2TB.

**Q: Files disappeared from scratch!**
A: Scratch files inactive for 60+ days are automatically removed. Always copy important data to LTS.

## Software

**Q: How do I install software?**
A: Use Spack: `spack install <package>`. See [Software Guide](../software/index.md).

**Q: Can you install software for me?**
A: HTCF provides self-service installation via Spack. Post in **#general** on Slack only if software unavailable.

**Q: Should I use Conda or Spack?**
A: Both options are available. Spack is designed for HPC environments. Conda works but may have performance issues on HPC systems.

**Q: My Python package won't install**
A: Try: (1) Update pip, (2) Use virtual environment, (3) Check dependencies, (4) Use Spack version

## GPUs

**Q: How do I request a GPU?**
A: Add to job script:
```bash
#SBATCH -p gpu
#SBATCH --gpus=1
```
See [GPU Overview](../gpu/index.md).

**Q: Which GPU should I use (A100 vs V100)?**
A: A100 (80GB) for large models/datasets. V100 (32GB) for smaller workloads. Avoid specifying constraint unless a specific GPU type is required.

**Q: Can I use multiple GPUs?**
A: Yes, but the code must support multi-GPU (e.g., PyTorch DataParallel). Request with `--gpus=N`.

**Q: Why is my GPU utilization low?**
A: Usually CPU/data loading bottleneck. Increase `num_workers` in data loader or use faster storage.

## Best Practices

**Q: How much memory should I request?**
A: Start with estimate + 20% buffer. Use `sacct` to check actual usage and adjust. See [Resource Estimation](../best-practices/resource-estimation.md).

**Q: How many CPUs should I request?**
A: Only request what your code can use. Many programs don't parallelize. Test with small job first.

**Q: How do I make my jobs run faster?**
A: (1) Optimize code, (2) Use scratch storage, (3) Use array jobs for parallelism, (4) Right-size resources. See [Job Optimization](../best-practices/job-optimization.md).

**Q: Should I use interactive or batch jobs?**
A: Interactive for development/testing. Batch for production. See [Interactive vs Batch](../using/interactive.md).

## Getting Help

**Q: Who do I contact for help?**
A: HTCF support via your institution's support channels. Include job ID and error messages.

**Q: What information should I provide when asking for help?**
A:
- Job ID: `squeue -u $USER`
- Error messages from output files
- Job details: `scontrol show jobid -dd <JOBID>`
- What you've already tried

--8<-- "includes/getting-help.md"
