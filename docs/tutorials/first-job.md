# Tutorial: Your First Batch Job

## Learning Objectives

By the end of this tutorial, you will:

- Create a simple batch job script
- Submit a job to the HTCF cluster
- Monitor job execution
- View and interpret results
- Analyze job statistics with sacct

**Time Required:** 15 minutes
**Prerequisites:** HTCF account, SSH access, basic Linux commands

## Step 1: Log In to HTCF

Open your terminal and SSH to the HTCF login server:

```bash
$ ssh <username>@login.htcf.wustl.edu
```

Enter your WashU Key password when prompted.

## Step 2: Navigate to Scratch

All computational work should be done in scratch storage:

```bash
$ cd /scratch/<lab>/$USER
$ mkdir -p first-job-tutorial
$ cd first-job-tutorial
$ pwd
```

Expected output: `/scratch/<lab>/<username>/first-job-tutorial`

## Step 3: Create a Job Script

Create a file called `hello_job.sh`:

```bash
$ nano hello_job.sh
```

Copy and paste this content:

```bash
#!/bin/bash

#SBATCH --job-name=hello_job
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=5:00
#SBATCH --output=hello_job_%j.out

# Print job information
echo "Job started at: $(date)"
echo "Job ID: $SLURM_JOB_ID"
echo "Running on host: $(hostname)"
echo "Running in directory: $(pwd)"
echo ""

# Do some work
echo "Hello from the HTCF cluster!"
echo "This is my first batch job."
echo ""

# Simulate some computation
echo "Simulating work (sleeping for 10 seconds)..."
sleep 10

# Finish
echo "Job completed at: $(date)"
```

Save and exit (in nano: `Ctrl+O`, `Enter`, `Ctrl+X`).

Make the script executable:

```bash
$ chmod +x hello_job.sh
```

## Step 4: Submit the Job

Submit your job to the cluster:

```bash
$ sbatch hello_job.sh
```

Expected output:

```
Submitted batch job 123456
```

The number is the **job ID** - save this!

## Step 5: Monitor Your Job

Check the status of your job:

```bash
$ squeue -u $USER
```

Expected output:

```
JOBID PARTITION     NAME     USER ST       TIME  NODES
123456   general hello_jo  yourname  R       0:05      1
```

Job states:
- **PD** (Pending) - Waiting for resources
- **R** (Running) - Currently executing
- **CG** (Completing) - Finishing up

Wait a few seconds and check again. When the job is done, it will disappear from `squeue`.

## Step 6: Check Results

List the files in the directory:

```bash
$ ls -lh
```

Expected files:
- `hello_job.sh` - The job script
- `hello_job_123456.out` - Output file (with the job ID)

View the output:

```bash
$ cat hello_job_123456.out
```

Expected output:

```
Job started at: Tue Jan  7 14:23:45 CST 2026
Job ID: 123456
Running on host: n042
Running in directory: /scratch/<lab>/<username>/first-job-tutorial

Hello from the HTCF cluster!
This is my first batch job.

Simulating work (sleeping for 10 seconds)...
Job completed at: Tue Jan  7 14:23:55 CST 2026
```

## Step 7: Analyze Job Statistics

View job statistics:

```bash
$ sacct -j 123456
```

Output:

```
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
123456       hello_job    general   mylab              1  COMPLETED      0:0
```

This shows job state (COMPLETED), allocated CPUs, and exit code (0:0 = success).

## Congratulations!

Successfully completed:
- ✓ Created a batch job script
- ✓ Submitted it to the cluster with sbatch
- ✓ Monitored its execution with squeue
- ✓ Retrieved and viewed output
- ✓ Analyzed job statistics with sacct

## What You Learned

**SBATCH Directives:**
- `--job-name`: Give your job a name
- `--cpus-per-task`: Request CPU cores
- `--mem`: Request memory
- `--time`: Set maximum runtime
- `--output`: Specify output file location

**Commands:**
- `sbatch`: Submit batch jobs
- `squeue`: Monitor running jobs
- `sacct`: View completed job history

**Environment Variables:**
- `$SLURM_JOB_ID`: Your job's unique ID
- `$USER`: Your username

## Troubleshooting

### Job Stays Pending

If your job stays in PD (pending) state:

```bash
$ squeue -u $USER -o "%.10i %.9P %.20j %.8u %.8T %.10M %.10l %.6D %.20R"
```

Look at the REASON column. Common reasons:
- **Resources**: No nodes available with requested resources
- **Priority**: Other jobs have higher priority

Try reducing resources or waiting.

### Job Fails Immediately

Check the output file for errors:

```bash
$ cat hello_job_*.out
```

Common issues:
- Missing `#!/bin/bash` shebang line
- Syntax errors in script
- Incorrect file paths

### No Output File

If no output file appears:

```bash
$ squeue -u $USER  # Check if still running
$ sacct -j 123456  # Check job history
```

The output file only appears after the job starts running.

## Next Steps

Now that you've submitted your first job, try:

1. **Modify the script** - Change the sleep time or add more commands
2. **Request more resources** - Try `--cpus-per-task=4` and `--mem=8G`
3. **Run real analysis** - Replace the echo commands with actual work
4. **Try GPU computing** - Continue to [Your First GPU Job](first-gpu-job.md)
5. **Process multiple files** - Learn [Array Jobs](array-jobs.md)

### Recommended Reading

- [Batch Jobs Guide](../using/batch.md) - Complete batch job documentation
- [Job Accounting](../using/accounting.md) - Detailed sacct usage
- [Resource Estimation](../best-practices/resource-estimation.md) - Right-size your jobs

--8<-- "includes/getting-help.md"
