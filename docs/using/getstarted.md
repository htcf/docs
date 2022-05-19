# Your HTCF Account

## Account Creation

To request a user account on the HTCF, please send an email containing the WUSTLKey username and department ID (Workday 'CC' number), for billing purposes.

!!! Note
    As stated in the [WUSTL and HTCF Policies](../policies.md#account-usage), **accounts and passwords cannot be shared. All users must have their own account.**

## Logging In

**WUSTLKey credentials** are used for authentication ([http://wustlkey.wustl.edu/](http://wustlkey.wustl.edu))

The login server, `login.htcf.wustl.edu` is accessible via ssh.  

!!! Note
    As stated in the [WUSTL and HTCF Policies](../policies.md#account-usage), **accounts and passwords cannot be shared. All users must have their own account.**

## First things first...

Before using the HTCF, it's important to read through and understand:

1. The [HTCF Storage](../storage/index.md)

2. Using [software](../software.md) on the HTCF

## Data & Data Storage

### Home Directories
Home directories are a small, fixed amount of storage per account.  They are kept on fault-tolerant storage and frequent snapshops are taken to prevent accidental data loss.  Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.  
[More info...](../storage/index.md#hds)

### Long Term Storage
LTS is used to store raw and "finished" project data.  The LTS directories are not available on the cluster nodes.  It is kept on fault-tolerant storage with snapshops.  
[More info...](../storage/index.md#lts)

### High Throughput Storage
High Throughput Storage is a distrubuted file system able to handle tens of GBs/sec of total throughput.

[More info...](../storage/index.md#hts)


### GUI Software

!!! Note
    As the HTCF is primarily a batch queuing system for high-throughput processing of data, use of GUI applications are not directly supported.  While use of GUI applications is possible using X forwarding, this can sometimes require significant desktop preparation and configuration which is beyond the scope of support.


## Jobs

### Resources

The number of CPUs and MBs of RAM per node can be found using the Slurm sinfo command:

    $ sinfo -N -p general -o '%n %c %m'

### Interactive 

Interactive sessions are for running interactive scripts, vizualization, any tasks that are too computational intensive to run on the login node not submitted via sbatch.  The defaults are: 1 CPU core, 1 GB RAM, and a time limit of 8 hours.

!!! Note
    The HTCF is primarily a batch queuing system.

    Interactive jobs are meant to function as daily workspaces.
    Because interactive jobs are by their nature, inefficient, they are not meant to be running continuously for more than 1 day.

    When using interactive tools such as rstudio or jupyter, please make sure the jobs are using the "interactive" queue  (using sbatch/srun parameters `-J interactive -p interactive`)

    **Jobs using interactive tools that are not in the interactive queue will be subject to cancellation in order to free up resources for batch jobs.**

    Tools such as Rscript can be used to run R programs in a batch fashion.
    It appears that jupyter notebooks can also be run in a [batch fashion](http://tritemio.github.io/smbits/2016/01/02/execute-notebooks/).

    Thanks for helping to ensure fairness for all folks on the HTCF.

An interactive session can be started using the Slurm `srun` command:

    $ srun --mem-per-cpu=<MBs> --cpus-per-task=<num> -J interactive -p interactive --pty /bin/bash -l

### Batch Job Submission

 * Determine resources
 * Create Job File
 * Create sbatch file with required resources
 * Submit
 * Monitor

#### Workflow

Jobs typically follow a generic workflow.

 1. Preprocessed Raw Data Enters LTS
 2. Raw Data is copied to scratch for processing
 3. Post processed data is copied to LTS
 4. Intermediate data generated in Step 2 is removed

#### Sbatch Examples

Create a job script (myjob.sbatch):
~~~~bash
#!/bin/bash

#SBATCH --cpus-per-task=1
#SBATCH --mem=1G

eval $( spack load --sh <program> )

program /scratch/lab/files/ABC.fasta /scratch/lab/files/ABC.out
~~~~

Submit the sbatch script:

    $ sbatch myjob.sbatch

View the job in the queue:

    $ squeue

### GPUs

The HTCF currently has a small number of GPUs.

A GPU is accessible using the following slurm parameters:

~~~~{.language-bash}
#SBATCH -p gpu
#SBATCH --gres=gpu
~~~~

