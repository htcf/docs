# HPC Applications

## Overview

Running MPI-based parallel applications and scientific simulations across multiple nodes.

**Official Documentation:**

- [MPI Standard](https://www.mpi-forum.org/docs/) - MPI specification and reference
- [OpenMPI Documentation](https://www.open-mpi.org/doc/) - OpenMPI implementation guide
- [MPICH Documentation](https://www.mpich.org/documentation/tutorials/) - MPICH implementation guide
- [OpenMP Specifications](https://www.openmp.org/specifications/) - Shared memory parallelism
- [Slurm MPI Guide](https://slurm.schedmd.com/mpi_guide.html) - Running MPI jobs with Slurm

## MPI Setup

### Load MPI
```bash
eval $(spack load --sh openmpi)
# or
eval $(spack load --sh mpich)
```

### Verify MPI
```bash
$ which mpirun
$ mpirun --version
```

## Example 1: Simple MPI Job

### MPI Program
```c
// hello_mpi.c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    MPI_Init(&argc, &argv);

    int world_size, world_rank;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    char processor_name[MPI_MAX_PROCESSOR_NAME];
    int name_len;
    MPI_Get_processor_name(processor_name, &name_len);

    printf("Hello from rank %d of %d on %s\n",
           world_rank, world_size, processor_name);

    MPI_Finalize();
    return 0;
}
```

### Compile and Run
```bash
#!/bin/bash

#SBATCH --job-name=mpi_hello
#SBATCH --nodes=2              # 2 nodes
#SBATCH --ntasks-per-node=16   # 16 tasks per node (32 total)
#SBATCH --cpus-per-task=1
#SBATCH --mem=32G
#SBATCH --time=1:00:00

eval $(spack load --sh openmpi gcc)

# Compile
mpicc -o hello_mpi hello_mpi.c

# Run
mpirun -n 32 ./hello_mpi
```

## Example 2: Multi-Node Scientific Application

### LAMMPS Molecular Dynamics
```bash
#!/bin/bash

#SBATCH --job-name=lammps
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=32   # 128 MPI tasks total
#SBATCH --cpus-per-task=1
#SBATCH --mem=120G
#SBATCH --time=24:00:00

eval $(spack load --sh lammps+mpi)

# Set up scratch directory
WORK_DIR=/scratch/mylab/$USER/lammps_${SLURM_JOB_ID}
mkdir -p $WORK_DIR
cd $WORK_DIR

# Copy input files (can read from LTS)
cp /lts/lab/input/* .

# Run LAMMPS
mpirun -n 128 lmp -in input.lammps

echo "Results in $WORK_DIR"
# Copy results to LTS from login node after job completes
```

!!! note "Copy Results After Job"
    LTS is read-only on compute nodes. After job completes:
    ```bash
    rsync -av /scratch/mylab/$USER/lammps_*/  /lts/lab/results/
    ```

## Example 3: R with MPI

```bash
#!/bin/bash

#SBATCH --job-name=r_mpi
#SBATCH --nodes=2
#SBATCH --ntasks-per-node=8    # 16 total tasks
#SBATCH --mem-per-cpu=4G
#SBATCH --time=4:00:00

eval $(spack load --sh r)
eval $(spack load --sh r-rmpi)
eval $(spack load --sh openmpi)

# Run R script with MPI
mpirun -n 16 R CMD BATCH --no-save script.R
```

See [R with MPI example](../software/examples/r-mpi/index.md) for details.

## Resource Specification

### Nodes and Tasks
```bash
#SBATCH --nodes=4              # Number of compute nodes
#SBATCH --ntasks-per-node=32   # MPI tasks per node
#SBATCH --cpus-per-task=1      # CPUs per MPI task
```

Total MPI ranks = nodes × ntasks-per-node = 4 × 32 = 128

### Memory
```bash
# Total memory per node
#SBATCH --mem=128G

# Or memory per CPU
#SBATCH --mem-per-cpu=4G
```

## Scaling Considerations

### Test Scaling
Run with increasing number of nodes to find optimal scaling:

```bash
# 1 node (32 cores)
#SBATCH --nodes=1 --ntasks-per-node=32

# 2 nodes (64 cores)
#SBATCH --nodes=2 --ntasks-per-node=32

# 4 nodes (128 cores)
#SBATCH --nodes=4 --ntasks-per-node=32
```

Analyze speedup:
```bash
$ sacct -j <JOBID>
```

### Communication Patterns
- **Embarrassingly parallel**: Scales linearly
- **Frequent communication**: May not scale past certain point
- **I/O bound**: Scaling limited by storage

## Best Practices

### Use Scratch
```bash
# In job script: work in scratch
WORK_DIR=/scratch/mylab/$USER/sim_${SLURM_JOB_ID}
mkdir -p $WORK_DIR
cd $WORK_DIR

# Process
mpirun -n 128 application

echo "Results in $WORK_DIR"
```

After job completes, copy from login node:
```bash
rsync -av /scratch/mylab/$USER/sim_*/ /lts/lab/results/
```

### Checkpoint Long Jobs
Save checkpoints periodically:
```bash
# In your simulation
if (step % 10000 == 0) {
    save_checkpoint("checkpoint.dat");
}
```

Resume from checkpoint:
```bash
if [ -f checkpoint.dat ]; then
    mpirun -n 128 app --restart checkpoint.dat
else
    mpirun -n 128 app --new
fi
```

### Monitor Performance
```bash
# In job script
mpirun -n 128 app > output.log 2>&1

# Check timing
grep "Time" output.log
```

## Troubleshooting

**MPI not found:**
```bash
eval $(spack load --sh openmpi)
```

**Wrong number of processes:**
Check `--nodes` × `--ntasks-per-node` = expected total

**Poor scaling:**
- Check communication overhead
- Verify problem size appropriate for cores
- Test with fewer nodes

--8<-- "includes/getting-help.md"
