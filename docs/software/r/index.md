# R

Labs can install R using Spack. 

R packages can be installed using the packages' install instructions.

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

    Please read and understand [libPaths: Search Paths for Packages](https://rdrr.io/r/base/libPaths.html), **specifically R_LIBS_USER and R_LIBS_SITE**, before using R.  It may be best to include the R version in the path such as `/ref/<lab>/software/r_packages/%v`

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
