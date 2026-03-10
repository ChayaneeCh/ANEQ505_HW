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


```

### Alpha Rarefaction Plot ~={red}(1 point)=~
- Chose the input sequencings depths (min and max) for generating the alpha rarefaction plot: 

```
#go to the cow directory

qiime diversity alpha-rarefaction \
--i-table dada2/cow_table_dada2_filtered300.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization alpha_rarefaction_curves_16S.qzv \
--p-min-depth ADD MIN RAREFACTION DEPTH \
--p-max-depth ADD MAX RAREFACTION DEPTH
```


### Run Core Metrics ~={red}(1 point)=~

```
qiime diversity core-metrics-phylogenetic \
--i-table INSERT FILTERED TABLE HERE \
--i-phylogeny INSERT FILE HERE \
--m-metadata-file INSERT FILE HERE \
--p-sampling-depth INSERT SEQ DEPTH HERE \
--output-dir core_metrics_results
```


### Visualize alpha diversity plots
- generate a plot to visualize the observed features ~={red}(1 point)=~
```
qiime diversity alpha-group-significance \
--i-alpha-diversity core_metrics_results/FILENAME.qza \
--m-metadata-file metadata/cow_metadata.txt \
--o-visualization core_metrics_results/OUTPUT-FILENAME.qzv
```

- generate a plot to visualize faith's PD ~={red}(2 points)=~
```
## insert the entire code chunk for generating this visualization 


```



## Homework questions ~={red}(10 points)=~

1. what is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample)
2. what did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why? 
3. Which cow body location had more observed features? Which has the lowest?
4. What is the main difference between Faiths PD and Shannons alpha diversity metrics?  
5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information?
6. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different?
7. Does it seem like there are any groupings in the beta diversity? What are the groupings? 
8. Why do you think these samples are grouping together? 
9. What test can you run to determine if the groups are significantly different?
10. What command would you use to run that test?

```
#insert command for running the test you suggest from question 7



```