---
title: "RNASeq_Wrangle.Rmd"
author: "Danielle Collins"
date: "10/2/2018"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Load in data
```{r}
library(dplyr)
 setwd("~/RNASeqExample/")
 samples <- read.csv('sample_info.csv',header = TRUE, sep = ",", quote = "\"", dec = ".", fill = TRUE, row.names = 1)
 genes <- read.csv('expression_results.csv',header = TRUE, sep = ",", quote = "\"", dec = ".", fill = TRUE, row.names = 1)
```

```{r}
d <- density(genes$KITA_02) # returns the density data
 plot(d) # plots the results
```
```{r}
d <- density(samples$PF_BASES) # returns the density data
 plot(d) # plots the results
```

```{r}
plot(log2(genes$KITA_01[(genes$KITA_01>10 |genes$KITA_03>10 )]),log2(genes$KITA_03[(genes$KITA_01>10 |genes$KITA_03>10 )]))
```

##Heatmap Creation

```{r}
library(ggplot2)
library(reshape2)
samples$uid=rownames(samples)
genes_summary<-data.frame(
 UID=rownames(samples),
 min=minBySample <- sapply(genes, function(x) min(x[x > 0])),
 max=maxBySample <- sapply(genes, function(x) max(x))
 )
corr<-cor(genes)
melted_corr <- melt(corr)
 head(melted_corr)
 ggplot(melted_corr , aes(x = Var1, y = Var2)) +
 geom_raster(aes(fill = value)) +
 scale_fill_gradient2(low="green", mid="white", high="red", midpoint=0.5) + theme_classic()
```

## Heatmap 2: slight modifications

```{r}
library(ggplot2)
library(reshape2)
library(plotly)
corr<-cor(genes)
melted_corr <- melt(corr)
p<-ggplot(melted_corr , aes(x = Var1, y = Var2)) + geom_raster(aes(fill = value)) + scale_fill_gradient2(low="green", mid="white", high="red", midpoint=0.5) + theme( plot.title = element_blank(),axis.text.x = element_blank(), axis.text.y = element_blank(), axis.title.y = element_blank(), axis.title.x = element_blank())
ggplotly(p)
```

## Dendrogram without annotation

```{r}
genes_sample <- t(genes[c(rep(FALSE,19),TRUE), ])
clusters <- hclust(dist(genes_sample))
 plot(clusters)
```

## Dendrogram with annotation 

```{r}
library('dendextend')
dend <- as.dendrogram(clusters)
dend <- rotate(dend, 1:93)
dend <- color_branches(dend, k=4)
par(cex=0.5) # reduces font
plot(dend)
```

## 3D Principle Component Analysis 

```{r}
library(dplyr)
setwd("~/RNASeqExample/")
samples <- read.csv('sample_info.csv',header = TRUE, sep = ",", quote = "\"", dec = ".", fill = TRUE, row.names = 1)
genes <- read.csv('expression_results.csv',header = TRUE, sep = ",", quote = "\"", dec = ".", fill = TRUE, row.names = 1)
min(genes[genes>0])
genes.log <-log2(genes+8.05e-12)
genes.log.small <- genes.log[seq(1, nrow(genes.log), 20), ]
pca <- prcomp(genes.log.small,center = TRUE,scale. = TRUE)
std_dev <- pca$sdev
  pr_var <- std_dev^2
  pr_var[1:10]
  prop_varex <- pr_var/sum(pr_var)
pcadf<-data.frame(pca$rotation)
pcadf$sample<-rownames(pcadf)
samples$sample<-rownames(samples)
join_KIT <- inner_join(pcadf, samples)
plot_ly(join_KIT, x = ~PC2, y = ~PC3, z = ~PC4, color = ~Kit) %>%
 add_markers() %>%
 layout(scene = list(xaxis = list(title = 'PC2'),
 yaxis = list(title = 'PC3'),
 zaxis = list(title = 'PC4')))
```

