~={red}(1point)=~ for Alpha Rarefaction Plot
Run Core Metrics ~={red}(1 point; .25pts per line)=~
Make alpha diversity plots ~={red}(3points)=~
~={red}10 points=~ for the questions 

~={red}15 points total=~
------------------------------------------------------------------

Due: 

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice recording commands and editing code to match your analysis.
2. Perform alpha rarefaction and determine an appropriate sequencing depth.
3. Run core metrics, generate plots for alpha and beta diversity
--------------------------------------------------

**Cow Site Data Workflow**, part 3

Load qiime2 in a terminal session after you go into the **cow** folder

```
# Insert the two commands to activate qiime2
module purge
module load qiime2/2024.10_amplicon
```

### Alpha Rarefaction Plot ~={red}(1 point)=~
- Chose the input sequencings depths (min and max) for generating the alpha rarefaction plot: 

```
#go to the cow directory

qiime diversity alpha-rarefaction \
--i-table dada2/cow_table_dada2_filtered300.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization alpha_rarefaction_curves_16S.qzv \
--p-min-depth 3 \
--p-max-depth 11000
```


### Run Core Metrics ~={red}(1 point)=~

```
qiime diversity core-metrics-phylogenetic \
--i-table dada2/cow_table_dada2_filtered300.qza \
--i-phylogeny tree/tree_gg2.qza \
--m-metadata-file metadata/cow_metadata.txt \
--p-sampling-depth 1500 \
--output-dir core_metrics_results
```


### Visualize alpha diversity plots
- generate a plot to visualize the observed features ~={red}(1 point)=~
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/observed_features_vector.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/observed_features_statistics.qzv
```

- generate a plot to visualize faith's PD ~={red}(2 points)=~
```
## insert the entire code chunk for generating this visualization 
qiime diversity alpha-group-significance \  
--i-alpha-diversity core-metrics-results/faith_pd_vector.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--o-visualization core-metrics-results/faiths_pd_statistics.qzv
```



## Homework questions ~={red}(10 points)=~

1. what is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample)

	cow_table_dada2_filtered300.qzv

2. what did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why? 

	The rarefaction curves plateau around 1500–2000 sequencing depth, indicating that most diversity is captured by this point. Hence, a sampling depth of **1500** is appropriate for downstream core-metrics analysis.

3. Which cow body location had more observed features? Which has the lowest?

	**Fecal** samples show the highest number of observed features, while **nasal** samples have the lowest number when control samples are excluded.

4. What is the main difference between Faiths PD and Shannons alpha diversity metrics?  

	**Faith's PD** measure diversity using diversity using phylogenetic relationships between organisms. It calculates the total branch length of the phylogenetic tree represented in a sample, reflecting the evolutionary diversity of the community. On the other hand, **Shannon’s alpha diversity** measure diversity based on species richness and evenness. It considers both how many species are present and how evenly they are distributed in the sample. It does not use phylogenetic information.

5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information?

	The diversity metrics produced by the core-metrics pipeline that require phylogenetic information are **Faith’s Phylogenetic Diversity (Faith’s PD)**, **Unweighted UniFrac**, and **Weighted UniFrac**.

6. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different?

	The two body sites with the highest Faith’s PD alpha diversity are **fecal and skin**. According to the Kruskal–Wallis pairwise test, these groups are **significantly different**, as indicated by both the p-value (1.820117e-04) and the q-value (3.033528e-04) which are both less than 0.05.

7. Does it seem like there are any groupings in the beta diversity? What are the groupings? 

	Yes, there are some groupings in the beta diversity plots. Based on the unweighted and weighted unifrac, **fecal samples** form a distinct cluster and group tightly together, separate from the other body sites. **Skin and udder samples** appear close to each other. **Nasal and oral samples** also group together, but their clustering is less tight and more spread out compared to the other samples.

8. Why do you think these samples are grouping together? 

	These samples are grouping together because samples from the same or similar body sites tend to have similar microbial communities. Each body location provides a different environment which relate to oxygen, moisture, nutrients, and host contract. **Fecal samples** cluster together because they come from gut environment inside the body, while **skin and udder** group closely because they are both external body surfaces. **Nasal and oral samples** also group together because they are both associated with the upper respiratory and mouth regions.

9. What test can you run to determine if the groups are significantly different?

	The test that can be run to determine if the groups are significantly different is **PERMANOVA (Permutational Multivariate Analysis of Variance)**. It tests whether beta diversity differs significantly between groups based on a chosen sample grouping variable.

10. What command would you use to run that test?

	The commands below test whether microbial communities differ significantly between groups (body sites) using beta diversity distance matrices. The **beta-group-significance** method performs a statistical test (**PERMANOVA** by default) to determine whether community composition differs between groups defined in the metadata. Here, the metadata column **body_site** is used to test whether body site is associated with significant differences in **Unweighted UniFrac** and **Bray–Curtis** distances.

```
#insert command for running the test you suggest from question 7

# unweighted unifrac significance
qiime diversity beta-group-significance \
--i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
--m-metadata-file metadata/cow_metadata.txt \
--m-metadata-column body_site \
--o-visualization core-metrics-results/unweighted_unifrac_distance_matrix.qzv

# bray curtis significance  
qiime diversity beta-group-significance \  
--i-distance-matrix core-metrics-results/bray_curtis_distance_matrix.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--m-metadata-column body_site \  
--o-visualization core-metrics-results/bray_curtis_distance_matrix.qzv
```