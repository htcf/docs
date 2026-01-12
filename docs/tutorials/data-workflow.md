# Tutorial: Complete Data Workflow

## Learning Objectives

Learn the standard data lifecycle on HTCF:

1. Store raw data in LTS (long-term storage)
2. Process data (read directly from LTS or copy to scratch for high-performance I/O)
3. Run computational jobs
4. Copy results back to LTS
5. Clean up scratch

**Time Required:** 25 minutes
**Level:** Beginner
**Prerequisites:** [Your First Batch Job](first-job.md)

## Why This Workflow Matters

--8<-- "includes/storage-workflow.md"

## Step 1: Understand Storage Types

### LTS (Long-Term Storage)
- **Purpose**: Permanent data storage
- **Location**: `/lts/<lab-name>/`
- **Speed**: Moderate
- **Quota**: Lab-specific
- **Backup**: Yes
- **Access**: Read-only on compute nodes, read/write on login node
- **Cleaning**: Never

### Scratch (HTS)
- **Purpose**: Temporary high-performance storage
- **Location**: `/scratch/<labname>/<username>/`
- **Speed**: Fast
- **Quota**: 2TB default
- **Backup**: No
- **Cleaning**: Files unused for 60 days are cleaned

!!! warning "Scratch is Temporary"
    Files on scratch are automatically moved to the trash after 60 days of inactivity. Always copy important data back to LTS!

## Step 2: Create Sample Data in LTS

For this tutorial, we'll simulate having data in LTS:

```bash
# Navigate to the LTS directory (adjust path for the lab)
$ cd /lts/<lab>

# If LTS access is not available, use the home directory for this tutorial
$ cd ~
$ mkdir -p tutorial-data
$ cd tutorial-data

# Create sample data files
$ echo "Sample,Value" > data1.csv
$ echo "A,10" >> data1.csv
$ echo "B,20" >> data1.csv
$ echo "C,30" >> data1.csv

$ echo "Sample,Value" > data2.csv
$ echo "X,15" >> data2.csv
$ echo "Y,25" >> data2.csv
$ echo "Z,35" >> data2.csv

$ ls -lh
```

Note the data location (e.g., `/lts/<lab>/tutorial-data` or `~/tutorial-data`).

## Step 3: Create Processing Script

Create a script that processes data:

```bash
$ cd /scratch/mylab/$USER
$ mkdir -p data-workflow-tutorial
$ cd data-workflow-tutorial
$ nano process_data.sh
```

Add this content:

```bash
#!/bin/bash

#SBATCH --job-name=data_workflow
#SBATCH --cpus-per-task=2
#SBATCH --mem=4G
#SBATCH --time=15:00
#SBATCH --output=workflow_%j.out

# IMPORTANT: Set the data source location
# Replace with the actual LTS path or use ~/ for home directory
DATA_SOURCE=~/tutorial-data          # CHANGE THIS
RESULTS_DEST=$DATA_SOURCE/results

echo "=== Data Workflow Tutorial ==="
echo "Job ID: $SLURM_JOB_ID"
echo "Start time: $(date)"
echo ""

# Step 1: Create working directory in scratch
WORK_DIR=/scratch/<lab>/$USER/workflow_${SLURM_JOB_ID}
mkdir -p $WORK_DIR
cd $WORK_DIR
echo "Working directory: $WORK_DIR"
echo ""

# Step 2: Copy input data to scratch
echo "Step 1: Copying data from LTS to scratch..."
rsync -av --progress $DATA_SOURCE/ $WORK_DIR/input/
echo "✓ Data copied"
echo ""

# Step 3: Process data
echo "Step 2: Processing data..."
mkdir -p output

# Simple processing: count lines and sum values
for file in input/*.csv; do
    filename=$(basename $file)
    echo "  Processing $filename..."

    # Count lines
    lines=$(wc -l < $file)

    # Sum the second column (values)
    sum=$(tail -n +2 $file | cut -d',' -f2 | awk '{sum+=$1} END {print sum}')

    # Create output
    echo "File: $filename" > output/result_$filename
    echo "Lines: $lines" >> output/result_$filename
    echo "Sum: $sum" >> output/result_$filename
    echo "" >> output/result_$filename
done

echo "✓ Processing complete"
echo ""

# Step 4: Copy results back to LTS
echo "Step 3: Copying results back to LTS..."
mkdir -p $RESULTS_DEST
rsync -av --progress $WORK_DIR/output/ $RESULTS_DEST/
echo "✓ Results saved to $RESULTS_DEST"
echo ""

# Step 5: Clean up scratch
echo "Step 4: Cleaning up scratch..."
cd /scratch/mylab/$USER
rm -rf $WORK_DIR
echo "✓ Scratch cleaned"
echo ""

echo "=== Workflow Complete ==="
echo "End time: $(date)"
echo "Results location: $RESULTS_DEST"
```

**Important**: Edit the `DATA_SOURCE` variable to match the data location!

## Step 4: Submit the Workflow Job

```bash
$ sbatch process_data.sh
Submitted batch job 123458
```

## Step 5: Monitor Progress

```bash
$ squeue -u $USER
$ tail -f workflow_123458.out  # Watch progress in real-time
```

Press `Ctrl+C` to stop watching.

## Step 6: Verify Results

After the job completes:

```bash
$ cat workflow_123458.out
```

Expected output:

```
=== Data Workflow Tutorial ===
Job ID: 123458
Start time: Tue Jan  7 15:00:00 CST 2026

Working directory: /scratch/yourname/workflow_123458

Step 1: Copying data from LTS to scratch...
sending incremental file list
data1.csv
data2.csv
✓ Data copied

Step 2: Processing data...
  Processing data1.csv...
  Processing data2.csv...
✓ Processing complete

Step 3: Copying results back to LTS...
sending incremental file list
result_data1.csv
result_data2.csv
✓ Results saved to ~/tutorial-data/results

Step 4: Cleaning up scratch...
✓ Scratch cleaned

=== Workflow Complete ===
End time: Tue Jan  7 15:00:25 CST 2026
Results location: ~/tutorial-data/results
```

Check the results:

```bash
$ ls ~/tutorial-data/results/
$ cat ~/tutorial-data/results/result_data1.csv
```

## Step 7: Verify Scratch is Clean

```bash
$ ls /scratch/mylab/$USER/workflow_123458
```

Should return: `No such file or directory` (because the directory was cleaned up).

## Summary

### Storage Best Practices

✓ **Store permanent data in LTS** - Raw data, results, important files
✓ **Use scratch for computation** - Fast I/O during processing
✓ **Clean up scratch** - Remove temporary files after jobs
✓ **Use rsync for transfers** - Efficient, resumable file transfers

### rsync Basics

```bash
# Basic sync
rsync -av source/ destination/

# With progress
rsync -av --progress source/ destination/

# Dry run (see what would be copied)
rsync -av --dry-run source/ destination/
```

### Workflow Pattern

**In job script:**
```bash
# 1. Create working directory in scratch
WORK_DIR=/scratch/mylab/$USER/job_$SLURM_JOB_ID
mkdir -p $WORK_DIR

# 2. Jobs can read directly from LTS (read-only)
cd $WORK_DIR
./process_script.sh /lts/lab/data/ output/

echo "Results in $WORK_DIR/output/"
```

**After job completes (from login node):**
```bash
# Copy results to LTS and clean up
rsync -av /scratch/mylab/$USER/job_*/output/ /lts/lab/results/
rm -rf /scratch/mylab/$USER/job_*
```

## Common Patterns

### Pattern 1: Large Dataset

For large datasets, process in chunks. From the login node:

```bash
# Stage subset to scratch
rsync -av /lts/lab/data/subset1/ /scratch/mylab/$USER/data/

# Submit job (reads from scratch, writes to scratch)
sbatch process_subset.sh

# After job completes, copy results back (from login node)
rsync -av /scratch/mylab/$USER/results/ /lts/lab/results/subset1/

# Repeat for subset2, subset3, etc.
```

### Pattern 2: Multiple Input Files

Process specific files only:

```bash
# Copy specific files
rsync -av /lts/lab/data/*.fastq /scratch/mylab/$USER/input/

# Or use include/exclude patterns
rsync -av --include="*.fastq" --exclude="*" /lts/lab/data/ /scratch/mylab/$USER/input/
```

### Pattern 3: Iterative Development

Keep data in scratch during development. All commands from login node:

```bash
# Copy once (login node)
rsync -av /lts/lab/data/ /scratch/mylab/$USER/dev-data/

# Develop and test - submit multiple jobs
sbatch test_run.sh
# Review results, adjust, repeat...

# After final run completes, copy results back (login node)
rsync -av /scratch/mylab/$USER/results/ /lts/lab/final-results/

# Clean up when completely done
rm -rf /scratch/mylab/$USER/dev-data
```

## Troubleshooting

### Disk Quota Exceeded

**Error:** `rsync: write failed: Disk quota exceeded`

**Solution:**

```bash
# Check scratch usage
$ du -sh /scratch/mylab/$USER/*

# Remove old data
$ rm -rf /scratch/mylab/$USER/old-project

# Check quota
$ storage-info
```

### Permission Denied

**Error:** `rsync: failed to set permissions: Permission denied`

**Solution:**

```bash
# Add --no-perms flag
rsync -av --no-perms /lts/lab/data/ /scratch/mylab/$USER/data/
```

### LTS is Slow

LTS is on network storage and can be slow for many small files.

**Solution:**

- Use scratch for computation
- Archive many small files: `tar czf archive.tar.gz directory/`
- Copy archives, then extract on scratch

## Next Steps

- [Array Jobs](array-jobs.md) - Process multiple datasets in parallel
- [Storage Documentation](../storage/index.md) - Detailed storage information
- [Data Management Best Practices](../best-practices/data-management.md)
- [Job Optimization](../best-practices/job-optimization.md) - I/O optimization

--8<-- "includes/getting-help.md"
