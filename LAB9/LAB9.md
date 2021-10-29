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
    ##               expr      min       lq     mean   median       uq     max neval
    ##     fun1(n = 1000) 35.27426 37.37109 43.37551 42.02885 59.37468 7.88178   100
    ##  fun1alt(n = 1000)  1.00000  1.00000  1.00000  1.00000  1.00000 1.00000   100

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
    ##        expr    min       lq     mean   median       uq      max neval
    ##     fun2(x) 11.375 10.71445 10.51034 10.07345 11.26773 3.865816   100
    ##  fun2alt(x)  1.000  1.00000  1.00000  1.00000  1.00000 1.000000   100
