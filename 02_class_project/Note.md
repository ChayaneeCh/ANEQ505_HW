```
sinteractive --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

```
module purge
module load qiime2/2026.1_amplicon
```

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
qiime dada2 denoise-paired \
--i-demultiplexed-seqs ../demux/demux_run2.qza \
--p-trim-left-f 0 \
--p-trim-left-r 0 \
--p-trunc-len-f 250 \
--p-trunc-len-r 250 \
--p-n-threads 6 \
--o-representative-sequences seqs_dada2.qza \
--o-denoising-stats dada2_stats_run2.qza \
--o-table table_run2.qza \
--o-base-transition-stats base-transition-stats.qza
```

visualization of each of these stats outputs

```
qiime metadata tabulate \
--m-input-file dada2_stats_run2.qza \
--o-visualization dada2_stats_run2.qzv
```