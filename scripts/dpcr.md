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
    select(chamber, sample_desc, droplet_num, blue_conc, green_conc, experiment_name)
  
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
    mutate(sample = as.character(sample))
  
  metadata[[i]] <- tmp
}

tidy_metadata <- bind_rows(metadata)
```


``` r
tidy_full_data <- full_join(tidy_data, tidy_metadata)
```

```
## Joining with `by = join_by(chamber, experiment_name)`
```

``` r
head(tidy_full_data)
```

```
## # A tibble: 6 × 10
##   chamber sample_desc droplet_num blue_conc green_conc experiment_name    sample
##   <chr>   <chr>             <dbl>     <dbl>      <dbl> <chr>              <chr> 
## 1 A1      1                 17574     27780       413. 2024_1107_organoi… 1     
## 2 B1      2                 16871     34116       456. 2024_1107_organoi… 2     
## 3 C1      3                 16604     35182       441. 2024_1107_organoi… 3     
## 4 D1      4                 16177     11030       360. 2024_1107_organoi… 4     
## 5 E1      5                 15347      3945       335. 2024_1107_organoi… 5     
## 6 F1      6                 15596      5158       272. 2024_1107_organoi… 6     
## # ℹ 3 more variables: dilution_factor <dbl>, blue <chr>, green <chr>
```

