LAB9
================

This function generates a n x k dataset with all its entries distributed
poission with mean lambda.

``` r
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
    ##     fun1(n = 1000) 34.94115 35.38844 42.78611 43.57873 53.73534 12.66564   100
    ##  fun1alt(n = 1000)  1.00000  1.00000  1.00000  1.00000  1.00000  1.00000   100

Find the column max (hint: Checkout the function max.col()).
