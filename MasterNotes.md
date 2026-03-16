# 3/12 
Goals: run FASTQC, trimmommatic, then FASTQC again
What this accomplishes: finding correct parameters to trim data, improve the quality of our data
FASTQC Dori path: /dsr84/viral_genomics/raw
FASTQC Luby path: /ycy201/viral_genomics/raw
Cleaned reads Dori path: /dsr84/viral_genomics/cleanedreads
Cleaned reads Luby path: /ycy201/viral_genomics/trimmedreads

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

# 3/16
Goals: Upload FASTQC file to bucket, Upload gzipped raw file to bucket, run trimmomatic, run FASTQC again
Actually accomplished:
## 1. could not upload gzip file to bucket
It said dsr84@georgetown does not have access to upload to bucket ???

## 2. Upload FASTQC to bucket 
gcloud storage cp gs://gu-biology-dept-class/SRR6996011.sra_2_fastqc.html gs://gu-biology-dept-class

## 3. Parameters for trimmomatic
Based on graph, quality is worst after base 186 (all in the red) 
We are planning to use the same parameters for HW 4
Script Luby path: Cleaned reads Luby path: /ycy201/viral_genomics/trimmedreads/slurm 
Script:
#!/bin/bash
#SBATCH --job-name="trim_viral_genomics"
#SBATCH --output=/home/yc1201/viral_genomics/trimmed_reads/%x.o%j
#SBATCH --mail-type=END,FAIL --mail-user=yc1201@georgetown.edu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=03:00:00
#SBATCH --mem=10G

#Load Trimmomatic module ("aliases" needed for GU HPC setup here)
shopt -s expand_aliases
module load trimmomatic

#Define paths and variables
adapters=/home/yc1201/HW4/HW4_input_files/TruSeq3-PE.fa
R1=/home/yc1201/viral_genomics/raw/SRR6996011/SRR6996011.sra_1.fastq
R2=/home/yc1201/viral_genomics/raw/SRR6996011/SRR6996011.sra_2.fastq
output_R1_PE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmPE1.fq.gz
output_R2_PE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmPE1.fq.gz
output_R1_SE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmSE1.fq.gz
output_R2_SE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmSE1.fq.gz

trimmomatic PE -threads 4 \
        $R1 \
	$R2 \
	$output_R1_PE $output_R1_SE \
        $output_R2_PE $output_R2_SE \
        ILLUMINACLIP:$adapters:2:30:10 \
        LEADING:10 \
        TRAILING:10 \
        SLIDINGWINDOW:4:15 \
        MINLEN:75

## 4. run FASTQC again
