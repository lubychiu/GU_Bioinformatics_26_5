# GU_Bioinformatics_26
This project implemented a viral metagenomics workflow to process raw sequencing data, improve read quality, assemble contigs, and identify and characterize viral populations in Swedish bog that had been under permafrost.

Raw sequencing data (SRR6996011) were retrieved using SRA tools and converted to FASTQ format. Trimmomatic and FASTQC were used to assess and trim poor-quality reads.  

High-quality reads were assembled into contigs using MEGAHIT, producing 21,900 contigs with a total assembly length of ~12.8 Mbp and a maximum contig length of 65.8 kb. Viral contigs were identified using VirSorter2, followed by length-based filtering (>5 kb) to improve downstream alignment reliability, resulting in 128 candidate viral contigs.

These contigs were clustered into viral operational taxonomic units (vOTUs) using vclust with a 95% ANI threshold, approximating species-level groupings. vOTU seeds were extracted and assessed for completeness and quality using CheckV. 

vOTUs were aligned to reference data using Bowtie2, and relative abundances were visualized in R as a clustered heatmap.
