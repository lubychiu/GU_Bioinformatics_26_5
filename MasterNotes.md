# 3/12 
Goals: run FASTQC, trimmommatic, then FASTQC again
What this accomplishes: finding correct parameters to trim data, improve the quality of our data
FASTQC Dori path: /dsr84/viral_genomics/raw
FASTQC Luby path: /ycy201/viral_genomics/raw
Cleaned reads Dori path: /dsr84/viral_genomics/cleanedreads
Cleaned reads Luby path: /ycy201/viral_genomics/cleanedreads

Actually accomplished: 
## 1. loaded conda environment 
$ module load anaconda 3
$ conda create -n sra_env -c bioconda sra-tools
-> y
$ conda activate sra_env
## 2. loaded data
entered /viral_genomics/raw
$ prefetch SAMN08784153
change files into FASTQ format:
$ fasterq-dump *.sra
## 3. Run FASTQC on raw data
Enter interactive mode on a compute node (from where you are) with: $srun --pty bash
$ module load fastqc
Confirm fastqc is available: $ fastqc -h
OUTPUT OF FASTQC GOING TO /viral_genomics/cleanedreads
$ THIS IS INCOMPLETE. FILL IN.  
## Future tasks:
1. $ gzip *.fastq 2. Upload to bucket (Dori do both of these)
