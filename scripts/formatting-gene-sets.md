---
title: "Formatting Consensomes into Gene Sets"
author: "Dave Bridges"
date: "February 19, 2019"
output:
  html_document:
    highlight: tango
    keep_md: yes
    number_sections: yes
    toc: yes
  pdf_document:
    highlight: tango
    keep_tex: yes
    number_sections: yes
    toc: yes
---



# Purpose

To generate gene lists based on experimentally derived transriptional changes for predicting transcriptional networks. This script was most recently run on Tue Feb 19 08:28:00 2019.

# Human to Mouse Gene Mapping


```r
human_mouse_table_file <- 'http://www.informatics.jax.org/downloads/reports/HOM_MouseHumanSequence.rpt'
human_mouse_table <- read.table(human_mouse_table_file, sep="\t", header=T)
human.to.mouse <- 
  human_mouse_table %>%
  select(Common.Organism.Name, Symbol, HomoloGene.ID) %>%
  distinct(HomoloGene.ID,Common.Organism.Name, .keep_all = T) %>%
  spread(Common.Organism.Name, Symbol) %>%
  rename("mouse"="mouse, laboratory") %>%
  mutate(human = as.character(human),
         mouse = as.character(mouse))
```

For mouse to human mapping I used the MGI human to mouse table at http://www.informatics.jax.org/downloads/reports/HOM_MouseHumanSequence.rpt for conversion.

# Generating Consensomes

Downloaded the consensomes on 2019-02-10 with these search criteria from the query tool at https://beta.signalingpathways.org/ominer/query.jsf and saved in the **consensomes** folder.

* Target Gene(s) of Interest: Consensome
* Omics Category: Transcriptomics
* Category Receptors; Nuclear Receptors; went through each
* Species: House Mouse
* Physiological System: All

Several receptors did not have enough data to assemble a consensome.


```r
consensome.file.list = list.files(path='consensomes-downloaded',pattern="*.csv",
                                  full.names=T)
library(readr)
consensome.data = lapply(consensome.file.list, read_csv, skip=2,
                         col_types = cols(
  ID = col_double(),
  Family = col_factor(levels=NULL),
  `Physiological System` = col_factor(levels=NULL),
  Organ = col_factor(levels=NULL),
  Species = col_factor(levels=NULL),
  Gene = col_factor(levels=NULL),
  Percentile = col_double(),
  `Discovery Rate` = col_double(),
  GMFC = col_double(),
  cPValue = col_double(),
  DOI = col_factor(levels=NULL),
  X12 = col_logical()
)) 

consensome.dataset <- do.call(rbind.data.frame, consensome.data)

consensome.data.sig <- consensome.dataset %>% filter(Percentile>99)

consensome.merged.data <- 
  left_join(consensome.data.sig, human.to.mouse, by=c("Gene"="mouse")) %>% 
  filter(!(is.na(human))) %>%
  group_by(Family) %>%
  select(human)

consensome.filename <- 'Consensomes - Mice.csv'
write.csv(consensome.data.sig, consensome.filename)

#convert to a list
consensome.list <- split(consensome.merged.data$human, consensome.merged.data$Family)
```

The consensome file that was downloaded are found in consensomes-downloaded/AMjW8Lrezy.1.csvconsensomes-downloaded/bzZQTlQOpH.1.csvconsensomes-downloaded/eKFA8s5YRX.1.csvconsensomes-downloaded/KJBzdZDiD4.1.csvconsensomes-downloaded/nHAplbKZVf.1.csvconsensomes-downloaded/PJF8NEyazd.1.csvconsensomes-downloaded/QdbZ7Mi2ts.1.csvconsensomes-downloaded/SIzL9WMaTt.1.csvconsensomes-downloaded/YjauVBA6hX.1.csv.  These mouse genes were then mapped to human genes using the Jackson laboratory HomoloGene tables, found at http://www.informatics.jax.org/downloads/reports/HOM_MouseHumanSequence.rpt.  We filtered for only genes in the top 1% of each consensome to make the gene sets.  The final consemsomes were was saved as Consensomes - Mice.csv.


# Session Information


```r
sessionInfo()
```

```
## R version 3.5.0 (2018-04-23)
## Platform: x86_64-apple-darwin15.6.0 (64-bit)
## Running under: macOS  10.14.2
## 
## Matrix products: default
## BLAS: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRblas.0.dylib
## LAPACK: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRlapack.dylib
## 
## locale:
## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] readr_1.3.1    bindrcpp_0.2.2 dplyr_0.7.8    tidyr_0.8.2   
## [5] knitr_1.21    
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_1.0.0       bindr_0.1.1      magrittr_1.5     hms_0.4.2       
##  [5] tidyselect_0.2.5 R6_2.3.0         rlang_0.3.1      stringr_1.3.1   
##  [9] tools_3.5.0      xfun_0.4         htmltools_0.3.6  yaml_2.2.0      
## [13] digest_0.6.18    assertthat_0.2.0 tibble_2.0.0     crayon_1.3.4    
## [17] purrr_0.2.5      glue_1.3.0       evaluate_0.12    rmarkdown_1.11  
## [21] stringi_1.2.4    compiler_3.5.0   pillar_1.3.1     pkgconfig_2.0.2
```
