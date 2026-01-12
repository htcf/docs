# Policies

## WUSTL Computer Use
[http://wustl.edu/policies/compolicy.html](http://wustl.edu/policies/compolicy.html)

## Account Usage
As stated in the above WUSTL Policy: *"Do not use the password of others or access files under false identity."*
Accounts and passwords cannot be shared. All users must have their own account.

## Account Renewal
HTCF user accounts are automatically renewed annually from the original activation date unless otherwise instructed.

## Account Removal
Home directories of expired accounts are removed 90 days after expiration.

## Storage Policies

### Scratch Data Cleaning

In order to ensure top performance of /scratch it is important to clean it regularly to remove stale data.  Therefore, the following weekly automated tasks are performed on /scratch:

- User files on scratch that have not been modified for more than **60 days** are **garbage collected** and placed in a "trash" location.
- After 30 days in the trash location, user files are **purged** from the system.  Once purged, there is no way files can be restored.

!!! note "Modification Time"
    "Modified" refers to the file's modification time (mtime). Reading a file does NOT reset this timer. To preserve files without changing their content, use `touch filename` to update the modification time.

Files needed for more than 60 days must be copied to an LTS bucket.

Garbage-collected files are stored in /scratch/trash/&lt;date_of_collection&gt;/.

Garbage-collected files can be restored by moving them out of this directory.

A list of garbage-collected files per user can be found in /scratch/trash/&lt;date_of_collection&gt;/filelists/&lt;username&gt;.

**The HTCF is not responsible for data loss from automated scrubs.  Labs are responsible for monitoring their files and transferring their data from scratch to long term storage.**

### Data Limits

Each member of the HTCF belongs to at least two Unix groups.  The primary group is your personal group, having the same name as your HTCF username.  The secondary group is the laboratory or similar entity that you are primarily associated with.

**Policy: Scratch user data limits**

- Size Limit - 2TB
- Inode Limit (Number of files) - 2,000,000

Example

**Username:**  johnsmith

To determine personal scratch usage:

~~~~
   $ storage-info
~~~~

## Login Node Policy

The HTCF login node is for editing files, submitting jobs, and moving data. File transfers, downloads, and light configuration tasks are acceptable. Computational work (running analysis scripts, processing data, software compilation) should be done in interactive or batch jobs. Computational processes on the login node may be terminated.  

## General Availability

Effort will be made to keep our resources available. Although the support personnel will do their best to keep the facility running at all times, we cannot guarantee to promptly resolve problems outside office hours, during weekends, and public holidays. Nevertheless, please notify us of whenever they arise.

## General Maintenance
 
Occasionally, it is necessary as part of maintaining a reliable service to update system software and replace faulty hardware. Sometimes it will be possible to perform these tasks transparently by means of queue reconfiguration in a way that will not disrupt running jobs or interactive use, or significantly inconvenience users. Some tasks however, particularly those affecting storage or login nodes, may require temporary interruption of service.

## Running Jobs

* Jobs that perform excessive I/O (writing to files more frequently than once per second, or reading/writing to `/home` repeatedly) may be terminated. Use `/scratch` for all job I/O operations.
* Jobs using more CPUs or memory than requested will be terminated.
* Request memory accurately. Under-requesting causes job crashes. Over-requesting prevents other users from running jobs.
