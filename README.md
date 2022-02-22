
<!-- README.md is generated from README.Rmd. Please edit that file -->

# testidentical

``` r
library(testidentical)
```

These are equal

``` r
# Same env, different files, simple function (no if {} branch)
identical(test1_simple, test2_simple)
#> [1] TRUE
```

No way to make these equal

``` r
# Same env, different files, more complex function
identical(test1_if, test2_if)
#> [1] FALSE

identical(test1_if, test2_if, ignore.environment = TRUE)
#> [1] FALSE
```

^ This eventually calls `R_body_no_src()` which doesn’t recursively
remove srcref/srcfile/wholeSrcref attributes.
<https://github.com/wch/r-source/blob/9bb47ca929c41a133786fa8fff7c70162bb75e50/src/main/util.c#L631>

This is a problem because the `if {}` closures of `test1_if()` and
`tests_if()` contain srcref info

``` r
if_body1 <- body(test1_if)[[2]]
if_body2 <- body(test2_if)[[2]]

# they look the same
if_body1
#> if (TRUE) {
#>     x
#> } else {
#>     NULL
#> }
if_body2
#> if (TRUE) {
#>     x
#> } else {
#>     NULL
#> }

# but this closure had srcref info that wasn't removed!
attributes(if_body1[[3]])
#> $srcref
#> $srcref[[1]]
#> {
#> 
#> $srcref[[2]]
#> x
#> 
#> 
#> $srcfile
#> /Users/davis/Desktop/r/playground/packages/testidentical/R/test1.R 
#> 
#> $wholeSrcref
#> .packageName <- "testidentical"
#> #line 1 "/Users/davis/Desktop/r/playground/packages/testidentical/R/test1.R"
#> #' @export
#> test1_simple <- function(x) {
#>   x
#> }
#> 
#> #' @export
#> test1_if <- function(x) {
#>   if (TRUE) {
#>     x
#>   }

attributes(if_body2[[3]])
#> $srcref
#> $srcref[[1]]
#> {
#> 
#> $srcref[[2]]
#> x
#> 
#> 
#> $srcfile
#> /Users/davis/Desktop/r/playground/packages/testidentical/R/test2.R 
#> 
#> $wholeSrcref
#> .packageName <- "testidentical"
#> #line 1 "/Users/davis/Desktop/r/playground/packages/testidentical/R/test1.R"
#> #' @export
#> test1_simple <- function(x) {
#>   x
#> }
#> 
#> #' @export
#> test1_if <- function(x) {
#>   if (TRUE) {
#>     x
#>   } else {
#>     NULL
#>   }
#> }
#> #line 1 "/Users/davis/Desktop/r/playground/packages/testidentical/R/test2.R"
#> #' @export
#> test2_simple <- function(x) {
#>   x
#> }
#> 
#> #' @export
#> test2_if <- function(x) {
#>   if (TRUE) {
#>     x
#>   }
```

So those differing attributes caused the functions to not look
identical.

``` r
# removeSource() will recursively remove srcref/srcfile/wholeSrcref attributes
identical(
  removeSource(test1_if), 
  removeSource(test2_if)
)
#> [1] TRUE

# See, it was removed before comparison
attributes(body(removeSource(test1_if))[[2]][[3]])
#> NULL
```
