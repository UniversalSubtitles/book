---
title: "Matrix Algebra Notation"
author: "Rafa"
date: "January 31, 2015"
output: html_document
layout: page
---



# Introduciton

Here we introduce the basics of matrix notation. At first this will all seem over-complicated but once we get to the examples you will start to appreciate its power of using this notation to both explain and derive solutions as well as implement them as R code. 

# The language of linear models

Linear algebra notation actually  simplifies the mathematical descriptions and manipulations of linear models as well are coding in R. We will show you the basics of this notation and then show some example in R.

The main point of this entire exercise is to show how we can write the models above using matrix notation and then explain how this is useful for solving the least squares equation. We start by simply defining notation and matrix multiplication, but bare with us, we eventually get back to the practical application.

## Motivation

Linear algebra was created by mathematicians to solve systems of linear equations such as this:

$$
\begin{aligned}
a + b + c &= 6\\\
3a - 2b + c &= 2\\\
2a + b  - c &= 1
\end{aligned}
$$

It provides very useful machinery to solve these problems generally. We will learn how we can write and solve this system using matrix algebra notation:


$$
\begin{pmatrix}
1&1&1\\\
3&-2&1\\\
2&1&-1
\end{pmatrix}
\begin{pmatrix}
a\\\
b\\\
c
\end{pmatrix} =
\begin{pmatrix}
6\\\
2\\\
1
\end{pmatrix}
\implies
\begin{pmatrix}
a\\\
b\\\
c
\end{pmatrix} =
\begin{pmatrix}
1&1&1\\\
3&-2&1\\\
2&1&-1
\end{pmatrix}^{-1}
\begin{pmatrix}
6\\\
2\\\
1
\end{pmatrix}
$$

This section explains the notation used above. It turns that we can borrow this notation for linear models in statistics as well.




## Vectors, Matrices and Scalars

In the examples above the random variables associated with the data were represented by $Y_1,\dots,Y_n$. We can think of this as a vector. In fact, in R we are already doing this:


```r
library(UsingR)
y=father.son$fheight
head(y)
```

```
## [1] 65.04851 63.25094 64.95532 65.75250 61.13723 63.02254
```
In math we can also use just one symbol and we usually use bold to distinguish it from the individual entries:

$$ \mathbf{Y} = \begin{pmatrix}
Y_1\\\
Y_2\\\
\vdots\\\
Y_N
\end{pmatrix}
$$

For reasons that will become clear soon, default representation of data vectors have dimension $N\times 1$ as opposed to $1 \times N$.

Note: we don't always use bold because commonly one can tell what is a matrix from the context.

Similarly we can use math notation to represent the covariates or predictors. In the case of the two, with the second one just being the square of the first.

$$
\mathbf{X}_1=\begin{pmatrix}
x_{1,1}\\\
\vdots\\\
x_{N,1}
\end{pmatrix}\mbox{ and }
\mathbf{X}_2=\begin{pmatrix}
x_{1,2}\\\
\vdots\\\
x_{N,2}
\end{pmatrix}
$$

Note that, for the object falling example $x_{1,1}= t_i$ and $x_{i,1}=t_i^2$ with $t_i$ the time of the i-th observation. Also note that vectors can be thought of as $N\times 1$ matrices 

For reasons that will become clear soon, it is convenient to representing  these in matrices:

$$
\mathbf{X} = [ \mathbf{X}_1 \mathbf{X_2} ] = \begin{pmatrix}
x_{1,1}&x_{1,2}\\\
\vdots\\\
x_{N,1}&x_{N,2}
\end{pmatrix}
$$

This matrix has dimension $N \times 2$. We can create this matrix in R this way


```r
n <- 25
tt <- seq(0,3.4,len=n) ##time in secs, t is a base function
X <- cbind(x1=tt,x2=tt^2)
head(X)
```

```
##             x1         x2
## [1,] 0.0000000 0.00000000
## [2,] 0.1416667 0.02006944
## [3,] 0.2833333 0.08027778
## [4,] 0.4250000 0.18062500
## [5,] 0.5666667 0.32111111
## [6,] 0.7083333 0.50173611
```

```r
dim(X)
```

```
## [1] 25  2
```

Note that we can also use this notation to denote an arbitrary number of covariates with the following $N\times p$ matrix:

$$
\mathbf{X} = \begin{pmatrix}
  x_{1,1}&\dots & x_{1,p} \\\
  x_{2,1}&\dots & x_{2,p} \\\
   & \vdots & \\\
  x_{N,1}&\dots & x_{N,p} 
  \end{pmatrix}
$$

Just as an example, we show you how to make one in R, now using `matrix` instead of `cbind`:


```r
N <- 100; p <- 5
X <- matrix(1:(N*p),N,p)
head(X)
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1  101  201  301  401
## [2,]    2  102  202  302  402
## [3,]    3  103  203  303  403
## [4,]    4  104  204  304  404
## [5,]    5  105  205  305  405
## [6,]    6  106  206  306  406
```

```r
dim(X)
```

```
## [1] 100   5
```

Note that the columns are filled by column. The `byrow=TRUE` argument let's us change that:


```r
N <- 100; p <- 5
X <- matrix(1:(N*p),N,p,byrow=TRUE)
head(X)
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    2    3    4    5
## [2,]    6    7    8    9   10
## [3,]   11   12   13   14   15
## [4,]   16   17   18   19   20
## [5,]   21   22   23   24   25
## [6,]   26   27   28   29   30
```

Finally, we define a scalar. A scalar is just a number. So why a special name? We want to distinguish it from vectors and matrices. We usually use lower case and don't bold. In the next section we will understand why we make this distinction.




