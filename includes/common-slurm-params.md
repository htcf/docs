## Common SLURM Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| `--cpus-per-task` or `-c` | Number of CPUs per task | `#SBATCH -c 4` |
| `--mem` | Total memory per node | `#SBATCH --mem=16G` |
| `--mem-per-cpu` | Memory per CPU | `#SBATCH --mem-per-cpu=4G` |
| `--time` or `-t` | Time limit (D-HH:MM:SS) | `#SBATCH -t 24:00:00` |
| `--partition` or `-p` | Partition to submit to | `#SBATCH -p general` |
| `--job-name` or `-J` | Job name for identification | `#SBATCH -J myjob` |
| `--output` or `-o` | Standard output file path | `#SBATCH -o output_%j.txt` |
| `--error` or `-e` | Standard error file path | `#SBATCH -e error_%j.txt` |
| `--array` | Job array specification | `#SBATCH --array=1-100` |
| `--nodes` or `-N` | Number of nodes | `#SBATCH -N 2` |
| `--ntasks` or `-n` | Number of tasks | `#SBATCH -n 16` |
