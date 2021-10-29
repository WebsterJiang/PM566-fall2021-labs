LAB9
================

## Problem 2:

This function generates a n x k dataset with all its entries distributed
poission with mean lambda.

``` r
set.seed(123)
fun1 <- function(n = 100, k = 4, lambda = 4) {
  x <- NULL
  for (i in 1:n)
    x <- rbind(x, rpois(k, lambda))
  x
}

fun1alt <- function(n = 100, k = 4, lambda = 4) {
  matrix(rpois(n*k,lambda), nrow=n, ncol=k)
}

# Benchmarking
microbenchmark::microbenchmark(
  fun1(n=1000),
  fun1alt(n=1000),unit="relative"
)
```

    ## Unit: relative
    ##               expr      min       lq     mean   median       uq      max neval
    ##     fun1(n = 1000) 34.29902 36.79315 44.90346 43.52127 56.84852 15.67115   100
    ##  fun1alt(n = 1000)  1.00000  1.00000  1.00000  1.00000  1.00000  1.00000   100

Find the column max (hint: Checkout the function max.col()).

``` r
# Data Generating Process (10 x 10,000 matrix)
set.seed(1234)
x <- matrix(rnorm(1e4), nrow=10)

# Find each column's max value
fun2 <- function(x) {
  apply(x, 2, max)
}

fun2alt <- function(x) {
  # position of the max value per row of x
  idx <-max.col(t(x))
  
  # Do something to get the actual max value
  # x[cbind(1,15)]-x[1,15]
  # Want to access x[1,16], x[4,1]
  # x[rbind(c(1,16),c(4,1))]
  # want to access x[4,16],x[4,1]
  # x[cbind(4,c(16,1))]
  x[cbind(idx,1:ncol(x))]
  
}

# Do we get the same?
all(fun2(x) == fun2alt(x))
```

    ## [1] TRUE

``` r
x <- matrix(rnorm(5e4), nrow=10)
# Benchmarking
microbenchmark::microbenchmark(
  fun2(x),
  fun2alt(x), unit = "relative"
)
```

    ## Unit: relative
    ##        expr      min       lq     mean   median       uq      max neval
    ##     fun2(x) 11.18767 10.76174 9.256478 9.522459 9.050508 4.825406   100
    ##  fun2alt(x)  1.00000  1.00000 1.000000 1.000000 1.000000 1.000000   100

## Problem 3: Parallelize everything

``` r
library(parallel)
my_boot <- function(dat, stat, R, ncpus = 1L) {
  
  # Getting the random indices
  n <- nrow(dat)
  idx <- matrix(sample.int(n, n*R, TRUE), nrow=n, ncol=R)
 
  # Making the cluster using `ncpus`

  cl <- makePSOCKcluster(ncpus)    
  # STEP 2: GOES HERE
  clusterSetRNGStream(cl, 123) # Equivalent to `set.seed(123)`
  clusterExport(cl, c("stat","dat","idx"), envir = environment())
                
                
          
    # STEP 3: THIS FUNCTION NEEDS TO BE REPLACES WITH parLapply
  ans <- parLapply(cl=cl, seq_len(R), function(i) {
    stat(dat[idx[,i], , drop=FALSE])
  })
  
  # Coercing the list into a matrix
  ans <- do.call(rbind, ans)
  
  # STEP 4: GOES HERE
  stopCluster(cl)
  
  ans
  
}
```

1.Use the previous pseudocode, and make it work with parallel. Here is
just an example for you to try:

``` r
library(parallel)
# Bootstrap of an OLS
my_stat <- function(d) coef(lm(y ~ x, data=d))

# DATA SIM
set.seed(1)
n <- 500; R <- 1e4

x <- cbind(rnorm(n)); y <- x*5 + rnorm(n)

# Checking if we get something similar as lm
ans0 <- confint(lm(y~x))
ans1 <- my_boot(dat = data.frame(x, y), my_stat, R = R, ncpus = 2L)

# You should get something like this
t(apply(ans1, 2, quantile, c(.025,.975)))
```

    ##                   2.5%      97.5%
    ## (Intercept) -0.1386903 0.04856752
    ## x            4.8685162 5.04351239

``` r
##                   2.5%      97.5%
## (Intercept) -0.1372435 0.05074397
## x            4.8680977 5.04539763
ans0
```

    ##                  2.5 %     97.5 %
    ## (Intercept) -0.1379033 0.04797344
    ## x            4.8650100 5.04883353

``` r
##                  2.5 %     97.5 %
## (Intercept) -0.1379033 0.04797344
## x            4.8650100 5.0488335
```

2.Check whether your version actually goes faster than the non-parallel
version:

``` r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 1L))
```

    ##    user  system elapsed 
    ##   0.084   0.019   4.131

``` r
system.time(my_boot(dat = data.frame(x, y), my_stat, R = 4000, ncpus = 2L))
```

    ##    user  system elapsed 
    ##   0.122   0.022   3.030
