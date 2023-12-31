Sugarcane Decomposition
================
Jessica Mulcrone
2023-09-04

## Task:

To determine if elevated-lipid sugarcane lines decompose differently
than wild type sugarcane. Results will provide insight into sugarcane
lipid decomposition and determine possible ecological and environmental
impacts of growing elevated-lipid sugarcane crops.

## Overview:

Energy crops such as sugarcane are engineered to contain higher lipid
contents (up to 3% lipid as opposed to .25%) in order to provide more
oil per plant. Plants with elevated lipid content may affect soil
organic matter (SOM) through a variety of mechanisms including
decomposition rate. It is possible that high-lipid sugarcane material
would decompose more slowly than wild-type due to the lower quality
lipid substrates.

Leaf and root litter bags from three sugarcane lines (one wild type
\[WT\], two elevated-lipid \[17T and 1566\]) were deployed at the
University of Illinois Energy Farm in Urbana, IL, USA and collected in
subsets at several intervals throughout the year. Mass lost from these
bags was measured, corrected for soil contamination, and combined with
carbon and nitrogen data from an elemental analyzer to determine
differences in mass lost and percent decomposition of nutrients across
lines.

## Data Sources:

Mock data that was used to check the R script is used in this analysis
for confidentiality reasons. A link to the paper with the real-world
data will be updated when forthcoming and published.

## Data Cleaning and Manipulation:

Libraries were imported and working directory was set.

``` r
library(dplyr)
library(ggplot2)
library(readxl)
library(ggpubr)
library(rstatix)
library(tidyverse)
library(knitr)

rm (list = ls())

setwd ("/Users/mulcron3/Desktop/sugarcane_data_8_25")
```

<style type="text/css">
pre {
  max-height: 300px;
  overflow-y: auto;
}
&#10;pre[class] {
  max-height: 100px;
}
</style>

Three types of data were imported: 1. Litter bag weights at each
retrieval. 2. EA data for percent carbon (C) and nitrogen (N) in each
sample at each retrieval as well as soil and stock litter. 3. Ashing
data describing percentage of sample at each retrieval that is litter
(organic matter) vs soil contamination, and the organic matter
percentage of each soil sample.

``` r
data.percentashsoil <- read_excel("sugarcane_ash_mock.xlsx", sheet="soil_final")
data.percentashleaf <- read_excel("sugarcane_ash_mock.xlsx", sheet="leaf_final")
data.percentashroot <- read_excel("sugarcane_ash_mock.xlsx", sheet="root_final")
data.leafweight <- read_excel("sugarcane_litterbag_weights_mock.xlsx", sheet="leaf_dry_weights")
data.rootweight <- read_excel("sugarcane_litterbag_weights_mock.xlsx", sheet="root_dry_weights")
data.leafcandn_2 <- read_excel("sugarcane_C_N_mock.xlsx", sheet="cn_leaves")
data.rootcandn_2 <- read_excel("sugarcane_C_N_mock.xlsx", sheet="cn_roots")
data.soil <- read_excel("sugarcane_C_N_mock.xlsx", sheet="cn_soil")
```

Soil ashing data reps were averaged then split into soil contaminating
root samples and soil contaminating leaf samples. Unnecessary columns
were deleted.

``` r
mean.soilash <- summarize(group_by(data.percentashsoil, Obs,
                                   litter_source, block, Sample_ID),
                          mean_soilash = mean (ash_mass_pct))

mean.soilashleaves <-filter(mean.soilash, grepl("leaf", Sample_ID))

mean.soilashroots <- filter(mean.soilash, grepl("root", Sample_ID))
```

Leaf and root ashing data reps were averaged then split between samples
and stock. Unnecessary columns were deleted and the stock mean column
was renamed to differentiate from sample mean columns.

``` r
mean.leafashraw <- summarise(group_by(data.percentashleaf, Obs, litter_source,
                                      litter_line, litter_date, block, litterbag_number),
                             mean_leafash = mean (ash_mass_pct),
                             n_leafash=n()) 

mean.stockleafashraw <- filter(mean.leafashraw, Obs == "Stock")

mean.stockleafash <- summarise (group_by(mean.stockleafashraw,
                                         litter_source, litter_line,
                                         mean_leafash))

names(mean.stockleafash)[names(mean.stockleafash) 
                         == 'mean_leafash'] <- 'mean_stockleafash'

mean.leafash <- filter (mean.leafashraw, Obs != "Stock")

mean.rootashraw <- summarise (group_by(data.percentashroot, Obs, litter_source,
                                       litter_line, litter_date, block,litterbag_number),
                              mean_rootash = mean (ash_mass_pct),
                              n_leafash=n())

mean.stockrootashraw <- filter (mean.rootashraw, Obs == "Stock")

mean.stockrootash <- summarise (group_by(mean.stockrootashraw,
                                         litter_source, litter_line,
                                         mean_rootash))

names(mean.stockrootash)[names(mean.stockrootash) 
                         == 'mean_rootash'] <- 'mean_stockrootash'

mean.rootash <- filter (mean.rootashraw, Obs != "Stock")
```

Columns for percent ash of samples’ corresponding contamination soil and
percent ash of the litter sample were added.

``` r
litterleaf.correspondingsoilash <- merge(mean.leafash, mean.soilashleaves, 
                                         by = c('litter_source', 'block'))

litterleaf.compiled <- merge(litterleaf.correspondingsoilash, 
                             mean.stockleafash, 
                             by = c('litter_source', 'litter_line'))

litterroot.correspondingsoilash <- merge(mean.rootash, mean.soilashroots, 
                                         by = c('litter_source', 'block'))

litterroot.compiled <- merge(litterroot.correspondingsoilash, 
                             mean.stockrootash, 
                             by = c('litter_source', 'litter_line'))
```

Percent carbon (C) and nitrogen (N) for soils, roots, and leaves were
calculated.

``` r
leaf_c_and_n_means <- summarize(group_by(data.leafcandn_2, Obs,litterbag_number,
                                         litter_source, block),
                                mean_c = mean (C),
                                mean_n = mean (N))

root_c_and_n_means <- summarize(group_by(data.rootcandn_2, Obs, litterbag_number,
                                         litter_source, block),
                                mean_c = mean (C),
                                mean_n = mean (N))

soil_c_and_n_means <- summarize(group_by(data.soil,
                                         litter_source, block, Sample_ID),
                                mean_c = mean (C),
                                mean_n = mean (N))
```

Leaf C and N data was split by chamber and field.

``` r
c_and_n_means_chamber01 <- filter (leaf_c_and_n_means, 
                                   litter_source == "cham_01")

c_and_n_means_field01 <- filter (leaf_c_and_n_means, 
                                 litter_source == "field_01")

soil_c_and_n_means_field01 <- filter (soil_c_and_n_means, 
                                      litter_source == "field_01")


soil_c_and_n_means_chamber01 <- filter (soil_c_and_n_means, 
                                        litter_source == "cham_01")
```

Soil C and N data was split by leaves and roots.

``` r
soil_c_and_n_chamber01_leaves <- filter (soil_c_and_n_means_chamber01, 
                                         Sample_ID < 11)


soil_c_and_n_chamber01_roots <- filter (soil_c_and_n_means_chamber01, 
                                        Sample_ID > 10)
```

C and N soil data was merged with the C and N litter data, and columns
were renamed to be unique for combination with litter data.

``` r
chamber01_c_and_n_leaf_compiled <- merge (c_and_n_means_chamber01,
                                          soil_c_and_n_chamber01_leaves,
                                          by = c('block', 'litter_source'))

field01_c_and_n_leaf_compiled <- merge (c_and_n_means_field01,
                                        soil_c_and_n_means_field01,
                                        by = c('block', 'litter_source'))

root_c_and_n_compiled <- merge (root_c_and_n_means, soil_c_and_n_chamber01_roots,
                                by = c('block', 'litter_source'))


colnames(chamber01_c_and_n_leaf_compiled)[4]  <- "litterbag_number"
colnames(chamber01_c_and_n_leaf_compiled)[5]  <- "mean_c_leaf"
colnames(chamber01_c_and_n_leaf_compiled)[6]  <- "mean_n_leaf"
colnames(chamber01_c_and_n_leaf_compiled)[7]  <- "soil_ID"
colnames(chamber01_c_and_n_leaf_compiled)[8]  <- "mean_c_soil"
colnames(chamber01_c_and_n_leaf_compiled)[9]  <- "mean_n_soil"

colnames(field01_c_and_n_leaf_compiled)[4]  <- "litterbag_number"
colnames(field01_c_and_n_leaf_compiled)[5]  <- "mean_c_leaf"
colnames(field01_c_and_n_leaf_compiled)[6]  <- "mean_n_leaf"
colnames(field01_c_and_n_leaf_compiled)[7]  <- "soil_ID"
colnames(field01_c_and_n_leaf_compiled)[8]  <- "mean_c_soil"
colnames(field01_c_and_n_leaf_compiled)[9]  <- "mean_n_soil"

colnames(root_c_and_n_compiled)[4]  <- "litterbag_number"
colnames(root_c_and_n_compiled)[5]  <- "mean_c_root"
colnames(root_c_and_n_compiled)[6]  <- "mean_n_root"
colnames(root_c_and_n_compiled)[7]  <- "soil_ID"
colnames(root_c_and_n_compiled)[8]  <- "mean_c_soil"
colnames(root_c_and_n_compiled)[9]  <- "mean_n_soil"
```

## Key Questions:

1)  Do elevated-lipid sugarcane leaf samples decompose more slowly than
    wild type?
2)  Do elevated-lipid sugarcane root samples decompose more slowly than
    wild type?
3)  Does the carbon in elevated-lipid sugarcane leaf or root samples
    decompose differently than wild type?
4)  Does the nitrogen in elevated-lipid sugarcane leaf samples decompose
    differently than wild type?
5)  Are there concerning outliers in the data?

## Analysis, Key Results, and Visualizations:

Percent actually litter (as opposed to soil) was calculated using
Equation from literature for leaves and roots (Blair et al. 1988 sBB,
<DOI:10.1016/0038-0717(88)90154-X>).

``` r
litterleaf.compiled <- litterleaf.compiled %>% 
  mutate(percent_actually_litterleaf
         =(((100-mean_leafash) - (100-mean_soilash))
           / ((100-mean_stockleafash) - (100-mean_soilash))
           *100))

litterroot.compiled<- litterroot.compiled %>% 
  mutate(percent_actually_litterroot
         =(((100-mean_rootash) - (100-mean_soilash))
           / ((100-mean_stockrootash) - (100-mean_soilash))
           *100))
```

Percent of sample decomposed was calculated for leaves and roots.

``` r
# merge weight data from retrieved sample bags with the percent actually litter data

litterleaf.compiled <-merge(data.leafweight, select(litterleaf.compiled, -c(Obs.x, Obs.y, litter_date)),
                            by = c("litterbag_number", "litter_source",
                                   "litter_line", "block"))

litterroot.compiled <-merge(data.rootweight, select(litterroot.compiled, -c(Obs.x, Obs.y, litter_date)),
                            by = c("litterbag_number", "litter_source",
                                   "litter_line", "block"))

# multiply percent actually litter by weights of retrieved sample bags and subtract results from initial weight

litterleaf.compiled <- litterleaf.compiled %>% 
  mutate(post_weight_thats_actually_litter
         = (post_litter_wt_g * percent_actually_litterleaf/100))

litterleaf.compiled <- litterleaf.compiled %>% 
  mutate(pre_minus_adj_post
         = (pre_litter_wt_g - post_weight_thats_actually_litter))

litterroot.compiled <- litterroot.compiled %>% 
  mutate(post_weight_thats_actually_litter
         = (post_litterbag_wt_g * percent_actually_litterroot/100))

litterroot.compiled <- litterroot.compiled %>% 
  mutate(pre_minus_adj_post
         = (pre_litter_wt_g - post_weight_thats_actually_litter))

# divide adjusted post weight by initial weight

litterleaf.compiled <- litterleaf.compiled %>% 
  mutate(pct_mass_lost_leaf
         = (pre_minus_adj_post / pre_litter_wt_g))

litterroot.compiled <- litterroot.compiled %>% 
  mutate(pct_mass_lost_root
         = (pre_minus_adj_post / pre_litter_wt_g))
```

Leaf data was split between chamber grown and field grown.

``` r
litterleafcham01.compiled <- filter (litterleaf.compiled, 
                                     litter_source == "cham_01")

litterleaffield01.compiled <- filter (litterleaf.compiled, 
                                      litter_source == "field_01")
```

## Figure 1. Percent Mass Lost Over Time

<img src="sugarcane-markdown_files/figure-gfm/plot perc mass lost over time-1.png" width="33%" /><img src="sugarcane-markdown_files/figure-gfm/plot perc mass lost over time-2.png" width="33%" /><img src="sugarcane-markdown_files/figure-gfm/plot perc mass lost over time-3.png" width="33%" />

*Do elevated-lipid sugarcane leaf samples decompose more slowly than
wild type?* *Do elevated-lipid sugarcane root samples decompose more
slowly than wild type?*

**Elevated-lipid sugarcane leaves decompose faster than wild type.**
Both the 17T and 1566 lines had a higher percent mass lost at each
timepoint. **Elevated-lipid sugarcane roots sometimes composed faster
than wild type and sometimes slower.** The 1566 line had higher percent
mass lost than WT at all time points while the 17T line had lower
percent mass lost at all time points.

Data for percent actually litter was combined with the C and N data

``` r
#trim down the percentage actual leaf data in prep to combine with C N data 

litterleafcham01.compiled_2 <- litterleafcham01.compiled %>% select(-c(litter_source, Obs,
                                                                       litter_date, zip_color, block,
                                                                       retrieve_date, rand_num, Jack_post_litter_rewt_g,
                                                                       note, Sample_ID))


litterleaffield01.compiled_2 <- litterleaffield01.compiled %>% select(-c(litter_source, Obs,
                                                                         litter_date, zip_color, block,
                                                                         retrieve_date, rand_num, Jack_post_litter_rewt_g,
                                                                         note, Sample_ID))

litterroot.compiled_2 <- litterroot.compiled %>% select(-c(litter_source, Obs,
                                                           litter_date, zip_color, block,
                                                           retrieve_date, rand_num,
                                                           note, Sample_ID))

#combine the trimmed percent actual leaf data and C N data

chamber01_c_and_n_leaf_compiled_2 <- merge (chamber01_c_and_n_leaf_compiled, 
                                            litterleafcham01.compiled_2, 
                                            by = c('litterbag_number'))

field01_c_and_n_leaf_compiled_2 <- merge (field01_c_and_n_leaf_compiled, 
                                          litterleaffield01.compiled_2, 
                                          by = c('litterbag_number'))

root_c_and_n_compiled_2 <- merge (root_c_and_n_compiled, 
                                  litterroot.compiled_2, 
                                  by = c('litterbag_number'))
```

Percent C and N left in each retrieved litter bag was calculated.

``` r
chamber01_c_and_n_leaf_compiled_3 <- chamber01_c_and_n_leaf_compiled_2 %>% 
  mutate(percent_c_leaf = (((mean_c_leaf/100) -
                              ((1 - (percent_actually_litterleaf/100)) * (mean_c_soil/100)))
                           / (percent_actually_litterleaf/100)))

field01_c_and_n_leaf_compiled_3 <- field01_c_and_n_leaf_compiled_2 %>% 
  mutate(percent_c_leaf = (((mean_c_leaf/100) -
                              ((1 - (percent_actually_litterleaf/100)) * (mean_c_soil/100)))
                           / (percent_actually_litterleaf/100)))

root_c_and_n_compiled_3 <- root_c_and_n_compiled_2 %>% 
  mutate(percent_c_root = (((mean_c_root/100) -
                              ((1 - (percent_actually_litterroot/100)) * (mean_c_soil/100)))
                           / (percent_actually_litterroot/100)))

chamber01_c_and_n_leaf_compiled_4 <- chamber01_c_and_n_leaf_compiled_3 %>% 
  mutate(percent_n_leaf = (((mean_n_leaf/100) -
                              ((1 - (percent_actually_litterleaf/100)) * (mean_n_soil/100)))
                           / (percent_actually_litterleaf/100)))


field01_c_and_n_leaf_compiled_4 <- field01_c_and_n_leaf_compiled_3 %>% 
  mutate(percent_n_leaf = (((mean_n_leaf/100) -
                              ((1 - (percent_actually_litterleaf/100)) * (mean_n_soil/100)))
                           / (percent_actually_litterleaf/100)))


root_c_and_n_compiled_4 <- root_c_and_n_compiled_3 %>% 
  mutate(percent_n_root = (((mean_n_root/100) -
                              ((1 - (percent_actually_litterroot/100)) * (mean_n_soil/100)))
                           / (percent_actually_litterroot/100)))
```

Mass of C and N in each retrieved litter bag was calculated.

``` r
chamber01_c_and_n_leaf_compiled_5 <- chamber01_c_and_n_leaf_compiled_4 %>% 
  mutate(mass_c_leaf= (percent_c_leaf * (percent_actually_litterleaf/100)
                       * post_litter_wt_g ))

field01_c_and_n_leaf_compiled_5 <- field01_c_and_n_leaf_compiled_4 %>% 
  mutate(mass_c_leaf= (percent_c_leaf * (percent_actually_litterleaf/100)
                       * post_litter_wt_g ))

root_c_and_n_compiled_5 <- root_c_and_n_compiled_4 %>% 
  mutate(mass_c_root= (percent_c_root * (percent_actually_litterroot/100)
                       * post_litterbag_wt_g ))

chamber01_c_and_n_leaf_compiled_6 <- chamber01_c_and_n_leaf_compiled_5 %>% 
  mutate(mass_n_leaf= (percent_n_leaf * (percent_actually_litterleaf/100)
                       * post_litter_wt_g ))

field01_c_and_n_leaf_compiled_6 <- field01_c_and_n_leaf_compiled_5 %>% 
  mutate(mass_n_leaf= (percent_n_leaf * (percent_actually_litterleaf/100)
                       * post_litter_wt_g ))

root_c_and_n_compiled_6 <- root_c_and_n_compiled_5 %>% 
  mutate(mass_n_root= (percent_n_root * (percent_actually_litterroot/100)
                       * post_litterbag_wt_g ))
```

C and N data was filtered for time point zero, labeled as such, and
merged back with corresponding retrieved litterbags.

``` r
#--Filter C and N percent for time zero

leaf_cham01_initial_c_and_n <- filter (chamber01_c_and_n_leaf_compiled_6, 
                                       retrieve_time == 0)

#--Trim data for retrieve time 0

leaf_cham01_initial_c_and_n_2 <- leaf_cham01_initial_c_and_n %>% 
  select(litter_line, block, percent_c_leaf,
         percent_n_leaf, mass_c_leaf, mass_n_leaf) 

#--Rename coloumns for retrieve time 0 to prepare to merge 

colnames(leaf_cham01_initial_c_and_n_2)[3]  <- "per_c_initial"
colnames(leaf_cham01_initial_c_and_n_2)[4]  <- "per_n_initial"
colnames(leaf_cham01_initial_c_and_n_2)[5]  <- "mass_c_int"
colnames(leaf_cham01_initial_c_and_n_2)[6]  <- "mass_n_int"

#--Filter C and N data for all other time points

leaf_cham01_c_and_n <- filter (chamber01_c_and_n_leaf_compiled_6, 
                               retrieve_time != 0)

#--Merge renamed time 0 and all other time points 

leaf_cham01_c_and_n_2 <- merge (leaf_cham01_c_and_n,leaf_cham01_initial_c_and_n_2,
                                by = c('litter_line', 'block'))

leaf_field01_initial_c_and_n <- filter (field01_c_and_n_leaf_compiled_6, 
                                        retrieve_time == 0)

leaf_field01_initial_c_and_n_2 <- leaf_field01_initial_c_and_n %>% 
  select(litter_line, block, percent_c_leaf,
         percent_n_leaf, mass_c_leaf, mass_n_leaf) 

colnames(leaf_field01_initial_c_and_n_2)[3]  <- "per_c_initial"
colnames(leaf_field01_initial_c_and_n_2)[4]  <- "per_n_initial"
colnames(leaf_field01_initial_c_and_n_2)[5]  <- "mass_c_int"
colnames(leaf_field01_initial_c_and_n_2)[6]  <- "mass_n_int"


leaf_field01_c_and_n <- filter (field01_c_and_n_leaf_compiled_6, 
                                retrieve_time != 0)

leaf_field01_c_and_n_2 <- merge (leaf_field01_c_and_n,leaf_field01_initial_c_and_n_2,
                                 by = c('litter_line', 'block'))

root_initial_c_and_n <- filter (root_c_and_n_compiled_6, 
                                retrieve_time == 0)  

root_initial_c_and_n_2 <- root_initial_c_and_n %>% 
  select(litter_line, block, percent_c_root,
         percent_n_root, mass_c_root, mass_n_root ) 

colnames(root_initial_c_and_n_2)[3]  <- "per_c_initial"
colnames(root_initial_c_and_n_2)[4]  <- "per_n_initial"
colnames(root_initial_c_and_n_2)[5]  <- "mass_c_int"
colnames(root_initial_c_and_n_2)[6]  <- "mass_n_int"


root_c_and_n <- filter (root_c_and_n_compiled_6, 
                        retrieve_time != 0)

root_c_and_n_2 <- merge (root_c_and_n,root_initial_c_and_n_2,
                         by = c('litter_line', 'block'))
```

Relative percent was calculated.

``` r
leaf_cham01_c_and_n_3 <- leaf_cham01_c_and_n_2 %>% 
  mutate (per_c_rel = (mass_c_leaf / mass_c_int))

leaf_cham01_c_and_n_4 <- leaf_cham01_c_and_n_3 %>% 
  mutate (per_n_rel = (mass_n_leaf / mass_n_int))

leaf_field01_c_and_n_3 <- leaf_field01_c_and_n_2 %>% 
  mutate (per_c_rel = (mass_c_leaf / mass_c_int))

leaf_field01_c_and_n_4 <- leaf_field01_c_and_n_3 %>% 
  mutate (per_n_rel = (mass_n_leaf / mass_n_int))

root_c_and_n_3 <- root_c_and_n_2 %>% 
  mutate (per_c_rel = (mass_c_root / mass_c_int))

root_c_and_n_4 <- root_c_and_n_3 %>% 
  mutate (per_n_rel = (mass_n_root / mass_n_int))
```

## Figure 2. Relative Percent Carbon and Nitrogen Remaining

<img src="sugarcane-markdown_files/figure-gfm/plot mass c lost scatter-1.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot mass c lost scatter-2.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot mass c lost scatter-3.png" width="30%" />

<img src="sugarcane-markdown_files/figure-gfm/plot mass n lost scatter-1.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot mass n lost scatter-2.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot mass n lost scatter-3.png" width="30%" />

*Does the carbon in elevated-lipid sugarcane leaf or root samples
decompose differently than wild type? Does the nitrogen in
elevated-lipid sugarcane leaf samples decompose differently than wild
type?*

**The carbon and nitrogen in elevated-lipid sugarcane leaf and root
samples decompose more quickly than the carbon and nitrogen in wild type
litter.** Patterns for graphs are identical because of the simplicity of
mock data set to confirm different elements of the script.

## Figure 3. Relative Percent Carbon and Nitrogen Remaining

<img src="sugarcane-markdown_files/figure-gfm/plot rel carbon-1.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot rel carbon-2.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot rel carbon-3.png" width="30%" />

<img src="sugarcane-markdown_files/figure-gfm/plot rel nitrogen-1.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot rel nitrogen-2.png" width="30%" /><img src="sugarcane-markdown_files/figure-gfm/plot rel nitrogen-3.png" width="30%" />

*Are there concerning outliers in the data?*

**There are no concerning outliers in the data.** The box plots results
are lines rather than boxes because mock data percent mass lost is the
same for each litter line and type combination at each retrieval time.

## Conclusions and Recomendations:

Results show that in leaves, wild type sugarcane litter decomposes more
slowly than elevated-lipid litter. This was the opposite of the
experiment’s hypothesis. In roots, wild type sugarcane litter decomposed
more quickly than one of the elevated-lipid lines and more slowly than
the other. If the lower quality lipid substrate is slowing down
decomposition, other factors may be counteracting the effect as
decomposition involves many interconnected steps.

Carbon and nitrogen in both leaves and roots decomposed more quickly in
elevated-lipid lines than wild type. These results support the
experiment’s hypothesis, but do not immediately explain the results of
overall faster decomposition in wild type litter. These results could
point to litter elements other than carbon or nitrogen making up the
bulk of decomposition (although carbon decomposing in an opposite
pattern to overall litter weight is unlikely to occur in real-world
data). When analyzing real-world data, the carbon and nitrogen relative
percentages may indicate which steps of the decomposition process are
driving results.

The data showed no concerning outliers and produced the expected
results, indicating that the script is performing the desired
calculations.

### Supplemental Material

Calculations Flowchart

<img src="flowchart.png" width="100%" />
