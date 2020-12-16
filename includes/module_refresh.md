!!! Note
    When first starting out with your new spack installation, some system-installed *spack external packages* might still need to have their module files generated.
    Usually, this is the case when seeing `module load` errors like:

        Lmod has detected the following error:  The following module(s) are unknown: .... 

    This can usually be fixed with a "refreshing" of the modules:

        $ spack module lmod refresh

