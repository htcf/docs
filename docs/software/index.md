# Software

The HTCF starts as somewhat of a blank slate for each lab.  However, this doesn't mean labs can't get up and running in a matter of minutes!

Each lab has its own dedicated space to install and manage software.  This reference space is located in `/ref/<lab>/software`.

Software building and installation on the HTCF is primarily self-service.

Labs are free to use their `/ref` software directory to install software using whatever means is most comfortable.

At the lab level, options for installing common software include [Spack](#spack) and virtual environments, depending on the software requirements.

## Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

!!! Note
    Software compilation and installation should be done in interactive jobs, not on the login node. Use `srun` to start an interactive session before running `spack install`.

### Tutorial

See [the official spack tutorial](https://spack-tutorial.readthedocs.io/en/latest/)

### Initialization

To create a lab (or personal) instance of the spack package manager:

1.  Follow the installation instructions in the [spack documentation](https://spack.readthedocs.io/en/latest/), placing spack in a directory under `/ref/<lab_name>/software`

2.  As a convenience, if the path `/ref/<lab_name>/software/spack/bin` exists, it will automatically be added to `$PATH` on login.   You may wish to make a symlink from `/ref/<lab_name>/software/<my_spack_dir_name>` to `/ref/<lab_name>/software/spack`, to take advantage of the automatic `$PATH` addition.  Alternatively, feel free to place spack instances in other locations (and add those instance `bin/` paths to your `$PATH` in `$HOME/.bashrc`).  

3.  After installing spack (and updating your `.bashrc` if necessary), logout and log back in.  This will ensure the spack command is available in the `$PATH`.

### Installing Software

see [the official spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#cmd-spack-install)

!!!Warning
    Please install software from within a Slurm job.

!!!Note
    Some compiling requires large amounts of RAM.  Using `--mem-per-cpu` with >= 4G is sometimes needed.

    **In the case of software that requires the "qt" package, more than --mem-per-cpu=10G could be needed.**

!!!Note
    When installing within a slurm job, be sure to tell Spack how many CPUs are available.

    For example, after [getting an interactive session](../getting-started.md#interactive-sessions):

        spack install -j ${SLURM_CPUS_ON_NODE} ..... 

!!!Note
    Some software can take an extremely long time to install (such as qt and llvm).  In these cases, an sbatch job will be needed rather than an interactive job:
    
        #!/bin/bash
        #SBATCH -c <num>
        #SBATCH --mem-per-cpu=<num>G
        spack install -j ${SLURM_CPUS_ON_NODE} ....
        

### Using the software

Once spack has built software, the bash shell needs to have the proper environment variables set to access the software.

This is accomplished using the `spack load` command.

Given a [spec](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies), a spack command can be used to generate the appropriate environment variables to "load" spack-installed software.

To set the environment variables:

```
$ eval $( spack load --sh <spec> )
```

To unset (unload) these variables:

```
$ eval $( spack unload --sh <spec> )
```

To simply view the environment variables that **would be set** without actually setting them:

```
$ spack load --sh <spec>
```

See [the official spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies) for more information on specs.

These commands can be placed in an sbatch file to be used in a job.

```
# !/bin/bash

eval $( spack load --sh <spec> )

```

### Example Workflow

For a complete walkthrough of searching, installing, and loading packages with Spack, see the [Spack Basic Usage Guide](https://spack.readthedocs.io/en/latest/basic_usage.html).

Quick example:

```bash
# Search for a package
$ spack list <package-name>

# See available versions
$ spack versions <package-name>

# Install specific version
$ spack install <package-name>@<version>

# Load in a job script
eval $(spack load --sh <package-name>)
```

## What if the software I want is not available through Spack

When needed software is not readily accessible via Spack, there are a few options.

1. **Follow the installation instructions from the software creator.**
 
     Sometimes, this can be very quick and straightforward.
   
     Sometimes, this can be very painful.
   
     Sometimes, it can be a good idea to pass judgement on the quality of software based on the quality of the installation process and documentation. :wink:

2. **Create a custom spack package**

     Spack can be a wonderful tool for creating and maintaining software. [Plenty of documentation is provided for creating and maintaining custom packages](https://spack.readthedocs.io/en/latest/packaging_guide.html), though a firm understanding of python is needed.

## R considerations

[Please see the R page for more information.](r/index.md)

## Manual Installation

Sometimes it's just easier to follow the installation steps provided by software creator.

If the software depends on other software, it might be that the dependency software could be installed via [spack](#spack).

For example, if a piece of software is not available via Spack, but requires samtools to be installed:

    # Install samtools via spack
    $ spack install samtools

    # load samtools before installation
    $ eval $( spack load --sh samtools )
    
    (proceed with the manual installation)

!!! Remember
    The best place to install software is in reference storage: `/ref/<lab>/software`.

## What about conda?

Feel free to use conda if it is required/preferred.

The HTCF does not handle Conda support requests.  For conda support, please see [https://docs.conda.io/en/latest/help-support.html](https://docs.conda.io/en/latest/help-support.html)
