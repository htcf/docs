# Your HTCF Account

## Account Creation

To request a user account on the HTCF, please send an email containing the WUSTLKey username and department ID (Workday 'CC' number), for billing purposes.

## Logging In

**[WUSTLKey credentials](http://wustlkey.wustl.edu)** are used for authentication.

The login server, `login.htcf.wustl.edu` is accessible via ssh.  

As stated in the [WUSTL and HTCF Policies](../policies.md#account-usage), **accounts and passwords cannot be shared. All users must have their own account.**

## First things first...

Before using the HTCF, it's important to read through and understand:

1. The [HTCF Storage](../storage/index.md)

2. Using [software](../software.md) on the HTCF

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

The HTCF currently has a small number of GPUs, currently 6 NVIDIA A100 80GB and 2 V100 32GB.
The default time limit of a GPU job is set to 8 hours.

A GPU is accessible using the following slurm parameters:

    -p gpu --gpus=<num>

In an sbatch:

~~~~{.language-bash}
#SBATCH -p gpu
#SBATCH --gpus=<num>
~~~~

You can verify the GPU is being utilized by the job with the nvidia-smi command, this example runs within your job allocation:

!!! Note
    The job must be submitted as a batch job to utilize the nvidia-smi command. 

~~~~{.language-bash}
$ srun --ntasks-per-node=1 --jobid=12345 nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.60.13    Driver Version: 525.60.13    CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA A100 80G...  Off  | 00000000:17:00.0 Off |                    0 |
| N/A   39C    P0    62W / 300W |      0MiB / 81920MiB |     23%      Default |
|                               |                      |             Disabled |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0                12043      C   guppy_basecaller                 4753MiB |
+-----------------------------------------------------------------------------+
~~~~
