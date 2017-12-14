Many times you might have a command that you’d like to run on a lot of samples.  To run these sequentially, you might have a script like this:

```bash
#!/bin/bash

module load spades

spades.py --careful --pe1-1 samp1-r1.fastq --pe1-2 samp1-r2.fastq -o assembly_1
spades.py --careful --pe1-1 samp2-r1.fastq --pe1-2 samp2-r2.fastq -o assembly_2
…
spades.py --careful --pe1-1 samp2000-r1.fastq --pe1-2 samp2000-r2.fastq -o assembly_2000
```

By submitting this on the HTCF as an “array job”, you have the potential of running each of these commands in parallel (all at the same time) rather than sequentially (one at a time).  The best case scenario could be that all 2000 commands finish in the time it takes 1 command to run!  More information regarding SLURM job arrays is available at [http://slurm.schedmd.com/job_array.html](http://slurm.schedmd.com/job_array.html).

* * *

##### Run a command or set of commands on file names that are sequentially numbered

|  Input File 1 | | Input File 2 | 
| ------------- | | ----------|
|sample1-r1.fastq | | sample1-r2.fastq | 
|sample2-r1.fastq | | sample2-r2.fastq | 
|…   | | …  | 
|sample2000-r1.fastq |  | sample2000-r2.fastq | 

Step 1:  Create an sbatch file

```bash
#!/bin/bash

#SBATCH --array=1-2000%20   # This will create 2000 tasks numbered 1-2000 and allow 20 concurrent jobs to run

module load spades

ID=${SLURM_ARRAY_TASK_ID}  # Number between 1 and 2000 

spades.py --careful --pe1-1 samp${ID}-r1.fastq --pe1-2 samp${ID}-r2.fastq -o assembly_${ID}
```

##### Run a command or set of commands on file names that aren’t numbered sequentially
* * * 
|Input File 1  | | Input File 2 | 
| ------ | | ------ | 
| sampleAAA-r1.fastq | | sampleAAA-r2.fastq | 
| sampleAAB-r1.fastq | | sampleAAB-r2.fastq | 
| … | |  … |  
|SampleZZZ-r1.fastq | | SampleZZZ-r2.fastq | 

Step 1:  Create a “lookup” file, lookup.txt
```bash
AAA
AAB
…
ZZZ
```

Step 2: Find the number of lines in lookup file.  This will be the number of tasks in your array job.

~~~~
$ wc -l lookup.txt
2000 lookup.txt
~~~~
Step 2: Create an sbatch file, grabbing the “ID” in line $SLURM_ARRAY_TASK_ID from the lookup file
```bash
#!/bin/bash

#SBATCH --array=1-2000%20   # This will create 2000 tasks numbered 1-2000 and allow 20 concurrent jobs to run

module load spades

ID=$( sed -n ${SLURM_ARRAY_TASK_ID}p lookup.txt )

spades.py --careful --pe1-1 samp${ID}-r1.fastq --pe1-2 samp${ID}-r2.fastq -o assembly_${ID}
```


##### Run a command or set of commands using a complex lookup file
* * *

Step 1:  Create a “lookup” file, lookup.txt
```
5 sampleAAA_R1 0.0001 5000 
5 sampleAAA_R2 0.0001 5000
5 sampleBBB_R1 0.0005 1000 
5 sampleBBB_R2 0.0005 1000 
```

Step 2:  Find the number of lines in lookup file.  This will be the number of tasks in your array job.

~~~~
$ wc -l lookup.txt
2000 lookup.txt
~~~~
Step 3: Create an sbatch file, each job is created using the template with information from the lookup file.
```bash
#!/bin/bash

#SBATCH --array=1-2000%20   # This will create 2000 tasks numbered 1-2000 and allow 20 concurrent jobs to run

module load seqtk

read part1 part2 part3 part4 < <( sed -n ${SLURM_ARRAY_TASK_ID}p lookup.txt )
 
seqtk sample -s${part1} /path/to/file.${part2}.fastq ${part3} > /path/to/file.${part2}_${part4}.fastq
```
