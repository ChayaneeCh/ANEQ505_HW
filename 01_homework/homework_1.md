**ANEQ Homework #1: Practice Importing Data, Demultiplexing Reads, and Denoising**

**Due Feb 12th at midnight**

**Name:**

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
cd ADD PATH TO DADA2 DIRECTORY

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_cow.qza \
--p-trim-left-f NUMBER \
--p-trim-left-r NUMBER \
--p-trunc-len-f NUMBER \
--p-trunc-len-r NUMBER \
--p-n-threads 6 \
--o-representative-sequences cow_seqs_dada2.qza \
--o-denoising-stats cow_dada2_stats.qza \
--o-table cow_table_dada2.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file cow_dada2_stats.qza \
--o-visualization YOUR_OUTPUT_FILENAME_HERE.qzv

qiime feature-table summarize \
--i-table cow_table_dada2.qza \
--m-sample-metadata-file ../metadata/cow_metadata.txt \
--o-visualization YOUR_OUTPUT_FILENAME_HERE.qzv

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2.qza \
--o-visualization YOUR_OUTPUT_FILENAME_HERE.qzv
```

	
Briefly **describe** the key information from each denoising output file:
1. Representative Sequences
2. Denoising Stats
3. Denoised Table

**Answer the following questions:**  
1. What is the mean reads per sample?
2. How long are the reads?
3. What is the maximum length of all your sequences?
4. Which sample (not including extraction controls starting with EC) lost the highest % of reads?
5. Why did you chose to trim or truncate where you did?

**To submit your homework from this document:**
write all of your commands here, then use command+P (for mac) or control+P (for windows) and search Git: commit. click it. then search for Git: Push and click it. go to your github online to check that it pushed correctly. we will check your github for homework credit. 