# Q1 - 2021 Changes

The beginning of 2021 will see quite a few (pretty cool, we think) changes within the HTCF.  Much effort has been made to ease the transitions that are necessary to make these changes.  Please contact us with any questions or concerns.

## What's changing?

There are essentially 4 changes (with more detail below that): 

**OS Upgrade**
:   While the HTCF *backend* servers have been continually upgraded over the years, the cluster nodes and login server are long overdue for an OS upgrade.

    !!! Note
        Many of the current software modules rely on system libraries.  These libraries will be upgraded when the OS is upgraded.
    
        It it likely that some software packages will have errors due to the change in system libraries.  Don't worry though, we've got a solution for that.... :point_down:

**Self-Service Software** :minibus:
:   As the HTCF user base grows, so do the software installation needs.  Maintaining the software and modules for all labs can leave a web of dependency, version, and architecture conflicts that are difficult to untangle.  The most sensible, sustainable, and flexible approach going forward is for HTCF support to move over to the passenger seat and leave the driving to the labs.  Labs should now feel empowered to install and manage their own software (with us in the passenger seat to help navigate).

    Infrastructure is in place to ease in this, including an **initial 1TB of space per lab** for software storage called /ref.  Speaking of which.... :point_down:

**/ref**
:   A new form of LTS storage called /ref will be introduced.  /ref is storage space for software and reference databases (think NCBI databases or software-provided reference sequences).  Each lab will be given an initial 1TB of reference space, this space can be expanded at LTS prices.  Speaking of new forms of LTS.... :point_down:

**LTOS** :cloud:
:   **Object Storage LTS** as been quietly available for a while, but now it's official.  This is the *cloud storage* form of local (ie. high speed) LTS (ie. not Amazon) that uses the [Amazon S3 API](https://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html) for access.  It's well suited for large amounts of data that don't change much (e.g. raw sequencing runs).

    **Unlike traditional LTS**:

    - LTOS buckets don't have a size limit
    - They're available from each node (no more login server copying back and forth!)  Aggregate transfer rates can be much faster than traditional /lts.
    - Before you get too excited, there's a learning curve...because it's more like cloud storage, you won't find it in a traditional filesystem directory.  With the right use case though, we're confident that the learning curve will be worth it!

    More info can be found [here]().


### OS Upgrade

All nodes (including the login server) will be upgraded from Ubuntu 16.04 to Ubuntu 20.04.

!!! Warning
    Many of the current software modules rely on system libraries.  These libraries will be upgraded when the OS is upgraded.

    It it likely that some software packages will have errors due to the change in system libraries.


| Date | Change | % nodes containing upgraded OS |
| - | - | - |
| January XX | New slurm partition **`beta`** will be made available for trying out jobs on a small number of nodes that use the upgraded OS. | 5% |
| January XX | Login Server will be upgraded to new OS | 5% | 
| January XX - XX | Rolling OS upgrades of idle nodes (up to 50% of nodes) will continue. | 5% - 50% |
| January XX | Slurm partition **`general`** will be renamed **`legacy`** and **`beta`** will be renamed to **`general`**. | 50% |
| January XX | Slurm partition **`general`** will become the default interactive and batch job partition. | 50% |
| January XX - XX | Rolling OS upgrades of idle nodes (up to 95% of nodes) will continue. | 50% - 95% |
| February XX | | 100% |

### Self-service software and modules

The system-wide software and modules in directories `/opt/apps` and `/opt/htcf` will be frozen (read-only) and use of software in these directories is **deprecated.**

Replacing these directories will be a form of LTS call /ref, lab reference space located in `/ref/<lab>/software` and `/ref/<lab>/modulefiles`.  Each lab will receive 1TB of space in /ref for storage of reference material including software and reference sequence databases.  [More information about /ref can be found here.](storage/ref.md)

Software installation will now be self-service.  To aid in this, [documentation has been added](software_new.md) and infrastructue is in place for lab-managed [spack installations](software_new.md#spack).  Thousands of packages are available for installation using [spack](https://spack.readthedocs.io/).

!!! Help
    We are here to help with the migration from the deprecated modules system to your labs' new /ref software.  Rest assured that the deprecated software will not be removed until migration is complete.

| Date | Change | % nodes containing upgraded OS |
| - | - | - |
| January XX | Each lab given 1TB of /ref space ([**availabe on all OS-upgraded nodes**](#cluster-wide-operating-system-upgrade)) to begin testing: <br/> `srun --mem=X000 -c X -p beta --pty /bin/bash -l`

 | 

### /ref

!!! Note
    Keep in mind, while /ref is a form of LTS, it **is not /lts** (e.g. it isn't backed up)


### LTOS

