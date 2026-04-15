```
sinteractive --time=01:00:00 --partition=amilan --nodes=1 --ntasks=2 --qos=normal
```

```
module purge
module load qiime2/2026.1_amplicon
```

import run2
```
qiime tools import \
--type "SampleData[PairedEndSequencesWithQuality]" \
--input-format PairedEndFastqManifestPhred33V2 \
--input-path manifest/manifest_run2.txt \
--output-path demux/demux_run2.qza
```

**Check Data Quality:**
```
cd demux
```

```
qiime demux summarize \
--i-data demux_run2.qza \
--o-visualization demux_run2.qzv
```