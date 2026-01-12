# Batch Jobs

## Overview

Batch jobs are the primary way to execute computational work on the HTCF cluster. Unlike interactive jobs, batch jobs are submitted to the scheduler and run when resources become available. This approach allows for:

- Efficient resource utilization
- Running jobs without staying logged in
- Automatic job execution and management
- Better resource allocation across all users

!!! note
    The HTCF is primarily a batch queuing system designed for batch job submission. Batch jobs are the typical method for production computational work.

### When to Use Batch Jobs

Use batch jobs for:

- Production computational workflows
- Long-running jobs (more than a few hours)
- Multiple sample processing
- Automated workflows that don't require interaction
- Any resource-intensive computation

For short development tasks, interactive visualization, or software testing, use [interactive jobs](interactive.md) instead.

## Batch Job Lifecycle

A typical batch job follows this lifecycle:

1. **Create job script** - Write the script that does the computational work
2. **Create sbatch file** - Define resource requirements with SBATCH directives
3. **Submit job** - Submit using `sbatch` command
4. **Pending** - Job waits in queue for resources
5. **Running** - Job executes on allocated nodes
6. **Complete** - Job finishes and outputs are saved

## Creating Job Scripts

### Basic Job Script Structure

A batch job script is a shell script with special SBATCH directives that tell Slurm what resources the job needs.

```bash
#!/bin/bash

#SBATCH --job-name=myjob          # Job name for identification
#SBATCH --cpus-per-task=4         # Number of CPU cores
#SBATCH --mem=16G                 # Total memory
#SBATCH --time=24:00:00           # Time limit (D-HH:MM:SS)
#SBATCH --partition=general       # Partition to use
#SBATCH --output=myjob_%j.out     # Standard output file (%j = job ID)
#SBATCH --error=myjob_%j.err      # Standard error file

# Load required software
eval $(spack load --sh <program>)

# Navigate to working directory (use your actual lab and paths)
cd /scratch/mylab/$USER/project/

# Run the computational work
program input.txt output.txt

# Exit successfully
exit 0
```

!!! tip "Shebang Line"
    Start the script with `#!/bin/bash` to ensure it runs with the bash shell.

### Resource Specification

--8<-- "includes/common-slurm-params.md"

#### Memory Specification

You can specify memory in two ways:

- `--mem=<size>` - Total memory per node (e.g., `--mem=16G`)
- `--mem-per-cpu=<size>` - Memory per CPU core (e.g., `--mem-per-cpu=4G`)

!!! warning "Memory Limits"
    Jobs that exceed their requested memory will be killed automatically. Include a safety margin (10-20%) above estimated needs.

#### Time Limits

Specify time limits in the format `D-HH:MM:SS` or just `HH:MM:SS`:

```bash
#SBATCH --time=2-12:00:00    # 2 days, 12 hours
#SBATCH --time=48:00:00      # 48 hours
#SBATCH --time=1:30:00       # 1 hour, 30 minutes
```

!!! note "Default Time Limit"
    The general partition has no time limit. Time limits can be specified for jobs to enable better scheduling.

### Output Files

By default, Slurm creates output files named `slurm-<jobid>.out`. You can customize this:

```bash
#SBATCH --output=logs/job_%j.out        # %j is replaced with job ID
#SBATCH --error=logs/job_%j.err         # Separate error file
#SBATCH --output=logs/job_%j_%x.out     # %x is replaced with job name
```

Create the logs directory before submitting:

```bash
mkdir -p logs
```

## Submitting Jobs

### Basic Submission

Submit a job script using the `sbatch` command:

```bash
$ sbatch myjob.sh
Submitted batch job 123456
```

The command returns a job ID that can be used to monitor and manage the job.

### Command Line Options

Sbatch file parameters can be overridden from the command line:

```bash
$ sbatch --mem=32G --cpus-per-task=8 myjob.sh
$ sbatch --job-name=test_run myjob.sh
$ sbatch --time=6:00:00 myjob.sh
```

### Job Dependencies

Run jobs sequentially by specifying dependencies:

```bash
# Submit first job
$ job1=$(sbatch --parsable first_job.sh)

# Submit second job that waits for first to complete successfully
$ sbatch --dependency=afterok:$job1 second_job.sh

# Submit third job that runs after first, regardless of success/failure
$ sbatch --dependency=after:$job1 third_job.sh
```

Common dependency types:

- `afterok:jobid` - Start after job completes successfully
- `afterany:jobid` - Start after job completes (any exit status)
- `after:jobid` - Start after job starts
- `afternotok:jobid` - Start only if job fails

## Array Jobs

Array jobs allow you to run the same job many times with different inputs, potentially running in parallel. This is ideal for processing multiple samples or parameter sweeps.

!!! tip "Parallel Processing"
    Array jobs can dramatically reduce total runtime by processing multiple items simultaneously instead of sequentially.

### Basic Array Job

```bash
#!/bin/bash

#SBATCH --job-name=array_job
#SBATCH --array=1-100%20          # 100 tasks, max 20 concurrent
#SBATCH --cpus-per-task=1
#SBATCH --mem=4G
#SBATCH --output=logs/array_%A_%a.out    # %A = job ID, %a = array task ID

# The task ID is available as $SLURM_ARRAY_TASK_ID
ID=${SLURM_ARRAY_TASK_ID}

echo "Processing sample ${ID}"

# Use the ID to process different inputs
process_sample.sh sample_${ID}.fastq
```

### Array Jobs with Lookup Files

For non-sequential file names, use a lookup file:

```bash
#!/bin/bash

#SBATCH --array=1-2000%20

# Read the sample name from line $SLURM_ARRAY_TASK_ID of lookup.txt
SAMPLE=$(sed -n ${SLURM_ARRAY_TASK_ID}p lookup.txt)

# Process the sample
process_sample.sh ${SAMPLE}.fastq
```

Create the lookup file:

```bash
$ cat lookup.txt
sampleAAA
sampleAAB
sampleAAC
...
```

For more array job examples, see our [Array Jobs Tutorial](../tutorials/array-jobs.md).

## Monitoring Jobs

### Checking Job Status

Use `squeue` to view jobs:

```bash
# View all jobs for a user
$ squeue -u $USER

# View specific job
$ squeue -j 123456

# View jobs with custom format
$ squeue -u $USER -o "%.10i %.9P %.30j %.8u %.8T %.10M %.6D %R"
```

Common job states:

- **PD** (Pending) - Job is waiting for resources
- **R** (Running) - Job is currently executing
- **CG** (Completing) - Job is in the process of completing
- **CD** (Completed) - Job has finished successfully
- **F** (Failed) - Job terminated with non-zero exit code
- **CA** (Cancelled) - Job was cancelled by user or admin

### Viewing Job Details

For running jobs, use `scontrol`:

```bash
$ scontrol show jobid -dd 123456
```

This shows detailed information including:

- Resource allocations
- Node assignments
- Job script contents
- Output file locations
- Job state and reason

### Monitoring Output in Real Time

Watch the job's output as it runs:

```bash
# View output file
$ tail -f slurm-123456.out

# Follow both output and error files
$ tail -f logs/job_123456.out logs/job_123456.err
```

### Cancelling Jobs

Cancel jobs using `scancel`:

```bash
# Cancel a specific job
$ scancel 123456

# Cancel all jobs for a user
$ scancel -u $USER

# Cancel all pending jobs for a user
$ scancel -u $USER -t PENDING

# Cancel array job tasks
$ scancel 123456_10        # Cancel task 10
$ scancel 123456_[1-50]    # Cancel tasks 1-50
```

## Job Examples

### Simple Single-Core Job

For jobs that don't benefit from parallelization:

```bash
#!/bin/bash

#SBATCH --job-name=single_core
#SBATCH --cpus-per-task=1
#SBATCH --mem=4G
#SBATCH --time=12:00:00
#SBATCH --output=single_%j.out

eval $(spack load --sh my-program)

cd /scratch/mylab/$USER/analysis/

my-program input.txt > output.txt
```

### Multi-Core Job

For parallel programs that use multiple CPU cores:

```bash
#!/bin/bash

#SBATCH --job-name=multi_core
#SBATCH --cpus-per-task=16
#SBATCH --mem=64G
#SBATCH --time=48:00:00
#SBATCH --output=multi_%j.out

eval $(spack load --sh parallel-program)

cd /scratch/mylab/$USER/analysis/

# Many programs use OMP_NUM_THREADS or similar
export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

parallel-program --threads $SLURM_CPUS_PER_TASK input.txt output.txt
```

!!! tip "Using $SLURM_CPUS_PER_TASK"
    Use the `$SLURM_CPUS_PER_TASK` environment variable in the script to avoid changing the number in multiple places.

### High-Memory Job

For jobs requiring large amounts of RAM:

```bash
#!/bin/bash

#SBATCH --job-name=high_mem
#SBATCH --cpus-per-task=4
#SBATCH --mem=250G              # Request 250GB RAM
#SBATCH --partition=general     # Use general partition (max 250GB)
#SBATCH --time=24:00:00
#SBATCH --output=highmem_%j.out

eval $(spack load --sh memory-intensive-program)

cd /scratch/mylab/$USER/large_dataset/

memory-intensive-program --memory 240G input.dat output.dat
```

### GPU Job

For jobs requiring GPU acceleration:

--8<-- "includes/gpu-params.md"

```bash
#!/bin/bash

#SBATCH --job-name=gpu_job
#SBATCH --partition=gpu
#SBATCH --gpus=1                # Request 1 GPU
#SBATCH --cpus-per-task=4
#SBATCH --mem=32G
#SBATCH --time=8:00:00          # Set appropriate time limit
#SBATCH --output=gpu_%j.out

# Load CUDA and your GPU-enabled software
eval $(spack load --sh cuda)
eval $(spack load --sh py-pytorch)

cd /scratch/mylab/$USER/ml_project/

# Verify GPU is available
nvidia-smi

# Run GPU program
python train_model.py --gpu 0
```

For more GPU information, see the [GPU Computing documentation](../gpu/index.md).

### Long-Running Job with Checkpointing

For jobs that may exceed time limits or need to be restarted:

```bash
#!/bin/bash

#SBATCH --job-name=long_run
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=48:00:00
#SBATCH --output=long_%j.out

eval $(spack load --sh simulation-program)

cd /scratch/mylab/$USER/simulation/

# Check if checkpoint exists from previous run
if [ -f checkpoint.dat ]; then
    echo "Restarting from checkpoint"
    simulation-program --restart checkpoint.dat
else
    echo "Starting new simulation"
    simulation-program --new
fi

# Program should save checkpoint periodically
```

## Common Patterns

### Loading Software

The HTCF uses Spack for software management. Load software in your job script:

```bash
# Load single package
eval $(spack load --sh python)

# Load multiple packages
eval $(spack load --sh python)
eval $(spack load --sh py-numpy)
eval $(spack load --sh py-scipy)

# Load specific version
eval $(spack load --sh python@3.11)
```

See the [Software documentation](../software/index.md) for more details.

### Working with Scratch Storage

Follow the standard data workflow:

--8<-- "includes/storage-workflow.md"

Example in a job script:

```bash
#!/bin/bash

#SBATCH --job-name=data_workflow
#SBATCH --cpus-per-task=4
#SBATCH --mem=16G
#SBATCH --output=workflow_%j.out

# Jobs can read directly from LTS (read-only on compute nodes)
# For high-performance I/O, copy to scratch first
cd /scratch/mylab/$USER/project/

eval $(spack load --sh my-analysis-tool)
my-analysis-tool /lts/mylab/raw_data/ output/

echo "Job complete! Results in /scratch/mylab/$USER/project/output/"
```

!!! important "Copy Results After Job Completes"
    LTS is read-only on compute nodes. After your job completes, copy results from the login node (use your actual lab and paths):
    ```bash
    rsync -av /scratch/mylab/$USER/project/output/ /lts/mylab/results/
    rm -rf /scratch/mylab/$USER/project/  # Clean up scratch
    ```

### Job Chaining

Run a series of dependent jobs:

```bash
# Submit preprocessing job
preprocess_id=$(sbatch --parsable preprocess.sh)
echo "Submitted preprocessing: $preprocess_id"

# Submit main analysis after preprocessing succeeds
analysis_id=$(sbatch --parsable --dependency=afterok:$preprocess_id analysis.sh)
echo "Submitted analysis: $analysis_id"

# Submit postprocessing after analysis succeeds
postprocess_id=$(sbatch --parsable --dependency=afterok:$analysis_id postprocess.sh)
echo "Submitted postprocessing: $postprocess_id"

# Submit cleanup after everything completes (success or failure)
sbatch --dependency=afterany:$postprocess_id cleanup.sh
echo "Submitted cleanup"
```

## Troubleshooting

### Job Won't Start (Pending)

If your job stays in **PD** (pending) state, check the reason:

```bash
$ squeue -u $USER -o "%.10i %.9P %.20j %.8u %.8T %.10M %.10l %.6D %.20R"
```

Common reasons:

- **Resources** - No nodes have the requested resources available
- **Priority** - Other jobs have higher priority
- **Dependency** - Job is waiting for another job to complete

### Job Fails Immediately

If your job starts and fails quickly:

1. Check the output and error files
2. Verify the shebang line is correct: `#!/bin/bash`
3. Check that required software is loaded
4. Verify input file paths are correct
5. Test the commands interactively first

```bash
# Check error log
$ cat slurm-123456.out

# View job details
$ sacct -j 123456
```

### Out of Memory Errors

If your job is killed due to memory:

```bash
$ sacct -j 123456
```

Check the `MaxRSS` and `State` columns in the output.

Solution:

- Increase memory request with `--mem=` or `--mem-per-cpu=`
- Add 10-20% safety margin above actual usage
- Consider if data can be processed in chunks

### Time Limit Exceeded

If your job hits the time limit:

```bash
$ sacct -j 123456
```

Check the `Elapsed` and `Timelimit` columns in the output.

Solution:

- Increase time limit with `--time=`
- Implement checkpointing for very long jobs
- Optimize code to run faster
- Consider if task can be split into array jobs

For more detailed troubleshooting, see the [Job Errors](../troubleshooting/job-errors.md) documentation.

## Best Practices

### Resource Estimation

- Start with small test jobs to determine actual resource needs
- Use `sacct` to analyze completed jobs and right-size future jobs
- See [Resource Estimation](../best-practices/resource-estimation.md) for detailed guidance

### Job Organization

- Use meaningful job names with `--job-name=`
- Organize output files in dedicated directories
- Include the job ID in output filenames with `%j`

### Efficiency

- Use array jobs instead of many individual jobs when processing multiple items
- Utilize job dependencies for workflows instead of polling
- Request only the resources needed for the job
- See [Job Optimization](../best-practices/job-optimization.md) for tips

## Next Steps

- [Interactive Jobs](interactive.md) - For development and testing
- [Job Accounting](accounting.md) - Analyze completed jobs
- [Quick Start Tutorial](../tutorials/first-job.md) - Step-by-step first job
- [Array Jobs Tutorial](../tutorials/array-jobs.md) - Parallel processing
- [GPU Computing](../gpu/index.md) - Using GPUs
- [Best Practices](../best-practices/index.md) - Optimize your workflow

--8<-- "includes/getting-help.md"
