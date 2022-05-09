# RStudio

Rstudio Server can be run as an interactive job and accessed via an SSH tunnel.

## Support

!!!Reminder
    **The HTCF does not handle Rstudio support requests**

    After installation, support requests for lab-installed software such as rstudio should be directed to:

    - Google
    - Software support forums or mailing lists
    - The software developers
    
## Building and Installing

Note: The Spack `rstudio` package is the **Destop Version** of rstudio and is **NOT** for use on the HTCF.

1. A custom [Spack](../../software.md#spack) package needs to be created: `spack create rstudio-server`

2. An rstudio-server [package.py can be found here](package.py).

## Running rstudio-server as a job

A sample `rstudio.sbatch` script [can be found here](rstudio.sbatch).

!!!Note
    The rstudio.sbatch script will likely need to be modified to add CPU and/or Memory requirements.
