# RStudio

Rstudio Server can be run as an interactive job and accessed via an SSH tunnel.

!!!Warning
    The Spack `rstudio` package is the **Destop Version** of rstudio and is **NOT for use on the HTCF**.

## Support

!!!Reminder
    **The HTCF does not handle Rstudio support requests**

    After installation, support requests for lab-installed software such as rstudio should be directed to:

    - Google
    - Software support forums or mailing lists
    - The software developers
    
## Before building

The `texlive` package may need modifying before installing rstudio-server.
If there is a texlive error during install, the following two lines should be commented out in the texlive package (`spack edit texlive`):

    #    version('live', sha256='74eac0855e1e40c8db4f28b24ef354bd7263c1f76031bdc02b52156b572b7a1d',
    #        url='ftp://tug.org/historic/systems/texlive/2021/install-tl-unx.tar.gz')

## Building and Installing

!!!Warning
    Compiling rstudo requires large amounts of RAM.  When setting up the Slurm jobs be sure to include:

        --mem-per-cpu=12G --cpus-per-task=<NUM>

    in the Slurm parameters. (*More than 1 CPU will make the building faster but could cause longer waiting in the queue.*)

    Also, be sure to tell Spack how many CPUs are available:

        spack install -j ${SLURM_CPUS_PER_TASK} ....

1. A custom [Spack](../../software.md#spack) package needs to be created: `spack create rstudio-server`

2. An rstudio-server [package.py can be found here](package.py).

## Running rstudio-server as a job

A sample `rstudio.sbatch` script [can be found here](rstudio.sbatch).

!!!Note
    The rstudio.sbatch script will likely need to be modified to add CPU and/or Memory requirements.
