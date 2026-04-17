```
sinteractive --time=02:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
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

**Denoise**
```
cd ../dada2
```

```
#!/bin/bash
#SBATCH --job-name=5hr_denoise
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=5:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=denoise(5hr)_slurm-%j.out
#SBATCH --qos=normal

module purge
module load qiime2/2026.1_amplicon

#Path
cd /scratch/alpine/c837238655@colostate.edu/soil_project/dada2_5hr

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

**maybe change metadata file first!!!-Let's check**

DADA2 TABLE FILE:
```
qiime feature-table summarize \
--i-table table_run2.qza \
--o-visualization table.qzv \
--m-sample-metadata-file ../metadata/metadata.tsv
```

DADA2 SEQS FILE:
```
qiime feature-table tabulate-seqs \
--i-data seqs_dada2.qza \
--o-visualization seqs.qzv
```

### Taxonomy, Taxa Barplots, Filtering, Phylogenetic Tree

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

 make taxonomy into a visualization:
```
qiime metadata tabulate \  
--m-input-file taxonomy_gg2.qza \  
--o-visualization taxonomy_gg2.qzv
```

going to group by **type_days** (in a metadata column):
```
qiime feature-table group \  
--i-table ../dada2/table_run2.qza \  
--m-metadata-file ../metadata/metadata.txt \  
--m-metadata-column type_days(EDIT HERE!!!) \  
--p-mode mean-ceiling \  
--p-axis sample \  
--o-grouped-table ../dada2/table_type_days.qza
```