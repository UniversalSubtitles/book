---
layout: page
title: Adjusting for Batch Effects with Linear Models
---




## Data Example

The R markdown document for this section is available [here](https://github.com/genomicsclass/labs/tree/master/batch/adjusting_with_linear_models.Rmd).

To illustrate how we can adjust for batch effects using statistical methods, we will create a data example in which the outcome of interest is confounded with batch, but not completely. We will also select an outcome for which we have an expectation of what genes should be diferentially expressed. Namely, we make sex the outcome of interest and expect genes on the Y chromosome to be diferentially expressed. We may also see genes from the X chromosome as differentially expressed since some escape X inactivation. The example dataset is below.


```r
##available from course github repository
library(GSE5859Subset)
data(GSE5859Subset)
```

To illustrate the confounding, we will pick some genes to show in a heatmap plot. We pick all Y chromosome genes, some genes that we see correlate with batch, and then some randomly selected genes.


```r
library(rafalib)
library(RColorBrewer)
library(genefilter)

batch <- factor(format(sampleInfo$date,"%m"))

chr <- geneAnnotation$CHR

tt<-rowttests(geneExpression,batch)

ind1 <- which(chr=="chrY") ##real differences
ind2 <- setdiff(c(order(tt$dm)[1:25],order(-tt$dm)[1:25]),ind1)

set.seed(1)
ind0 <- setdiff(sample(seq(along=tt$dm),50),c(ind2,ind1))
geneindex<-c(ind2,ind0,ind1)
mat<-geneExpression[geneindex,]
mat <- mat -rowMeans(mat)
```

The object `mat` contains the subset of the data we will show in the plots. By looking at an image we see the Y chromosome genes as well as those most affected by month:


```r
icolors <- colorRampPalette(rev(brewer.pal(11,"RdYlBu")))(100)
mypar(1,1)
image(t(mat),xaxt="n",yaxt="n",col=icolors)
```

![Image of gene expression data for genes selected to show difference in group as well as the batch effect, along with some randomly chosen genes.](images/R/adjusting_with_linear_models-tmp-image_of_subset-1.png) 

In what follows, we will imitate the typical analysis we would do in practice. We will act as if we don't know which genes are supposed to be differentially expressed between males and females. 

#### Exploratory Data Analysis for Evaluation

Another reason we are using the dataset described above for illustrating different approaches is because we actually have a reasonable idea of what to expect. Autosomal (not on chrX or chrY) genes on the list are likely false positives and chrY are likely true positives. ChrX genes could go either way. This gives us the opportunity to compare different procedures. Since in practice we rarely know the "truth", these evaluations are not possible. Simulations are therefore commonly used for evaluation purposes: we know the truth because we construct the data. But simulations are at risk of not capturing all the nuances of real experimental data. This dataset is an experimental dataset. 

In the next sections we will use the histogram p-values to evaluate the specificity (low false positive rates) of the batch adjustment procedures presented here. Because the autosomal genes are not expected to be differentially expressed, we should see a a flat p-value histogram. To evaluate sensitivity (low false negative rates), we will report the number of the reported genes on chrX and chrY. Below are the results for when we don't adjust and report genes with q-values smaller than 0.1. We also include a volcano plot with a horizontal dashed line separating the genes called significant from those that are not, and colors used to highlight chrX and chrY genes.


```r
library(qvalue)
res <- rowttests(geneExpression,as.factor( sampleInfo$group ))
mypar(1,2)
hist(res$p.value[which(!chr%in%c("chrX","chrY") )],main="",ylim=c(0,1300))

plot(res$dm,-log10(res$p.value))
points(res$dm[which(chr=="chrX")],-log10(res$p.value[which(chr=="chrX")]),col=1,pch=16)
points(res$dm[which(chr=="chrY")],-log10(res$p.value[which(chr=="chrY")]),col=2,pch=16,xlab="Effect size",ylab="-log10(p-value)")
legend("bottomright",c("chrX","chrY"),col=1:2,pch=16)
qvals <- qvalue(res$p.value)$qvalue
index <- which(qvals<0.1)
abline(h=-log10(max(res$p.value[index])))
```

![p-value histogram and volcano plot for comparison between sexes. The Y chromosome genes (considered to be positives) are highlighted in red. The X chromosome genes (a subset is considered to be positive) are shown in green.](images/R/adjusting_with_linear_models-tmp-pvalue_hist_and_volcano_plots-1.png) 

```r
cat("Total genes with q-value < 0.1:",length(index))
```

```
## Total genes with q-value < 0.1: 59
```

```r
cat("Number of selected genes on chrY:", sum(chr[index]=="chrY",na.rm=TRUE))
```

```
## Number of selected genes on chrY: 8
```

```r
cat("Number of selected genes on chrX:", sum(chr[index]=="chrX",na.rm=TRUE))
```

```
## Number of selected genes on chrX: 12
```

The histogram is not flat. Instead, low p-values are over-represented. More than half of the genes on the final list are autosomal.

## Adjusting for Batch Effects with Linear Models

The R markdown document for this section is available [here](https://github.com/genomicsclass/labs/tree/master/batch/adjusting_with_linear_models.Rmd).

We have already observed that processing date has an effect on gene expression.  We will therefore try to _adjust_ for this by including it in a model.  When we perform a t-test comparing the two groups, it is equivalent to fitting the following linear model:

{$$}Y_{ij} = \alpha_j + x_i \beta_{j} + \varepsilon_{ij}{/$$}

to each gene {$$}j{/$$} with {$$}x_i=1{/$$} if subject {$$}i{/$$} is female and 0 otherwise. Note that {$$}\beta_{j}{/$$} represents the estimated difference for gene {$$}j{/$$} and {$$}\varepsilon_{ij}{/$$} represents the within group variation. So what is the problem?

The theory we described in the linear models chapter assumes that the error terms are independent. We know that this is not the case for all genes because we know the error terms from October will be more alike to each other than the June error terms. We can _adjust_ for this by including a term that models this effect:


{$$}Y_{ij} = \alpha_j + x_i \beta_{j} + z_i \gamma_j+\varepsilon_{ij}.{/$$}

Here {$$}z_i=1{/$$} if sample {$$}i{/$$} was processed in October and 0 otherwise and {$$}\gamma_j{/$$} is the month effect for gene {$$}j{/$$}. This an example of how linear models give us much more flexibility than procedures such as the t-test.

We construct a model matrix that includes batch.

```r
sex <- sampleInfo$group
X <- model.matrix(~sex+batch)
```

Now we can fit a model for each gene. For example, note the difference between the original model and one that has been adjusted for batch:


```r
j <- 7635
y <- geneExpression[j,]
X0 <- model.matrix(~sex) 
fit <- lm(y~X0-1)
summary(fit)$coef
```

```
##                 Estimate Std. Error   t value     Pr(>|t|)
## X0(Intercept)  6.9555747  0.2166035 32.112008 5.611901e-20
## X0sex         -0.6556865  0.3063237 -2.140502 4.365102e-02
```

```r
X <- model.matrix(~sex+batch)
fit <- lm(y~X)
summary(fit)$coef
```

```
##                Estimate Std. Error    t value     Pr(>|t|)
## (Intercept)  7.26329968  0.1605560 45.2384140 2.036006e-22
## Xsex        -0.04023663  0.2427379 -0.1657616 8.699300e-01
## Xbatch10    -1.23089977  0.2427379 -5.0709009 5.070727e-05
```

We then fit this new model for each gene. For instance, we can use `sapply` to recover the estimated coefficient and p-value in the following way:


```r
res <- t( sapply(1:nrow(geneExpression),function(j){
  y <- geneExpression[j,]
  fit <- lm(y~X-1)
  summary(fit)$coef[2,c(1,4)]
} ) )


##turn into data.frame so we can use the same code for plots as above
res <- data.frame(res)
names(res) <- c("dm","p.value")

mypar(1,2)
hist(res$p.value[which(!chr%in%c("chrX","chrY") )],main="",ylim=c(0,1300))

plot(res$dm,-log10(res$p.value))
points(res$dm[which(chr=="chrX")],-log10(res$p.value[which(chr=="chrX")]),col=1,pch=16)
points(res$dm[which(chr=="chrY")],-log10(res$p.value[which(chr=="chrY")]),col=2,pch=16,xlab="Effect size",ylab="-log10(p-value)")
legend("bottomright",c("chrX","chrY"),col=1:2,pch=16)
qvals <- qvalue(res$p.value)$qvalue
index <- which(qvals<0.1)
abline(h=-log10(max(res$p.value[index])))
```

![p-value histogram and volcano plot for comparison between sexes after adjustement for month. The Y chromosome genes (considered to be positives) are highlighted in red. The X chromosome genes (a subset is considered to be positive) are shown in green.](images/R/adjusting_with_linear_models-tmp-pvalue_hist_and_volcano_plots2-1.png) 

```r
cat("Total genes with q-value < 0.1:",length(index))
```

```
## Total genes with q-value < 0.1: 17
```

```r
cat("Number of selected genes on chrY:", sum(chr[index]=="chrY",na.rm=TRUE))
```

```
## Number of selected genes on chrY: 6
```

```r
cat("Number of selected genes on chrX:", sum(chr[index]=="chrX",na.rm=TRUE))
```

```
## Number of selected genes on chrX: 9
```

There is a great improvement in specificity (less false positives) without much loss in sensitivity (we still find many chrY genes). However, we still see some bias in the histogram. In the following sections we will see that month does not perfectly account for the batch effect and that better estimates are possible.


## A Note on Computing Efficiency

The R markdown document for this section is available [here](https://github.com/genomicsclass/labs/tree/master/batch/adjusting_with_linear_models.Rmd).

In the code above, the design matrix does not change within the iterations we are computing {$$}(X^\top X)^{-1}{/$$} repeatedly and applying to each gene. Instead we can perform this calculation in one matrix algebra calculation by computing it once and then obtaining all the betas by multiplying {$$}(X^\top X)^{-1}X^\top Y{/$$} with the columns of {$$}Y{/$$} representing genes in this case. The `limma` package has an implementation of this idea (using the QR decomposition). Notice how much faster this is:


```r
library(limma)
X <- model.matrix(~sex+batch)
fit <- lmFit(geneExpression,X)
```

The estimated regression coefficients for each gene are obtained like this:

```r
dim( fit$coef)
```

```
## [1] 8793    3
```
We have one estimate for each gene. To obtain p-values for one of these, we have to construct the ratios:


```r
k <- 2 ##second coef
ses <- fit$stdev.unscaled[,k]*fit$sigma
ttest <- fit$coef[,k]/ses
pvals <- 2*pt(-abs(ttest),fit$df)
```

#### Combat

 Combat [NEED CITATION] is a popular method and is based on using linear models to adjust for batch effects. It fits a hierarchical model (we will learn about these in the next section) to estimate and remove row specific batch effects. Combat uses a modular approach. In a first step, what is considered to be a batch effect is removed:


```r
library(sva) #available from Bioconductor
```

```
## Loading required package: mgcv
## Loading required package: nlme
## This is mgcv 1.8-6. For overview type 'help("mgcv-package")'.
```

```r
mod <- model.matrix(~sex)
cleandat <- ComBat(geneExpression,batch,mod)
```

```
## Found 2 batches
## Adjusting for 1 covariate(s) or covariate level(s)
## Standardizing Data across genes
## Fitting L/S model and finding priors
## Finding parametric adjustments
## Adjusting the Data
```

Then the results can be used to fit a model with our variable of interest:



```r
res<-genefilter::rowttests(cleandat,factor(sex))
```

In this case, the results are less specific than what we obtain by fitting the simple linear model:


```r
mypar(1,2)
hist(res$p.value[which(!chr%in%c("chrX","chrY") )],main="",ylim=c(0,1300))

plot(res$dm,-log10(res$p.value))
points(res$dm[which(chr=="chrX")],-log10(res$p.value[which(chr=="chrX")]),col=1,pch=16)
points(res$dm[which(chr=="chrY")],-log10(res$p.value[which(chr=="chrY")]),col=2,pch=16,xlab="Effect size",ylab="-log10(p-value)")
legend("bottomright",c("chrX","chrY"),col=1:2,pch=16)
qvals <- qvalue(res$p.value)$qvalue
index <- which(qvals<0.1)
abline(h=-log10(max(res$p.value[index])))
```

![p-value histogram and volcano plot for comparison between sexes for Combat. The Y chromosome genes (considered to be positives) are highlighted in red. The X chromosome genes (a subset is considered to be positive) are shown in green.](images/R/adjusting_with_linear_models-tmp-pvalue_hist_and_volcano_plots3-1.png) 

```r
cat("Total genes with q-value < 0.1:",length(index))
```

```
## Total genes with q-value < 0.1: 68
```

```r
cat("Number of selected genes on chrY:", sum(chr[index]=="chrY",na.rm=TRUE))
```

```
## Number of selected genes on chrY: 8
```

```r
cat("Number of selected genes on chrX:", sum(chr[index]=="chrX",na.rm=TRUE))
```

```
## Number of selected genes on chrX: 16
```
