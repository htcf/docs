# Storage at the HTCF

The HTCF provides three types of storage:


### HDS: Home Directory Storage (/home)

Home directories are a small, fixed amount of storage per account.  They are kept on fault-tolerant storage and frequent snapshops are taken to prevent accidental data loss.  Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.

Each HTCF user account has 20GB home directory.  This directory can be used to store scripts, development tools, etc.  Home directories are located in "/home/**WUSTL_KEY_ID**" and are available on all nodes.

*Please keep your home directory access to a minimum in your jobs.*

By default, home directories will not be readable or writeable by other users of HTCF.  If you wish to change this default, you can use the chmod command to give the appropriate level of access to your lab members or other collaborative users.

You can check the amount of home directory with the `du` command.  It may take a few moments for the command to complete.  For example:

~~~~{.language-bash}
du -csh $HOME
10G /home/<username>
10G total
~~~~


* * *

### LTS: Long Term Storage (/lts)

Long term storage is lab project space to store raw sequencing and completed data, the directories are not available on nodes for computational use.  It is available in terabyte increments billed yearly.  It is kept on fault-tolerant storage with snapshops.  Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.  To see how much space your lab is using, use the example command below, replace lab with your labname.  

~~~~{.language-bash}
~$ df -h $( grep -v '^#' /etc/fstab | awk '{ print $2 }' | grep '/lts/mylab' )
Filesystem                                   Size  Used Avail Use% Mounted on
/dev/mylab/seq                           1.0T  165G  860G  17% /lts/lab/seq
/dev/mylab/data1                          10T  6.4T  3.7T  64% /lts/lab/data1
/dev/mylab/data2                          15T   14T  1.1T  93% /lts/lab/data2
~~~~

* * *

### HTS: High Throughput Storage (/scratch)

HTCF high throughput storage is over 700TB, BeeGFS distrubuted file system able to handle more than 19GB/second of total throughput.  This storage is *temporary scratch space* and is **not** backed up.  We absolutely **cannot** recover anything in /scratch once removed.

*Data stored in /scratch is subject to the [Scratch Data Cleaning Policy](https://htcf.wustl.edu/docs/policies/#scratch-data-cleaning).

Jobs must utilize this space for inputs and outputs to provide the best performance possible.  Running jobs that read/write from your home directory will cause slowness and login issues for all users.

We will ask users to clean up older files on /scratch if it is needed to improve system performance.

The best use of the HTS is to follow a workflow similar to this:

1.  Copy your starting (raw) data from LTS over to HTS.
2.  Submit jobs to the cluster that process your data, creating intermediate or final data.
3.  Copy your final data (and job files used to create that data) over to LTS.
4.  Remove all working data from HTS

Results that are generated on this storage need to be promptly copied to LTS. 

We have a general quota of 2TB per user in /scratch to prevent the filesystem from filling up.  At ~95% /scratch becomes very slow, at 100%, all operations on the cluster stop.  To check the amount of space you're using in /scratch, you can use the command below as an example, replace the support username with your username.

~~~~{.language-bash}
~$ beegfs-ctl --getquota --uid support
      user/group     ||           size          ||    chunk files
     name     |  id  ||    used    |    hard    ||  used   |  hard
--------------|------||------------|------------||---------|---------
       support|  1000|| 325.48 GiB |   2.00 TiB ||        0|        0
~~~~


### Quota Increase Requests

To temporarily increase your scratch quota, please email with the following information:

1. Reason for the increase
2. Amount of additional space
3. Duration additional space will be required


### Recommendations
* No important source code, scripts, libraries, executables should be kept in /scratch
* Do not make symlinks from your home directory to folders in /scratch
* Work with large files before breaking up into smaller files

### Publishing Files

To temporarily generate a URL for a collaborator to download files, you can use the following commands.  If there are any errors, please ensure the directory and file permissions on the files you wish to share allow all users read access.  The URL will be valid for 90 days, if a shorter duration is required please contact us.

To share a directory:
~~~~
~$ module load htcf
~$ serve /path/to/directory
~~~~
To share a specific file:
~~~
~$ module load htcf
~$ serve /path/to/file
~~~

For long term hosting of publicly accessible data, please contact your department IT.

### Copying Files Using Rsync

Using rsync to transfer to scratch and LTS is recommended.  Rsync can resume failed copies, be re-run to ensure all of the data has been transferred, and will also transfer incremental changes.  This will save a substantial amount of time if you need to verify that all files have been successfully copied.

When using this command, please note that the absense of a trailing slash means the directory, with a trailing slash means the contents of that directory.  Here are a few examples:

~~~~
~$ rsync -aHv --progress /directory/to/transfer /destination/directory/location/
~~~~

The above example would put the directory named "transfer" into the directory named "location"

~~~~
~$ rsync -aHv --progress /directory/to/transfer/ /destination/directory/location/
~~~~

The above example would put the contents of the directory named "transfer" into the directory named "location".

### Disk Quota Exceeded Errors

If you receive `disk quota exceeded` messages, please check each storage location to ensure you have enough disk space available.  Examples of the `du`, `df`, and `beegfs-ctl` commands to view utilization are provided for each storage type above. 

