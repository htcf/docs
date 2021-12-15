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

- User files on scratch that have not been modified for more than **60 days** are **garbage collected** and placed in a “trash” location.
- After 30 days in the trash location, user files are **purged** from the system.  Once purged, there is no way files can be restored.

Please ensure that any files you need for more than 60 days are safely copied to an LTS bucket.

Garbage-collected files are stored in /scratch/trash/&lt;date_of_collection&gt;/.

You can restore your garbage-collected files by moving them out of this directory.

A list of your garbage-collected files can be found in /scratch/trash/&lt;date_of_collection&gt;/filelists/&lt;username&gt;.

**The HTCF is not responsible for data loss from automated scrubs.  Labs are responsible for monitoring their files and transferring their data from scratch to long term storage.**

### Data Limits

Each member of the HTCF belongs to at least two Unix groups.  The primary group is your personal group, having the same name as your HTCF username.  The secondary group is the laboratory or similar entity that you are primarily associated with.

**Policy: Scratch user data limits**

Size Limit - 2TB
Inode Limit (Number of files) - 2,000,000

Example

**Username:**  johnsmith

To determine your personal scratch usage:

~~~~
   $ beegfs-ctl --getquota --uid johnsmith
~~~~

## Login Node Policy

The HTCF login node is to be used for job composition, software installation, and staging of job data.  Any computational processes found running longer than 30 minutes can be terminated.  

## General Availability 

Effort will be made to keep our resources available. Although the support personnel will do their best to keep the facility running at all times, we cannot guarantee to promptly resolve problems outside office hours, during weekends, and public holidays. Nevertheless, please notify us of whenever they arise.

## General Maintenance
 
Occasionally, it is necessary as part of maintaining a reliable service to update system software and replace faulty hardware. Sometimes it will be possible to perform these tasks transparently by means of queue reconfiguration in a way that will not disrupt running jobs or interactive use, or significantly inconvenience users. Some tasks however, particularly those affecting storage or login nodes, may require temporary interruption of service.

## Running Jobs

* Jobs that improperly perform excessive I/O, or utilize unreserved CPU time will be terminated. 
* Please be accurate when requesting memory, not requesting enough memory will result in your process crashing, requesting too much memory will prevent other users from running jobs.

