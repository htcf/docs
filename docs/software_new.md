# Software on the HTCF

Software-wise, the HTCF starts as a blank slate for each lab.  However, this doesn't mean labs can't get up and running in a matter of minutes.

Each lab has its own dedicated space to install and manage software.  This [reference space](storage/ref.md) is located in [`/ref/<lab>/software`](storage/ref.md#software).

## Self-service Installation

Software building and installation on the HTCF is primarily self-service.

Labs are free to use their [/ref software](storage/ref.md) directory to install software using whatever means is most comfortable.

At the lab level, **use of [Spack](#spack) to install common software is encouraged**.  Virtual environments can also be used if the software is well suited.

### Spack

[Spack](https://spack.readthedocs.io) is a package management tool designed to support multiple versions and configurations of software on a wide variety of platforms and environments.

More info [can be found here](spack.md)

#### What if the software I want is not available through Spack

When needed software is not readily accessible via Spack, there are a few options.

1. **Follow the installation instructions from the software creator.**
 
     Sometimes, this can be very quick and straightforward.
   
     Sometimes, this can be very painful.
   
     Sometimes, it can be a good idea to pass judgement on the quality of software based on the quality of the installation process and documentation. :wink:

2. **Create a custom spack package**

   Spack can be a wonderful tool for creating and maintaining software. [Plenty of documentation is provided for creating and maintaining custom packages](https://spack.readthedocs.io/en/latest/packaging_guide.html), though a firm understanding of python is needed.

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
    $ eval $( spack load --sh samtools )
    
    (proceed with the manual installation)

!!! Remember
    The best place to install software is your [reference storage](storage/ref.md) `/ref/lab/sofware`.

After manual software installation, it's good practice to then create a [module file](#manual-module-files)

#### Manual module files

Module files that are manually created go in your [reference storage](storage/ref.md) in `/ref/<lab>/software/modules`.

[The lmod documentation](https://lmod.readthedocs.io/en/latest/015_writing_modules.html) is the best place to learn about creating module files.
