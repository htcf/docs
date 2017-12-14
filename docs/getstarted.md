### Getting Started
* * *

### Getting an Account

To request an account on the HTCF computational cluster, please email for more information.


### Access

 * The HTCF is directly accessible via WUCON (wired network on the Medical campus) or WUSM-Secure wireless
 * External Access/Off Campus/Eduroam will require use of the WUSM VPN ([https://msvpn.wusm.wustl.edu](https://msvpn.wusm.wustl.edu)), please email for instructions
 * Uses WUSTL KEY for authentication ([http://wustlkey.wustl.edu/](http://wustlkey.wustl.edu))

### Using the HTCF Submit Node
* * *

The login server is htcf.wustl.edu

~~~~{.language-bash}
~$ ssh WUSTL_KEY_ID@htcf.wustl.edu
~~~~

If you’re using an X server locally, you can connect with -Y for *some* graphical applications.
Once logged in, you can create an interactive workspace or submit batch jobs.

~~~~{.language-bash}
~$ ssh -Y WUSTL_KEY_ID@htcf.wustl.edu 
~~~~

### Data & Data Storage
* * * 
#### Home Directories
Each HTCF user account has 20GB home directory. This directory can be used to store scripts, development tools, etc. Home directories are located in "/home/WUSTL_KEY_ID" and are available on all nodes. They are kept on fault-tolerant storage and frequent snapshops are taken to prevent accidental data loss. Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.

#### Long Term Storage
LTS is used to store raw and "finished" project data.  The LTS directories are not available on the cluster nodes.  Long term storage is lab project space, available in terabyte increments. It is kept on fault-tolerant storage with snapshops. Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.

#### High Throughput Storage
HTCF high throughput storage is a large, distrubuted file system able to handle more than 6GB/second of total throughput. The HTS is scratch space and is not backed up. High thoughput storage is temporary. We absolutely cannot recover anything in /scratch once removed.

*Data stored in /scratch is subject to the [Scratch Data Cleaning Policy](https://htcf.wustl.edu/docs/policies/#scratch-data-cleaning).

*More information is available on our [Data & Data Storage Page](storage.md)*

### Software

List the software available on HTCF
~~~~{.language-bash}
module avail
~~~~

To load the software:
~~~~{.language-bash}
module load bowtie2
~~~~

To request additional software modules, please contact us.

*More information is available on our [Software Page](software.md)*

### Workflow

Jobs typically follow a generic workflow.

 * A - Preprocessed Raw Data Enters LTS
 * B - Raw Data is copied to scratch for processing
 * C - Post processed data is copied to LTS
 * D - Intermediate data generated in Step B is removed


### Partitions


Partition    |  Max Memory | Duration    | Max CPUs in Queue |
:----------- |  :----------: | :---------: | :---------------: |
debug        |      256GB    |  no limit   |         2250      |
interactive  |      256GB      |   8 hours   |         2250      |


### Jobs

* * *

#### Interactive 

Interactive sessions are for running interactive scripts, vizualization, any tasks that are too computational intensive to run on the login node not submitted via sbatch.  The defaults are: 1 CPU core, 1 GB RAM, and a time limit of 8 hours.

You can create an interactive session by running:

~~~~{.language-bash}
~$ interactive
~~~~

or you can modify the following command to fit your requirements:

~~~~{.language-bash}
srun --mem=2000 --cpus-per-task=1 -J interactive -p interactive --pty /bin/bash -l
~~~~

#### Batch Job Submission

 * Determine resources
 * Create Job File
 * Create sbatch file with required resources
 * Submit
 * Monitor

### Sbatch Examples

Create a job script (myjob.sbatch):
~~~~bash
#!/bin/bash

#SBATCH --cpus-per-task=1
#SBATCH --mem=1G

ml program

program /scratch/lab/files/ABC.fasta /scratch/lab/files/ABC.out
~~~~

Submit the sbatch script.

~~~~bash
sbatch myjob.sbatch
~~~~

View the job in the queue

~~~~
user@htcf:~$ squeue

JOBID PARTITION NAME USER ST TIME NODES NODELIST(REASON)
106  debug  example  example R   0:13   1    n067
~~~~

