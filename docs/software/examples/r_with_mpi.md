# Example: R+MPI 

The following is an example of preparing for and running a job consisting of an R script that requires:

  - mpi (`Rmpi`) to parallalize the work
  - bioinformatics packages `sva` and `edgeR`

## Install the Spack managed software (R, mpich, Rmpi)

Get an interactive session with plenty of resources to help install software quickly:

    srun --mem-per-cpu=4G -c 8 -J interactive -p interactive --pty /bin/bash -l


Install (but don't load) software that will be managed by Spack (specifying exact versions if desired):

    # R
    spack install r@<version>

    # mpich
    spack install mpich

    # Rmpi
    spack install r-rmpi ^r@<version> ^mpich

!!! Note 

    These 3 separate commands aren't actually needed.  The same could be accomplished with just 1 command:

        spack install r-rmpi ^r@<version> ^mpich
        
    With 3 separate commands, Spack considers all three "explicitly" installed.

    With 1 command, only r-rmpi is considered "explicity" installed.  The rest are "implicitly" installed.

    [See here for more information.](https://spack.readthedocs.io/en/latest/basic_usage.html#marking-packages-explicit-or-implicit)
    
    
Install libpng (which seems to be needed to install the `sva` R package later on)

    spack install libpng

## Install R libraries (sva and edgeR)

Get an interactive session (if not already in one) with plenty of resources to help install software quickly:

    srun --mem-per-cpu=4G -c 8 -J interactive -p interactive --pty /bin/bash -l

Ensure a clean environment (ie. unload any spack packages and/or modules that might be loaded from previous work):

    eval $(spack unload --sh --all)

Load R (specifying version if desired):

    eval $(spack load --sh r@<version>)

The `sva` R library seems to need `libpng` to be loaded:

    eval $(spack load --sh libpng)

Decide on a shared location and name for the software and create the directory

For example: 

    mkdir /ref/jdlab/software/projecta

Using an R environment variable, tell R where that location is so it will install packages there:

    export R_LIBS_SITE=/ref/jdlab/software/projecta


!!! Important

    - This `R_LIBS_SITE` environment variable needs to be set any time R packages are **installed** for the project.
    - This `R_LIBS_SITE` environment variable needs to be set any time the R packages need to be **loaded** in a job.

Install `sva` and `edgeR` into `R_LIBS_SITE`:

    $ R
    > install.packages("BiocManager")
    ...
    ...
    ...
    > BiocManager::install("edgeR")
    ...
    ...
    ...
    > BiocManager::install("sva")
    ...
    ...
    ...
    
Put it all together in a job

In the `.sbatch` job:
    #!/bin/bash

    #SBATCH -n X
    #SBATCH --mem-per-cpu=XG
    #SBATCH --cpus-per-task=X

    eval $(spack load r-rmpi ^r@<version> ^mpich)
    export R_LIBS_SITE=/ref/jdlab/software/projecta
    
    mpiexec -usize $SLURM_NTASKS -np 1 Rscript /path/to/rscript.R
    

