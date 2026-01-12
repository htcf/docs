# Tutorial: Debugging Jobs

## Learning Objectives

Learn to debug jobs that fail or behave unexpectedly:

- Test code interactively before batch submission
- Interpret job output and error files
- Use Slurm tools to diagnose problems
- Scale from test runs to production

**Time Required:** 20 minutes
**Level:** Beginner
**Prerequisites:** [Your First Batch Job](first-job.md)

## The Development Workflow

Most job problems can be avoided by testing interactively first:

```
1. Interactive Test    →    2. Small Batch Test    →    3. Production Run
   (1 sample, minutes)        (few samples)              (all samples)
```

## Step 1: Start an Interactive Session

Before submitting a batch job, test your commands interactively:

```bash
$ srun --mem=4G --cpus-per-task=2 -J interactive -p interactive --pty /bin/bash -l
```

Now you're on a compute node where you can run commands and see immediate feedback.

## Step 2: Test Your Commands

Run the same commands you plan to put in your batch script:

```bash
# Load software
$ eval $(spack load --sh my-program)

# Verify it's available
$ which my-program
/ref/mylab/software/spack/.../bin/my-program

# Test with one file
$ my-program /lts/mylab/data/sample1.txt output/

# Check output
$ ls output/
sample1_results.txt

# Verify results look correct
$ head output/sample1_results.txt
```

!!! tip "Test Small First"
    Use a single sample or a subset of data. If it works for one, it will likely work for all.

## Step 3: Create Your Batch Script

Once commands work interactively, put them in a script:

```bash
#!/bin/bash

#SBATCH --job-name=my_analysis
#SBATCH --cpus-per-task=2
#SBATCH --mem=4G
#SBATCH --time=1:00:00
#SBATCH --output=logs/job_%j.out
#SBATCH --error=logs/job_%j.err

# Create log directory if needed
mkdir -p logs

eval $(spack load --sh my-program)

cd /scratch/mylab/$USER/project/

my-program /lts/mylab/data/sample1.txt output/

echo "Job completed successfully"
```

## Step 4: Submit a Test Job

Submit with a small test case:

```bash
$ mkdir -p logs
$ sbatch my_job.sh
Submitted batch job 123456
```

Monitor it:

```bash
$ squeue -u $USER
```

## Step 5: Check Job Output

After the job completes, check the output:

```bash
# View standard output
$ cat logs/job_123456.out

# View errors (if any)
$ cat logs/job_123456.err

# Check job status
$ sacct -j 123456
```

## Common Problems and Solutions

### Problem: "Command not found"

**Symptoms:**
```
bash: my-program: command not found
```

**Cause:** Software not loaded in batch script.

**Solution:** Add `eval $(spack load --sh <package>)` before using the command.

**Debug:** In interactive session, verify `which my-program` returns a path.

---

### Problem: Job Killed (Exit Code 137)

**Symptoms:**
```
slurmstepd: error: Detected 1 oom-kill event
```

**Cause:** Job exceeded memory limit.

**Solution:** Increase memory request:
```bash
#SBATCH --mem=16G    # Increase from 4G
```

**Debug:** Check actual usage with `seff <JOBID>` after a successful smaller run.

---

### Problem: Job Times Out

**Symptoms:**
```
TIMEOUT
```

**Cause:** Job exceeded time limit.

**Solution:** Increase time or optimize code:
```bash
#SBATCH --time=4:00:00    # Increase from 1:00:00
```

**Debug:** Test with subset to estimate full runtime.

---

### Problem: Files Not Found

**Symptoms:**
```
Error: Cannot open file '/path/to/file.txt'
```

**Cause:** Wrong path or file doesn't exist.

**Solution:**
1. Use absolute paths in scripts
2. Verify paths exist before job runs:
```bash
$ ls /lts/mylab/data/sample1.txt
```

**Debug:** Add `pwd` and `ls` commands to your script to see working directory.

---

### Problem: Job Succeeds But No Output

**Symptoms:** Job shows COMPLETED but output files are missing.

**Causes:**
1. Output written to unexpected location
2. Output redirected incorrectly
3. Program silently failed

**Debug Checklist:**
```bash
# Where did the job run?
$ grep "Working directory" logs/job_123456.out

# Check scratch for output
$ ls /scratch/mylab/$USER/

# Check if program wrote anywhere
$ find /scratch/mylab/$USER -name "*.txt" -mmin -60
```

**Solution:** Always print working directory and output location in your script:
```bash
echo "Working directory: $(pwd)"
echo "Output will be in: $PWD/output/"
```

---

### Problem: Permission Denied

**Symptoms:**
```
Permission denied
```

**Cause:** Trying to write to read-only location (like LTS from compute node).

**Solution:** Write to `/scratch`, copy to LTS after job completes from login node.

---

### Problem: Array Job Partial Failure

**Symptoms:** Some array tasks complete, others fail.

**Debug:**
```bash
# Check which tasks failed
$ sacct -j 123456
```

Look for tasks with non-zero exit codes.

**Solution:** Check logs for specific failed tasks:
```bash
$ cat logs/job_123456_15.err   # Check task 15
```

## Debugging Commands Reference

```bash
# Job status
$ sacct -j <JOBID>

# Job efficiency (memory, CPU usage)
$ seff <JOBID>

# Detailed job info (while running)
$ scontrol show jobid -dd <JOBID>

# View output in real-time
$ tail -f logs/job_<JOBID>.out

# Cancel a stuck job
$ scancel <JOBID>
```

## Adding Debug Output to Scripts

For troublesome jobs, add debugging output:

```bash
#!/bin/bash
#SBATCH --job-name=debug_test
#SBATCH --output=logs/%j.out
#SBATCH --error=logs/%j.err

set -x  # Print each command before running (verbose mode)

echo "=== Debug Info ==="
echo "Job ID: $SLURM_JOB_ID"
echo "Node: $(hostname)"
echo "Working dir: $(pwd)"
echo "Date: $(date)"
echo "=================="

# Your commands here
eval $(spack load --sh my-program)
echo "Loaded software, path is: $(which my-program)"

my-program input.txt output.txt
EXIT_CODE=$?

echo "Program exit code: $EXIT_CODE"
echo "Output files:"
ls -la output.txt

exit $EXIT_CODE
```

## Scaling Up Safely

Once your test job works:

### From 1 Sample to Many

```bash
# Test: 1 sample
my-program sample1.txt output1.txt

# Scale: array job for all samples
#SBATCH --array=1-100%20    # 100 samples, 20 at a time
```

### Estimating Resources

Use test job results to size production jobs:

```bash
# Check test job usage
$ seff 123456
Job ID: 123456
CPU Utilized: 00:05:23
Memory Utilized: 2.1 GB
```

For 100 samples:
- Memory: Same per task (2.1 GB → request 4 GB with buffer)
- Time: 5 min × 100 = 500 min, but with array parallelism much faster

## Next Steps

- [Array Jobs](array-jobs.md) - Process multiple samples
- [Job Errors Troubleshooting](../troubleshooting/job-errors.md) - Detailed error reference
- [Resource Estimation](../best-practices/resource-estimation.md) - Right-size your jobs

--8<-- "includes/getting-help.md"
