---
title: Common cases when the G matrix determinant is zero
date: 2019-11-27
permalink: /posts/2019/11/Gmatrix-0Det
tags:
  - R
  - Gmatrix
---


Common cases when the G matrix determinant is zero

Here, I show common genetic reasons where can lead to genomic
relationship matrix determinant equal to 0. They are commonly based on population structure or repeated information. The cases are not always true, but they can represent why you have a strong linear dependence in
your G matrix (Van Raden 2008), and, therefore, a determinant equal 0 with not unique inverse for the matrix. After identifying the reason, you can take action in order to manage the data before building the G matrix.


If you know any other common reason, please let me know and I can add to
the list. :)

Data used are real and from freely available data sets from `BGLR` R
package and `AGHmatrix`.

``` r
library(AGHmatrix) #for potato and pine data set and G functions
library(BGLR) #for wheat data set
library(ggfortify) #for PCAs
```

    ## Loading required package: ggplot2

G matrix with determinant higher than 0
----------------------------

An example of an invertable G matrix, the data set has characteristics
as no repeated individuals, no apparent population structure, and much
more markers than individuals.

``` r
data(snp.sol)
dim(snp.sol)
```

    ## [1]  571 3895

``` r
G_OK <- Gmatrix(snp.sol, ploidy=4)
```

    ## Initial data: 
    ##  Number of Individuals: 571 
    ##  Number of Markers: 3895 
    ## 
    ## Missing data check: 
    ##  Total SNPs: 3895 
    ##   0 SNPs dropped due to missing data threshold of 1 
    ##  Total of: 3895  SNPs 
    ## MAF check: 
    ##  No SNPs with MAF below 0 
    ## Monomorphic check: 
    ##  No monomorphic SNPs 
    ## Summary check: 
    ##  Initial:  3895 SNPs 
    ##  Final:  3895  SNPs ( 0  SNPs removed) 
    ##  
    ## Completed! Time = 2.245  seconds

``` r
det(G_OK)
```

    ## [1] -5.523282e-288

``` r
G_OK_Inv <- solve(G_OK)
PCs <- prcomp(G_OK, scale=TRUE)
autoplot(PCs)
```

![](https://rramadeu.github.io/images/zero_det_Gmatrix/unnamed-chunk-1-1.png)

1\) Repeated individuals
----------------------------

I repeat the first line in `snp.sol` (as a clone or repeated
individual), notice the off-diagonal value close to 1 in this case.

``` r
snp.sol1 <- rbind(snp.sol[1,],snp.sol)
G1 <- Gmatrix(snp.sol1, ploidy=4)
```

    ## Initial data: 
    ##  Number of Individuals: 572 
    ##  Number of Markers: 3895 
    ## 
    ## Missing data check: 
    ##  Total SNPs: 3895 
    ##   0 SNPs dropped due to missing data threshold of 1 
    ##  Total of: 3895  SNPs 
    ## MAF check: 
    ##  No SNPs with MAF below 0 
    ## Monomorphic check: 
    ##  No monomorphic SNPs 
    ## Summary check: 
    ##  Initial:  3895 SNPs 
    ##  Final:  3895  SNPs ( 0  SNPs removed) 
    ##  
    ## Completed! Time = 2.06  seconds

``` r
det(G1)
```

    ## [1] 0

``` r
#G1_Inv <- solve(G1) #error!
G1[1:4,1:4] #off-diagonal values close to 1
```

    ##                       MSH228-6  Manistee   MSS297-3
    ##          1.00038749 1.00038749 0.1536395 0.09283224
    ## MSH228-6 1.00038749 1.00038749 0.1536395 0.09283224
    ## Manistee 0.15363950 0.15363950 0.9382326 0.15281292
    ## MSS297-3 0.09283224 0.09283224 0.1528129 1.03410691

2\) More individuals than markers
----------------------------

I subset the `snp.sol` letting it with just 500 markers and all the 571
individuals.

``` r
snp.sol2 <- snp.sol[,1:500]
G2 <- Gmatrix(snp.sol2, ploidy=4)
```

    ## Initial data: 
    ##  Number of Individuals: 571 
    ##  Number of Markers: 500 
    ## 
    ## Missing data check: 
    ##  Total SNPs: 500 
    ##   0 SNPs dropped due to missing data threshold of 1 
    ##  Total of: 500  SNPs 
    ## MAF check: 
    ##  No SNPs with MAF below 0 
    ## Monomorphic check: 
    ##  No monomorphic SNPs 
    ## Summary check: 
    ##  Initial:  500 SNPs 
    ##  Final:  500  SNPs ( 0  SNPs removed) 
    ##  
    ## Completed! Time = 0.169  seconds

``` r
det(G2)
```

    ## [1] 0

``` r
#G2_Inv <- solve(G2) #error!
```

3\) Population Structure
----------------------------

`wheat` data set has a population structure with 2 subpopulations.

``` r
data(wheat)
dim(wheat.X)
```

    ## [1]  599 1279

``` r
G3 <- Gmatrix(wheat.X, ploidy=2)
```

    ## Initial data: 
    ##  Number of Individuals: 599 
    ##  Number of Markers: 1279 
    ## 
    ## Missing data check: 
    ##  Total SNPs: 1279 
    ##   0 SNPs dropped due to missing data threshold of 1 
    ##  Total of: 1279  SNPs 
    ## MAF check: 
    ##  No SNPs with MAF below 0 
    ## Monomorphic check: 
    ##  No monomorphic SNPs 
    ## Summary check: 
    ##  Initial:  1279 SNPs 
    ##  Final:  1279  SNPs ( 0  SNPs removed) 
    ##  
    ## Completed! Time = 0.62  seconds

``` r
det(G3)
```

    ## [1] 0

``` r
#G3_Inv <- solve(G3) #error!
PCs <- prcomp(G3, scale=TRUE)
autoplot(PCs)
```

![](https://rramadeu.github.io/images/zero_det_Gmatrix/unnamed-chunk-4-1.png)

4\) Population Structure at the family level
----------------------------

`snp.pine` data set has a population structure at the family level (each
cluster in the PCA).

``` r
data(snp.pine)
dim(snp.pine)
```

    ## [1]  926 4853

``` r
G4 <- Gmatrix(snp.pine, ploidy=2)
```

    ## Initial data: 
    ##  Number of Individuals: 926 
    ##  Number of Markers: 4853 
    ## 
    ## Missing data check: 
    ##  Total SNPs: 4853 
    ##   0 SNPs dropped due to missing data threshold of 1 
    ##  Total of: 4853  SNPs 
    ## MAF check: 
    ##  No SNPs with MAF below 0 
    ## Monomorphic check: 
    ##  No monomorphic SNPs 
    ## Summary check: 
    ##  Initial:  4853 SNPs 
    ##  Final:  4853  SNPs ( 0  SNPs removed) 
    ##  
    ## Completed! Time = 5.838  seconds

``` r
det(G4)
```

    ## [1] 0

``` r
#G4_Inv <- solve(G4) #error!
PCs <- prcomp(G4, scale=TRUE)
autoplot(PCs)
```

![](https://rramadeu.github.io/images/zero_det_Gmatrix/unnamed-chunk-5-1.png)

References for packages and data sets
----------------------------

``` r
citation("AGHmatrix")
citation("BGLR")
citation("ggfortify")
```

  - Amadeu, R. R., C. Cellon, J. W. Olmstead, A. A. F. Garcia, M. F. R.
    Resende, and P. R. Muñoz. 2016. AGHmatrix: R Package to Construct
    Relationship Matrices for Autotetraploid and Diploid Species: A
    Blueberry Example. The Plant Genome 9.

  - Perez, P., and de los Campos, G., 2014 Genome-Wide Regression and
    Prediction with the BGLR Statistical Package. Genetics 198 (2):
    483-495.

  - VanRaden, P.M., 2008. Efficient methods to compute genomic
    predictions. Journal of dairy science, 91(11), pp.4414-4423.

  - Yuan Tang, Masaaki Horikoshi, and Wenxuan Li. ggfortify: Unified
    Interface to Visualize Statistical Result of Popular R Packages. The
    R Journal 8.2 (2016): 478-489.
