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
file_names
```

```
## [1] "2024_1108_organoid5_allgenes_Results.csv"                    
## [2] "2024_1209_organoid6_ccl20_rpp30_gp2_Results.csv"             
## [3] "2024_1219_organoid6_mcell_markers_Results.csv"               
## [4] "2025_0123_organoid7_mcell_markers_Results.csv"               
## [5] "2025_0124_organoid7_mcell_marker_polyaltr_repeat_Results.csv"
```

