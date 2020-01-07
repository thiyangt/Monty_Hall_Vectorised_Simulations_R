
<!-- README.md is generated from README.Rmd. Please edit that file    -->

<!-- Origin: https://github.com/hrbrmstr/ggalt/blob/master/README.Rmd -->

<!-- Thanks to Bob Rudis for sharing.                                 -->

# Monty Hall Problem - Vectorised Simulations

## `data.table` Simulation

``` r
set.seed(663948)
simNum <- 10000
doorNum <- 3

library(data.table)

simdt <- CJ(sim=1:simNum, door=1:doorNum)

mhdt <-  simdt[, `:=`(guess=sample(c(0, 0, 1)), true=sample(c(0, 0, 1)), switch=0), .(sim)][
    order(sim, -guess, true)][
    rowid(sim)==doorNum, switch:=1][
    , .(stayWin = sum(guess*true)/simNum, switchWin=sum(switch*true)/simNum)]

mhdt
##    stayWin switchWin
## 1:  0.3343    0.6657
```

## Tidyverse Simulation

``` r
library(tidyr)
library(dplyr)

simtv <- expand_grid(sim=1:simNum, door=1:doorNum)

mhtv <- simtv %>%
    group_by(sim) %>%
    mutate(guess=sample(c(0, 0, 1)), true=sample(c(0, 0, 1))) %>%
    arrange(sim, -guess, true) %>%
    mutate(switch = as.numeric(row_number(sim)==doorNum)) %>%
    ungroup %>%
    summarise(stayWin = sum(guess*true)/simNum,
              switchWin=sum(switch*true)/simNum)

mhtv
## # A tibble: 1 x 2
##   stayWin switchWin
##     <dbl>     <dbl>
## 1   0.336     0.664
```

## Base R Simulation

``` r
mhbrGrid <- expand.grid(door=1:doorNum, sim=1:simNum, guess=NA, true=NA, switch=0)

mhbr <- do.call(rbind, lapply(split(mhbrGrid, mhbrGrid[, "sim"]), function(x){ 
  x$guess <- sample(c(0, 0, 1))
  x$true <- sample(c(0, 0, 1))
  mh <- x[with(x, order(sim, -guess, true)), ]
  mh$switch[nrow(x)] <- 1
  mh$stayWin <- with(mh, guess*true)
  mh$switchWin <- with(mh, switch*true)
  mh
  }))

rbind(list(stayWin = sum(mhbr$stayWin)/simNum, 
           switchWin = sum(mhbr$switchWin)/simNum))
##      stayWin switchWin
## [1,] 0.3311  0.6689
```

``` r
rDoors <- function(sims, doors){
  unlist(lapply(1:sims, function(x) sample(c(0, 0, 1), replace = FALSE))[])
}

# Note that for this method to work the "sim" variable must be sorted in order and
# then the door (which is why the order matters to the expand.grid() function)
mhbrGrid <- expand.grid(door=1:doorNum, sim=1:simNum)
mhbrGrid$true <- rDoors(simNum, doorNum)
mhbrGrid$guess <- rDoors(simNum, doorNum)

mhbr <- mhbrGrid[with(mhbrGrid, order(sim, -guess, true)), ]
mhbr$switch <- rep(c(0, 0, 1), simNum)
mhbr$stayWin <- with(mhbr, guess*true)
mhbr$switchWin <- with(mhbr, switch*true)

rbind(list(stayWin = sum(mhbr$stayWin)/simNum, 
           switchWin = sum(mhbr$switchWin)/simNum))
##      stayWin switchWin
## [1,] 0.3337  0.6663
```
