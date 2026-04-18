```
sinteractive --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

```
module purge
module load qiime2/2026.1_amplicon
```

### Demultiplexing

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
--output-path demux/demux.qza
```

Check Data Quality:
```
cd demux
```

```
qiime demux summarize \
--i-data demux.qza \
--o-visualization demux.qzv
```

### Denoising (2026 version)

```
cd ../dada2
```

```
#!/bin/bash
#SBATCH --job-name=denoise
#SBATCH --nodes=1
#SBATCH --ntasks=12
#SBATCH --partition=amilan
#SBATCH --time=3:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=chayanee.chanpanich@colostate.edu
#SBATCH --output=denoise_slurm-%j.out
#SBATCH --qos=normal

module purge
module load qiime2/2026.1_amplicon

#Path
cd /scratch/alpine/c837238655@colostate.edu/soil_project/dada2

# DENOISE (2026 version)
qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 12 \
--o-representative-sequences rep_seqs.qza \
--o-denoising-stats dada2_stats.qza \
--o-table table.qza \
--o-base-transition-stats base-transition-stats.qza

# visualization of each of these stats outputs:
qiime metadata tabulate \
--m-input-file dada2_stats.qza \
--o-visualization dada2_stats.qzv

# DADA2 TABLE FILE (2026 version):
qiime feature-table summarize \
  --i-table table.qza \
  --o-feature-frequencies feature-frequencies.qza \
  --o-sample-frequencies sample-frequencies.qza \
  --o-summary dada2_visual_summary.qzv

qiime feature-table tabulate-seqs \
--i-data rep_seqs.qza \
--o-visualization rep_seqs.qzv
```
