---
title: 2018-future weather data arrange process_3
author: Weilu Wang
date: '2018-10-05'
slug: 2018-future-weather-data-arrange-process-3
categories:
  - R
  - Data management
tags:
  - R
  - Learning exprience
  - Weather data
lastmod: '2018-10-05T14:31:20+08:00'
layout: post
type: post
highlight: no
---

**Insert Lead paragraph here.**

In this tutorial, we will further introduce how to extracting future weather data with R language.


### Manipulating data with R

The following R commands are simple.

```
setwd("E:/R_work/WeatherData/Future_weather_data/RCP2.6/")
library(openxlsx)
library(tidyverse)
library(reshape)

Future_weather <- function(num, station){
name <- dir()
file <- name[num]
site <- openxlsx::read.xlsx(file, sheet = 1) %>%
mutate(Station = station)}

Jiangsu_RCP85 <- rbind(Dafeng, Dongshan, Dongtai, Funing, Gaoyou, Kunshan,
	Lianshui, Liyang, Lvsi, Nanjing, Nantong, Rugao, Sheyang, Sihong,
	Suining, Wuxi, Yutai)

Jiangsu_RCP85 <- Jiangsu_RCP85[, c(11, 1:10)]

rm(Dafeng, Dongshan, Dongtai, Funing, Gaoyou, Kunshan,
	Lianshui, Liyang, Lvsi, Nanjing, Nantong, Rugao, Sheyang, Sihong,
	Suining, Wuxi, Yutai)

Jiangsu_RCP26_2 <- Jiangsu_RCP26[, c(1, 2, 5, 6, 8, 7, 9:11)]


colnames(Jiangsu_RCP26_2) <- c("Station", "Year", "Dateseq", "Ra", "MinT", "MaxT", "Ea_VPD", "MeanWIN", "PRE")

Jiangsu_RCP85_2_20s <- filter(Jiangsu_RCP85_2, Year >= 2010 & Year <= 2040)
Jiangsu_RCP85_2_50s <- filter(Jiangsu_RCP85_2, Year >= 2041 & Year <= 2070)
Jiangsu_RCP85_2_80s <- filter(Jiangsu_RCP85_2, Year >= 2071 & Year <= 2099)
Weather_oryza_90s <- filter(weather_oryza, Year >= 1981 & Year <= 2010)


Weather_oryza_90s$MaxT[which(Weather_oryza_90s$MaxT == 3276.6)]<-35
Weather_oryza_90s$MeanWIN[which(Weather_oryza_90s$MeanWIN == 3276.6)]<-0
Weather_oryza_90s$Ea_VPD[which(Weather_oryza_90s$Ea_VPD >= 100)]<-0

a <- as.data.frame(a <- lapply(Weather_oryza_90s, mean)) 做计算用
```

### Function used for calculating values

1. apply() function

We use `apply()` over a matrice.

```
apply(X, MARGIN, FUN)

a_m1 <- apply(m1, 2, sum)
```

2. lapply() function

l in `lapply()` stands for list. The difference between lapply() and apply() lies between the output return. The output of lapply() is a list. lapply() can be used for other objects like data frames and lists.

```
movies <- c("SPYDERMAN","BATMAN","VERTIGO","CHINATOWN")
movies_lower <-lapply(movies, tolower)
str(movies_lower)
```

3. sapply() function

`sapply()` function does the same jobs as lapply() function but returns a vector.

4. tapply() function
The function `tapply()` computes a measure (mean, median, min, max, etc..) or a function for each factor variable in a vector.

```
tapply(X, INDEX, FUN = NULL)
Arguments:
-X: An object, usually a vector
-INDEX: A list containing factor
-FUN: Function applied to each element of x
```




