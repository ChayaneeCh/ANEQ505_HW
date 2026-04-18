# Load QIIME2
```
sinteractive --time=02:00:00 --partition=amilan --nodes=1 --ntasks=6 --qos=normal

module purge

module load qiime2/2024.10_amplicon
```
# Import files
```
qiime tools import \
--type "SampleData[PairedEndSequencesWithQuality]" \
--input-format PairedEndFastqManifestPhred33V2 \
--input-path manifest/manifest_run2_16S.tsv \
--output-path demux/demux.qza
```
# Check data quality
```
cd demux

qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv
```
# Denoise
## Slurm script contents
```
#!/bin/bash
#SBATCH --job-name=denoise
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/dada2

# Denoise with DADA2
qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux.qza \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 12 \
--o-representative-sequences rep_seqs.qza \
--o-denoising-stats dada2_stats.qza \
--o-table table.qza

#Visualize the denoising results:
qiime metadata tabulate \
--m-input-file dada2_stats.qza \
--o-visualization dada2_stats.qzv

qiime feature-table summarize \
--i-table table.qza \
--m-sample-metadata-file ../metadata/metadata.tsv \
--o-visualization table.qzv

qiime feature-table tabulate-seqs \
--i-data rep_seqs.qza \
--o-visualization rep_seqs.qzv
```
## Run Slurm script
```
sbatch denoise.sh
```
# Filter out large ASVs (off-target taxa)
## Slurm script contents
```
#!/bin/bash
#SBATCH --job-name=filter300
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/dada2

# Filter out large ASVs (off-target taxa)
qiime feature-table filter-seqs \
--i-data rep_seqs.qza \
--m-metadata-file rep_seqs.qza \
--p-where 'length(sequence) < 300' \
--o-filtered-data rep_seqs_filtered300.qza

qiime feature-table tabulate-seqs \
--i-data rep_seqs_filtered300.qza \
--o-visualization rep_seqs_filtered300.qzv

qiime feature-table filter-features \
--i-table table.qza \
--m-metadata-file rep_seqs_filtered300.qza \
--o-filtered-table table_filtered300.qza
  
qiime feature-table summarize \
--i-table table_filtered300.qza \
--m-sample-metadata-file ../metadata/metadata.tsv \
--o-visualization table_filtered300.qzv
```
# Taxonomic classification
## Download pre-trained Greengenes2 classifier
```
mkdir taxonomy

cd taxonomy

wget --no-check-certificate https://ftp.microbio.me/greengenes_release/2024.09/2024.09.backbone.v4.nb.qza
```
## Taxonomic classification Slurm script
```
#!/bin/bash
#SBATCH --job-name=taxonomy
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/taxonomy

# Classify using Greengenes2 pre-trained classifier
qiime feature-classifier classify-sklearn \
--i-reads ../dada2/rep_seqs_filtered300.qza \
--i-classifier 2024.09.backbone.v4.nb.qza \
--o-classification taxonomy_gg2_filtered300.qza

# Generate taxonomy visualization
qiime metadata tabulate \
--m-input-file taxonomy_gg2_filtered300.qza \
--o-visualization taxonomy_gg2_filtered300.qzv
```
## Run Slurm script
```
sbatch taxonomy.sh
```
Inspect output for off-target taxa.
# Filter feature table by taxonomy and generate taxa barplots
```
#!/bin/bash
#SBATCH --job-name=filter_taxonomy_barplots
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/taxonomy

# Filter feature table to remove mitochondria and chloroplasts
qiime taxa filter-table \
--i-table ../dada2/table_filtered300.qza \
--i-taxonomy taxonomy_gg2_filtered300.qza \
--p-exclude mitochondria,chloroplast,sp004296775 \
--p-include c__ \
--o-filtered-table ../dada2/table_nomitochloro_gg2_filtered300.qza

# Filter representative sequences to match filtered feature table
qiime feature-table filter-seqs \
--i-data ../dada2/rep_seqs_filtered300.qza \
--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \
--o-filtered-data ../dada2/rep_seqs_nomitochloro_gg2_filtered300.qza

# Generate taxa barplot for all samples
cd ../taxaplots

qiime taxa barplot \
--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered300.qza \
--o-visualization taxa_barplot_nomitochloro_gg2_filtered300.qzv

# Check taxa barplots of lab process controls only
qiime feature-table filter-samples \
--i-table ../dada2/table_nomitochloro_gg2_filtered300.qza \
--m-metadata-file ../metadata/metadata.tsv \
--p-where "[Treatment]='Lab Control'" \
--o-filtered-table ../dada2/table_controls.qza

qiime taxa barplot \
--i-table ../dada2/table_controls.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered300.qza \
--m-metadata-file ../metadata/metadata.tsv \
--o-visualization taxa_barplot_controls.qzv
```
## Run Slurm script
```
sbatch filter_taxonomy_barplots.sh
```
Inspect controls for patterns that look like real samples. (In our case, lab controls look like very low-abundance, which suggests very minor contamination.)
# Phylogenetic tree
## Get Greengenes2 backbone for SEPP insertion tree
```
cd ../tree

wget https://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.sepp-reference.qza
```
## Phylogenetic tree Slurm script
```
#!/bin/bash
#SBATCH --job-name=sepp_tree
#SBATCH --nodes=1
#SBATCH --ntasks=24
#SBATCH --partition=amilan
#SBATCH --time=04:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/tree

# SEPP fragment insertion tree
qiime fragment-insertion sepp \
--i-representative-sequences ../dada2/rep_seqs_nomitochloro_gg2_filtered300.qza \
--i-reference-database 2022.10.backbone.sepp-reference.qza \
--o-tree tree_gg2.qza \
--o-placements tree_gg2_placements.qza \
--p-threads 24
```
## Run Slurm script
```
sbatch sepp_tree.sh
```
# Rarefaction
## Filter lab controls from samples
```
# Run in outer drought_soils directory
qiime feature-table filter-samples \
--i-table dada2/table_nomitochloro_gg2_filtered300.qza \
--m-metadata-file metadata/metadata.tsv \
--p-where "NOT [Treatment] IN ('Lab Control') " \
--o-filtered-table dada2/table_nomitochloro_nocontrol.qza
```
## Generate alpha rarefaction curve
```
cd alpha_rarefaction

qiime diversity alpha-rarefaction \
--i-table ../dada2/table_nomitochloro_nocontrol.qza \
--m-metadata-file ../metadata/metadata.tsv \
--p-max-depth 35000 \
--o-visualization alpha_rarefaction_curves.qzv
```
Looking at this alpha rarefaction curve, it appears that 14,000 reads is a decent rarefaction threshold to start at.
# Core metrics
## Core metrics Slurm script (--p-sampling-depth 14000)
```
#!/bin/bash
#SBATCH --job-name=core_metrics
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2024.10_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils

qiime diversity core-metrics-phylogenetic \
--i-table dada2/table_nomitochloro_nocontrol.qza \
--i-phylogeny tree/tree_gg2.qza \
--m-metadata-file metadata/metadata.tsv \
--p-sampling-depth 14000 \
--output-dir core_metrics_results

qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/observed_features_vector.qza \
--m-metadata-file metadata/metadata.tsv \
--o-visualization core_metrics_results/observed_features_statistics.qzv

qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/faith_pd_vector.qza \
--m-metadata-file metadata/metadata.tsv \
--o-visualization core_metrics_results/faiths_pd_statistics.qzv
```
## Run Slurm script
```
sbatch core_metrics.sh
```
# Differential abundance analysis
Note that ANCOM-BC2 is only available in versions of QIIME2 from 2026 onward.
Make a new directory called `ancombc2` in the `drought_soils` folder.
```
#!/bin/bash
#SBATCH --job-name=ancombc2
#SBATCH --nodes=1
#SBATCH --ntasks=8
#SBATCH --partition=amilan
#SBATCH --time=02:00:00
#SBATCH --mail-type=ALL
#SBATCH --output=slurm-%j.out
#SBATCH --qos=normal
#SBATCH --mail-user=sarah.spotten@colostate.edu

# Activate QIIME2
module purge
module load qiime2/2026.1_amplicon

# Change directory
cd /scratch/alpine/$USER/aneq505/drought_soils/ancombc2

# Filter to same depth as alpha rarefaction
qiime feature-table filter-samples \
--i-table ../dada2/table_nomitochloro_nocontrol.qza \
--p-min-frequency 14000 \
--o-filtered-table table_14k.qza

# Filter out low abundance and low prevalence ASVs
qiime feature-table filter-features \
--i-table table_14k.qza \
--p-min-frequency 50 \
--p-min-samples 20 \
--o-filtered-table table_14k_abund.qza

# Collapse features to species level
qiime taxa collapse \
--i-table table_14k_abund.qza \
--i-taxonomy ../taxonomy/taxonomy_gg2_filtered300.qza \
--p-level 7 \
--o-collapsed-table table_14k_abund_l7.qza

# Run ANCOM-BC2
qiime composition ancombc2 \
--i-table table_14k_abund_l7.qza \
--m-metadata-file ../metadata/metadata.tsv \
--p-fixed-effects-formula Treatment \
--o-ancombc2-output ancombc2_results_treatment_l7.qza

# Visualize ANCOM-BC2 results
qiime composition tabulate \
--i-data ancombc2_results_treatment_l7.qza \
--o-visualization ancombc2_results_treatment_l7.qzv
  
qiime composition ancombc2-visualizer \
  --i-data ancombc2_results_treatment_l7.qza \
  --o-visualization ancombc2_barplot_treatment_l7.qzv
```
## Run Slurm script
```
sbatch ancombc2.sh
```