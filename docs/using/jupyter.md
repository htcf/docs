```
#!/bin/bash

#SBATCH --mem=1G
#SBATCH --cpus-per-task=1
#SBATCH --time 8:00:00
#SBATCH --output jupyter-lab-%J.out 

. env/bin/activate

unset XDG_RUNTIME_DIR
port=$(shuf -i9000-9999 -n1)


echo -e "
    Run in a new local terminal window to create an SSH tunnel to $host
    -----------------------------------------------------------------
    ssh -N -L $port:$HOSTNAME:$port $USER@htcf.wustl.edu
    -----------------------------------------------------------------

    Then in the desktop browser, follow the http://127.0.0.1..... address shown at the bottom of the jupyter lab command
    "

# Launch jupyter lab
jupyter lab --no-browser --port=$port --ip=$HOSTNAME
```
