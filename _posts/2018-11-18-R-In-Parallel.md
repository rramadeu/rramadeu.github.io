---
title: 'R Parallel with Genomic Prediction Application'
date: 2018-10-18
permalink: /posts/2018/11/RParallel/
tags:
  - R
  - parallel
---

Material developed for a workshop on *R Parallel with Genomic Selection Application* for the Genomic Selection Studies group at University of Florida, Gainesville, FL.

Material developed for a workshop on *R Parallel with Genomic Selection Application* for the Genomic Selection Studies group at University of Florida, Gainesville, FL. If you find any issue with it or have suggestions, please let me know. Here, we show how to run in parallel a cross-validation analysis in genomic prediction purposes with R and in a HPC. Dataset adapted from *Endelman, Jeffrey B., et al. "Genetic variance partitioning and genome-wide prediction with allele dosage information in autotetraploid potato." Genetics (2018): genetics-300685.* The objective is to compare the predictive ability of two different methodologies (Ridge Regression and Lasso) using `BGLR` package. The `SolData.Rdata` used in this tutorial can be found at [here](https://github.com/rramadeu/Tutorials_File/tree/master/WorkshopRParallel)

Data Set
--------

``` r
load("data/SolData.Rdata")
ls()
```

    ## [1] "clones" "M"      "y"

``` r
str(clones); str(y); str(M)
```

    ##  chr [1:549] "MSZ194-2" "W14184-5" "MSP239-1" "W9632-1" "AF4648-2" ...

    ##  num [1:549] 52.3 48 52.3 54 62.8 ...

    ##  int [1:549, 1:3895] 3 3 2 2 1 1 2 2 3 2 ...
    ##  - attr(*, "dimnames")=List of 2
    ##   ..$ : chr [1:549] "MSZ194-2" "W14184-5" "MSP239-1" "W9632-1" ...
    ##   ..$ : chr [1:3895] "solcap_snp_c1_1" "solcap_snp_c1_1000" "solcap_snp_c1_10000" "solcap_snp_c1_10001" ...

`Y` is the response variable, `M` is the molecular information, `Clones` are the name of the genotypes.

Creating training and testing populations
-----------------------------------------

Here we randomly split the dataset into 5 groups (1 to 5) to follow a 5-fold validation scheme. Hereafter, `trn` for training population and `tst` for testing population.

``` r
tst.pop <- rep(1:5,length=549)
set.seed(1234)
tst.pop <- sample(tst.pop,549,FALSE)
```

Evaluating without loop for k=1
-------------------------------

Due to time constraint, we will show all the analysis with really low number of iteractions/burnIns.

``` r
library(BGLR)

pa <- rep(NA,2) #vector to collect the predictive ability
rmse <- rep(NA,2) #vector to collect the RMSE

start.time <- Sys.time()
y.trn <- y
y.trn[tst.pop==1] <- NA

## Ridge Regression
ETA_1 <- list(PED1=list(X=M, model="BRR",saveEffects=FALSE))
fit <- BGLR(y.trn, ETA=ETA_1, 
            nIter = 100, burnIn = 50,thin = 5, 
            verbose = FALSE, saveAt = paste0("output/model_1"))
pa[1] <- cor(fit$yHat[tst.pop==1],y[tst.pop==1])
rmse[1] <- mean((fit$yHat[tst.pop==1]-y[tst.pop==1])^2)

## Bayesian Lasso
ETA_2 <- list(PED1=list(X=M, model="BL",saveEffects=FALSE))
fit <- BGLR(y.trn, ETA=ETA_2,
            nIter = 100, burnIn = 50,thin = 5, 
            verbose = FALSE, saveAt = paste0("output/model_2"))
pa[2] <- cor(fit$yHat[tst.pop==1],y[tst.pop==1])
rmse[2] <- mean((fit$yHat[tst.pop==1]-y[tst.pop==1])^2)

end.time <- Sys.time()
(time.taken <- end.time - start.time)
```

    ## Time difference of 5.254325 secs

Evaluating with a for loop for all k's
--------------------------------------

``` r
pa <- matrix(NA,2,5) #matrix to collect the predictive ability
rmse <- matrix(NA,2,5) #matrix to collect the RMSE
time <- rep(NA,5) #vector to collect time taken

## Building models outside the loop
ETA_1 <- list(PED1=list(X=M, model="BRR",saveEffects=FALSE))
ETA_2 <- list(PED1=list(X=M, model="BL",saveEffects=FALSE))

for(k in 1:5){
  start.time <- Sys.time()
  y.trn <- y
  y.trn[tst.pop==k] <- NA
  
  fit <- BGLR(y.trn, ETA=ETA_1, 
              nIter = 100, burnIn = 50,thin = 5, 
              verbose = FALSE, saveAt = paste0("output/model_1_",k))
  pa[1,k] <- cor(fit$yHat[tst.pop==k],y[tst.pop==k])
  rmse[1,k] <- mean((fit$yHat[tst.pop==k]-y[tst.pop==k])^2)
  
  ## Bayesian Lasso
  fit <- BGLR(y.trn, ETA=ETA_2,
              nIter = 100, burnIn = 50,thin = 5, 
              verbose = FALSE, saveAt = paste0("output/model_2_",k))
  pa[2,k] <- cor(fit$yHat[tst.pop==k],y[tst.pop==k])
  rmse[2,k] <- mean((fit$yHat[tst.pop==k]-y[tst.pop==k])^2)
  
  end.time <- Sys.time()
  time[k] <- end.time - start.time
}

print(pa)
```

    ##           [,1]      [,2]      [,3]      [,4]      [,5]
    ## [1,] 0.3401508 0.3769742 0.2584141 0.1628996 0.1822812
    ## [2,] 0.3392227 0.3157449 0.1920118 0.2049281 0.1780244

``` r
print(rmse)
```

    ##          [,1]     [,2]     [,3]     [,4]     [,5]
    ## [1,] 142.5198 174.0918 156.8485 165.2986 138.5694
    ## [2,] 143.1469 176.6930 171.2460 166.5368 146.3603

``` r
print(time)
```

    ## [1] 3.341892 3.327272 3.142947 3.033428 3.065488

``` r
sum(time)
```

    ## [1] 15.91103

Evaluating with a lapply loop for all k's
-----------------------------------------

``` r
## Building models outside the loop
ETA_1 <- list(PED1=list(X=M, model="BRR",saveEffects=FALSE))
ETA_2 <- list(PED1=list(X=M, model="BL",saveEffects=FALSE))

## Index data.frame to loop
index <- expand.grid(k=1:5,ETA=1:2)
x <- 1:nrow(index)

## Creating a function to put in the loop
fit.BGLR <- function(x){
  ## Collecting Loop Info
  k=index$k[x]
  ind.eta = index$ETA[x]  

  ## Which ETA
  if(index$ETA[x]==1){
    ETA=ETA_1
  }else{
    ETA=ETA_2
  }
  
  ## Which k-fold
  y.trn <- y
  y.trn[tst.pop==k] <- NA
  
  fit <- BGLR(y.trn, ETA=ETA,
              nIter = 100, burnIn = 50,thin = 5, 
              verbose = FALSE, saveAt = paste0("output/model_",ind.eta,"_",k))
  pa <- cor(fit$yHat[tst.pop==k],y[tst.pop==k])
  rmse <- mean((fit$yHat[tst.pop==k]-y[tst.pop==k])^2)
  
  return(data.frame(k,ind.eta,pa,rmse))
}

## With lapply function
system.time(
  output <- lapply(x,fit.BGLR)
)
```

    ##    user  system elapsed 
    ##  15.688   0.320  16.056

``` r
## Basically the same time as the for loop
```

Evaluating with a parLapply loop for all k's
--------------------------------------------

Differences to `lapply`: you need to call the packages within the function, export all the pertinent objects for the nodes, the output is printed.

``` r
## Building models outside the loop
ETA_1 <- list(PED1=list(X=M, model="BRR",saveEffects=FALSE))
ETA_2 <- list(PED1=list(X=M, model="BL",saveEffects=FALSE))

## Index data.frame to loop
index <- expand.grid(k=1:5,ETA=1:2)
x <- 1:nrow(index)

## Creating a function to put in the loop
fit.BGLR <- function(x){
  
  ## You need to call the library inside the function
  library(BGLR)
  
  ## Collecting Loop Info
  k=index$k[x]
  ind.eta = index$ETA[x]  

  ## Which ETA
  if(index$ETA[x]==1){
    ETA=ETA_1
  }else{
    ETA=ETA_2
  }
  
  ## Which k-fold
  y.trn <- y
  y.trn[tst.pop==k] <- NA
  
  fit <- BGLR(y.trn, ETA=ETA,
              nIter = 100, burnIn = 50,thin = 5, 
              verbose = FALSE, saveAt = paste0("output/model_",ind.eta,"_",k))
  pa <- cor(fit$yHat[tst.pop==k],y[tst.pop==k])
  rmse <- mean((fit$yHat[tst.pop==k]-y[tst.pop==k])^2)
  
  print(data.frame(k,ind.eta,pa,rmse))
}

## With lapply function
library(parallel)

## 2 cores
no_cores <- 2
cl <- makeCluster(no_cores)
clusterExport(cl=cl,varlist = list("y","index","ETA_1","ETA_2","tst.pop"))
system.time(
  out <- parLapply(cl, x, fit.BGLR)
)
```

    ##    user  system elapsed 
    ##   0.005   0.000  11.456

``` r
stopCluster(cl)
```

If you want to save as the parallel works occur, add this line before the end of the loop (only Linux/MAC):

``` r
system(paste0("echo '",paste(k,ind.eta,pa,rmse),"' >> output.txt"))
```

It is a good procedure to always let one CPU free, to do it automatically in the Slurm environment:

``` r
no_cores <- as.integer(Sys.getenv("SLURM_CPUS_ON_NODE"))-1
```

Slurm file example in the HPC (with HPG limits)
-----------------------------------------------

``` bash
#!/bin/bash
#SBATCH --job-name=MyJob    # Job name
#SBATCH --mail-type=END,FAIL          # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=myemail@domain.com     # Where to send mail   
#SBATCH --nodes=1                     # Run on a single node
#SBATCH --ntasks=1                    # Run on a single task (single R file)
#SBATCH --cpus-per-task=10             # number of threads per node (MAX=32)
#SBATCH --mem=128gb                     # Job memory request per node (MAX=128gb)
#SBATCH --time=96:00:00               # Time limit hrs:min:sec (MAX=96h)
#SBATCH --output=%j.log   # Standard output and error log
#SBATCH --account=user
#SBATCH --qos=user-b

date
 
module load R/3.5.1
 
Rscript MyScript.R

date
```

An way to handle unpredicted errors in the loop with try() and to not crash the loop
------------------------------------------------------------------------------------

Here, `ETA_3` is a wrong model and it results an error in the `BGLR()` function. Then, in this case, the `try()` function will collect an `try-error` object. In the following function `fit.BGLR()`, we included an `try()` encapsulating `BGLR()` function. If it is an error, return just `NA` values for our output, if not, keep going and compute the statistics.

``` r
## Building models outside the loop
ETA_1 <- list(PED1=list(X=M, model="BRR",saveEffects=FALSE))
ETA_2 <- list(PED1=list(X=M, model="BL",saveEffects=FALSE))

## Creating ETA_3 as an errorenous model
fit <- NULL
ETA_3 <- "error"
fit <- BGLR(y.trn, ETA=ETA_3,
            nIter = 100, burnIn = 50,thin = 5, 
            verbose = FALSE, saveAt = paste0("output/model_3"))
fit <- NULL
fit <- try(BGLR(y.trn, ETA=ETA_3,
            nIter = 100, burnIn = 50,thin = 5, 
            verbose = FALSE, saveAt = paste0("output/model_3")))
class(fit)

## Index data.frame to loop
index <- expand.grid(k=1:5,ETA=1:3)
x <- 1:nrow(index)

## Handling it with try()
fit <- try(BGLR(y.trn, ETA=ETA_3,
       nIter = 100, burnIn = 50,thin = 5, 
       verbose = FALSE, saveAt = paste0("output/model_3")),silent = TRUE
)

class(fit)

## Index data.frame to loop
index <- expand.grid(k=1:5,ETA=1:3)
x <- 1:nrow(index)

## Creating a function to put in the loop
fit.BGLR <- function(x){
  
  ## You need to call the library inside the function
  library(BGLR)
  
  ## Collecting Loop Info
  k=index$k[x]
  ind.eta = index$ETA[x]
  pa=NA
  rmse=NA

  ## Which ETA
  if(index$ETA[x]==1)
    ETA=ETA_1
  if(index$ETA[x]==2)
    ETA=ETA_2
  if(index$ETA[x]==3)
    ETA=ETA_3
  
  ## Which k-fold
  y.trn <- y
  y.trn[tst.pop==k] <- NA
  
  fit <- try(BGLR(y.trn, ETA=ETA,
              nIter = 100, burnIn = 50,thin = 5, 
              verbose = FALSE, saveAt = paste0("output/model_",ind.eta,"_",k)))
  if(class(fit)!="BGLR"){
    print(data.frame(k,ind.eta,pa,rmse))
  }else{
    pa <- cor(fit$yHat[tst.pop==k],y[tst.pop==k])
    rmse <- mean((fit$yHat[tst.pop==k]-y[tst.pop==k])^2)
    print(data.frame(k,ind.eta,pa,rmse))
  }
}

## With lapply function
library(parallel)
## 2 cores
no_cores <- 2
cl <- makeCluster(no_cores)
clusterExport(cl=cl,varlist = list("y","index","ETA_1","ETA_2","ETA_3","tst.pop"))
system.time(
   out <- parLapply(cl, x, fit.BGLR)
)
stopCluster(cl)
print(out)
```

RAM Memory Tips
---------------

-   R is really bad at RAM Memory management!
-   In each loop always try to load the low possible ammount of data to save RAM memory
-   In R environment, always prefer `RData` format for outputs and inputs
-   In the our example if a big `M` is used with several models, one option is to save the each model in a `RData` file and load the correct one with `load(past0(...))` in each loop. This save memmory and you can use more CPUs in the process
-   When setting your parallel process, debug the code! Check if everything is correct. In our example, a good procedure is first run it with a partial dataset and a low number of iteractions to check if everything is correct. It is really frustrating to send a process, wait days, and realize that bugs in the code

Real example folder for HPG
----------------------------
[clone this](https://github.com/rramadeu/Tutorials_File/tree/master/WorkshopRParallel)
