# Research Reproducibility

## Documentation

**Document Your Workflow**
```bash
# README.md in project directory
## Project: Analysis Pipeline
## Date: 2026-01-07
## Software versions:
- Python 3.11
- NumPy 1.24
- Custom scripts in scripts/

## Steps:
1. Preprocess: sbatch preprocess.sh
2. Analyze: sbatch analyze.sh
3. Visualize: python plot_results.py
```

**Job Scripts as Documentation**
Job scripts serve as executable documentation:
```bash
#!/bin/bash
# Analysis of dataset XYZ
# Author: Your Name
# Date: 2026-01-07

#SBATCH --job-name=analysis_xyz
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G

# Load specific versions
eval $(spack load --sh python@3.11)
eval $(spack load --sh py-numpy@1.24)

# Process data
python analyze.py --input data.csv --output results.csv
```

## Version Control

If you wanted to use version control for tracking code and script changes, Git is one option.

For Git basics, see:

- [Pro Git Book](https://git-scm.com/book/en/v2) - Complete Git guide
- [GitHub Quickstart](https://docs.github.com/en/get-started/quickstart) - Getting started with GitHub
- [Git Basics](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository) - Initialize and commit changes

**Track Software Versions**
```bash
# Save environment
spack find > environment.txt

# Or in job script
eval $(spack load --sh python)
python --version > versions.txt
pip list >> versions.txt
```

## Environment Management

**Spack Environments**
```bash
# Create environment
spack env create myproject
spack env activate myproject
spack add python py-numpy py-pandas
spack install
```

**Document Dependencies**
```bash
# Save environment spec
spack env export > spack.yaml
```

## Results Tracking

**Unique Job IDs**
```bash
# Use job ID in output names
#SBATCH --output=results_${SLURM_JOB_ID}.out

# Create result directories with job ID
mkdir -p results_${SLURM_JOB_ID}
```

**Save Parameters**
```python
# Save run parameters
import json
params = {
    'input': 'data.csv',
    'model': 'random_forest',
    'n_estimators': 100
}
with open(f'params_{job_id}.json', 'w') as f:
    json.dump(params, f)
```

## Best Practices

1. **Use version control** for code
2. **Document software versions** used
3. **Save parameters** with results
4. **Use meaningful names** for files and jobs
5. **Archive complete environments** for important results

--8<-- "includes/getting-help.md"
