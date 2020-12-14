# Software on the HTCF

When it comes to software, the HTCF starts as a blank slate for each lab.  However, this doesn't mean labs can't get up and running in a matter of minutes.

Each lab has its very own dedicated space to install and manage software.  This [reference space](storage/ref.md) is located in [/ref/*\<lab\>*/software](storage/ref.md#software).

After software is installed a [software modules](#software-modules) system is available for convenient way to *load* and *unload* software, as needed. 

## Self-service Installation

Software building and installation on the HTCF is primarily self-service.

Labs are free to use their [/ref software](storage/ref.md) directory to install software using whatever means is most comfortable.

At the lab level, use of [Spack](#spack) to install common software is encouraged.  [Virtual environments](#virtual-environments) can also be used if the software is well suited.

### Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

#### Spack Setup for Your Lab or Group

To create a lab instance of the spack package manager:

1.  Download and untar spack release into `/ref/<lab>/software`

2.  Rename (or make a symlink from) `/ref/<lab>/software/spack-VERSION` to `/ref/<lab>/software/spack`.

3.  Logout and log back in.  This will ensure the `spack` command is available in the PATH.

#### The software I want is not available through Spack

When needed software is not readily accessible via Spack, there are a few options.

1. **Following the installation instructions from the software creator.**
 
   Sometimes, this can be very quick and straightforward.
   
   Sometimes, this can be very painful.
   
   Sometimes, it can be a good idea to pass judgement on the quality of software by the quality of the installation process and documentation. :wink:

2. **Create a custom spack package**

   Spack can be a wonderful tool for creating and maintaining software. [Plenty of documentation is provided for creating and maintaining custom packages](https://spack.readthedocs.io/en/latest/packaging_guide.html), though a firm understanding of python is needed.


### Virtual Environments

(links to python virtual environments go here)


## Using the software

### Software Modules

### Using Spack-generated modules

Example sbatch job

```
#!/bin/bash

source <(spack module lmod loads 
```
