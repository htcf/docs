# Storage

The HTCF provides three types of storage:


### HDS: Home Directory Storage (/home)

Home directories are a small, fixed amount of storage per account.  They are kept on fault-tolerant storage and frequent snapshops are taken to prevent accidental data loss.  Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.

Each HTCF account has a 20GB home directory.  This directory can be used to store scripts, development tools, etc.  Home directories are located in `/home/<WUSTLKEY_ID>` and are available on all nodes.

!!!Note
    Home directory space is not a high-speed resource like /scratch space.

    Please keep home directory access to a minimum in batch jobs.

By default, home directories will not be readable or writeable by other users of HTCF.  Feel free to change this default, if desired.

To check home directory usage:

    $ du -sh $HOME

### LTS: Long Term Storage (/lts)

Long term storage is lab project space to store raw sequencing and completed data, the directories are not available on nodes for computational use.  It is available in terabyte increments billed monthly.  It is kept on fault-tolerant storage with snapshops.  Copies of the latest daily snapshots are kept offsite for disaster recovery purposes.  To see how much space your lab is using, use the example command below, replace lab with your labname.  

To check LTS usage:

    $ df -h /lts/<lab_name>/<bucket_name>


### HTS: High Throughput Storage (/scratch)

High Throughput Storage is a distrubuted file system able to handle tens of GBs/sec of total throughput.  This storage is *temporary scratch space* and is **not** backed up.  Once data is removed from /scratch, it cannot be recovered.

*Data stored in /scratch is subject to the [Scratch Data Cleaning Policy](/policies/#scratch-data-cleaning).*

Jobs utilize this space for inputs and outputs to provide the best performance possible.  Running jobs that read/write from the home directory will cause slowness and login issues for all users.

!!!Note
    Users will be asked to clean up older files on /scratch if it is needed to improve system performance.

The best use of the HTS is to use a workflow similar to the following:

1.  Copy starting (raw) data from LTS to HTS.
2.  Submit jobs to the cluster that process the data, creating intermediate and/or finished data.
3.  Copy the finished data (and job files used to create that data) over to LTS.
4.  Remove all working data from HTS

Results that are generated on this storage need to be promptly copied to LTS. 

There is a quota of 2TB per user in /scratch to prevent the filesystem from filling up.  At >85% /scratch can become very slow.  To check the amount of space being used, use the following command:

    $ beegfs-ctl --getquota --uid $USER

#### Quota Increase Requests

!!!Note
    A quota increase is not garaunteed.  If excess capacity is not available, a quota increase cannot be granted.

If excess capacity is available.  A temporary increase in the scratch quota can be requested.  To request more scratch space, email following information:

1. Reason for the increase
2. Amount of additional space
3. Duration additional space will be required


#### Recommendations

* No important source code, scripts, libraries, executables should be kept in `/scratch`
* Do not make symlinks from your home directory to folders in `/scratch`

### Sharing Files Publicly

Globus can be used to share data from LTS or Home directories.  The Globus "Collection" is called "HTCF Home Directories". Please see [the globus documentation for more information](https://docs.globus.org/how-to/get-started/)

For long term hosting of publicly accessible data, please contact WUSTL IT.

### Copying Files Using Rsync

Using rsync to transfer to scratch and LTS is recommended.  Rsync can resume failed copies, be re-run to ensure all of the data has been transferred, and will also transfer incremental changes.  This will save a substantial amount of time if you need to verify that all files have been successfully copied.

When using this command, please note that the absense of a trailing slash means the directory, with a trailing slash means the contents of that directory.  Here are a few examples:
[More info...](https://stackoverflow.com/questions/31278098/slashes-and-the-rsync-command)

### Disk Quota Exceeded Errors

If you receive `disk quota exceeded` messages, please check each storage location to ensure you have enough disk space available.
