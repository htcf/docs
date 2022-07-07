# R

It's best to install R using Spack, then [after setting up the R search path](#search-paths), install R packages using the R command line interface, following the packages' install instructions.

This is the best way to ensure the proper installation of the latest (or most appropriate) versions of the R libraries.

## Support

!!!Reminder
    Beyond basic R installation and basic R package installation, **the HTCF cannot handle R support requests.**

    After installation, support requests for R software and packages should be directed to:

    - Google
    - Software support forums or mailing lists
        - [https://www.r-project.org/help.html](https://www.r-project.org/help.html)
        - [https://www.biostars.org/](https://www.biostars.org/)
    - The software developers
    
## Search paths

!!!Warning
    ** A thorough understanding of R search paths is necessary for successful installation and use of R packages.**

    It's best to store R packages in a shared location (such as `/ref/<lab>/software/...`) rather than `$HOME/R`.  R packages in `$HOME` aren't as easily shared with other lab members and can quickly fill up the space in `$HOME`.

    **The shared location must exist (`mkdir <dir>`) prior to starting the R command line.**

    Please read and understand [libPaths: Search Paths for Packages](https://rdrr.io/r/base/libPaths.html), **specifically R_LIBS_USER and R_LIBS_SITE**, before using R.  It may be best to include the R version in the path such as `/ref/<lab>/software/r_packages/%v`

For example, to set up a shared directory of R (version 4.1.X) packages and install into it:

    $ mkdir -p /ref/<labname>/software/r-envs/<project_name>/4.1
    $ export R_LIBS_SITE=/ref/<labname>/software/r-envs/<project_name>/%v
    $ eval $( spack load --sh r@4.1.1 )
    $ R
    ...
    > install.packages('<pkgname>')

To use these R libraries in an Rscript job:

    #!/bin/bash

    export R_LIBS_SITE=/ref/<labname>/software/r-envs/<project_name>/%v
    eval $( spack load --sh r@4.1.1 )

    Rscript ........

## Troubleshooting Package Installation

Unfortunately, trial and error may be required when installing R packages.

### Environment variables

It seems some R packages such as `Rsamtools` and `Rhtslib` require extra environment variables in order to find their dependencies during install.

To do this, prior to installation, the variables `$LIBRARY_PATH` and `$C_INCLUDE_PATH` can be set as follows:

    export LIBRARY_PATH=$_LIBRARY_PATH
    export C_INCLUDE_PATH=$_C_INCLUDE_PATH

This should help install some R packages.

!!!Warning
    **Most** R packages **DO NOT** need these environment variables, and they might actually **FAIL** to install properly if these variables are set.

### Installing within rstudio

!!!Warning
    Some have reported problems installing software from within rstudio.

    It may be best to always install software from command-line R (from within an interactive job).
