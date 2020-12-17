# Q1 - 2021 Changes

## Operating System

All nodes (including the login server) will be upgraded from Ubuntu 16.04 to Ubuntu 20.04.

## Software

The system-wide software and modules in directories /opt/apps and /opt/htcf will be frozen (read-only) and use of software in these directories is **deprecated.**

Replacing these directories will be a form of LTS call /ref, lab reference space located in `/ref/<lab>/software` and `/ref/<lab>/modulefiles`.  Each lab will receive 1TB of space in /ref for storage of reference material including software and reference sequence databases.  [More information about /ref can be found here.](storage/ref.md)

Software installation will now be self-service.  To aid in this, [documentation has been added](software_new.md) and infrastructue is in place for lab-managed [spack installations](software_new.md#spack).  Thousands of packages are available for installation using [spack](https://spack.readthedocs.io/).

!!! Help
    We are here to help with the migration from the deprecated modules system to your labs' new /ref software.  Rest assured that the deprecated software will not be removed until migration is complete.

## LTS


