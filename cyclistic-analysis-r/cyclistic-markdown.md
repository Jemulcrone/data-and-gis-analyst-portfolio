Cyclistic Case Study
================
Jessica Mulcrone
2023-09-03

## Task:

The purpose of this analysis is to determine how annual and casual
riders use Cyclistic Bikes differently in order to lay the groundwork
for a marketing plan to convert more casual riders into annual riders.

## Overview:

Cyclistic Bikes is a bike-share company in Chicago with over 5,800
bicycles and 600 docking stations. In addition to standard bikes,
Cyclistic offers assistive options which 8% of their riders utilize.
Most customers ride for pleasure but 30% of the user base rent bikes to
commute. Cyclistic Bikes has historically focused on appealing to new
customers with the flexibility of three pricing plans: single-ride,
full-day, and membership, but are now interested in converting existing
casual riders into members.

The Cyclistic data analytics team will investigate how annual and casual
riders use Cyclistic differently, why casual riders might buy an annual
membership, and how social media can influence casual riders to become
members. This report addresses the first of these analysis goals: how do
annual and casual riders use Cyclist Bikes differently?

## Data Sources:

Trip data was made available to the public from Motivate International
Inc. and was downloaded from
<https://divvy-tripdata.s3.amazonaws.com/index.html>. Records from
August 2022 through July 2023 were used to generate the most current
analysis.

## Data Cleaning and Manipulation:

Libraries were imported and working directory was set

``` r
library(dplyr)
library(ggplot2)
library(readxl)
library(ggpubr)
library(rstatix)
library(tidyverse)
library(knitr)

rm (list = ls())

setwd ("/Users/mulcron3/Desktop/cyclist_data")
```

Column headers on all sheets were confirmed to match before sheets were
imported into R and combined.

``` r
data.1 <- read_csv("202207-divvy-tripdata.csv")
data.2 <- read_csv("202208-divvy-tripdata.csv")
data.3 <- read_csv("202209-divvy-publictripdata.csv")
data.4 <- read_csv("202210-divvy-tripdata.csv")
data.5 <- read_csv("202211-divvy-tripdata.csv")
data.6 <- read_csv("202212-divvy-tripdata.csv")
data.7 <- read_csv("202301-divvy-tripdata.csv")
data.8 <- read_csv("202302-divvy-tripdata.csv")
data.9 <- read_csv("202303-divvy-tripdata.csv")
data.10 <- read_csv("202304-divvy-tripdata.csv")
data.11 <- read_csv("202305-divvy-tripdata.csv")
data.12 <- read_csv("202306-divvy-tripdata.csv")

cyclist_compiled <- bind_rows(data.1, data.2)
cyclist_compiled_2 <- bind_rows(data.3, data.4)
cyclist_compiled_3 <- bind_rows(data.5, data.6)
cyclist_compiled_4 <- bind_rows(data.7, data.8)
cyclist_compiled_5 <- bind_rows(data.9, data.10)
cyclist_compiled_6 <- bind_rows(data.11, data.12)
cyclist_compiled_7 <- bind_rows(cyclist_compiled, cyclist_compiled_2)
cyclist_compiled_8 <- bind_rows(cyclist_compiled_3, cyclist_compiled_4)
cyclist_compiled_9 <- bind_rows(cyclist_compiled_5, cyclist_compiled_6)
cyclist_compiled_10 <- bind_rows(cyclist_compiled_7, cyclist_compiled_8)
cyclist_compiled_final <- bind_rows(cyclist_compiled_10, cyclist_compiled_9)
```

Date-time columns were separated while ensuring consistent formatting.

``` r
cyclist_compiled_final$start_date <- as.Date(cyclist_compiled_final$started_at)
cyclist_compiled_final$start_time <- format(cyclist_compiled_final$started_at,"%H:%M:%S")
cyclist_compiled_final$end_date <- as.Date(cyclist_compiled_final$ended_at)
cyclist_compiled_final$end_time <- format(cyclist_compiled_final$ended_at,"%H:%M:%S")
```

The data frame was trimmed and rider ID’s were checked for duplicates
(none).

``` r
cyclist_compiled_final %>% 
  mutate(across(where(is.character), str_trim))
```

``` r
cyclist_compiled_final[!duplicated(cyclist_compiled_final[c('ride_id')]), ]
```

A column start day-of-week was calculated by extracting day-of-week from
start date.

``` r
cyclist_compiled_final$start_day_of_week<-weekdays(as.Date(cyclist_compiled_final$start_date,'%Y-%m-%d'))
cyclist_compiled_final$end_day_of_week<-weekdays(as.Date(cyclist_compiled_final$end_date,'%Y-%m-%d'))
```

A column for ride length was calculated by subtracting start time from
finish time

``` r
cyclist_compiled_final$ride_length<-cyclist_compiled_final$ended_at-cyclist_compiled_final$started_at
```

A column was created for month of start date

``` r
cyclist_compiled_final$start_month <- format(as.Date(cyclist_compiled_final$start_date, format="%Y/%m/%d"),"%m")
```

Start day-of-week and ride length columns were converted to numbers for
calculation and visualization purposes.

``` r
cyclist_compiled_final$ride_length <- as.numeric(cyclist_compiled_final$ride_length)
cyclist_compiled_final$start_month <- as.numeric(cyclist_compiled_final$start_month)
```

Rides with lengths of over one day were investigated and filtered out of
the data set as likely either data or user errors\*.

``` r
rides_over_one_day <-filter(cyclist_compiled_final, ride_length >= 86400)
cyclist_compiled_final <- filter(cyclist_compiled_final, ride_length < 86400)
```

\*When large ride lengths were not filtered, monthly max ride lengths
hover around a day for members and 10-20 days for casual riders.
Excessive max ride length may be a data mistake or riders not returning
bikes to stations. Rides over one day made up 0.09% of trips and were
much more common among casual riders. They were removed to avoid very
long trips artificially inflating average casual ride length.

## Key Questions:

1)  Do casual riders and members differ on which days of the week they
    are most likely to use Cylcistic?
2)  Do casual riders and members differ on ride length and is the
    pattern dependent on day of week?
3)  Does daily popularity change month-to-month for casual users or
    members?
4)  Do patterns of ride length change month-to-month for casual users or
    members?
5)  Are casual riders and members most likely to begin a trip from
    different places?  
6)  Do casual riders and members prefer different bike types?

## Analysis, Key Results, and Visualizations:

Calculate percentage of each Cyclstic users type overall and by season

``` r
nrow(filter(cyclist_compiled_final, member_casual == "member"))
nrow(cyclist_compiled_final)

nrow(filter(cyclist_compiled_final, member_casual == "member" & (start_month == 1 | start_month == 2 | start_month == 3)))
nrow(filter(cyclist_compiled_final, start_month == 1 | start_month == 2 | start_month == 3))

nrow(filter(cyclist_compiled_final, member_casual == "member" & (start_month == 4 | start_month == 5 | start_month == 6)))
nrow(filter(cyclist_compiled_final, start_month == 4 | start_month == 5 | start_month == 6))

nrow(filter(cyclist_compiled_final, member_casual == "member" & (start_month == 7 | start_month == 8 | start_month == 9)))
nrow(filter(cyclist_compiled_final, start_month == 7 | start_month == 8 | start_month == 9))

nrow(filter(cyclist_compiled_final, member_casual == "member" & (start_month == 10 | start_month ==11 | start_month == 12)))
nrow(filter(cyclist_compiled_final, start_month == 10 | start_month == 11 | start_month == 12))
```

Calculate mean ride length by overall, month, and season for all riders,
causal riders, and members

``` r
ride_length_mean <- mean(cyclist_compiled_final$ride_length)
ride_length_mean_jan <- mean(subset(cyclist_compiled_final, start_month == 1)$ride_length)
ride_length_mean_feb <- mean(subset(cyclist_compiled_final, start_month == 2)$ride_length)
ride_length_mean_march <- mean(subset(cyclist_compiled_final, start_month == 3)$ride_length)
ride_length_mean_april <- mean(subset(cyclist_compiled_final, start_month == 4)$ride_length)
ride_length_mean_may <- mean(subset(cyclist_compiled_final, start_month == 5)$ride_length)
ride_length_mean_june <- mean(subset(cyclist_compiled_final, start_month == 6)$ride_length)
ride_length_mean_july <- mean(subset(cyclist_compiled_final, start_month == 7)$ride_length)
ride_length_mean_aug <- mean(subset(cyclist_compiled_final, start_month == 8)$ride_length)
ride_length_mean_sept <- mean(subset(cyclist_compiled_final, start_month == 9)$ride_length)
ride_length_mean_oct <- mean(subset(cyclist_compiled_final, start_month == 10)$ride_length)
ride_length_mean_nov <- mean(subset(cyclist_compiled_final, start_month == 11)$ride_length)
ride_length_mean_dec <- mean(subset(cyclist_compiled_final, start_month == 12)$ride_length)
ride_length_mean_winter <- mean(subset(cyclist_compiled_final, start_month == 1 | start_month == 2 | start_month == 3)$ride_length)
ride_length_mean_spring <- mean(subset(cyclist_compiled_final, start_month == 4 | start_month == 5 | start_month == 6)$ride_length)
ride_length_mean_summer <- mean(subset(cyclist_compiled_final, start_month == 7 | start_month == 8 | start_month == 9)$ride_length)
ride_length_mean_fall <- mean(subset(cyclist_compiled_final, start_month == 10 | start_month == 11 | start_month == 12)$ride_length)

ride_length_mean_cas <- mean(subset(cyclist_compiled_final, member_casual == 'casual')$ride_length)
ride_length_mean_jan_cas <- mean(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'casual')$ride_length)
ride_length_mean_feb_cas <- mean(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'casual')$ride_length)
ride_length_mean_march_cas <- mean(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'casual')$ride_length)
ride_length_mean_april_cas <- mean(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'casual')$ride_length)
ride_length_mean_may_cas <- mean(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'casual')$ride_length)
ride_length_mean_june_cas <- mean(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'casual')$ride_length)
ride_length_mean_july_cas <- mean(subset(cyclist_compiled_final, start_month == 7& member_casual == 'casual')$ride_length)
ride_length_mean_aug_cas <- mean(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'casual')$ride_length)
ride_length_mean_sept_cas <- mean(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'casual')$ride_length)
ride_length_mean_oct_cas <- mean(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'casual')$ride_length)
ride_length_mean_nov_cas <- mean(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'casual')$ride_length)
ride_length_mean_dec_cas <- mean(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'casual')$ride_length)
ride_length_mean_winter_cas <- mean(subset(cyclist_compiled_final, member_casual == 'casual'& (start_month == 1 | start_month == 2 | start_month == 3))$ride_length)
ride_length_mean_spring_cas <- mean(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 4 | start_month == 5 | start_month == 6))$ride_length)
ride_length_mean_summer_cas <- mean(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 7 | start_month == 8 | start_month == 9))$ride_length)
ride_length_mean_fall_cas <- mean(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 10 | start_month == 11 | start_month == 12))$ride_length)

ride_length_mean_mem <- mean(subset(cyclist_compiled_final, member_casual == 'member')$ride_length)
ride_length_mean_jan_mem <- mean(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'member')$ride_length)
ride_length_mean_feb_mem <- mean(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'member')$ride_length)
ride_length_mean_march_mem <- mean(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'member')$ride_length)
ride_length_mean_april_mem <- mean(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'member')$ride_length)
ride_length_mean_may_mem <- mean(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'member')$ride_length)
ride_length_mean_june_mem <- mean(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'member')$ride_length)
ride_length_mean_july_mem <- mean(subset(cyclist_compiled_final, start_month == 7 & member_casual == 'member')$ride_length)
ride_length_mean_aug_mem <- mean(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'member')$ride_length)
ride_length_mean_sept_mem <- mean(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'member')$ride_length)
ride_length_mean_oct_mem <- mean(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'member')$ride_length)
ride_length_mean_nov_mem <- mean(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'member')$ride_length)
ride_length_mean_dec_mem <- mean(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'member')$ride_length)
ride_length_mean_winter_mem <- mean(subset(cyclist_compiled_final, member_casual == 'member'& (start_month == 1 | start_month == 2 | start_month == 3))$ride_length)
ride_length_mean_spring_mem <- mean(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 4 | start_month == 5 | start_month == 6))$ride_length)
ride_length_mean_summer_mem <- mean(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 7 | start_month == 8 | start_month == 9))$ride_length)
ride_length_mean_fall_mem <- mean(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 10 | start_month == 11 | start_month == 12))$ride_length)
```

Calculate max ride length by overall, month, and season for all riders,
causal riders, and members

``` r
ride_length_max <- max(cyclist_compiled_final$ride_length)
ride_length_max_jan <- max(subset(cyclist_compiled_final, start_month == 1)$ride_length)
ride_length_max_feb <- max(subset(cyclist_compiled_final, start_month == 2)$ride_length)
ride_length_max_march <- max(subset(cyclist_compiled_final, start_month == 3)$ride_length)
ride_length_max_april <- max(subset(cyclist_compiled_final, start_month == 4)$ride_length)
ride_length_max_may <- max(subset(cyclist_compiled_final, start_month == 5)$ride_length)
ride_length_max_june <- max(subset(cyclist_compiled_final, start_month == 6)$ride_length)
ride_length_max_july <- max(subset(cyclist_compiled_final, start_month == 7)$ride_length)
ride_length_max_aug <- max(subset(cyclist_compiled_final, start_month == 8)$ride_length)
ride_length_max_sept <- max(subset(cyclist_compiled_final, start_month == 9)$ride_length)
ride_length_max_oct <- max(subset(cyclist_compiled_final, start_month == 10)$ride_length)
ride_length_max_nov <- max(subset(cyclist_compiled_final, start_month == 11)$ride_length)
ride_length_max_dec <- max(subset(cyclist_compiled_final, start_month == 12)$ride_length)
ride_length_max_winter <- max(subset(cyclist_compiled_final, start_month == 1 | start_month == 2 | start_month == 3)$ride_length)
ride_length_max_spring <- max(subset(cyclist_compiled_final, start_month == 4 | start_month == 5 | start_month == 6)$ride_length)
ride_length_max_summer <- max(subset(cyclist_compiled_final, start_month == 7 | start_month == 8 | start_month == 9)$ride_length)
ride_length_max_fall <- max(subset(cyclist_compiled_final, start_month == 10 | start_month == 11 | start_month == 12)$ride_length)

ride_length_max_cas <- max(subset(cyclist_compiled_final, member_casual == 'casual')$ride_length)
ride_length_max_jan_cas <- max(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'casual')$ride_length)
ride_length_max_feb_cas <- max(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'casual')$ride_length)
ride_length_max_march_cas <- max(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'casual')$ride_length)
ride_length_max_april_cas <- max(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'casual')$ride_length)
ride_length_max_may_cas <- max(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'casual')$ride_length)
ride_length_max_june_cas <- max(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'casual')$ride_length)
ride_length_max_july_cas <- max(subset(cyclist_compiled_final, start_month == 7 & member_casual == 'casual')$ride_length)
ride_length_max_aug_cas <- max(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'casual')$ride_length)
ride_length_max_sept_cas <- max(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'casual')$ride_length)
ride_length_max_oct_cas <- max(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'casual')$ride_length)
ride_length_max_nov_cas <- max(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'casual')$ride_length)
ride_length_max_dec_cas <- max(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'casual')$ride_length)
ride_length_max_winter_cas <- max(subset(cyclist_compiled_final, member_casual == 'casual'& (start_month == 1 | start_month == 2 | start_month == 3))$ride_length)
ride_length_max_spring_cas <- max(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 4 | start_month == 5 | start_month == 6))$ride_length)
ride_length_max_summer_cas <- max(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 7 | start_month == 8 | start_month == 9))$ride_length)
ride_length_max_fall_cas <- max(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 10 | start_month == 11 | start_month == 12))$ride_length)

ride_length_max_mem <- max(subset(cyclist_compiled_final, member_casual == 'member')$ride_length)
ride_length_max_jan_mem <- max(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'member')$ride_length)
ride_length_max_feb_mem <- max(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'member')$ride_length)
ride_length_max_march_mem <- max(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'member')$ride_length)
ride_length_max_april_mem <- max(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'member')$ride_length)
ride_length_max_may_mem <- max(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'member')$ride_length)
ride_length_max_june_mem <- max(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'member')$ride_length)
ride_length_max_july_mem <- max(subset(cyclist_compiled_final, start_month == 7 & member_casual == 'member')$ride_length)
ride_length_max_aug_mem <- max(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'member')$ride_length)
ride_length_max_sept_mem <- max(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'member')$ride_length)
ride_length_max_oct_mem <- max(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'member')$ride_length)
ride_length_max_nov_mem <- max(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'member')$ride_length)
ride_length_max_dec_mem <- max(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'member')$ride_length)
ride_length_max_winter_mem <- max(subset(cyclist_compiled_final, member_casual == 'member'& (start_month == 1 | start_month == 2 | start_month == 3))$ride_length)
ride_length_max_spring_mem <- max(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 4 | start_month == 5 | start_month == 6))$ride_length)
ride_length_max_summer_mem <- max(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 7 | start_month == 8 | start_month == 9))$ride_length)
ride_length_max_fall_mem <- max(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 10 | start_month == 11 | start_month == 12))$ride_length)
```

Calculate mode day of the week by overall, month, and season for all
riders, causal riders, and members

``` r
find_mode <- function(x) {
  u <- unique(x)
  tab <- tabulate(match(x, u))
  u[tab == max(tab)]
}

day_of_week_mode <- find_mode(cyclist_compiled_final$start_day_of_week)
day_of_week_mode_jan <- find_mode(subset(cyclist_compiled_final, start_month == 1)$start_day_of_week)
day_of_week_mode_feb <- find_mode(subset(cyclist_compiled_final, start_month == 2)$start_day_of_week)
day_of_week_mode_march <- find_mode(subset(cyclist_compiled_final, start_month == 3)$start_day_of_week)
day_of_week_mode_april <- find_mode(subset(cyclist_compiled_final, start_month == 4)$start_day_of_week)
day_of_week_mode_may <- find_mode(subset(cyclist_compiled_final, start_month == 5)$start_day_of_week)
day_of_week_mode_june <- find_mode(subset(cyclist_compiled_final, start_month == 6)$start_day_of_week)
day_of_week_mode_july <- find_mode(subset(cyclist_compiled_final, start_month == 7)$start_day_of_week)
day_of_week_mode_aug <- find_mode(subset(cyclist_compiled_final, start_month == 8)$start_day_of_week)
day_of_week_mode_sept <- find_mode(subset(cyclist_compiled_final, start_month == 9)$start_day_of_week)
day_of_week_mode_oct <- find_mode(subset(cyclist_compiled_final, start_month == 10)$start_day_of_week)
day_of_week_mode_nov <- find_mode(subset(cyclist_compiled_final, start_month == 11)$start_day_of_week)
day_of_week_mode_dec <- find_mode(subset(cyclist_compiled_final, start_month == 12)$start_day_of_week)
day_of_week_mode_winter <- find_mode(subset(cyclist_compiled_final, start_month == 1 | start_month == 2 | start_month == 3)$start_day_of_week)
day_of_week_mode_spring <- find_mode(subset(cyclist_compiled_final, start_month == 4 | start_month == 5 | start_month == 6)$start_day_of_week)
day_of_week_mode_summer <- find_mode(subset(cyclist_compiled_final, start_month == 7 | start_month == 8 | start_month == 9)$start_day_of_week)
day_of_week_mode_fall <- find_mode(subset(cyclist_compiled_final, start_month == 10 | start_month == 11 | start_month == 12)$start_day_of_week)

day_of_week_mode_cas <- find_mode(subset(cyclist_compiled_final, member_casual == 'casual')$start_day_of_week)
day_of_week_mode_jan_cas <- find_mode(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_feb_cas <- find_mode(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_march_cas <- find_mode(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_april_cas <- find_mode(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_may_cas <- find_mode(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_june_cas <- find_mode(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_july_cas <- find_mode(subset(cyclist_compiled_final, start_month == 7 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_aug_cas <- find_mode(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_sept_cas <- find_mode(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_oct_cas <- find_mode(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_nov_cas <- find_mode(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_dec_cas <- find_mode(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'casual')$start_day_of_week)
day_of_week_mode_winter_cas <- find_mode(subset(cyclist_compiled_final, member_casual == 'casual'& (start_month == 1 | start_month == 2 | start_month == 3))$start_day_of_week)
day_of_week_mode_spring_cas <- find_mode(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 4 | start_month == 5 | start_month == 6))$start_day_of_week)
day_of_week_mode_summer_cas <- find_mode(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 7 | start_month == 8 | start_month == 9))$start_day_of_week)
day_of_week_mode_fall_cas <- find_mode(subset(cyclist_compiled_final, member_casual == 'casual' & (start_month == 10 | start_month == 11 | start_month == 12))$start_day_of_week)

day_of_week_mode_mem <- find_mode(subset(cyclist_compiled_final, member_casual == 'member')$start_day_of_week)
day_of_week_mode_jan_mem <- find_mode(subset(cyclist_compiled_final, start_month == 1 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_feb_mem <- find_mode(subset(cyclist_compiled_final, start_month == 2 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_march_mem <- find_mode(subset(cyclist_compiled_final, start_month == 3 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_april_mem <- find_mode(subset(cyclist_compiled_final, start_month == 4 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_may_mem <- find_mode(subset(cyclist_compiled_final, start_month == 5 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_june_mem <- find_mode(subset(cyclist_compiled_final, start_month == 6 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_july_mem <- find_mode(subset(cyclist_compiled_final, start_month == 7 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_aug_mem <- find_mode(subset(cyclist_compiled_final, start_month == 8 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_sept_mem <- find_mode(subset(cyclist_compiled_final, start_month == 9 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_oct_mem <- find_mode(subset(cyclist_compiled_final, start_month == 10 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_nov_mem <- find_mode(subset(cyclist_compiled_final, start_month == 11 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_dec_mem <- find_mode(subset(cyclist_compiled_final, start_month == 12 & member_casual == 'member')$start_day_of_week)
day_of_week_mode_winter_mem <- find_mode(subset(cyclist_compiled_final, member_casual == 'member'& (start_month == 1 | start_month == 2 | start_month == 3))$start_day_of_week)
day_of_week_mode_spring_mem <- find_mode(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 4 | start_month == 5 | start_month == 6))$start_day_of_week)
day_of_week_mode_summer_mem <- find_mode(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 7 | start_month == 8 | start_month == 9))$start_day_of_week)
day_of_week_mode_fall_mem <- find_mode(subset(cyclist_compiled_final, member_casual == 'member' & (start_month == 10 | start_month == 11 | start_month == 12))$start_day_of_week)
```

### Table 1. Summary Table

<img src="summary_table.png" width="50%" />

Most rides were taken by members at 61 percent. Members took more rides
than casual users every season, though the difference was highest in the
winter and lowest in the summer. The mean ride length for members was
consistently shorter than causal users. Weekdays were the most popular
for members and weekends were most popular for casual users, consistent
across seasons.

### Figure 1. Mean Ride Length by Day of Week

<img src="cyclistic-markdown_files/figure-gfm/means overall-1.png" width="30%" /><img src="cyclistic-markdown_files/figure-gfm/means overall-2.png" width="30%" /><img src="cyclistic-markdown_files/figure-gfm/means overall-3.png" width="30%" />

*Do casual riders and members differ on ride length and is the pattern
dependent on day of week?*

**Ride length was longest on weekends for all Cyclistic users, though
the difference was smaller for members than causal riders. Members had
shorter average ride lengths in general. This pattern may be due to
member’s commutes being shorter than casual user’s pleasure rides.**

### Figure 2. Mean Ride Length by Day of Week Monthly

<img src="cyclistic-markdown_files/figure-gfm/means by month-1.png" width="100%" style="display: block; margin: auto;" /><img src="cyclistic-markdown_files/figure-gfm/means by month-2.png" width="100%" style="display: block; margin: auto;" /><img src="cyclistic-markdown_files/figure-gfm/means by month-3.png" width="100%" style="display: block; margin: auto;" />

*Do patterns of ride length change month-to-month for casual users or
members?*

Ride length is nearly always longest on the weekends for both casual
users and members across the year, with casual users consistently
showing larger differences between weekends and weekdays. Members have
the largest difference between weekends and weekdays in warmer months,
while casual users have the largest difference in colder months.
**Members might be riding more for pleasure on the weekends in warmer
months and casual rider weekday ride length might be greater in summer
months due to tourists vacationing on weekdays in addition to
weekends.**

### Figure 3. Rides per Day of Week

<img src="cyclistic-markdown_files/figure-gfm/number of rides overall-1.png" width="30%" /><img src="cyclistic-markdown_files/figure-gfm/number of rides overall-2.png" width="30%" /><img src="cyclistic-markdown_files/figure-gfm/number of rides overall-3.png" width="30%" />

*Do casual riders and members differ on which days of the week they are
most likely to use Cylcistic?*

When rides per day are split between casual riders and members, an
inverse pattern occurs; casual users ride most on weekends while members
ride most on weekdays. There was less variation for overall riders
compared to data split between casual riders and members. **This pattern
is likely due to members commuting to work while casual users ride for
pleasure.**

### Figure 4. Rides per Day of Week by Month

<img src="cyclistic-markdown_files/figure-gfm/number of rides overall by month-1.png" width="100%" style="display: block; margin: auto;" /><img src="cyclistic-markdown_files/figure-gfm/number of rides overall by month-2.png" width="100%" style="display: block; margin: auto;" /><img src="cyclistic-markdown_files/figure-gfm/number of rides overall by month-3.png" width="100%" style="display: block; margin: auto;" />

*Do patterns of daily popularity change month-to-month for casual users
or members?*

Member preference for weekday trips does not hold in July or October and
weekend trips instead become more popular. Casual rider preference for
weekend trips also fails to hold in winter months and weekday trips
become more popular. **This is likely because in colder months trips are
much more likely to be for business (commutes) and in warmer months
they’re much more likely to be for pleasure (taken on weekends).**

In August casual riders rode on weekdays almost as much as weekends.
**This might have been caused by a large number of tourists vacationing
during the weekdays as well, or more casual riders might have been using
bikes to commute.**

Calculate most popular start station for casual riders and members

``` r
find_mode(subset(cyclist_compiled_final, member_casual == "casual" & start_station_name != "NA")$start_station_name)
```

    ## [1] "Streeter Dr & Grand Ave"

``` r
find_mode(subset(cyclist_compiled_final, member_casual == "member" & start_station_name != "NA")$start_station_name)
```

    ## [1] "Kingsbury St & Kinzie St"

### Figure 5. Most Popular Start Stations

<img src="start-station.png" width="100%" />

*Are casual riders and members most likely to begin a trip from
different places?*

Casual riders were most likely to begin their trip from Streeter Drive
and Grand Avenue, which is the Navy Pier station. Members were most
likely to begin their trip closer to downtown from the station at
Kingsbury Street and Kinzie Street. **This suggests casual riders are
likely to be tourists as Navy Pier is a popular Chicago tourist
destination.**

### Figure 6. Bike Type Popularity and Ride Lenght by Bike Type

<img src="cyclistic-markdown_files/figure-gfm/type pop and ride length by type-1.png" width="50%" /><img src="cyclistic-markdown_files/figure-gfm/type pop and ride length by type-2.png" width="50%" />

*Do casual riders and members prefer different bike types?*

Both casual riders and members prefer electric bikes to classic bikes,
though the preference is stronger for casual riders. Docked bikes are
only used by casual riders. Docked bikes had by far the longest rides.
Casual users were more likely to use classic bikes for longer, perhaps
for pleasure or exercise rather than commutes. There was no data
provided about accessibility bikes.

## Recomendations:

Casual riders and members use Cyclistic for different purposes. Members
take shorter rides on weekdays while casual users take longer rides on
weekends. This pattern suggests that members are usually riding to
commute while casual users mostly ride for pleasure. Month-to-month data
supports the business vs. pleasure theory as members take more long
weekend rides in the warmer weather and far more weekday rides in the
winter. Starting station data provides further insight into causal
riders as the most popular starting station was at Navy Pier, while the
most popular starting station for members was downtown, suggesting
casual riders are often tourists.

Given these findings, further analysis may benefit from breaking casual
riders into groups of likely, possible, and unlikely to be converted
when researching what might turn a casual user into a member. The
popularity of the Navy Pier starting station suggests that a sizeable
number of casual riders are tourists and would be very unlikely to
convert. Oppositely, casual riders most likely to become members might
be found by filtering for member-like behavior, such as short weekday
rides most of the year, as they may benefit most from membership.
Inversely, members could be filtered for casual-rider behavior (mostly
weekend use) and interviewed about why membership is worthwhile for
non-commuters. The prevalence of commuting in membership also suggests
that convincing casual riders to bike to work may cause conversion and
is a potential avenue for the team researching social media influence on
the Cyclistic community.

One further suggestion is to incorporate information on accessibility
bikes if available might provide further insight and suggestions as they
make up 8% of ridership but were not included in the dataset.
