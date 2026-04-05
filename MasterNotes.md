# 3/12 
Goals: run FASTQC, trimmommatic, then FASTQC again
What this accomplishes: finding correct parameters to trim data, improve the quality of our data
- FASTQC Dori path: /dsr84/viral_genomics/raw
- FASTQC Luby path: /yc1201/viral_genomics/raw
- Cleaned reads Dori path: /dsr84/viral_genomics/trimmed_reads
- Cleaned reads Luby path: /yc1201/viral_genomics/trimmed_reads

## 1. Loaded conda environment 
```bash
$ module load anaconda 3
$ conda create -n sra_env -c bioconda sra-tools
-> y
$ conda activate sra_env
```
## 2. Loaded data
```bash
# entered /viral_genomics/raw
$ prefetch SAMN08784153
# change files into FASTQ format:
$ fasterq-dump *.sra
``` 
## 3. Run FASTQC on raw data
```bash
# Enter interactive mode on a compute node (from where you are)
$ srun --pty bash
# load fastqc
$ module load fastqc
# Confirm fastqc is available:
$ fastqc -h
$ fastqc -o fastqc_out SRR6996011.fastq
```
Output FASTQC Luby path: /yc1201//viral_genomics/fastqc_out

## Future tasks:
1. $ gzip *.fastq 2. Upload to bucket (Dori do both of these)

# 3/16
Goals: Upload FASTQC file to bucket, Upload gzipped raw file to bucket, run trimmomatic, run FASTQC again

## 1. zip FASTQC files and upload to bucket
```bash
$ gzip *.fastq
$ gcloud storage cp SRR6996011.sra_2_fastqc.gzip gs://gu-biology-dept-class
```
## 2. Could not upload gzip file to bucket
It said dsr84@georgetown does not have access to upload to bucket ???
Solution: Ensure full path to bucket, including /. gs://gu-biology-dept-class/

```bash
$ gcloud storage cp SRR6996011.sra_2_fastqc.gzip gs://gu-biology-dept-class/
```

## 3. Upload FASTQC to bucket 
```bash
$ gcloud storage cp SRR6996011.sra_2_fastqc.html gs://gu-biology-dept-class/
```
## 4. Run trimmomatic 
Based on graph, quality is worst after base 186 (all in the red) 
We are planning to use the same parameters for HW 4
Script Luby path: Cleaned reads Luby path: /yc1201/viral_genomics/trimmedreads/slurm 
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

## 5. Run FASTQC again
$fastqc -o trimmed_fastqc_out SRR6996011_R1_TrmPE1.fq.gz RR6996011_R2_TrmPE1.fq.gz
$fastqc -o trimmed_fastqc_out SRR6996011_R1_TrmSE1.fq.gz RR6996011_R2_TrmSE1.fq.gz

## 6. Downloading fastqc outputs to local computer
### Incorrect 
FROM LOCAL COMPUTER, $ gcloud compute scp/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R1_TrmPE1.fastqc.html ~/Downloads/
We got errors saying the source must be remote if the file is local, but are unsure what that means ^

### What worked: 
FROM LOCAL COMPUTER, 
$ gcloud compute scp m12-controller:/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R1_TrmPE1.fastqc.html ~/Downloads/

Then the same for R2 PE
$ gcloud compute scp m12-controller/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R2_TrmPE1.fastqc.html ~/Downloads/

## 7. Evaluating trimmomatic results
R1 looks great! R2 not so much, so we're going to trim them both again using the same parameters

## 8. Run trimmomatic again 
SCRIPT:
#!/bin/bash
#SBATCH --job-name="trim_viral_genomics2"
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
R1=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmPE1.fq.gz
R2=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmPE1.fq.gz
output_R1_PE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmPE2.fq.gz
output_R2_PE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmPE2.fq.gz
output_R1_SE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmSE2.fq.gz
output_R2_SE=/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmSE2.fq.gz

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


New outputs: PE2

## 9. Run FASTQC on the second-trimmed files
fastqc -o trimmed_fastqc_out2 SRR6996011_R1_trmPE2.fq.gz SRR6996011_R2_trmPE2.fq.gz
Oh they still don't look great, probably because using the exact same parameters doesn't do anything. 

# 3/17
## 1. Re-run trimmomatic on the raw files with a changed window parameter. 
We changed SLIDINGWINDOW to 2:15 instead of 4:15.
That didn't work! Let's try again. 

## 2. Reorganize files 
### Dori organization
Overall folder: viral_genomics
Within viral_genomics: fastqc, logs, megahit, raw, slurmscripts, trimmed_reads

### Luby organization
Overall folder: viral_genomics
Within viral_genomics: raw, fastqc_out, trimmed_reads, megahit

## 3. Re-run trimmomatic on the raw files with a changed window parameter, again.
We changed SLIDINGWINDOW to 2:20 instead of 2:15. Reads are now of an adequately high quality.

## 4. Megahit
#!/bin/bash
#SBATCH --job-name=megahit_SRR6996011   	# how job appears in the queue
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8                 
#SBATCH --mem=32G                         
#SBATCH --time=03:00:00                   
#SBATCH --output=/home/dsr84/viral_genomics/logs/megahit_test1.%j.out      
#SBATCH --error=/home/dsr84/viral_genomics/logs/megahit_test1.%j.err       

#note %j = job ID

#==== Load mamba/conda module (students: no need to change) ====
module load mamba
source $(mamba info --base)/etc/profile.d/conda.sh

#Activate the environment where you had MEGAHIT installed
conda activate megahit-env

#==== Set paths and filenames (students: edit this block!) ====

#Directory where the cleaned reads live
READDIR=/home/dsr84/viral_genomics/trimmed_reads

#Input read files (paired-end)
READ1=${READDIR}/EC-12_R1_trmPE5.fq.gz
READ2=${READDIR}/EC-12_R2_trmPE5.fq.gz

#Output directory (give it a name, it will be created by MEGAHIT)
OUTDIR=/home/dsr84/viral_genomics/megahit/megahit_out

#==== Run MEGAHIT ====

megahit \
  -1 ${READ1} \
  -2 ${READ2} \
  -t ${SLURM_CPUS_PER_TASK} \
  -o ${OUTDIR}

echo "Done. Contigs should be in: ${OUTDIR}/final.contigs.fa"

## 5. Assess assmebled reads
$ grep ">" final.contigs.fa | wc -l 
Output: 21900 reads 

Upload assembled reads to github
FROM LOCAL COMPUTER
$gcloud compute scp m12-controller:/home/dsr84/viral_genomics/megahit/megahit_out/final.contigs.fa ~/Desktop/
Uploaded to GitHub

$ module load mamba/
$ mamba activate megahit-env
$ mamba install -c bioconda seqkit
Say Y
$ seqkit stats -a final.contigs.fa

### Seqkit stats: 
file: final.contigs.fa
format: FASTA
type: DNA
num_seqs: 21900
sum_len: 12,797,626 (total length of all contigs)
min_len: 205 (not helpful)
avg_len: 584.4 (probably not helpful) 
max_len: 65,854 (helpful!)
Q1: 305
Q2: 388
Q3: 536
sum_gap: 0
N50: 573
N50_num: 1,533
Q20(%): 0
Q30(%): 0
AvqQual: 0 (that doesn't seem good)
GC%: 49.42
sum_n: 0

Dori: Created $ nano seqkit_stats in /home/dsr84/viral_genomics/megahit/megahit_out

# 3/19
Goals: Run virsorter, cluster

## 1. Virsorter
### Setup:
$ module load mamba
$ mamba create -y -n vs2-env -c conda-forge -c bioconda virsorter
$ mamba activate vs2-env
$ rm -rf db 	
Run set up for database:
$ virsorter setup -d db -j 4

### Script:
luby script
#!/bin/bash
#SBATCH --job-name=final_contigs_visorter1
#SBATCH --nodes=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=20G
#SBATCH --time=03:00:00
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=yc1201@georgetown.edu
#SBATCH --output=/home/yc1201/viral_genomics/visorter1.%j.out
#SBATCH --error=/home/yc1201/viral_genomics/visorter1.%j.err

 === Load mamba ===
module load mamba
conda activate vs2-env

Activate the environment where you had VirSorter2 installed
mamba activate vs2-env

=== Set paths and filenames ===
Set up directories
INDIR=/home/yc1201/viral_genomics/megahit/
OUTROOT=/home/yc1201/viral_genomics/visorter/
mkdir -p "${OUTROOT}"

SAMPLE_ID=sample5
INPUT="${INDIR}/dori_luby_final.contigs.fa"
OUTDIR="${OUTROOT}/vs2-${SAMPLE_ID}"

mkdir -p "${OUTDIR}"

==== Run virsorter2 with >5kb cutoff and DNA virus categories first
echo "Running VirSorter2 on ${INPUT}"
virsorter run \
  -w "${OUTDIR}" \
  -i "${INPUT}" \
  --db-dir /home/yc1201/viral_genomics/virsorter-db \
  --keep-original-seq \
  --include-groups dsDNAphage,NCLDV,ssDNA \
  --min-length 5000
echo "Done."

Dori's version (all on one line): virsorter run -w "/home/dsr84/viral_genomics/virsorter/vs2-SRR6996011" -i /home/dsr84/viral_genomics/megahit/megahit_out/final.contigs.fa --keep-original-seq --include-groups dsDNAphage,NCLDV,ssDNA --min-length 5000
(It worked!)

### Filtering for 5kB
==== Counting how many contigs
Need to use
mamba install -c bioconda seqkit
before
seqkit seq -m 5000 final-viral-combined.fa | grep -c ">"
which gives 128.
Then,
seqkit seq -m 5000 final-viral-combined.fa > final-viral-combined_min5kb.fa

# 3/23 

## Clustering using vclust

$ module load mamba
$ mamba create -n votu-env -c bioconda -c conda-forge vclust
$ mamba activate votu-env

$ vclust prefilter -i /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa -o fltr.txt
$ vclust align -i /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa -o ani.tsv --filter fltr.txt
$ vclust cluster -i ani.tsv -o clusters.tsv --ids ani.ids.tsv --metric ani --ani 0.95 --out-repr

$ awk '{print $2}' clusters.tsv | sort -u > votu_seeds.txt

$ mamba deactivate
$ mamba activate megahit-env

$ seqkit grep -f votu_seeds.txt /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa > votus_final.fna

$ wc -l votu_seeds.txt
$ grep -c ">" votus_final.fna

Final location for all files from clustering: /home/yc1201/viral_genomics/clustering

## Upload to class bucket
From inside the clustering directory, rename the final file to avoid confusion
$ mv votus_final.fna luby_dori_votus_final.fna
$ gcloud storage cp luby_dori_votus_final.fna gs://gu-biology-dept-class/

## CheckV
### Set up checkv database
$ module load checkv	
$ checkv download_database ./

### Script luby

#!/bin/bash
#SBATCH --job-name=checkv
#SBATCH --output=/home/yc1201/viral_genomics/checkv-%j.out
#SBATCH --error=/home/yc1201/viral_genomics/checkv-%j.err
#SBATCH --time=03:00:00
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=16G
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=yc1201@georgetown.edu

 ==== Load checkv program module (students: no need to change) ====

module load checkv


 ==== Set variables, paths, and filenames (students: edit this block!) ====

CHECKVDB="/home/yc1201/viral_genomics/checkv/checkv-db-v1.5"

SAMPLE_ID="vOTUs"
INPUT="/home/yc1201/viral_genomics/clustering/luby_dori_votus_final.fna"
OUTDIR="/home/yc1201/viral_genomics/checkv/${SAMPLE_ID}"

mkdir -p "${OUTDIR}"

 ==== run checkv (students: no need to change. The second line is the command) ====
echo "Running CheckV on ${INPUT}"
checkv end_to_end "${INPUT}" "${OUTDIR}" -d "${CHECKVDB}" -t ${SLURM_CPUS_PER_TASK}
echo "Done."

Our contigs look good! Mostly lower quality but we have a few with "comeplete" or "high quality"

## Check quality_summary_votus.tsv
$ less quality_summary_votus.tsv

## Grab pooled vOTUs from the bucket
$ gcloud storage cp gs://gu-biology-dept-class/ClassProject/votus_10kb_6samples.fna /home/yc1201/viral_genomics/bowtie

# 3/26
## Bowtie
### Script
#!/bin/bash
#SBATCH --job name=bowtie2_vOTUs
#SBATCH --output=/home/yc1201/viral_genomics/bowtie-%j.out
#SBATCH --error=/home/yc1201/viral_genomics/bowtie-%j.err
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=yc1201@georgetown.edu
#SBATCH --time=8:00:00
#SBATCH --mem=16G

 ---------SET UP----------
SAMPLE="sample6_yc1201"
INDEX="/home/yc1201/viral_genomics/bowtie/votu_index"
OUTPUTDIR="/home/yc1201/viral_genomics/bowtie"

--------- LOAD MODULES ----------
module purge
module load bowtie2/2.5.4

 --------- RUN BOWTIE2 AND PIPE TO SAMTOOLS ----------

-----First make output and log directories; move into OUTPUTDIR
mkdir -p "${OUTPUTDIR}"
cd "${OUTPUTDIR}"
mkdir -p logs

echo "Running bowtie2 on sample ${SAMPLE}"

bowtie2 -p 8 -x "${INDEX}" -1 "/home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R1_trmPE5.fq.gz" -2 " /home/yc1201/viral_genomics/trimmed_reads/SRR6996011_R2_trmPE5.fq.gz" \
| samtools view -bS - > "${SAMPLE}.bam"

echo "Finished running bowtie2 and performing compression"

---------sort and index files
echo "Sorting"
samtools sort "${SAMPLE}.bam" > "${SAMPLE}_sorted.bam"

echo "Indexing"
samtools index "${SAMPLE}_sorted.bam"

## After Bowtie
BAM file (binary code takes up less space than SAM file) was uploaded to the bucket
File was too large for normal cp function, so was downloaded to local computer and uploaded to gcloud bucket GUI

## Data visualization!
Combined vOTUs were converted into an excel spreadsheet and uploaded to R under the variable name votus. 
Dori script: 

tpm_threshold <- 10                      # keep vOTUs with max TPM > this
heatmap_colors <- c("#440154", "#31688e", "#35b779", "#fde725")

library(readxl)
library(pheatmap)

================================= Keep Contig and TPM columns only
tpm_cols <- grepl("TPM$", names(votus))
cov_tpm <- votus[ , c("Contig", names(votus)[tpm_cols])]

================================= Remove S1k141_26921_full
cov_tpm <- subset(cov_tpm, Contig != "S1k141_26921||full")

================================= Optional filter: drop very low-abundance vOTUs
cov_tpm$max_tpm <- apply(cov_tpm[ , -1], 1, max, na.rm = TRUE)
cov_tpm <- subset(cov_tpm, max_tpm > tpm_threshold)
cov_tpm$max_tpm <- NULL

=================================  If everything got filtered out (threshold too high), warn and stop
if (nrow(cov_tpm) == 0) {
  stop("No vOTUs passed the TPM threshold. Try lowering tpm_threshold.")
}

================================= Make matrix for heatmap (rows = vOTUs, cols = samples)
mat <- as.matrix(cov_tpm[ , -1])
rownames(mat) <- cov_tpm$Contig

================================= Log-transform for nicer color scaling
mat_log <- log10(mat + 1)

================================= Draw heat map
pheatmap(mat_log,
         cluster_rows = TRUE,
         cluster_cols = TRUE,
         scale = "none",
         color = colorRampPalette(heatmap_colors)(100),
         fontsize_row = 4,
         fontsize_col = 8,
         main = "vOTU relative abundance (log10 TPM + 1)")

### Resulting heatmap 

# Notes
Lines in scripts with ------ should be commented 

# Organization
## Dori
directory: viral_genomics
subdirectories: checkv, class_votus, db, fastqc, logs, megahit, raw, slurmscripts, trimmed_reads, virsorter, votus
checkv: files after running checkv
class_votus: file: votus_10kb_6samples.fna (combined votus)
db: conda_envs, group, hmm, rbs for virsorter
fastqc: output fastqc files before and after trimming
logs: logs and errors for all jobs
megahit: output files for megahit
raw: raw data
slurmscripts: all scripts
trimmed_reads: trimmed reads after several iterations of trimming parameters 
virsorter: outputs of virsorter
votus: sorted votus

## Luby
directory: viral_genomics
subdirectories: bowtie, checkv, clustering, raw, visorter, fastqc_out, trimmed_reads, megahit, virsorter-db
bowtie: slurm script for bowtie, sample5_yc1201_sorted.bam, sample5_yc1201.bam, sample5_yc1201_sorted.bam.bai, votus_10kb_6samples.fna, votu_index.1.bt2, votu_index.2.bt2, votu_index.3.bt2, votu_index.4.bt2, votu_index.rev.1.bt2, votu_index.rev.2.bt2
checkv: vOTUs, slurm script for checkv, checkv-db-v1.5
clustering: luby_dori_votus_final.fna, votu_seeds.txt, fltr.txt, clusters.tsv, ani.tsv, ani.ids.tsv
raw: SRR6996011 data
visorter: vs2-sample5, slurm script for virsorter
fastqc_out: SRR6996011.sra_2_fastqc.html (after trimmomatic)
trimmed_reads: slurm script for trimmomatic, trimmed reads
megahit: dori_luby_final.contigs.fa, slurm script for megahit
virsorter-db: virsorter database

