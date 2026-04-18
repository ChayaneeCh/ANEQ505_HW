```
sinteractive --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

```
module purge
module load qiime2/2026.1_amplicon
```

```
module purge
module load qiime2/2024.10_amplicon
```

### Demultiplexing and Denoising

convert manifest (.csv) file to .txt file
```
tr ',' '\t' < manifest_run2_16S.csv > manifest_run2.txt
```

import run2
```
qiime tools import \
--type "SampleData[PairedEndSequencesWithQuality]" \
--input-format PairedEndFastqManifestPhred33V2 \
--input-path manifest/manifest_run2.txt \
--output-path demux/demux_run2.qza
```

Check Data Quality:
```
cd demux
```

```
qiime demux summarize \
--i-data demux_run2.qza \
--o-visualization demux_run2.qzv
```

**Denoise (2026 version)**
```
cd ../dada2
```

```
#!/bin/bash
#SBATCH --job-name=denoise
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=2:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=denoise_slurm-%j.out
#SBATCH --qos=normal

module purge
module load qiime2/2026.1_amplicon

#Path
cd /scratch/alpine/c837238655@colostate.edu/soil_project/dada2

qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_run2.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 12 \
--o-representative-sequences seqs_dada2.qza \
--o-denoising-stats dada2_stats_run2.qza \
--o-table table_run2.qza \
--o-base-transition-stats base-transition-stats.qza
```

visualization of each of these stats outputs:
```
qiime metadata tabulate \
--m-input-file dada2_stats_run2.qza \
--o-visualization dada2_stats_run2.qzv
```

DADA2 TABLE FILE (2026 version):
```
qiime feature-table summarize \
  --i-table table_run2.qza \
  --o-feature-frequencies feature-frequencies.qza \
  --o-sample-frequencies sample-frequencies.qza \
  --o-summary dada2_visual_summary.qzv
```

DADA2 SEQS FILE:
```
qiime feature-table tabulate-seqs \
--i-data seqs_dada2.qza \
--o-visualization seqs.qzv
```

### Taxonomy, Taxa Barplots, Filtering, Phylogenetic Tree

Go to the taxonomy folder:
```
cd /scratch/alpine/c837238655@colostate.edu/soil_project/taxonomy
```

download classifier:
```
wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```

classify taxonomy:
```
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/seqs_dada2.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2.qza
```

 make taxonomy into a visualization: *Got Killed => submit slurm job instead*
```
qiime metadata tabulate \
--m-input-file taxonomy_gg2.qza \
--o-visualization taxonomy_gg2.qzv
```

going to group by *Treatment* (in a metadata column):
```
qiime feature-table group \
--i-table ../dada2/table_run2.qza \
--m-metadata-file ../metadata/metadata.tsv \
--m-metadata-column Treatment \
--p-mode mean-ceiling \
--p-axis sample \
--o-grouped-table ../dada2/table_treatment.qza
```

Slurm Job
```
#!/bin/bash
#SBATCH --job-name=Taxa
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=4:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=Taxa_Treatment-%j.out
#SBATCH --qos=normal

module purge
module load qiime2/2026.1_amplicon

cd /scratch/alpine/c837238655@colostate.edu/soil_project/taxonomy

# Classify taxonomy
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/seqs_dada2.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2.qza

# Visualization
qiime metadata tabulate \
--m-input-file taxonomy_gg2.qza \
--o-visualization taxonomy_gg2.qzv

# Grouping by Treatment
qiime feature-table group \
--i-table ../dada2/table_run2.qza \
--m-metadata-file ../metadata/metadata.tsv \
--m-metadata-column Treatment \
--p-mode mean-ceiling \
--p-axis sample \
--o-grouped-table ../dada2/table_treatment.qza
```

Remove contaminating features:
```
qiime taxa filter-table \
--i-table ../dada2/table_treatment.qza \
--i-taxonomy taxonomy_gg2.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_treatment_nomitochloro.qza
```

**Taxa Bar Plot**

Go to the taxa plots directory: `cd ../taxaplots`
Generate the taxa bar plot:
```
qiime taxa barplot \
--i-table ../dada2/table_treatment_nomitochloro.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \
--o-visualization taxa_barplot_treatment_nomitochloro.qzv
```

*Check the controls*
Filter samples:
```
qiime feature-table filter-samples \
--i-table ../dada2/table_run2.qza \
--m-metadata-file ../metadata/metadata.tsv \
--p-where "[Treatment]='Control'" \
--o-filtered-table ../dada2/table_controls.qza
```
Create a taxa barplot with our table with just controls:
```
qiime taxa barplot \
--i-table ../dada2/table_controls.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \
--m-metadata-file ../metadata/metadata.tsv \
--o-visualization taxa_barplot_controls.qzv
```

*Create the full taxa bar plot of all the samples*
```
qiime taxa filter-table \
--i-table ../dada2/table_run2.qza \
--i-taxonomy taxonomy_gg2.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_nomitochloro.qza  
  
#visualize:   
qiime taxa barplot \
--i-table ../dada2/table_nomitochloro.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2.qza \
--m-metadata-file ../metadata/metadata.tsv \
--o-visualization taxa_barplot_all_samples.qzv
```

**Phylogenetic Trees**

Go to the tree directory: `cd ../tree`
Get the reference backbone:
```
wget https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza
```

Create Slurm Job:
```
#!/bin/bash  
#SBATCH --job-name=sepp  
#SBATCH --nodes=1  
#SBATCH --ntasks=24  
#SBATCH --partition=amilan  
#SBATCH --time=04:00:00  
#SBATCH --mail-type=ALL  
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=sepp_slurm-%j.out  
#SBATCH --qos=normal  
  
  
#Activate qiime  
module purge
module load qiime2/2026.1_amplicon
  
# go to your decomp directory  
cd /scratch/alpine/$USER/soil_project

#frament insertion sepp  
qiime fragment-insertion sepp \--i-representative-sequences dada2/seqs.qza \--i-reference-database tree/2022.10.backbone.sepp-reference.qza \--o-tree tree/tree_gg2.qza \--o-placements tree/tree_gg2_placements.qza \--p-threads 4
```