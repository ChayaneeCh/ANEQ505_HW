**ANEQ Homework #1: Practice Importing Data, Demultiplexing Reads, and Denoising**

**Due Feb 12th at midnight**

**Name: Chayanee Chanpanich** 

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------------
**Learning Objectives**

1. Practice independently running the first few steps of a Qiime2 workflow We will focus on importing data, demultiplexing, and denoising for homework 1.

2. Practice recording commands and editing code to match your analysis

3. Push to GitHub for credit

------------------------------------------------------------------------


**Cow Site Dataset**

20 adult dairy cattle were sampled (via swabbing) to determine the microbial community composition between 5 different body sites (nasal/skin/udder/fecal/oral).  We want you to determine if and how the microbial composition changes between sites over the course of multiple homework assignments.

We will first begin by copying raw sequencing data from a public folder on Alpine into a new directory (that you create) on your Alpine account.

**Steps:**

1. Log into Alpine using OnDemand and create a new directory for this new analysis in your _scratch_ directory. Hint: it should look something like: /scratch/alpine/$USER/cow/
2. Move into your new directory using OnDemand
3. Create the following sub-directories using OnDemand: 
	1. slurm
	2. taxonomy
	3. tree
	4. taxaplots
	5. dada2
	6. demux
	7. metadata
	8. core_metrics

4. Download the cow_barcodes.txt and cow_metadata.txt files from Canvas and upload them both to your metadata folder within your new cow directory. So, your filepath should look something like this: /scratch/alpine/$USER/cow/metadata

5. On OnDemand, go to your cow directory and open a new terminal

6. Copy the raw sequencing files from this public folder to **your new folder** using the terminal. Do **not** change the names of these files and folders. Hint: make sure you are in your new cow folder before you run this code (this will copy over the whole folder):  

```
cp -r /pl/active/courses/2024_summer/maw_2024/raw_reads .
```


5.    Launch an interactive session and load qiime2 within your cow directory. 

```
#launch an interactive session: 
ainteractive --ntasks=6 --time=02:00:00

#insert your code here to activate qiime. Hint: there should be 2 things you add here
module purge
module load qiime2/2024.10_amplicon
```


6.    Import the sequences/reads into a Qiime2-readable format (.qza). Note this might take 10-20 mins

```
qiime tools import \
--type EMPPairedEndSequences \
--input-path raw_reads \
--output-path cow_reads.qza
```


7.    Demultiplex the reads by submitting a job. Note this may take ~30 mins

a.    Go into your slurm directory using OnDemand. Create a new file named **demux.sh** so you can submit a job that will demultiplex your sequences quicker. Fill in the lines that need editing (denoted by capital letters or hashes) to this demultiplexing command and add that to your new script. 


```
#!/bin/bash
#SBATCH --job-name=demux
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=c837238655@colostate.edu

#What needs to go here in order to “turn on” qiime2? Hint: we do these 2 commands every time we activate qiime2!
module purge
module load qiime2/2024.10_amplicon

#change the following line if your file path looks different
cd /scratch/alpine/$USER/cow/demux

#Below is the command you will run to demultiplex the samples.

qiime demux emp-paired \
--m-barcodes-file ../metadata/cow_barcodes.txt \
--m-barcodes-column barcode \
--p-rev-comp-mapping-barcodes \
--p-rev-comp-barcodes \
--i-seqs ../cow_reads.qza \
--o-per-sample-sequences demux_cow.qza \
--o-error-correction-details cow_demux_error.qza

#visualize the read quality
qiime demux summarize \
--i-data demux_cow.qza \
--o-visualization demux_cow.qzv
```


 Run the script in your slurm directory as a job using: 
 ```
sbatch demux.sh
 ```

8.    Denoise. 

Fill in the blank to denoise your samples based on what you think should be trimmed (from the front of the reads) or truncated (from the ends of the reads) based on the demux_cow.qzv file. You can run this in the terminal or as a job.

```
cd /scratch/alpine/$USER/cow/dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_cow.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 6 \
--o-representative-sequences cow_seqs_dada2.qza \
--o-denoising-stats cow_dada2_stats.qza \
--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file cow_dada2_stats.qza \
--o-visualization cow_dada2_stats.qzv

qiime feature-table summarize \
--i-table cow_table_dada2.qza \
--m-sample-metadata-file ../metadata/cow_metadata.txt \
--o-visualization table.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization seqs.qzv
```

	
Briefly **describe** the key information from each denoising output file:
1. Representative Sequences \
		The representative sequences file, name as seqs.qzv, contains the unique amplicon sequence variants (ASVs) identified after DADA2 denoising. In this dataset, there are 4,653 representative sequences, each corresponding to a distinct microbial sequence variant. The sequences range in length from 250 to 427 bp, with a mean length of approximately 253 bp. Additionally, each sequence is assigned a unique Feature ID and can be exported or BLASTed against the NCBI nucleotide (nt) database for manual sequence identification. These representative sequences can be used for downstream analysis such as taxonomy assignment and phylogenetic tree construction.
2. Denoising Stats \
		The denoising stats file (cow_dada2_stats.qzv) summarizes how reads were processed through each step of the DADA2 pipeline for every sample. It reports the number of reads at each stage, including input, filtered, denoised, merged, and non-chimeric reads. Moreover, it also provide percentages such as the percentage of input reads that passed filtering, percentage of input reads successfully merged, and percentage of input reads that remained non-chimeric. These percentages indicate how much data was retained at each step and help evaluate data quality. Samples with lower percentage might have experienced poor read quality, poor overlap during merging, or higher levels of chimeric sequences.
3. Denoised Table \
		The denoised table (table.qzv.qza) consists of three main sections: Overview, Interactive Sample Detail, and Feature Detail, each serving a specific purpose in evaluating the dataset. Firstly, overview section summarizes the overall sequencing results, showing that 147 samples and 4,653 ASVs were retained after denoising, with a total of 1,634,012 reads. The frequency per sample statistics indicate variation in sequencing depth across samples, with a mean of 11,115.7 reads per sample and a maximum of 33,768 reads. This section is used to assess whether sequencing depth is sufficient and relatively balanced across samples. The frequency per feature summary helps to describe the distribution of ASVs, showing the presence of rare (minimum frequency = 2) versus highly abundant features (maximum frequency = 104,080). Secondly, interactive Sample Detail section allows visualization of individual sample read counts and comparison across metadata categories (e.g., swab_label, BarcodeSequence, cow_id etc.), which helps identify uneven sequencing depth that could affect downstream diversity analyses. Lastly, feature Detail section lists each ASV, its total frequency, and the number of samples in which it was observed, enabling identification of widely distributed ASVs versus those restricted to a few samples, which is important for interpreting microbial community structure and ecological patterns.

**Answer the following questions:**  
1. What is the mean reads per sample? \
	The mean reads per sample after denoising was **11,115.7 reads per sample (11,116 reads when rounded)**.
2. How long are the reads? \
	The reads are approximately **250 base pairs (bp) long** for both forward and reverse reads.
3. What is the maximum length of all your sequences? \
		The maximum length of all representative sequences is **427 nucleotides**.
4. Which sample (not including extraction controls starting with EC) lost the highest % of reads? \
		The sample **2019.3.14.cow.oral.20** lost the highest percentage of reads, retaining only 8.76% of its input reads after denoising (a loss of 91.24%).
5. Why did you chose to trim or truncate where you did? \
		Based on the demultiplexing quality plots (demux_cow.qzv), the minimum sequence length identified during subsampling was 251 bases, confirming that reads were approximately 250 bp long. The quality scores remained high (above Q30) across bases 1–250 for both forward and reverse reads, but the final base (position 251) of the reverse reads showed a sharp drop in quality (median = Q13). Therefore, I set **trim-left to 0 and truncated both reads at 250 bp** to remove the low-quality tail while retaining high-quality sequence data.

**To submit your homework from this document:**
write all of your commands here, then use command+P (for mac) or control+P (for windows) and search Git: commit. click it. then search for Git: Push and click it. go to your github online to check that it pushed correctly. we will check your github for homework credit. 