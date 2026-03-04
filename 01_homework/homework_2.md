**ANEQ Class Assignment #2: Practice Qiime2 Workflow**

Due: March 5th 2026 at midnight


**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice independently running a partial Qiime2 workflow on new data using both the command line and slurm to submit jobs. 
2. Practice recording commands and editing code to match your analysis.
3. Perform taxonomic assignments using greengenes2, filtering out unwanted taxonomy and large amplicons, run another job script for generating a SEPP tree.
--------------------------------------------------

**Cow Site Data Workflow**, part 2

Load qiime2 in a terminal session after you go into the taxonomy folder

```
# Insert the two commands to activate qiime2
module purge
module load qiime2/2024.10_amplicon
```

### Remove long (300+ base pair) amplicons from the representative sequences file and the feature table

```
# filter out any large amplicons from the seqs and table (because they are contaminates)

cd /scratch/alpine/$USER/cow/dada2

qiime feature-table filter-seqs \
--i-data cow_seqs_dada2.qza \
--m-metadata-file cow_seqs_dada2.qza \
--p-where 'length(sequence) < 300' \
--o-filtered-data cow_seqs_dada2_filtered300.qza

qiime feature-table tabulate-seqs \
--i-data cow_seqs_dada2_filtered300.qza \
--o-visualization cow_seqs_dada2_filtered300.qzv

qiime feature-table filter-features \
--i-table cow_table_dada2.qza \
--m-metadata-file cow_seqs_dada2_filtered300.qza \
--o-filtered-table cow_table_dada2_filtered300.qza
  
qiime feature-table summarize \
--i-table cow_table_dada2_filtered300.qza \
--m-sample-metadata-file ../metadata/cow_metadata.txt \
--o-visualization cow_table_dada2_filtered300.qzv
    
```


### Classify taxonomy using GreenGenes2

First get the Greengenes2 database:

```
cd /scratch/alpine/$USER/cow/taxonomy
```

```
wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

Classify taxonomy using GreenGenes2 classify the ASVs (takes about 5 mins). ~={red}(1point)=~
```
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/cow_seqs_dada2_filtered300.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2_filtered.qza
```

Visualize the taxonomy of your ASVs: ~={red}(1point)=~
```
qiime metadata tabulate \
--m-input-file taxonomy_gg2_filtered.qza \
--o-visualization taxonomy_gg2_filtered.qzv
```

- Filter mitochondria and chloroplast out to generate a filtered feature table, keep only ASVs with a class or lower taxonomy. fill in the blank (--p-exclude) to exclude these DNA. Fill in the blank to include only class level or below classifications. (~={red}1point)=~
```
qiime taxa filter-table \
--i-table ../dada2/cow_table_dada2_filtered300.qza \
--i-taxonomy taxonomy_gg2_filtered.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza
```

- Visualize the taxa bar plot
```
qiime taxa barplot \
--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \
--i-taxonomy taxonomy_gg2_filtered.qza \
--m-metadata-file ../metadata/cow_metadata.txt \
--o-visualization ../taxaplots/taxa_barplot_nomitochloro_gg2_filtered300.qzv
```

## Filtered Taxa Bar Plot Questions ~={red}(10 points)=~

**Question 1**: Attach a picture of your taxa bar plot, organized by cow sampling location (body_site) at the level 7 taxonomic level. What general trends do you notice?  \

The taxa bar plot below shows differences in microbial community composition across the different body sites. Most fecal samples are dominated by three main species: Faecousia sp000434635, Cryptobacteroides sp902787255, and Treponema_D porcinum. These species make up a large proportion of the microbial community in fecal samples. Fecal, skin, and udder samples also show a wide distribution of colors across taxa, indicating diverse microbial communities. In contrast, many oral and nasal samples are strongly dominated by one or two taxa, which appear as large blocks in the bar chart. Across all sample types, the lower portions of the bars contain many thin colored bands, indicating a high number of low-abundance taxa contributing to overall diversity.

![barplot](image/HW2_level-7-bars.svg)
![legend](image/HW2_level-7-legend.svg)


**Question 2**: What are the top 2 most abundant bacterial **classes** in the fecal samples? 

The two most abundant bacterial classes in the fecal samples are **Clostridia (Clostridia_258483)** and **Bacteroidia**.

**Question 3**: What highly abundant ASV is shared between both the udder and skin samples?

The highly abundant ASV shared between both the udder and skin samples is **Faecousia sp000434635**, classified within the class **Clostridia_258483** (d__Bacteria; p__Bacillota_A_368345; c__Clostridia_258483; o__Oscillospirales; f__Oscillospiraceae_88309; g__Faecousia; s__Faecousia sp000434635). This ASV appears at high relative abundance in both body sites on the taxa bar plot.

**Question 4**: Which samples (still sorted by body_site) have higher alpha diversity in terms of observed features?

Fecal, skin, and udder samples appear to have the highest diversity. However, alpha diversity is difficult to assess precisely from the species-level taxa bar plot because many taxa are present at low relative abundance across samples. Therefore, an alpha diversity analysis is needed to more accurately quantify microbial diversity.

**Question 5**: do all samples contain archaea as well?

No, not all samples contain Archaea. Archaea are present in some skin and udder samples, but they are not detected in every sample type.

**Question 6**: why do we filter out sp004296775?

sp004296775 is not a real bacterial taxon of interest in cow microbiome samples.  It represents another chloroplast.

**Question 7**: what is the difference between these two flags? 
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \

**--p-exclude** flag removes specific unwanted taxa which are mitochondria, chloroplast, and sp004296775 from the feature table. On the other hand, **--p-include c__** flag keeps only ASVs that are classified at least to the class level. Together, these filters remove contaminants and poorly classified sequences to improve the accuracy of downstream analyses.

**Question 8**: do the positive controls look the same as each other? Yes or No?

**Yes**, positive controls look similar to each other, showing consistent dominant taxa and similar relative abundance patterns across the samples.

**Question 9**: Do the negative/extraction controls (Samples labeled as EC), look like the positive controls? Yes or no? 

**No**, the negative/extraction controls (EC samples) do not look like the positive controls. The EC samples show different dominant taxa and overall relative abundance patterns compared to the positive controls, indicating distinct community compositions.

**Question 10**: do the negative/extraction controls (Samples labeled as EC), look like the real samples? Yes or no?

**No**, the negative/extraction controls have lower diversity and simpler community composition compared to the real samples. The EC samples are dominated by fewer taxa and do not show the complex microbial patterns observed in the biological samples.

## Phylogenetic tree ~={red}(1point)=~

Create a job script to run the phylogenetic tree building. Remember you must start a new terminal session, navigate to your slurm directory, and then submit the job. You do NOT need to start any other interactive sessions.This job will take about an hour. 

Go to OnDemand and create a new text file for your job script
```
nano tree.sh
```

```
#!/bin/bash
#SBATCH --job-name=tree
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=04:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal

#Activate qiime
#Insert the two commands you need to load qiime2
module purge
module load qiime2/2024.10_amplicon

#Get reference
wget --no-check-certificate -P ../tree https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza


#Command
qiime fragment-insertion sepp \
--i-representative-sequences ../dada2/cow_seqs_dada2_filtered300.qza \
--i-reference-database ../tree/2022.10.backbone.sepp-reference.qza \
--o-tree ../tree/tree_gg2.qza \
--o-placements ../tree/tree_placements_gg2.qza
```

- submit the job from the terminal
```
#submit the job
sbatch tree.sh
```
We will use this file in the next homework!

### Once this job finishes, copy and paste what the slurm email says here ~={red}(1 point)=~: 


Job ID: 24414230  
Cluster: alpine  
User/Group: [c837238655@colostate.edu](mailto:c837238655@colostate.edu)/[c837238655pgrp@colostate.edu](mailto:c837238655pgrp@colostate.edu)  
State: COMPLETED (exit code 0)  
Nodes: 1  
Cores per node: 8  
CPU Utilized: 02:25:04  
CPU Efficiency: 12.43% of 19:27:28 core-walltime  
Job Wall-clock time: 02:25:56  
Memory Utilized: 7.81 GB  
Memory Efficiency: 26.03% of 30.00 GB (3.75 GB/core)