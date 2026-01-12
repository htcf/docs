# Job Errors

## Job Won't Start

### Symptoms
Job stays in PD (pending) state.

### Check Reason
```bash
$ squeue -u $USER -o "%.18i %.9P %.20j %.8u %.2t %.10M %.6D %.20R"
```

### Common Causes

**Resources**
- No nodes have requested resources available
- Solution: Reduce resources or wait

**Priority**
- Other jobs have higher priority
- Solution: Wait or reduce resource requests

**Invalid Partition**
```bash
#SBATCH -p genral    # Typo!
```
Solution: Fix partition name (`general` or `gpu`)

## Job Fails Immediately

### Check Output
```bash
$ cat slurm-<JOBID>.out
$ sacct -j <JOBID>
```

### Common Causes

**Missing Shebang**
```bash
# Wrong - no shebang
#SBATCH --mem=4G

# Correct
#!/bin/bash
#SBATCH --mem=4G
```

**Command Not Found**
```
bash: python: command not found
```
Solution: Load the package first
```bash
eval $(spack load --sh python)
```

**File Not Found**
```
python: can't open file 'train.py': No such file or directory
```
Solution: Check paths with `pwd` and `ls` in script

**Permission Denied**
```
bash: ./myscript.sh: Permission denied
```
Solution: `chmod +x myscript.sh`

## Out of Memory

### Symptoms
```
slurmstepd: error: Detected 1 oom-kill event
OUT_OF_MEMORY
```

### Check Actual Usage
```bash
$ sacct -j <JOBID>
```

### Solution
Increase memory request:
```bash
#SBATCH --mem=32G    # Was 16G
```

Add 10-20% buffer above actual usage.

## Time Limit Exceeded

### Symptoms
```
CANCELLED AT <time> DUE TO TIME LIMIT
TIMEOUT
```

### Solution
```bash
#SBATCH --time=48:00:00    # Was 24:00:00
```

Or implement checkpointing for very long jobs.

## Node Failure

### Symptoms
```
NODE_FAIL
slurmstepd: error: *** JOB <jobid> ON <node> CANCELLED AT <time> DUE TO NODE FAILURE
```

### Solution
Resubmit the job - node hardware issue, not your fault.

## Common Exit Codes

| Exit Code | Meaning | Solution |
|-----------|---------|----------|
| 0:0 | Success | - |
| 1:0 | General error | Check output logs |
| 2:0 | Misuse of shell | Check script syntax |
| 126:0 | Command cannot execute | Check permissions |
| 127:0 | Command not found | Load required software |
| 137:0 | Killed (usually OOM) | Increase memory |

--8<-- "includes/getting-help.md"
