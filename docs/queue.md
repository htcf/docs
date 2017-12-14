### Queuing System - Slurm
* * *

The HTCF utlizes the Simple Linux Utility for Resource Management (Slurm).  Slurm documentation can be found at [http://slurm.schedmd.com/documentation.html](http://slurm.schedmd.com/documentation.html).

### Job Submission

There are two type of Slurm jobs, batch and interactive.

#### Batch Jobs

The steps needed to submit batch jobs are:

1.  Create a "job script".  This is the file that actually does the work of the job.
2.  Create a "sbatch script".  This file sets the Slurm parameters and prepares the environment for the job script.
3.  Launch the job using the "sbatch" command



**salloc** - Obtains a job allocation.
    
    --cpu-per-task - Number of CPUs required per task
    --dependency=<state:jobid>
    --job-name=<name>
    --mem=<MB> Memory required per node.
    --mem-per-cpu=<MB> Memory required per allocated CPU.
    
**sbatch** - Submits batch scripts for execution.

**srun** - Obtains job allocation and executes an application.


### Partitions
Partition    |  Max Memory | Duration    | Max CPUs in Queue |
:----------- |  :----------: | :---------: | :---------------: |
debug        |      256GB    |  no limit   |         1824      |
interactive  |      256GB      |   8 hours   |         1824      |

### Job Management

scancel

squeue

sinfo

### Job Accounting

sacct is the command to view all previously run job information.  You can get a list of viewable fields by running the command

~~~~
sacct -e
~~~~

To view a past jobs maximum used memory and duration
~~~~{.language-bash}
sacct -j JOBID --format=JobID,JobName,MaxRSS,Elapsed
~~~~

Scontrol can be used to view detailed information about your running job including the job script that was submitted.  Please send the output of this command if your currently running job is having issues. 
~~~~
~$ scontrol show jobid -dd 846115
JobId=846115 JobName=sleep.sh
   UserId=ericmartin(1002) GroupId=ericmartin(1002)
   Priority=3070 Nice=0 Account=htcfadmin QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=1 Reboot=0 ExitCode=0:0
   DerivedExitCode=0:0
   RunTime=00:00:09 TimeLimit=UNLIMITED TimeMin=N/A
   SubmitTime=2016-03-09T09:47:08 EligibleTime=2016-03-09T09:47:08
   StartTime=2016-03-09T09:47:09 EndTime=Unknown
   PreemptTime=None SuspendTime=None SecsPreSuspend=0
   Partition=debug AllocNode:Sid=n082:21380
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

module load bowtie2

sleep 100
~~~~

More information on usage is available at [http://slurm.schedmd.com/sacct.html](http://slurm.schedmd.com/sacct.html).

* * *

More information available here:
* [http://slurm.schedmd.com/overview.html](http://slurm.schedmd.com/overview.html)
* [http://slurm.schedmd.com/tutorials.html](http://slurm.schedmd.com/tutorials.html)
* [http://slurm.schedmd.com/faq.html](http://slurm.schedmd.com/faq.html)
