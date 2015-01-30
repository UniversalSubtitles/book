---
layout: page
title: Central Limit Theorem in practice
---



# Introduction



## Start by reading in the data

```r
url <- "https://raw.githubusercontent.com/genomicsclass/dagdata/master/inst/extdata/mice_pheno.csv"
filename <- tempfile()
download.file(url,destfile=filename,method="curl")
dat <- read.csv(filename)
```


```r
head(dat)
```

```
##   Sex Diet Bodyweight
## 1   F   hf      31.94
## 2   F   hf      32.48
## 3   F   hf      22.82
## 4   F   hf      19.92
## 5   F   hf      32.22
## 6   F   hf      27.50
```

Start by selecting only female mice


```r
hfPopulation <- dat[dat$Sex=="F" & dat$Diet=="hf",3]
chowPopulation <- dat[dat$Sex=="F" & dat$Diet=="chow",3]
```

We can compute the population parameters


```r
mu_hf <- mean(hfPopulation)
mu_chow <- mean(chowPopulation)
print(mu_hf - mu_chow)
```

```
## [1] 2.375517
```

Compute the population standard deviations as well


```r
sd_hf <- mean((hfPopulation-mu_hf)^2)
sd_chow <- mean((chowPopulation-mu_chow)^2)
```

These are values we do not get to see. We want to estimate them.
The central limit tells us that if we take a sample

```r
N <- 12
hf <- sample(hfPopulation,12)
chow <- sample(chowPopulation,12)
```
the average of each of these is approximately normal with average population mean and standard error population variance divided by $N$. In practice how large must $N$ be? Here we can study that because we actually have all the populations.


```r
Ns <- c(3,12,25,50)
B <- 10000 #number of simulations
res <- sapply(Ns,function(n){
  sapply(1:B,function(j){
    mean(sample(hfPopulation,n))-mean(sample(chowPopulation,n))
  })
})
```


```r
library(rafalib)
```

```
## Loading required package: RColorBrewer
```

```r
mypar2(2,2)
for(i in seq(along=Ns)){
  title <- paste("Avg=",signif(mean(res[,i]),3),"SD=",signif(sd(res[,i]),3))
  qqnorm(res[,i],main=title)
  qqline(res[,i])
}
```

![plot of chunk unnamed-chunk-9](assets/fig/unnamed-chunk-9-1.png) 


```r
mypar2(1,1)
boxplot(res,names=Ns)
```

![plot of chunk unnamed-chunk-10](assets/fig/unnamed-chunk-10-1.png) 



