# Tutorial: Running Array Jobs

## Learning Objectives

Learn to process multiple samples in parallel using Slurm array jobs:

- Understand array job syntax
- Use SLURM_ARRAY_TASK_ID
- Process multiple files simultaneously
- Use lookup files for non-sequential names

**Time Required:** 25 minutes
**Level:** Intermediate
**Prerequisites:** [Your First Batch Job](first-job.md)

## Why Use Array Jobs?

**Sequential Processing:**
```bash
process sample1.txt  # 10 minutes
process sample2.txt  # 10 minutes
process sample3.txt  # 10 minutes
# Total: 30 minutes
```

**Array Job (Parallel):**
```bash
process sample1.txt  # 10 minutes
process sample2.txt  # 10 minutes  } All at the same time!
process sample3.txt  # 10 minutes
# Total: 10 minutes (3x faster!)
```

## Example 1: Sequential File Names

### Step 1: Create Sample Data

```bash
$ cd /scratch/mylab/$USER
$ mkdir -p array-tutorial
$ cd array-tutorial

# Create 10 sample files
$ for i in {1..10}; do
    echo "This is sample $i" > sample_$i.txt
done

$ ls
```

You should see: `sample_1.txt`, `sample_2.txt`, ..., `sample_10.txt`

### Step 2: Create Array Job Script

```bash
$ nano process_array.sh
```

Add this content:

```bash
#!/bin/bash

#SBATCH --job-name=array_demo
#SBATCH --array=1-10           # Create 10 tasks (1 through 10)
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=5:00
#SBATCH --output=logs/array_%A_%a.out    # %A=job ID, %a=array task ID

# Create logs directory if it doesn't exist
mkdir -p logs

# The task ID is available as $SLURM_ARRAY_TASK_ID
TASK_ID=$SLURM_ARRAY_TASK_ID

echo "Processing task $TASK_ID"
echo "Job ID: $SLURM_JOB_ID"
echo "Array Task ID: $SLURM_ARRAY_TASK_ID"
echo "Running on: $(hostname)"
echo ""

# Use the task ID in the filename
INPUT_FILE="sample_${TASK_ID}.txt"
OUTPUT_FILE="result_${TASK_ID}.txt"

echo "Input: $INPUT_FILE"
echo "Output: $OUTPUT_FILE"
echo ""

# Process the file
echo "Processing $INPUT_FILE..."
cat $INPUT_FILE
wc -l $INPUT_FILE > $OUTPUT_FILE
echo "Processing time: 2 seconds" >> $OUTPUT_FILE
sleep 2

echo "Done!"
```

### Step 3: Submit the Array Job

```bash
$ sbatch process_array.sh
Submitted batch job 123459
```

### Step 4: Monitor Array Job

```bash
$ squeue -u $USER
```

You'll see multiple jobs with IDs like:
```
JOBID      PARTITION NAME           USER  ST TIME
123459_1   general   array_demo     user  R  0:01
123459_2   general   array_demo     user  R  0:01
123459_3   general   array_demo     user  R  0:01
...
```

The `_1`, `_2`, `_3` are the array task IDs.

### Step 5: Check Results

```bash
$ ls logs/
array_123459_1.out
array_123459_2.out
...
array_123459_10.out

$ ls result_*.txt
result_1.txt
result_2.txt
...
result_10.txt

$ cat logs/array_123459_1.out
```

### Step 6: Analyze Array Job

```bash
$ sacct -j 123459
```

Shows all array tasks and their individual statistics.

## Example 2: Non-Sequential File Names (Lookup File)

### Step 1: Create Sample Files with Random Names

```bash
$ cd /scratch/mylab/$USER/array-tutorial

# Create files with non-sequential names
$ echo "Data for sample AAA" > sample_AAA.txt
$ echo "Data for sample BBB" > sample_BBB.txt
$ echo "Data for sample CCC" > sample_CCC.txt
$ echo "Data for sample DDD" > sample_DDD.txt
$ echo "Data for sample EEE" > sample_EEE.txt
```

### Step 2: Create Lookup File

```bash
$ nano lookup.txt
```

Add one sample name per line:

```
AAA
BBB
CCC
DDD
EEE
```

### Step 3: Count Lines in Lookup File

```bash
$ wc -l lookup.txt
5 lookup.txt
```

We have 5 samples, so array will be `1-5`.

### Step 4: Create Array Job with Lookup

```bash
$ nano process_lookup.sh
```

Add this content:

```bash
#!/bin/bash

#SBATCH --job-name=lookup_array
#SBATCH --array=1-5              # Match number of lines in lookup.txt
#SBATCH --cpus-per-task=1
#SBATCH --mem=1G
#SBATCH --time=5:00
#SBATCH --output=logs/lookup_%A_%a.out

mkdir -p logs

# Read the sample name from line $SLURM_ARRAY_TASK_ID of lookup.txt
SAMPLE_NAME=$(sed -n ${SLURM_ARRAY_TASK_ID}p lookup.txt)

echo "Array Task ID: $SLURM_ARRAY_TASK_ID"
echo "Sample Name: $SAMPLE_NAME"
echo ""

# Use the sample name in filenames
INPUT_FILE="sample_${SAMPLE_NAME}.txt"
OUTPUT_FILE="result_${SAMPLE_NAME}.txt"

echo "Processing $INPUT_FILE..."
cat $INPUT_FILE
cat $INPUT_FILE > $OUTPUT_FILE
echo "Processed by task $SLURM_ARRAY_TASK_ID" >> $OUTPUT_FILE

echo "Done!"
```

### Step 5: Submit and Monitor

```bash
$ sbatch process_lookup.sh
$ squeue -u $USER
```

## Example 3: Limiting Concurrent Jobs

### Control How Many Tasks Run at Once

```bash
#SBATCH --array=1-100%10    # 100 tasks, max 10 running simultaneously
```

The `%10` means "run at most 10 tasks at a time."

### Why Limit Concurrent Jobs?

- Avoid overwhelming shared resources
- Be considerate to other users
- Prevent I/O bottlenecks

### Example Script

```bash
#!/bin/bash

#SBATCH --job-name=limited_array
#SBATCH --array=1-100%20        # 100 tasks, 20 at a time
#SBATCH --cpus-per-task=2
#SBATCH --mem=4G
#SBATCH --output=logs/limited_%A_%a.out

mkdir -p logs

ID=$SLURM_ARRAY_TASK_ID
echo "Processing task $ID"

# Your processing here
process_sample.sh sample_${ID}.fastq
```

## Example 4: Complex Lookup File

### Multiple Parameters per Sample

Create `complex_lookup.txt`:

```
sample1  0.001  100  control
sample2  0.005  200  treatment
sample3  0.01   150  control
sample4  0.001  300  treatment
```

### Process with Multiple Parameters

```bash
#!/bin/bash

#SBATCH --array=1-4
#SBATCH --output=logs/complex_%A_%a.out

mkdir -p logs

# Read the entire line
LINE=$(sed -n ${SLURM_ARRAY_TASK_ID}p complex_lookup.txt)

# Parse the line into variables
read SAMPLE PARAM1 PARAM2 CONDITION <<< "$LINE"

echo "Sample: $SAMPLE"
echo "Parameter 1: $PARAM1"
echo "Parameter 2: $PARAM2"
echo "Condition: $CONDITION"
echo ""

# Use in processing
my_analysis --sample $SAMPLE \
            --threshold $PARAM1 \
            --iterations $PARAM2 \
            --group $CONDITION \
            --output results_${SAMPLE}.txt
```

## Array Job Best Practices

### 1. Test with Small Array First

```bash
# Test with just 2 tasks
#SBATCH --array=1-2

# Once working, scale up
#SBATCH --array=1-1000%50
```

### 2. Use Meaningful Output Names

```bash
# Good: includes both job ID and task ID
#SBATCH --output=logs/job_%A_task_%a.out

# Bad: all tasks overwrite same file
#SBATCH --output=output.out
```

### 3. Create Logs Directory First

```bash
#!/bin/bash
#SBATCH --output=logs/array_%A_%a.out

# IMPORTANT: Create directory in script
mkdir -p logs

# Rest of script...
```

### 4. Check File Existence

```bash
INPUT_FILE="sample_${SLURM_ARRAY_TASK_ID}.fastq"

if [ ! -f "$INPUT_FILE" ]; then
    echo "ERROR: $INPUT_FILE not found!"
    exit 1
fi

# Process file...
```

## Common Issues

### Issue 1: Wrong Number of Tasks

**Problem:** Array size doesn't match number of files

**Solution:**

```bash
# Count files
$ ls sample_*.txt | wc -l
42

# Update array directive
#SBATCH --array=1-42
```

### Issue 2: All Tasks Write to Same File

**Problem:** Forgot `%a` in output filename

```bash
# Wrong (all tasks overwrite each other)
#SBATCH --output=output.out

# Correct
#SBATCH --output=logs/task_%a.out
```

### Issue 3: Lookup File Mismatch

**Problem:** sed returns empty string

**Check:**

```bash
# Verify lookup file
$ cat -n lookup.txt

# Test sed command
$ sed -n 1p lookup.txt
$ sed -n 2p lookup.txt
```

## Monitoring and Managing Array Jobs

### View All Array Tasks

```bash
$ squeue -u $USER
```

### Cancel Entire Array Job

```bash
$ scancel 123459
```

### Cancel Specific Array Tasks

```bash
# Cancel task 5
$ scancel 123459_5

# Cancel tasks 1-10
$ scancel 123459_[1-10]
```

### Check Array Job Statistics

```bash
$ sacct -j 123459
```

## Real-World Example: Process FASTQ Files

```bash
#!/bin/bash

#SBATCH --job-name=fastq_array
#SBATCH --array=1-20%5           # 20 samples, 5 at a time
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=4:00:00
#SBATCH --output=logs/fastq_%A_%a.out

mkdir -p logs results

# Get sample name from lookup
SAMPLE=$(sed -n ${SLURM_ARRAY_TASK_ID}p samples.txt)

echo "Processing sample: $SAMPLE"
echo "Task ID: $SLURM_ARRAY_TASK_ID"
echo "Start time: $(date)"

# Load required software
eval $(spack load --sh fastqc)
eval $(spack load --sh trimmomatic)

# Set file paths
INPUT="/lts/lab/raw_data/${SAMPLE}.fastq.gz"
OUTPUT="results/${SAMPLE}"
mkdir -p $OUTPUT

# Copy to scratch for processing
SCRATCH_DIR="/scratch/mylab/$USER/fastq_${SLURM_JOB_ID}_${SLURM_ARRAY_TASK_ID}"
mkdir -p $SCRATCH_DIR
rsync -av $INPUT $SCRATCH_DIR/

# Process on scratch
cd $SCRATCH_DIR
fastqc ${SAMPLE}.fastq.gz
trimmomatic SE ${SAMPLE}.fastq.gz ${SAMPLE}_trimmed.fastq.gz LEADING:3 TRAILING:3

# Copy results back
rsync -av $SCRATCH_DIR/ $OUTPUT/

# Cleanup
rm -rf $SCRATCH_DIR

echo "Complete: $(date)"
```

## Next Steps

- [Batch Jobs Guide](../using/batch.md) - Advanced batch job features
- [Job Optimization](../best-practices/job-optimization.md) - Optimize array jobs
- [Job Accounting](../using/accounting.md) - Analyze array job performance

--8<-- "includes/getting-help.md"
