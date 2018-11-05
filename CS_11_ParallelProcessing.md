---
title: "Parallel Computing with R"
subtitle: Write a parallel for loop
week: 11
type: Case Study
reading:
   - Parallel [Computing with the R Language in a Supercomputing Environment](https://link.springer.com/chapter/10.1007/978-3-642-13872-0_64)
   - CRAN Task View [High Performance and Parallel Computing with R](http://cran.r-project.org/web/views/HighPerformanceComputing.html)
tasks:
   - Reproject `world` dataset to a global equal area projection
   - Write a parallel `foreach()` loop to identify the a spatial relationships of each country
   - Set the output of the `foreach()` funtion to return a simple matrix
   - Confirm that your parallel loop returns the same answer as a typical "sequential" approach
---



# Reading

- Parallel [Computing with the R Language in a Supercomputing Environment](https://link.springer.com/chapter/10.1007/978-3-642-13872-0_64)
- CRAN Task View [High Performance and Parallel Computing with R](http://cran.r-project.org/web/views/HighPerformanceComputing.html)


# Tasks

- Write parallel for loops to process spatial data

## Background


```r
library(tidyverse)
library(spData)
library(sf)

## New Packages
library(foreach)
library(doParallel)
registerDoParallel(2)
getDoParWorkers() # check registered cores

#define working projection (EASE-Grid, https://nsidc.org/data/ease)
proj="+proj=cea +lon_0=0 +lat_ts=30 +x_0=0 +y_0=0 +ellps=WGS84 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs"
```

<div class="well">
<button data-toggle="collapse" class="btn btn-primary btn-sm round" data-target="#demo1">Show Hints</button>
<div id="demo1" class="collapse">

## Steps

Write an Rmd script that:

* Loads the `world` dataset in the `spData` package
   * Reproject the `world` dataset to the Equal-Area Scalable Earth Grid (EASE-Grid) ([EASE-Grid ](https://nsidc.org/data/ease)) using `st_transform()` and the proj4 projection in the code above
* Runs a parallel `foreach()` to loop over countries (`name_long`) that:
   * `filter` the world object to include only on country at a time.
   * use `st_is_within_distance` to find the distance from that country to all other countries in the `world` object within 100000m Set `sparse=F` to return a simple vector of `TRUE/FALSE` for countries within the distance.
   * set `.combine=rbind` to return a simple matrix.
* Confirm that you get the same answer without using foreach:
   * simply use `st_is_within_distance` with the transformed `world` object as both `x` and `y` object.
   * compare the results with `identical()`
   * if you are curious, you can also check the time difference with `system.time()`.
   
   
</div>
</div>



The first 10 rows/columns of the resulting matrix (e.g. `x_par[1:10,1:10]`) should look like this:

1       2       3       4       5       6       7       8       9       10    
------  ------  ------  ------  ------  ------  ------  ------  ------  ------
TRUE    FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE 
FALSE   TRUE    FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE 
FALSE   FALSE   TRUE    FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE 
FALSE   FALSE   FALSE   TRUE    TRUE    FALSE   FALSE   FALSE   FALSE   FALSE 
FALSE   FALSE   FALSE   TRUE    TRUE    FALSE   FALSE   FALSE   FALSE   FALSE 
FALSE   FALSE   FALSE   FALSE   FALSE   TRUE    TRUE    FALSE   FALSE   FALSE 
FALSE   FALSE   FALSE   FALSE   FALSE   TRUE    TRUE    FALSE   FALSE   FALSE 
FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   TRUE    TRUE    FALSE 
FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   TRUE    TRUE    FALSE 
FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   FALSE   TRUE  

Note that in this example the sequential version typically runs faster than the parallel version due to the relatively small size of the dataset and computation needed.


---

<div class="extraswell">
<button data-toggle="collapse" class="btn btn-link" data-target="#extras">
Extra time? Try this...
</button>
<div id="extras" class="collapse">

This approach could be used to identify which countries were 'close' to others.  For example, Identify which countries are within 10^{5}m of Costa Rica:


|Nearbye Countries |
|:-----------------|
|Panama            |
|Costa Rica        |
|Nicaragua         |

And plot them:

![](CS_11_ParallelProcessing_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

</div>
</div>
