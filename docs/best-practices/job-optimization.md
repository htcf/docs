# Job Optimization

## Parallel Strategies

### Array Jobs
Process multiple samples simultaneously:
```bash
#SBATCH --array=1-100%20    # 100 tasks, 20 at a time
```

### Multi-threading
Use CPU cores effectively:
```bash
#SBATCH --cpus-per-task=16
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
```

## I/O Optimization

**Use Scratch**

Jobs can read directly from LTS. For heavy I/O, copy to scratch first (from login node):
```bash
# On login node: stage data to scratch
rsync -av /lts/data/ /scratch/mylab/$USER/data/
sbatch process_job.sh
```

In your job script, read from scratch and write results to scratch. After job completes, copy results back from login node:
```bash
# On login node: copy results back
rsync -av /scratch/mylab/$USER/results/ /lts/results/
```

**Reduce File I/O**
- Process in memory when possible
- Write results once at end
- Archive many small files

## Memory Efficiency

**Process in Chunks**
```python
# Instead of loading all data
for chunk in pd.read_csv('data.csv', chunksize=10000):
    process(chunk)
```

**Clear Memory**
```python
import gc
gc.collect()
```

## Checkpoint Long Jobs

```bash
#!/bin/bash
if [ -f checkpoint.dat ]; then
    program --restart checkpoint.dat
else
    program --new
fi
```

## See Also

- [Array Jobs Tutorial](../tutorials/array-jobs.md) - Detailed guide for parallel processing
- [Data Workflow Tutorial](../tutorials/data-workflow.md) - Optimizing I/O with LTS and Scratch
- [Job Accounting](../using/accounting.md) - Measuring and analyzing job performance

--8<-- "includes/getting-help.md"
