# Jupyter Lab

Jupyter Lab can be installed via Spack:

    spack install py-jupyterlab

Example sbatch script:

```
#!/bin/bash

# ------ SLURM Parameters ------

# rstudio-server is interactive.  Use the interactive partition
# Add cpu or memory slurm parameters as needed

#SBATCH -p interactive

# ------ Make sure it's run as a job ------

if [ -z "${SLURM_JOBID}" ]; then
    echo "Error: Must be run as a job"
    exit 1
fi

# ------ Load environment ------

eval $(spack load --sh py-jupyterlab)

port=$(shuf -i9000-9999 -n1)

echo -e "
    In a local terminal, create SSH tunnel to $HOSTNAME
    -----------------------------------------------------------------
    ssh $USER@login.htcf.wustl.edu -N -L $port:$HOSTNAME:$port
    -----------------------------------------------------------------

    Then in the desktop browser, follow the http://127.0.0.1..... address shown at the bottom of the jupyter lab command
    "

# Launch jupyter lab
jupyter lab --no-browser --port=$port --ip=$HOSTNAME
```
