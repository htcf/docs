# Software on the HTCF

--8<-- "includes/coming_soon.md"

When it comes to software, the HTCF starts as a blank slate for each lab.  However, this doesn't mean labs can't get up and running in a matter of minutes.

Each lab has its very own dedicated space to install and manage software.  This [reference space](storage/ref.md) is located in [`/ref/<lab>/software`](storage/ref.md#software).

After software is installed, a [software modules system](#software-modules-lmod) is available as a convenient way to *load* and *unload* software, as needed. 

## Self-service Installation

Software building and installation on the HTCF is primarily self-service.

Labs are free to use their [/ref software](storage/ref.md) directory to install software using whatever means is most comfortable.

At the lab level, **use of [Spack](#spack) to install common software is encouraged**.  [Virtual environments](#virtual-environments) can also be used if the software is well suited.

### Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

#### Spack Setup for Your Lab or Group

To create a lab instance of the spack package manager:

1.  Download and untar a [spack release](https://github.com/spack/spack/releases) into `/ref/<lab>/software`

2.  Rename (or make a symlink from) `/ref/<lab>/software/spack-VERSION` to `/ref/<lab>/software/spack`.

3.  Logout and log back in.  This will ensure the `spack` command is available in the `$PATH`.

#### Installing software via Spack

Thousands of software packages are available for install using Spack using [`spack list`](https://spack.readthedocs.io/en/latest/basic_usage.html#listing-available-packages) provides that list:

    $ spack list

Often, installation is as simple as:

    $ spack install <software_name>

Multiple versions of software can be installed at the same time.  More information about software installation can be found in the [spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#installing-and-uninstalling).


#### What if the software I want is not available through Spack

When needed software is not readily accessible via Spack, there are a few options.

1. **Following the installation instructions from the software creator.**
 
     Sometimes, this can be very quick and straightforward.
   
     Sometimes, this can be very painful.
   
     Sometimes, it can be a good idea to pass judgement on the quality of software based on the quality of the installation process and documentation. :wink:

2. **Create a custom spack package**

   Spack can be a wonderful tool for creating and maintaining software. [Plenty of documentation is provided for creating and maintaining custom packages](https://spack.readthedocs.io/en/latest/packaging_guide.html), though a firm understanding of python is needed.


### Virtual Environments

Some programming languages, such as Python, provide a virtual environment mechanism which allow you to manage separate package installations for different projects.

####  Python

Using python virtual environments can be a convenient way to install python software not readily installable via slack.

##### Preparing

After you've [set up spack for the lab](#spack) and used it to install the latest python, pip, and wheel:

    $ spack install python
    $ spack install py-pip
    $ spack install py-wheel

The modules can be loaded:

    $ module load python py-pip py-wheel

--8<-- "includes/module_refresh.md"

##### Creating

The best explanation of creating and using python virtual environments can be found [in the official documentation](https://packaging.python.org/guides/installing-using-pip-and-virtual-environments/#creating-a-virtual-environment)

#### R considerations

### Manual Installation

Sometimes it's just easier to follow the installation steps provided by software creator.

If the software depends on other software, it might be that the dependency software could be installed via [spack](#spack).

For example, if a piece of software is not available via Spack, but requires samtools to be installed:

    # Install samtools via spack
    $ spack install samtools

    # load samtools before installation
    $ module load samtools
    
    (proceed with the manual installation)

!!! Remember
    The best place to install software is your [reference storage](storage/ref.md) `/ref/lab/sofware`.

After manual software installation, it's good practice to then create a [module file](#manual-module-files)

## Using the software

### Software Modules - LMOD

A *software modules system* may sound fancy, but it's really just a small set of convenience functions that help to make software readily available to your [shell environment](https://en.wikipedia.org/wiki/Unix_shell) by simplifying the modification of [environment variables](https://en.wikipedia.org/wiki/Environment_variable#True_environment_variables) such as [`$PATH`](https://en.wikipedia.org/wiki/Path_\(computing\)) and `$LD_LIBRARY_PATH`.

The modules system [Lmod](https://lmod.readthedocs.io/en/latest/) is available on the HTCF and any [spack installed packages](#using-spack-generated-modules) should automatically create the [module lua file](https://lmod.readthedocs.io/en/latest/015_writing_modules.html) needed for proper module installation.

--8<-- "includes/module_refresh.md"

#### Spack-generated modules

Example sbatch job

```
#!/bin/bash

source <(spack module lmod loads 
```

#### Manual module files

Module files that are manually created go in your [reference storage](storage/ref.md) in `/ref/<lab>/modulefiles/custom`.

[The lmod documentation](https://lmod.readthedocs.io/en/latest/015_writing_modules.html) is the best place to learn about creating module files.
