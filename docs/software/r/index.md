# R

Labs can install R using Spack. 

R packages can be installed using the packages' install instructions.

## Search paths

!!!Warning
    ** A thorough understanding of R search paths is necessary for successful installation and use of R packages.**

    It's best to store R packages in a shared location (such as `/ref/<lab>/software/...`) rather than `$HOME/R`.  R packages in `$HOME` aren't as easily shared with other lab members and can quickly fill up the space in `$HOME`.

    Please read and understand [libPaths: Search Paths for Packages](https://rdrr.io/r/base/libPaths.html), **specifically R_LIBS_USER and R_LIBS_SITE**, before using R.  It may be best to include the R version in the path such as `/ref/<lab>/software/r_packages/%v`

## Support

!!!Reminder
    Beyond basic R installation and R package installation, **the HTCF cannot handle R support requests.**

    After installation, support requests for R software and packages should be directed to:

    - Google
    - Software support forums or mailing lists
    - The software developers
    
