# Quick Reference

Common diagnostic commands for troubleshooting.

## Job Status

```bash
# Your jobs
squeue -u $USER

# Specific job
squeue -j <JOBID>

# All jobs with details
squeue -u $USER -o "%.10i %.9P %.30j %.8u %.8T %.10M %.6D %R"
```

## Job History

```bash
# Recent jobs
sacct

# Specific job
sacct -j <JOBID>

# Jobs from last 7 days
sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d)

# Failed jobs only
sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) --state=FAILED
```

## Job Efficiency

```bash
# Quick efficiency summary
seff <JOBID>

# Running job details
scontrol show jobid -dd <JOBID>
```

## Storage

```bash
# Check all storage quotas
storage-info

# Scratch usage by directory
du -sh /scratch/mylab/$USER/*

# Find large files
find /scratch/mylab/$USER -size +1G -ls

# Find old files (>30 days)
find /scratch/mylab/$USER -mtime +30 -ls
```

## Cluster Status

```bash
# Node availability
sinfo -N

# Partition summary
sinfo

# Available GPUs
sinfo -p gpu -o "%n %G"
```

## Software

```bash
# Available packages
spack find

# Search for package
spack list <name>

# Load package
eval $(spack load --sh <package>)

# Check if command is available
which <command>
```

## Job Control

```bash
# Cancel job
scancel <JOBID>

# Cancel all your jobs
scancel -u $USER

# Cancel pending jobs only
scancel -u $USER -t PENDING
```

## Output Files

```bash
# View job output
cat slurm-<JOBID>.out

# Follow output in real-time
tail -f slurm-<JOBID>.out

# Search for errors
grep -i error slurm-<JOBID>.out
```

## Interactive Session

```bash
# Basic session
srun -J interactive -p interactive --pty /bin/bash -l

# With more resources
srun --mem=8G --cpus-per-task=4 -J interactive -p interactive --pty /bin/bash -l

# GPU session
srun -p gpu --gpus=1 -J interactive --pty /bin/bash -l
```

## Common Exit Codes

| Code | Meaning | Likely Cause |
|------|---------|--------------|
| 0:0 | Success | Job completed normally |
| 1:0 | General error | Check output logs |
| 127:0 | Command not found | Software not loaded |
| 137:0 | Killed (OOM) | Exceeded memory limit |
| TIMEOUT | Time exceeded | Increase --time |
| OUT_OF_MEMORY | OOM killed | Increase --mem |

## When to Use What

| Symptom | Command |
|---------|---------|
| Job won't start | `squeue -j <JOBID>` - check REASON |
| Job failed | `sacct -j <JOBID>` then `cat slurm-*.out` |
| Job slow | `seff <JOBID>` - check efficiency |
| Storage full | `storage-info` then `du -sh *` |
| Can't find software | `spack list <name>` |
| Need to test code | Start interactive session |
