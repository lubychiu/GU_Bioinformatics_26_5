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
# y
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
# run fastqc 
$ fastqc -o fastqc_out SRR6996011.fastq
```
Output FASTQC Luby path: /yc1201//viral_genomics/fastqc_out

### 4. Evaluate FASTQC of raw data
- Total sequences: 7789043
- Sequences flagged as poor quality: 0
- Sequence length: 35-301. 
- % GC: 46
- Total bases: 1.7 Gbp
- Per-base N content: increases after 264 bp
- Sequence duplication level: 21.49 %
- No adapter content
- Overall read quality begins to be majority in the red area after 235 bp

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
Based on FASTQC on raw reads, quality is worst after base 186 (all in the red) 
We are planning to use the same parameters for HW 4

Initial trimming script can be found in trimmomatic1
Parameters:
```bash
ILLUMINACLIP:$adapters:2:30:10
LEADING:10
TRAILING:10
SLIDINGWINDOW:4:15
MINLEN:75
```

## 5. Run FASTQC again
```bash
$fastqc -o trimmed_fastqc_out SRR6996011_R1_TrmPE1.fq.gz RR6996011_R2_TrmPE1.fq.gz
$fastqc -o trimmed_fastqc_out SRR6996011_R1_TrmSE1.fq.gz RR6996011_R2_TrmSE1.fq.gz
```

## 6. Downloading fastqc outputs to local computer
### Incorrect 
```bash
# FROM LOCAL COMPUTER
$ gcloud compute scp/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R1_TrmPE1.fastqc.html ~/Downloads/
```
We got errors saying the source must be remote if the file is local, but are unsure what that means ^

### What worked: 
```bash
# FROM LOCAL COMPUTER 
$ gcloud compute scp m12-controller:/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R1_TrmPE1.fastqc.html ~/Downloads/
# Then the same for R2 PE
$ gcloud compute scp m12-controller/home/yc1201/viral_genomics/trimmed_reads/trimmed_fastqc_out/SRR6996011_R2_TrmPE1.fastqc.html ~/Downloads/
```

## 7. Evaluating trimmomatic results
R1 looks great! R2 not so much, so we're going to trim them both again using the same parameters

## 8. Run trimmomatic and FASTQC again 
Using the same parameters on the output of the first round.
New outputs: PE2
```bash
$ fastqc -o trimmed_fastqc_out2 SRR6996011_R1_trmPE2.fq.gz SRR6996011_R2_trmPE2.fq.gz
```
Oh they still don't look great, probably because using the exact same parameters doesn't do anything. 
This did not change our results, as the parameters have already trimmed as much as they ever will. 

# 3/17
## 1. Re-run trimmomatic on the raw files with a changed window parameter. 
We changed SLIDINGWINDOW to 2:15 instead of 4:15.
That didn't work! Let's try again. 

## 2. Re-run trimmomatic on the raw files with a changed window parameter, again.
We changed SLIDINGWINDOW to 2:20 instead of 2:15. Reads are now of an adequately high quality.
Slurm script can be found in final_trimmomatic. 

## 3. Evaluation of final trimmed reads
- Total paired end sequences surviving:	4883658, 62% of sequences
- Sequences flagged as poor quality: 0
- Sequence length: 75-301, compared to 35-301. Sequences under 50 bp were trimmed, and others were excluded for poor quality
- % GC: 44, previously 46
- Total bases: 838.7 Mbp, previously 1.7 Gbp, 49% of bases survived. 
- Per-base N content, 0
- No overrepresented sequences
- No adapter content
- R2 has some overlap in the red area after 240 bp, none for R1


## 4. Reorganize files 
### Dori organization
Overall folder: viral_genomics
Within viral_genomics: fastqc, logs, megahit, raw, slurmscripts, trimmed_reads

### Luby organization
Overall folder: viral_genomics
Within viral_genomics: raw, fastqc_out, trimmed_reads, megahit

## 5. Megahit
Megahit assembles quality reads into contigs. 
Slurm script can be found in megahit file.

## 6. Assess assmebled reads

```bash
$ grep ">" final.contigs.fa | wc -l
```
Output: 21900 reads 

Upload assembled reads to github
```bash
# FROM LOCAL COMPUTER
$ gcloud compute scp m12-controller:/home/dsr84/viral_genomics/megahit/megahit_out/final.contigs.fa ~/Desktop/
```
Uploaded to GitHub

```bash
$ module load mamba/
$ mamba activate megahit-env
$ mamba install -c bioconda seqkit
# Say Y
$ seqkit stats -a final.contigs.fa
```

### Seqkit stats: 
- file: final.contigs.fa
- format: FASTA
- type: DNA
- num_seqs: 21900
- sum_len: 12,797,626 (total length of all contigs)
- min_len: 205 (not helpful)
- avg_len: 584.4 (probably not helpful) 
- max_len: 65,854 (helpful!)
- Q1: 305
- Q2: 388
- Q3: 536
- sum_gap: 0
- N50: 573
- N50_num: 1,533
- Q20(%): 0
- Q30(%): 0
- AvqQual: 0 
- GC%: 49.42
- sum_n: 0

We decided to move forward with any sample that included a maximum contig of over 10,000 base pairs, so our data moved forward. 
Shorter contigs are to be expected in such a diverse and messy sample. The presence of a maximum contig of 65,854 bp indicates the presence of some identifiable viral contigs. 

### Organized stats
```bash
$ nano seqkit_stats in /home/dsr84/viral_genomics/megahit/megahit_out
```

# 3/19
Goals: Run virsorter, cluster
Virsorter identifies viral contigs out of our sample which may contain bacteria or other microorganisms. It does this using complex pattern recognition and identifying non-cellular genes.

## 1. Virsorter
Install virsorter
```bash
$ module load mamba
$ mamba create -y -n vs2-env -c conda-forge -c bioconda virsorter
$ mamba activate vs2-env
$ rm -rf db
```
Load database for virsorter
```bash
$ virsorter setup -d db -j 4 --conda-frontend conda
```

Luby's virsorter slurm script can be found in virsorter file. 

Dori's version (all on one line): 
```bash
$ virsorter run -w "/home/dsr84/viral_genomics/virsorter/vs2-SRR6996011" -i /home/dsr84/viral_genomics/megahit/megahit_out/final.contigs.fa --keep-original-seq --include-groups dsDNAphage,NCLDV,ssDNA --min-length 5000
```
(It worked!)

Final results are in a file called final-viral-combined.fa, not loaded here. This is the file that moves through the rest of the workflow. 

### Filtering for 5kB
Filtering for contigs over 5kB is performed after virsorter.
The purpose of this is to guarentee the highest chance of alignment software (bowtie2) being able to match viral contigs to their databases.
While there are viruses with genomes smaller than 5kB, they will not be present in this analysis. 

Filter for contigs <5000 kB and count how many contigs remain
```bash
$ mamba install -c bioconda seqkit
$ seqkit seq -m 5000 final-viral-combined.fa | grep -c ">"
```
Moving into a new file
```bash
seqkit seq -m 5000 final-viral-combined.fa > final-viral-combined_min5kb.fa
```
128 contigs transfered. 

# 3/23 
Goal: vOTU generation
vOTUs are clusters of similar viral sequences, assumed to be the same viral species. Unlike cellular organisms that universally contain the 16S rRNA gene as a marker, viral clusters must be identified based on overlapping sequences in key areas. 

## Clustering using vclust
Install vclust
```bash
$ module load mamba
$ mamba create -n votu-env -c bioconda -c conda-forge vclust
$ mamba activate votu-env
```

```bash
# Prefilter similar genome sequence pairs before conducting pairwise alignments
$ vclust prefilter -i /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa -o fltr.txt

# Align similar genome sequence pairs and calculate pairwise ANI measures.
$ vclust align -i /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa -o ani.tsv --filter fltr.txt

# Cluster genome sequences based on given ANI measure and minimum threshold (these files were generated in the previous steps)
$ vclust cluster -i ani.tsv -o clusters.tsv --ids ani.ids.tsv --metric ani --ani 0.95 --out-repr

# make a list of the vOTU headers
$ awk '{print $2}' clusters.tsv | sort -u > votu_seeds.txt
```
Entering vOTU seed sequences into a new file
```bash
# must deactivate mamba and enter into a new mamba environment 
$ mamba deactivate
$ mamba activate megahit-env

$ seqkit grep -f votu_seeds.txt /home/yc1201/viral_genomics_visorter/vs2-sample5/final-viral-combined_min5kb.fa > votus_final.fna

$ wc -l votu_seeds.txt
# should have the same number of clusters 
$ grep -c ">" votus_final.fna
```

Final location for all files from clustering: /home/yc1201/viral_genomics/clustering

## Upload to class bucket
From inside the clustering directory, rename the final file to avoid confusion

```bash
$ mv votus_final.fna luby_dori_votus_final.fna
$ gcloud storage cp luby_dori_votus_final.fna gs://gu-biology-dept-class/
```

## CheckV
CheckV assesses the quality and completeness of vOTUs
### Set up checkv database

```bash
$ module load checkv	
$ checkv download_database ./
```
Slurm script can be found in checkv file.

## Check quality_summary_votus.tsv
$ less quality_summary_votus.tsv

Our contigs look good! Mostly lower quality but we have a few with "comeplete" or "high quality"

## Grab pooled vOTUs from the bucket
$ gcloud storage cp gs://gu-biology-dept-class/ClassProject/votus_10kb_6samples.fna /home/yc1201/viral_genomics/bowtie

# 3/26
## Bowtie

Bowtie identifies viral species by aligning vOTUs to databases. Slurm script can be found in bowtie.

## After Bowtie
BAM file (binary code takes up less space than SAM file) was uploaded to the bucket
File was too large for normal cp function, so was downloaded to local computer and uploaded to gcloud bucket via GUI

## Data visualization!
Combined vOTUs were converted into an excel spreadsheet and uploaded to R under the variable name votus. 
Dori RStudio script: 

```bash
library(readxl)
votus <- ClassProject_votus.xlsx
tpm_threshold <- 10                      # keep vOTUs with max TPM > this
heatmap_colors <- c("#440154", "#31688e", "#35b779", "#fde725")      
library(pheatmap)

# Keep Contig and TPM columns only
tpm_cols <- grepl("TPM$", names(votus))
cov_tpm <- votus[ , c("Contig", names(votus)[tpm_cols])]

# Remove S1k141_26921_full
cov_tpm <- subset(cov_tpm, Contig != "S1k141_26921||full")

# Optional filter: drop very low-abundance vOTUs
cov_tpm$max_tpm <- apply(cov_tpm[ , -1], 1, max, na.rm = TRUE)
cov_tpm <- subset(cov_tpm, max_tpm > tpm_threshold)
cov_tpm$max_tpm <- NULL

# If everything got filtered out (threshold too high), warn and stop
if (nrow(cov_tpm) == 0) {
  stop("No vOTUs passed the TPM threshold. Try lowering tpm_threshold.")
}

# Make matrix for heatmap (rows = vOTUs, cols = samples)
mat <- as.matrix(cov_tpm[ , -1])
rownames(mat) <- cov_tpm$Contig

# Log-transform for nicer color scaling
mat_log <- log10(mat + 1)

# Draw heat map
pheatmap(mat_log,
         cluster_rows = TRUE,
         cluster_cols = TRUE,
         scale = "none",
         color = colorRampPalette(heatmap_colors)(100),
         fontsize_row = 4,
         fontsize_col = 8,
         main = "vOTU relative abundance (log10 TPM + 1)")
```

Resulting heatmap can be found in vOTU_Heatmap.png

# Final Organization
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

