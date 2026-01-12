# Resource Estimation

## Why It Matters

- **Faster start times** - Smaller requests start sooner
- **Better efficiency** - Cluster runs more jobs
- **Fair share** - Don't monopolize resources

## Quick Efficiency Check

Use `seff` for a quick, human-readable efficiency summary:

```bash
$ seff <JOBID>
```

This shows CPU and memory efficiency percentages at a glance. See [Job Accounting](../using/accounting.md#job-efficiency-analysis-with-seff) for detailed usage.

For multiple jobs, use `reportseff`:

```bash
$ reportseff --since yesterday
```

See [Analyzing Multiple Jobs](../using/accounting.md#analyzing-multiple-jobs-with-reportseff) for details.

## Iterative Refinement

1. **Overestimate initially**
```bash
#SBATCH --cpus-per-task=8
#SBATCH --mem=64G
#SBATCH --time=24:00:00
```

2. **Run and analyze**
```bash
$ sacct -j <JOBID>
```

3. **Adjust for next job**
- MaxRSS: 12GB → Request 16GB (+ 33% buffer)
- Elapsed: 2 hours → Request 3 hours
- Low CPU usage → Reduce CPUs

## Memory Estimation

```bash
# Check actual usage (look at MaxRSS column)
$ sacct -j <JOBID>

# Add 10-20% buffer
Actual: 12GB → Request: 16GB
```

## CPU Estimation

```bash
# Check efficiency with seff
$ seff <JOBID>

# If <70% efficiency, reduce CPUs
```

## Time Estimation

Test with subset, extrapolate:
```
Small test: 100 samples in 1 hour
Full job: 1000 samples → 10 hours + buffer = 12 hours
```

--8<-- "includes/getting-help.md"
