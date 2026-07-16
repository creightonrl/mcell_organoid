---
title: "M cell organoid digital PCR analysis"
author: "Rachel Creighton"
date: "2026-07-15"
output:
  html_document:
    keep_md: true
---


# Overview
In this project, we used digital PCR to measure the expression of M cell marker genes (GP2, SpiB, CCL20), interferon-stimulated genes (ISG15, MX1, IFI6, etc), and TLRs. 

We initially iterated through several previously published organoid media formulations to determine which resulted in the highest M cell marker expression. These experiments were performed in duodenal-derived organoids from a single donor.

# M cell marker expression
## Base media comparison
The first set of experiments, evaluating "differentiation" media published by Ding et al and "patterning" and "maturation" media published by He et al were repeated 3 times.


``` r
experiments <- paste("organoid5", "organoid6", "organoid7", sep = "|")
file_names <- list.files(path = here("data", "dpcr", "raw"),
                         pattern = experiments)

data <- list()
for (i in 1:length(file_names)) {
  tmp <- read_csv(here("data", "dpcr", "raw", file_names[i])) %>%
    mutate(experiment_name = str_extract(file_names[i], "^\\d{4}_\\d{4}_\\w{9}")) %>%
    rename(chamber = "ChamberID",
          sample_desc = "SampleName",
          droplet_num = "TotalNumberOfDroplets",
          blue_conc = "Blue_Channel_Concentration",
          green_conc = "Green_Channel_Concentration",
          experiment_name = "experiment_name") %>%
    mutate(chamber = str_sub(chamber, -2)) %>%
    select(chamber, sample_desc, droplet_num, blue_conc, green_conc, experiment_name) %>%
    pivot_longer(c(blue_conc, green_conc), names_to = "color", names_pattern = "(.*)_conc", values_to = "concentration")
  
  data[[i]] <- tmp
}

tidy_data <- bind_rows(data)
```


``` r
experiments <- paste("organoid5", "organoid6", "organoid7", sep = "|")
file_names <- list.files(path = here("data", "dpcr", "metadata"),
                         pattern = experiments)

metadata <- list()
for (i in 1:length(file_names)) {
  tmp <- readxl::read_xlsx(here("data", "dpcr", "metadata", file_names[i])) %>%
    mutate(experiment_name = str_extract(file_names[i], "^\\d{4}_\\d{4}_\\w{9}")) %>%
    mutate(sample = as.character(sample)) %>%
    pivot_longer(c(blue, green), names_to = "color", values_to = "gene")
  
  metadata[[i]] <- tmp
}

tidy_metadata <- bind_rows(metadata)
```


``` r
experiments <- paste("organoid5", "organoid6", "organoid7", sep = "|")
file_names <- list.files(path = here("data", "metadata"),
                         pattern = experiments)

organoid_metadata <- list()
for (i in 1:length(file_names)) {
  tmp <- read_csv(here("data", "metadata", file_names[i])) %>%
    mutate(organoid_experiment = str_extract(file_names[i], "^\\w{9}")) %>%
    mutate(sample = as.character(sample))
  
  organoid_metadata[[i]] <- tmp
}

tidy_organoid_metadata <- bind_rows(organoid_metadata)
```



``` r
tidy_full_data <- full_join(tidy_data, tidy_metadata) %>%
  filter(droplet_num >= 8000) %>%
  filter_out(sample == "NTC" | sample == "NRT") %>%
  drop_na() %>%
  mutate(orig_concentration = concentration * dilution_factor) %>%
  mutate(organoid_experiment = str_extract(experiment_name, "organoid\\d+$"))
```

```
## Joining with `by = join_by(chamber, experiment_name, color)`
```

``` r
head(tidy_full_data)
```

```
## # A tibble: 6 × 11
##   chamber sample_desc droplet_num experiment_name     color concentration sample
##   <chr>   <chr>             <dbl> <chr>               <chr>         <dbl> <chr> 
## 1 A1      1                 17574 2024_1107_organoid5 blue         27780  1     
## 2 A1      1                 17574 2024_1107_organoid5 green          413. 1     
## 3 B1      2                 16871 2024_1107_organoid5 blue         34116  2     
## 4 B1      2                 16871 2024_1107_organoid5 green          456. 2     
## 5 C1      3                 16604 2024_1107_organoid5 blue         35182  3     
## 6 C1      3                 16604 2024_1107_organoid5 green          441. 3     
## # ℹ 4 more variables: dilution_factor <dbl>, gene <chr>,
## #   orig_concentration <dbl>, organoid_experiment <chr>
```

``` r
normalized_data <- tidy_full_data %>%
  dplyr::select(c(organoid_experiment, sample, orig_concentration, gene)) %>%
  filter_out(orig_concentration == "Inf") %>%
  pivot_wider(names_from = gene, values_from = orig_concentration, values_fn = mean) %>%
  mutate(across(.cols = c("CCL20", "GP2", "SPIB"),
                .fns = ~ .x / RPP30,
                .names = "{.col}_norm"))

normalized_data <- full_join(normalized_data, tidy_organoid_metadata) %>%
  mutate(condition = paste(media, mcell_factors, sep = "_"))
```

```
## Joining with `by = join_by(organoid_experiment, sample)`
```


``` r
mean_data <- normalized_data %>%
  group_by(organoid_experiment, condition) %>%
  summarize(
    across(
      ends_with("_norm"),       
      ~ mean(.x, na.rm = TRUE),
      .names = "{.col}_mean"
    ))
```

```
## `summarise()` has regrouped the output.
## ℹ Summaries were computed grouped by organoid_experiment and condition.
## ℹ Output is grouped by organoid_experiment.
## ℹ Use `summarise(.groups = "drop_last")` to silence this message.
## ℹ Use `summarise(.by = c(organoid_experiment, condition))` for per-operation
##   grouping (`?dplyr::dplyr_by`) instead.
```

``` r
ggplot(normalized_data, aes(x = condition, y = GP2_norm, color = organoid_experiment)) +
    geom_jitter(width = 0.1) +
    geom_point(data = mean_data, 
               aes(x = condition, y = GP2_norm_mean), 
               size = 6, 
               shape = "\u2014") +
    scale_y_log10() +
    theme_classic() +
    labs(y = "GP2/RPP30")
```

```
## Warning in scale_y_log10(): log-10 transformation introduced infinite values.
## log-10 transformation introduced infinite values.
```

```
## Warning: Removed 5 rows containing missing values or values outside the scale range
## (`geom_point()`).
```

![](dpcr_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

