# Performance Issues

## Quick Diagnostics

Use `seff` for an instant efficiency overview:

```bash
$ seff <JOBID>
```

This immediately shows CPU and memory efficiency percentages. See [Job Efficiency Analysis](../using/accounting.md#job-efficiency-analysis-with-seff) for interpretation.

For multiple jobs, use `reportseff` to spot patterns:

```bash
$ reportseff --since yesterday
```

See [Analyzing Multiple Jobs](../using/accounting.md#analyzing-multiple-jobs-with-reportseff) for detailed usage.

## Job Running Slowly

### Check CPU Utilization
```bash
$ seff <JOBID>  # Quick summary
$ sacct -j <JOBID>
```

### Common Causes
- Not using requested CPUs (code not parallel)
- I/O bottleneck
- CPU-bound vs I/O-bound workload

### Solutions
- Use scratch for data
- Optimize code
- Request fewer CPUs if not parallelizing

## Low CPU Utilization

### Check
```bash
$ seff <JOBID>  # Shows CPU efficiency percentage
$ sacct -j <JOBID>
```

### If <50% efficiency:
- Code doesn't parallelize well
- Request fewer CPUs
- See [Resource Estimation](../best-practices/resource-estimation.md) for optimization tips

## Memory Issues

### Check Usage
```bash
$ seff <JOBID>  # Shows memory efficiency percentage
$ sacct -j <JOBID>
```

### If much less than requested (<50% efficiency):
- Reduce memory request
- Smaller jobs start faster

### If very close to limit (>95% efficiency):
- Increase memory to prevent OOM errors
- Add 20% safety margin

## I/O Bottlenecks

### Symptoms
- Job slow despite low CPU use
- Many small file operations

### Solutions
- Use scratch, not LTS
- Reduce file I/O operations
- Archive small files

--8<-- "includes/getting-help.md"
