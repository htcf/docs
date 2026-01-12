# Job Accounting

## Overview

Job accounting allows you to track and analyze your job history, resource usage, and performance metrics. This information is valuable for:

- Understanding actual resource usage vs requested resources
- Optimizing future job submissions by right-sizing requests
- Troubleshooting failed or inefficient jobs
- Tracking computational usage for labs or projects
- Improving overall workflow efficiency

Slurm provides two main commands for job accounting:

- **`sacct`** - View historical job information
- **`scontrol`** - View detailed information about currently running jobs

## Using sacct - Job History

### Basic Usage

View your recent jobs:

```bash
$ sacct
```

This shows a simple table with basic information about your recently completed jobs.

View jobs for a specific user:

```bash
$ sacct -u $USER
```

View a specific job:

```bash
$ sacct -j 123456
```

### Specifying Time Ranges

By default, `sacct` shows jobs from today. To see older jobs:

```bash
# Jobs from the last 7 days
$ sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d)

# Jobs from a specific date range
$ sacct --starttime=2024-01-01 --endtime=2024-01-31

# All jobs from a specific date
$ sacct --starttime=2024-01-15

# Jobs from the last month
$ sacct --starttime=$(date -d '1 month ago' +%Y-%m-%d)
```

### Output Format

The default `sacct` output shows JobID, JobName, Partition, Account, AllocCPUS, State, and ExitCode - usually sufficient for basic troubleshooting.

For more details (memory usage, elapsed time, etc.), use `seff`:

```bash
$ seff <JOBID>
```

This provides a quick efficiency summary including memory and CPU utilization.

### All Available Fields

See all available fields:

```bash
$ sacct -e
```

This displays a comprehensive list of fields you can include in your output format.

## Understanding Job Statistics

### Memory Usage

**MaxRSS** - Maximum Resident Set Size (actual memory used):

```bash
$ sacct -j 123456 --format=JobID,JobName,MaxRSS,ReqMem,State

       JobID    JobName     MaxRSS     ReqMem      State
------------ ---------- ---------- ---------- ----------
123456       myjob        8192K          16Gn  COMPLETED
```

In this example:
- MaxRSS: 8192K (8 MB) - actual memory used
- ReqMem: 16Gn (16 GB per node) - memory requested
- This job used much less memory than requested

!!! tip "Right-Sizing Memory"
    If MaxRSS is significantly less than ReqMem, you can request less memory in future jobs. Add a 10-20% safety margin above MaxRSS.

### Time Usage

**Elapsed** - Wall clock time (actual runtime):

```bash
$ sacct -j 123456 --format=JobID,JobName,Elapsed,Timelimit,State

       JobID    JobName    Elapsed  Timelimit      State
------------ ---------- ---------- ---------- ----------
123456       myjob        01:23:45   06:00:00  COMPLETED
```

In this example:
- Elapsed: 1 hour 23 minutes 45 seconds
- Timelimit: 6 hours requested
- This job finished in 23% of the requested time

### CPU Efficiency

Compare CPU time to wall time:

```bash
$ sacct -j 123456 --format=JobID,AllocCPUS,CPUTime,Elapsed,TotalCPU,State

       JobID  AllocCPUS    CPUTime    Elapsed   TotalCPU      State
------------ ---------- ---------- ---------- ---------- ----------
123456                4   05:35:00   01:23:45   05:12:34  COMPLETED
```

- **AllocCPUS**: 4 CPUs allocated
- **CPUTime**: 5:35:00 (Elapsed × AllocCPUS = theoretical max)
- **TotalCPU**: 5:12:34 (actual CPU time used)
- **CPU Efficiency**: 5:12:34 / 5:35:00 = 93% efficiency (excellent!)

!!! note "CPU Efficiency Interpretation"
    - **>90%** - Excellent, CPUs well utilized
    - **70-90%** - Good, reasonable efficiency
    - **<70%** - Poor, consider requesting fewer CPUs or optimizing code

### Job States

Common job states in `sacct`:

- **COMPLETED (CD)** - Job finished successfully (exit code 0)
- **FAILED (F)** - Job finished with non-zero exit code
- **TIMEOUT (TO)** - Job exceeded time limit
- **OUT_OF_MEMORY (OOM)** - Job exceeded memory limit
- **CANCELLED (CA)** - Job was cancelled by user or admin
- **NODE_FAIL (NF)** - Job terminated due to node failure

### Exit Codes

```bash
$ sacct -j 123456 --format=JobID,State,ExitCode,DerivedExitCode

       JobID      State ExitCode DerivedExitCode
------------ ---------- -------- ---------------
123456       FAILED       1:0             1:0
```

- **ExitCode**: `signal:exit_code`
- **0:0** - Success
- **1:0** - General error
- **137:0** - Killed (usually out of memory)
- **Other non-zero** - Application-specific error

## Using scontrol - Running Jobs

For currently running or recently completed jobs, `scontrol` provides detailed information:

```bash
$ scontrol show jobid -dd 123456
```

Example output:

```
JobId=846115 JobName=sleep.sh
   UserId=ericmartin(1002) GroupId=ericmartin(1002)
   Priority=3070 Nice=0 Account=htcfadmin
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
   DerivedExitCode=0:0
   RunTime=00:00:09 TimeLimit=UNLIMITED TimeMin=N/A
   SubmitTime=2016-03-09T09:47:08 EligibleTime=2016-03-09T09:47:08
   StartTime=2016-03-09T09:47:09 EndTime=Unknown
   PreemptTime=None SuspendTime=None SecsPreSuspend=0
   Partition=general AllocNode:Sid=n082:21380
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=n082
   BatchHost=n082
   NumNodes=1 NumCPUs=2 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
     Nodes=n082 CPU_IDs=3-4 Mem=2000
   MinCPUsNode=1 MinMemoryCPU=1000M MinTmpDiskNode=0
   Features=(null) Gres=(null) Reservation=(null)
   Shared=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/scratch/htcfadmin/eric/sleep.sh
   WorkDir=/scratch/htcfadmin/eric
   StdErr=/scratch/htcfadmin/eric/slurm-846115.out
   StdIn=/dev/null
   StdOut=/scratch/htcfadmin/eric/slurm-846115.out
   BatchScript=
#!/bin/bash

#SBATCH -n 2
#SBATCH -N 1

eval $(spack load --sh bowtie2)

sleep 100
```

!!! tip "When to Use scontrol"
    Use `scontrol show jobid -dd <JOBID>` when:

    - Your job is currently running and you need details
    - You need to see the actual job script that was submitted
    - You need to see the exact node and resources allocated
    - You're troubleshooting and need to provide info to support

## Job Efficiency Analysis with seff

The `seff` command provides a quick, human-readable summary of job efficiency, making it much easier to understand resource utilization compared to raw `sacct` output.

### Using seff

Check efficiency for a single completed job:

```bash
$ seff 123456
```

Example output:

```
Job ID: 123456
Cluster: htcf
User/Group: username/labgroup
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 8
CPU Utilized: 06:15:30
CPU Efficiency: 78.19% of 08:00:00 core-walltime
Job Wall-clock time: 01:00:00
Memory Utilized: 28.50 GB
Memory Efficiency: 71.25% of 40.00 GB
```

### Interpreting seff Output

**CPU Efficiency:**
- Shows percentage of requested CPU time actually used
- **>80%** - Excellent, CPUs well utilized
- **50-80%** - Good, reasonable efficiency
- **<50%** - Consider requesting fewer cores or improving parallelization

**Memory Efficiency:**
- Shows percentage of requested memory actually used
- **>75%** - Good, well-sized memory request
- **50-75%** - Acceptable, slight over-allocation
- **<50%** - Reduce memory request for future jobs
- **>95%** - Risky, increase memory to prevent OOM errors

### When to Use seff

Use `seff` when you need to:

- Quickly check if your job used resources efficiently
- Determine if you're requesting too much or too little memory
- Identify if your parallel code is using all requested cores
- Right-size resources before submitting array jobs or similar jobs

!!! tip "seff vs sacct"
    - **Use seff**: Quick efficiency check, easy to read, single job analysis
    - **Use sacct**: Multiple jobs, custom fields, scripting/automation, detailed forensics

## Analyzing Multiple Jobs with reportseff

The `reportseff` command analyzes efficiency for multiple jobs at once, making it ideal for reviewing job arrays or recent job history.

### Using reportseff

Check efficiency for all your jobs from the last 7 days:

```bash
$ reportseff
```

Check specific jobs:

```bash
$ reportseff 123456 123457 123458
```

Check efficiency for a date range:

```bash
$ reportseff --since 2024-01-01 --until 2024-01-31
```

Check only your jobs from yesterday:

```bash
$ reportseff --since yesterday
```

### Example Output

```
   JobID    State      Elapsed    CPUEff   MemEff
 123456    COMPLETED   01:30:00   92.3%    68.5%
 123457    COMPLETED   02:15:00   45.2%    85.1%
 123458    FAILED      00:05:00    5.1%    12.3%
 123459    COMPLETED   04:00:00   88.7%    95.2%
```

### Useful reportseff Options

```bash
# Show only jobs with low CPU efficiency (<50%)
$ reportseff --format JobID,State,CPUEff,MemEff | awk '$3 < 50'

# Check all jobs from a specific user
$ reportseff --user username

# Check efficiency for an entire array job
$ reportseff --job 123456

# Include additional columns
$ reportseff --format JobID,JobName,State,Elapsed,CPUEff,MemEff,MaxRSS
```

### Identifying Patterns with reportseff

**Pattern: Consistently low CPU efficiency**
```
JobID      State        CPUEff   MemEff
123456     COMPLETED    25%      70%
123457     COMPLETED    22%      68%
123458     COMPLETED    28%      72%
```
→ **Action**: Code may not be parallelizing well, reduce CPU request

**Pattern: Consistently high memory efficiency (>95%)**
```
JobID      State        CPUEff   MemEff
123459     COMPLETED    85%      98%
123460     COMPLETED    88%      97%
123461     COMPLETED    82%      99%
```
→ **Action**: Risk of OOM, increase memory request

**Pattern: High variability in resource usage**
```
JobID      State        MemEff
123462     COMPLETED    25%
123463     COMPLETED    88%
123464     COMPLETED    45%
```
→ **Action**: Input data size varies, consider dynamic allocation or separate job configs

### Combining seff and reportseff

Workflow for optimizing job arrays:

1. **Run reportseff** to identify problematic jobs:
   ```bash
   $ reportseff --since yesterday
   ```

2. **Use seff on outliers** for detailed analysis:
   ```bash
   $ seff 123458  # Low efficiency job
   ```

3. **Adjust resources** based on findings and resubmit

!!! note "Installing seff and reportseff"
    If `seff` or `reportseff` are not available on your system, they can typically be installed via:

    ```bash
    # seff comes with slurm-tools
    $ spack install slurm-tools

    # or check if they're already installed
    $ which seff reportseff
    ```

    Post in **#general** on Slack if you need these tools installed system-wide.

## Analyzing Past Jobs for Optimization

### Workflow for Right-Sizing Resources

1. **Submit initial job** with generous resource estimates
2. **Job completes** (or fails)
3. **Analyze with sacct** to see actual usage
4. **Adjust requests** for future jobs
5. **Iterate** until resources are optimized

### Example: Memory Optimization

Initial job (over-estimated):

```bash
#!/bin/bash
#SBATCH --mem=64G
#SBATCH --job-name=test_job
...
```

After completion, check actual usage:

```bash
$ sacct -j 123456 --format=JobID,JobName,MaxRSS,ReqMem,State

       JobID    JobName     MaxRSS     ReqMem      State
------------ ---------- ---------- ---------- ----------
123456       test_job     12000K         64Gn  COMPLETED
```

Analysis:
- MaxRSS: 12000K (~12 GB actual usage)
- ReqMem: 64G requested
- Optimized request: 16G (12G + 33% safety margin)

Optimized job:

```bash
#!/bin/bash
#SBATCH --mem=16G    # Reduced from 64G
#SBATCH --job-name=test_job
...
```

### Example: Time Optimization

Check actual runtime:

```bash
$ sacct -j 123456 --format=JobID,Elapsed,Timelimit

       JobID    Elapsed  Timelimit
------------ ---------- ----------
123456         02:34:12   12:00:00
```

- Actual: 2.5 hours
- Requested: 12 hours
- Optimized request: 4:00:00 (add buffer for variability)

### Example: CPU Optimization

Check CPU efficiency:

```bash
$ sacct -j 123456 --format=JobID,AllocCPUS,TotalCPU,Elapsed

       JobID  AllocCPUS   TotalCPU    Elapsed
------------ ---------- ---------- ----------
123456                8   01:23:45   02:15:30
```

- 8 CPUs allocated
- Theoretical max: 8 × 2:15:30 = 18:04:00
- Actual usage: 1:23:45 (only 8% efficiency!)
- Problem: Job doesn't parallelize well
- Optimized request: 2 CPUs (test to verify)

## Lab-Wide Usage Tracking

### View Usage for Multiple Users

Track usage for your entire lab:

```bash
$ sacct -u user1,user2,user3 --starttime=2024-01-01 --format=User,JobID,JobName,Elapsed,AllocCPUS,State
```

### Generate Monthly Reports

```bash
# All lab users for January 2024
$ sacct -u user1,user2,user3 \
    --starttime=2024-01-01 \
    --endtime=2024-01-31 \
    --format=User,JobID,JobName,AllocCPUS,Elapsed,State \
    --state=COMPLETED
```

### Calculate Total CPU Hours

```bash
# Get CPU hours for a user in a time period
$ sacct -u $USER \
    --starttime=2024-01-01 \
    --format=CPUTimeRaw,AllocCPUS,Elapsed \
    --parsable2 \
    --state=COMPLETED \
    | tail -n +2 \
    | awk -F'|' '{sum+=$1/3600} END {printf "Total CPU hours: %.2f\n", sum}'
```

## Exporting Data for Analysis

### CSV Export

Export job data to CSV for analysis in R, Python, or Excel:

```bash
$ sacct -u $USER \
    --starttime=2024-01-01 \
    --format=JobID,JobName,Partition,AllocCPUS,MaxRSS,Elapsed,State,ExitCode \
    --parsable2 > jobs_2024.csv
```

The `--parsable2` flag creates pipe-delimited output that's easy to import.

### Analyzing in R

```r
# Read job data
jobs <- read.csv("jobs_2024.csv", sep="|")

# Calculate memory efficiency
jobs$MaxRSS_MB <- as.numeric(gsub("K", "", jobs$MaxRSS)) / 1024
jobs$ReqMem_MB <- as.numeric(gsub("Gn", "", jobs$ReqMem)) * 1024
jobs$MemEfficiency <- jobs$MaxRSS_MB / jobs$ReqMem_MB

# Plot memory efficiency
hist(jobs$MemEfficiency,
     main="Memory Request Efficiency",
     xlab="Actual / Requested Memory")
```

### Analyzing in Python

```python
import pandas as pd
import matplotlib.pyplot as plt

# Read job data
jobs = pd.read_csv('jobs_2024.csv', sep='|')

# Filter completed jobs
completed = jobs[jobs['State'] == 'COMPLETED']

# Calculate statistics
print(f"Total jobs: {len(completed)}")
print(f"Total CPU cores used: {completed['AllocCPUS'].sum()}")

# Plot
completed['AllocCPUS'].hist(bins=20)
plt.xlabel('CPUs Allocated')
plt.ylabel('Number of Jobs')
plt.title('CPU Allocation Distribution')
plt.show()
```

## Useful sacct One-Liners

### Jobs that failed in the last week

```bash
$ sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) --state=FAILED --format=JobID,JobName,State,ExitCode
```

### Jobs that ran out of memory

```bash
$ sacct --starttime=$(date -d '30 days ago' +%Y-%m-%d) --state=OUT_OF_MEMORY --format=JobID,JobName,MaxRSS,ReqMem
```

### Jobs that timed out

```bash
$ sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) --state=TIMEOUT --format=JobID,JobName,Elapsed,Timelimit
```

### Top 10 memory-using jobs

```bash
$ sacct --starttime=$(date -d '30 days ago' +%Y-%m-%d) --format=JobID,JobName,MaxRSS | sort -k3 -hr | head -10
```

### Average runtime for a specific job name

```bash
$ sacct --starttime=$(date -d '30 days ago' +%Y-%m-%d) --name=myjob --format=Elapsed,State | grep COMPLETED
```

### Array job summary

```bash
$ sacct -j 123456 --format=JobID,JobName,State,MaxRSS,Elapsed | grep -E "^123456_"
```

## Best Practices

### Regular Analysis

- Review job statistics weekly to identify optimization opportunities
- Keep a log of typical resource needs for common workflows
- Share insights with lab members to improve overall efficiency

### Resource Right-Sizing Benefits

- **Faster queue times** - Smaller jobs often start sooner
- **Better cluster efficiency** - More jobs can run simultaneously
- **Fair share** - Don't monopolize resources unnecessarily

### Using Job Accounting for Troubleshooting

When a job fails:

1. Check job info: `sacct -j JOBID`
2. Check efficiency: `seff JOBID`
3. If currently running: `scontrol show jobid -dd JOBID`
4. Check output/error files
5. See [Troubleshooting Guide](../troubleshooting/job-errors.md) for solutions

## Common Patterns

### Create an Alias for Job Info

Add to your `.bashrc`:

```bash
alias jobinfo='seff'
```

Usage:

```bash
$ jobinfo 123456
```

### Script to Analyze Recent Jobs

```bash
#!/bin/bash
# analyze_jobs.sh - Show summary of recent jobs

echo "Jobs in last 7 days:"
sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) \
    --format=State --parsable2 --noheader | \
    sort | uniq -c | sort -rn

echo -e "\nFailed jobs:"
sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) \
    --state=FAILED,OUT_OF_MEMORY,TIMEOUT \
    --format=JobID,JobName,State,ExitCode
```

### Monitor Job in Real-Time

While a job is running, monitor resource usage:

```bash
# In one terminal - watch job status
watch -n 10 'squeue -j 123456 && scontrol show jobid 123456 | grep RunTime'

# In another terminal - watch output
tail -f slurm-123456.out
```

## Quick Reference

### Most Useful Commands

```bash
# Basic job history
sacct

# Specific job info
sacct -j 123456

# Job efficiency summary
seff 123456

# Running job details
scontrol show jobid -dd 123456

# Failed jobs last week
sacct --starttime=$(date -d '7 days ago' +%Y-%m-%d) --state=FAILED
```

## Next Steps

- [Batch Jobs](batch.md) - Learn to submit and manage batch jobs
- [Interactive Jobs](interactive.md) - Use interactive sessions
- [Resource Estimation](../best-practices/resource-estimation.md) - Optimize resource requests
- [Troubleshooting](../troubleshooting/job-errors.md) - Debug job failures

--8<-- "includes/getting-help.md"
