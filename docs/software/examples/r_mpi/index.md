# Example: Using MPI with slurm via R and Rmpi

Getting all the necessary software properly built, installed, and configured can be very difficult.

This combination of software/versions seems to work:

- Spack 0.18.0
- Slurm 20.11.9.1
- openmpi 4.1.3
- R 4.1.3
- r-rmpi 0.6-9.2

## Installing

Step 1. Get and [interactive session](../../../using/getstarted.md#interactive)

Step 2. Remove Slurm variables from the interactive session's environment (or Rmpi will break when it tries to install):

    $ for x in $(env|grep ^SLURM|cut -f1 -d=); do unset $x;done

Step 3. Install r-rmpi (with a very long spec):

    $ spack install r-rmpi@0.6-9.2 ^r@4.1.3 ^openmpi@4.1.3 schedulers=slurm legacylaunchers=true ^slurm@20-11-9-1


## Using

Example [sbatch file](r_mpi_example.sbatch) and [R script](r_mpi_example.r).

