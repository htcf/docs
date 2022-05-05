# Software

The HTCF starts as somewhat of a blank slate for each lab.  However, this doesn't mean labs can't get up and running in a matter of minutes!

Each lab has its own dedicated space to install and manage software.  This reference space is located in `/ref/<lab>/software`.

Software building and installation on the HTCF is primarily self-service.

Labs are free to use their `/ref` software directory to install software using whatever means is most comfortable.

At the lab level, **use of [Spack](#spack) to install common software is encouraged**.  Virtual environments can also be used if the software is well suited.

## Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

!!! Note
    Installation of software on the login node is unsupported.  Please build/install software from an interactive job.

### Tutorial

See [the official spack tutorial](https://spack-tutorial.readthedocs.io/en/latest/)

### Initialization

To create a lab instance of the spack package manager:

1.  Download and untar [a spack release](https://github.com/spack/spack/releases) into `/ref/<lab_name>/software`

2.  Rename (or make a symlink from) `/ref/<lab_name>/software/spack-VERSION` to `/ref/<lab_name>/software/spack`.

3.  Logout and log back in.  This will ensure the spack command is available in the PATH.


### Installing Software

see [the official spack documentation](https://spack.readthedocs.io/en/latest/basic_usage.html#cmd-spack-install)


### Using the software

Once spack has built software, the bash shell needs to have the proper environment variables set to access the software.

This is accomplished using the `spack load` command.  Spack packages can be "loaded" similar to the way modules are loaded.

Given a [spec](https://spack.readthedocs.io/en/latest/basic_usage.html#specs-dependencies), a spack command can be used to generate the appropriate environment variables to "load" spack-installed software.


To set the environment variables (similar to `module load ...`):

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
#!/bin/bash

eval $( spack load --sh <spec> )

```

### Example

Installing biom-format

Search for the name of the package:

    $ spack list biom
    ==> 7 packages.
    microbiomeutil  py-biomine    r-biomart   r-biomformat
    py-biom-format  r-biom-utils  r-biomartr

See what versions are available:

    $ spack versions py-biom-format
    ==> Safe versions (already checksummed):
      2.1.10  2.1.9  2.1.7  2.1.6
    ...

Install:

    $ spack install py-biom-format@2.1.10

Load the software in a job:

    #!/bin/bash
    
    eval $( spack load --sh py-biom-format@2.1.10 )

    ...

## What if the software I want is not available through Spack

When needed software is not readily accessible via Spack, there are a few options.

1. **Follow the installation instructions from the software creator.**
 
     Sometimes, this can be very quick and straightforward.
   
     Sometimes, this can be very painful.
   
     Sometimes, it can be a good idea to pass judgement on the quality of software based on the quality of the installation process and documentation. :wink:

2. **Create a custom spack package**

     Spack can be a wonderful tool for creating and maintaining software. [Plenty of documentation is provided for creating and maintaining custom packages](https://spack.readthedocs.io/en/latest/packaging_guide.html), though a firm understanding of python is needed.

## R considerations

For R software libraries, it's best to install R using Spack, then install R libraries using the R command line interface.

This is the best way to ensure the proper installation of the latest (or most appropriate) versions of the R libraries.

To create an R library directory that can be used by multiple people, the `$R_LIBS_SITE` variable can be used.
For example, to install R version 4.1.1 and create a shared R library directory:

    $ spack install r@4.1.1
    ...
    $ export R_LIBS_SITE=/ref/<labname>/software/r-envs/<project_name>/4.1.1
    $ mkdir -p $R_LIBS_SITE
    $ eval $( spack load --sh r@4.1.1 )
    $ R
    ...
    > install.packages('<pkgname>')
    > install.packages('<another_pkgname>')
    > install.packages(c('<and_another_pkgname>', '<one_more_pkgname>'))

To use these R libraries in an Rscript job:

    #!/bin/bash

    export R_LIBS_SITE=/ref/<labname>/software/r-envs/<project_name>/4.1.1
    eval $( spack load --sh r@4.1.1 )

    Rscript ........

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

After manual software installation, it's good practice to then create a [module file](#manual-module-files)

### Manual module files

Module files that are manually created go in reference storage in `/ref/<lab>/software/modules`.

[The lmod documentation](https://lmod.readthedocs.io/en/latest/015_writing_modules.html) is the best place to learn about creating module files.

## What about conda?

Feel free to use conda if it is required/preferred.

We are unable to support Conda.  For conda support, please see [https://docs.conda.io/en/latest/help-support.html](https://docs.conda.io/en/latest/help-support.html)

