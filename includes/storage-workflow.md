## Standard Data Workflow

Follow this workflow for efficient data management on the HTCF:

1. **Store raw data in LTS** - Long-term archival storage
   ```bash
   /lts/<lab>/raw_data/
   ```

2. **Process data** - Two options depending on needs:

   **Option A: Read directly from LTS** (for read-only access)
   ```bash
   # Jobs can read input data directly from LTS (read-only on compute nodes)
   sbatch --wrap="process_tool /lts/<lab>/raw_data/ /scratch/$USER/results/"
   ```

   **Option B: Copy to scratch first** (when data will be modified or for high-performance I/O)
   ```bash
   rsync -aHv --progress /lts/<lab>/raw_data/ /scratch/$USER/project/
   cd /scratch/$USER/project/
   sbatch process_data.sh
   ```

3. **Copy results back to LTS** - Archive processed results from the login node
   ```bash
   rsync -aHv --progress /scratch/$USER/project/results/ /lts/<lab>/results/
   ```

4. **Clean up scratch** - Remove working files
   ```bash
   rm -rf /scratch/$USER/project/
   ```

!!! warning "Scratch Data Cleaning Policy"
    Files on `/scratch` are subject to automatic removal after 60 days of inactivity. Always copy important results back to LTS or LTOS for long-term storage. See the [storage policies](../policies.md#scratch-data-cleaning) for more details.
