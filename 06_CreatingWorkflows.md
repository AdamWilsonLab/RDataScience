---
title: "Creating Workflows"

---



[<i class="fa fa-file-code-o fa-3x" aria-hidden="true"></i> The R Script associated with this page is available here](scripts/06_CreatingWorkflows.R).  Download this file and open it (or copy-paste into a new script) with RStudio so you can follow along.  


## Libraries


```r
library(knitr)
library(dplyr)
library(tidyr)
library(ggplot2)
library(raster)
library(rasterVis)
library(scales)
library(rgeos)

# load data for this course
# devtools::install_github("adammwilson/DataScienceData")
library(DataScienceData)
```

## Today's question

### How will future (projected) sea level rise affect Bangladesh?

1. How much area is likely to be flooded by rising sea level?
2. How many people are likely to be displaced?
3. Will sea level rise affect any major population centers?

## Bangladesh


```r
getData("ISO3")%>%
  as.data.frame%>%
  filter(NAME=="Bangladesh")
```

```
##   ISO3       NAME
## 1  BGD Bangladesh
```




### Download Bangladesh Border

Often good idea to keep data in separate folder.  You will need to edit this for your machine!

```r
datadir="~/Downloads/data"
if(!file.exists(datadir)) dir.create(datadir, recursive=T)
```
Download country border.

```r
bgd=getData('GADM', country='BGD', level=0,path = datadir)
```

Or load it from the data package.

```r
data(bangladesh)
bgd=bangladesh
```


```r
bgd%>%
  gSimplify(0.01)%>%
  plot()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

## Topography

SRTM Elevation data with `getData()` as 5deg tiles.  If you have trouble downloading using `getData()`, skip to the `data(bangladesh_dem)` line below


```r
bgdc=gCentroid(bgd)%>%coordinates()

dem1=getData("SRTM",lat=bgdc[2],lon=bgdc[1],path=datadir)
```


### Mosaicing/Merging rasters

Download the remaining necessary tiles


```r
dem2=getData("SRTM",lat=23.7,lon=85,path=datadir)
```

Use `merge()` to join two aligned rasters (origin, resolution, and projection).  Or `mosaic()` combines with a function.


```r
dem=merge(dem1,dem2)
```

Or, load it from the data package.

```r
data(bangladesh_dem)
dem=bangladesh_dem # rename for convenience
```


```r
plot(dem)
bgd%>%
  gSimplify(0.01)%>%
  plot(add=T)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

## Saving/exporting rasters

Beware of massive temporary files!


```r
inMemory(dem)
```

```
## [1] TRUE
```

```r
dem@file@name
```

```
## [1] "/private/var/folders/fh/g_hk6yxx4cj5c83096lj3g4r0000gn/T/Rtmp9gEdq9/raster/r_tmp_2017-08-21_132716_46406_48324.grd"
```

```r
file.size(sub("grd","gri",dem@file@name))*1e-6
```

```
## [1] NA
```

```r
showTmpFiles()
```

```
## --- none ---
```




```r
rasterOptions()
```

```
## format        : raster 
## datatype      : FLT4S 
## overwrite     : FALSE 
## progress      : none 
## timer         : FALSE 
## chunksize     : 1e+07 
## maxmemory     : 1e+09 
## tmpdir        : /var/folders/fh/g_hk6yxx4cj5c83096lj3g4r0000gn/T//RtmphO758X/raster// 
## tmptime       : 168 
## setfileext    : TRUE 
## tolerance     : 0.1 
## standardnames : TRUE 
## warn depracat.: TRUE 
## header        : none
```
Set with `rasterOptions(tmpdir = "/tmp")`



Saving raster to file: _two options_

Save while creating

```r
dem=merge(dem1,dem2,filename=file.path(datadir,"dem.tif"),overwrite=T)
```

Or after

```r
writeRaster(dem, filename = file.path(datadir,"dem.tif"))
```



### WriteRaster formats

Filetype  Long name	                      Default extension	  Multiband support
---       ---                             ---                 ---                                                     
raster	  'Native' raster package format	.grd	              Yes
ascii	    ESRI Ascii	                    .asc                No
SAGA	    SAGA GIS	                      .sdat	              No
IDRISI	  IDRISI	                        .rst	              No
CDF	      netCDF (requires `ncdf`)	      .nc	                Yes
GTiff	    GeoTiff (requires rgdal)	      .tif	              Yes
ENVI	    ENVI .hdr Labelled	            .envi	              Yes
EHdr	    ESRI .hdr Labelled	            .bil	              Yes
HFA	      Erdas Imagine Images (.img)   	.img	              Yes

`rgdal` package does even more...

### Crop to Coastal area of Bangladesh


```r
# crop to a lat-lon box
dem=crop(dem,extent(90,91,21.5,24),filename=file.path(datadir,"dem_bgd.tif"),overwrite=T)

plot(dem)
bgd%>%
  gSimplify(0.01)%>%
  plot(add=T)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

# Use ggplot

```r
gplot(dem,max=1e5)+
  geom_tile(aes(fill=value))+
  scale_fill_gradientn(
    colours=c("red","yellow","grey30","grey20","grey10"),
    trans="log1p",breaks= log_breaks(n = 5, base = 10)(c(1, 1e3)))+
  coord_equal(ylim=c(21.5,24),xlim=c(90,91))+
  geom_path(data=fortify(bgd),
            aes(x=long,y=lat,group=group),size=.5)
```

```
## Regions defined for each Polygons
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-18-1.png)<!-- -->



# Terrain analysis (an aside)

## Terrain analysis options

`terrain()` options:

* slope
* aspect
* TPI (Topographic Position Index)
* TRI (Terrain Ruggedness Index)
* roughness
* flowdir



Use an even smaller region:

```r
reg1=crop(dem,extent(90.6,90.7,23.25,23.4))
plot(reg1)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

The terrain indices are according to Wilson et al. (2007), as in [gdaldem](http://www.gdal.org/gdaldem.html).



### Calculate slope


```r
slope=terrain(reg1,opt="slope",unit="degrees")
plot(slope)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-20-1.png)<!-- -->



### Calculate aspect


```r
aspect=terrain(reg1,opt="aspect",unit="degrees")
plot(aspect)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-21-1.png)<!-- -->



### TPI (Topographic Position Index)

Difference between the value of a cell and the mean value of its 8 surrounding cells.


```r
tpi=terrain(reg1,opt="TPI")

gplot(tpi,max=1e6)+geom_tile(aes(fill=value))+
  scale_fill_gradient2(low="blue",high="red",midpoint=0)+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-22-1.png)<!-- -->
Negative values indicate valleys, near zero flat or mid-slope, and positive ridge and hill tops



<div class="well">
## Your turn

* Identify all the pixels with a TPI less than -5 or greater than 5.
* Use `plot()` to:
    * plot elevation for this region
    * overlay the valley pixels in blue
    * overlay the ridge pixels in red

Hint: use `transparent` to plot a transparent pixel and `add=T` to add a layer to an existing plot. 

<button data-toggle="collapse" class="btn btn-primary btn-sm round" data-target="#demo1">Show Solution</button>
<div id="demo1" class="collapse">


```r
plot(reg1)
plot(tpi>5,col=c("transparent","red"),add=T,legend=F)
plot(tpi<(-5),col=c("transparent","blue"),add=T,legend=F)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-23-1.png)<!-- -->

```r
#OR (ggplot solution, sort of)
rcl=matrix(c(-Inf,-5,1,
           -5,5,2,
           5,Inf,3),byrow=T,nrow=3)
regclass=reclassify(tpi,rcl)
gplot(regclass,max=1e6)+geom_tile(aes(fill=value))+
  scale_fill_gradient2(low="blue",high="red",midpoint=2)+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-23-2.png)<!-- -->
</div>
</div>


### TRI (Terrain Ruggedness Index)

Mean of the absolute differences between the value of a cell and the value of its 8 surrounding cells.


```r
tri=terrain(reg1,opt="TRI")
plot(tri)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-24-1.png)<!-- -->



### Roughness 

Difference between the maximum and the minimum value of a cell and its 8 surrounding cells.


```r
rough=terrain(reg1,opt="roughness")
plot(rough)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-25-1.png)<!-- -->




### Hillshade (pretty...)

Compute from slope and aspect (in radians). Often used as a backdrop for another semi-transparent layer.


```r
hs=hillShade(slope*pi/180,aspect*pi/180)

plot(hs, col=grey(0:100/100), legend=FALSE)
plot(reg1, col=terrain.colors(25, alpha=0.5), add=TRUE)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-26-1.png)<!-- -->



### Flow Direction

_Flow direction_ (of water), i.e. the direction of the greatest drop in elevation (or the smallest rise if all neighbors are higher). 

Encoded as powers of 2 (0 to 7). The cell to the right of the focal cell 'x' is 1, the one below that is 2, and so on:

32	64	    128
--- ---     ---     
16	**x**	  1
8   4       2




```r
flowdir=terrain(reg1,opt="flowdir")

plot(flowdir)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-27-1.png)<!-- -->
Much more powerful hydrologic modeling in [GRASS GIS](https://grass.osgeo.org) 

# Sea Level Rise



## Global SLR Scenarios


```r
slr=data.frame(year=2100,
               scenario=c("RCP2.6","RCP4.5","RCP6.0","RCP8.5"),
               low=c(0.26,0.32,0.33,0.53),
               high=c(0.54,0.62,0.62,0.97))
slr
```

```
##   year scenario  low high
## 1 2100   RCP2.6 0.26 0.54
## 2 2100   RCP4.5 0.32 0.62
## 3 2100   RCP6.0 0.33 0.62
## 4 2100   RCP8.5 0.53 0.97
```

[IPCC AR5 WG1 Section 13-4](https://www.ipcc.ch/pdf/assessment-report/ar5/wg1/drafts/fgd/WGIAR5_WGI-12Doc2b_FinalDraft_Chapter13.pdf)

## Storm Surges

Range from 2.5-10m in Bangladesh since 1960 [Karim & Mimura, 2008](http://www.sciencedirect.com/science/article/pii/S0959378008000447).  


```r
ss=c(2.5,10)
```

## Raster area

1st Question: How much area is likely to be flooded by rising sea levels? 

WGS84 data is unprojected, must account for cell area (in km^2)...

```r
area=raster::area(dem)
plot(area)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-30-1.png)<!-- -->


<div class="well">
## Your Turn

1. How much area is likely to be flooded by rising sea levels for two scenarios:
   * 0.26m SLR and 2.5m surge (2.76 total) - call this `flood1`
   * 0.97 SLR and 10m surge (10.97 total) - call this `flood2`
   
Steps:

* Identify which pixels are below thresholds
* Multiply by cell area
* Use `cellStats()` to calculate potentially flooded areas.

<button data-toggle="collapse" class="btn btn-primary btn-sm round" data-target="#demo2">Show Solution</button>
<div id="demo2" class="collapse">
## Identify pixels below thresholds


```r
flood1=dem<=2.76
flood2=dem<=10.97

plot(flood2,col=c("transparent","darkred"))
plot(flood1,col=c("transparent","red"),add=T)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-31-1.png)<!-- -->


## Multiply by area and sum


```r
flood1_area=flood1*area
flood2_area=flood2*area

cellStats(flood1_area,sum)
```

```
## [1] 1569.09
```

```r
cellStats(flood2_area,sum)
```

```
## [1] 18250.66
```

</div>
</div>

## Reclassification

Another useful function for raster processing is `reclass()`.


```r
rcl=matrix(c(-Inf,2.76,1,
           2.76,10.97,2,
           10.97,Inf,3),byrow=T,ncol=3)
rcl
```

```
##       [,1]  [,2] [,3]
## [1,]  -Inf  2.76    1
## [2,]  2.76 10.97    2
## [3,] 10.97   Inf    3
```

```r
regclass=reclassify(dem,rcl)

gplot(regclass,max=1e5)+
  geom_tile(aes(fill=as.factor(value)))+
  scale_fill_manual(values=c("red","orange","blue"),
                    name="Flood Class")+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-33-1.png)<!-- -->


Or, do reclassification 'on the fly in the plotting function


```r
gplot(dem,max=1e5)+
  geom_tile(aes(fill=cut(value,c(-Inf,2.76,10.97,Inf))))+
  scale_fill_manual(values=c("red","orange","blue"),
                    name="Flood Class")+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-34-1.png)<!-- -->



## Socioeconomic Data

Socioeconomic Data and Applications Center (SEDAC)
[http://sedac.ciesin.columbia.edu](http://sedac.ciesin.columbia.edu)
<img src="06_assets/sedac.png" alt="alt text" width="70%">

* Population
* Pollution
* Energy
* Agriculture
* Roads



### Gridded Population of the World

Data _not_ available for direct download (e.g. `download.file()`) and are only available globally.  

<img src="06_assets/sedacData.png" alt="alt text" width="80%">

The steps to aquire the full dataset are as follows:

* Log into SEDAC with an Earth Data Account
[http://sedac.ciesin.columbia.edu](http://sedac.ciesin.columbia.edu)
* Download Population Density Grid for 2015
* Crop and mask to the country boundary for Bangladesh

The masked data are available in the DataScienceData package in the `bangladesh_pop` dataset.

### Load population data

Use `raster()` to load a raster from disk.


```r
pop_global=raster(file.path(datadir,"gpw-v4-population-density-2015/gpw-v4-population-density_2015.tif"))
```


```r
data(bangladesh_population)
```

If the data package isn't working, download directly from github.


```r
tf=tempfile()
download.file("https://github.com/adammwilson/DataScienceData/raw/master/data/bangladesh_population.rda",destfile = tf)
load(tf)
```


```r
## make a virtual copy with a shorter name for convenience

pop=bangladesh_population
```

### Explore population data


```r
gplot(pop,max=1e5)+geom_tile(aes(fill=value))+
  scale_fill_gradientn(colours=c("grey90","grey60","darkblue","blue","red"),
                       trans="log1p",breaks= log_breaks(n = 5, base = 10)(c(1, 1e5)))+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-39-1.png)<!-- -->



### Resample to DEM

Compare the resolution and origin of `pop` and `dem`.


```r
pop
```

```
## class       : RasterLayer 
## dimensions  : 707, 560, 395920  (nrow, ncol, ncell)
## resolution  : 0.008333333, 0.008333333  (x, y)
## extent      : 88.00833, 92.675, 20.74167, 26.63333  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0 
## data source : in memory
## names       : gpw.v4.population.density_2015 
## values      : 19.83075, 154258.4  (min, max)
```

```r
dem
```

```
## class       : RasterLayer 
## dimensions  : 3000, 1200, 3600000  (nrow, ncol, ncell)
## resolution  : 0.0008333333, 0.0008333333  (x, y)
## extent      : 89.99958, 90.99958, 21.49958, 23.99958  (xmin, xmax, ymin, ymax)
## coord. ref. : +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0 
## data source : /Users/adamw/Downloads/data/dem_bgd.tif 
## names       : dem_bgd 
## values      : -27, 37  (min, max)
```

```r
res(pop)
```

```
## [1] 0.008333333 0.008333333
```

```r
res(dem)
```

```
## [1] 0.0008333333 0.0008333333
```

```r
origin(pop)
```

```
## [1] -6.536993e-13  2.629008e-13
```

```r
origin(dem)
```

```
## [1] -0.000416061 -0.000416207
```

```r
# Look at average cell area in km^2 
cellStats(raster::area(pop),"mean")
```

```
## [1] 0.7828593
```

```r
cellStats(raster::area(dem),"mean")
```

```
## [1] 0.007886292
```

So to work with these rasters (population and elevation), it is easiest to "adjust" them to have the same resolution.  But there is no good way to do this.  Do you aggregate the finer raster or resample the coarser one?

Assume equal density within each grid cell and resample

```r
pop_fine=pop%>%
  resample(dem,method="bilinear")

gplot(pop_fine,max=1e5)+geom_tile(aes(fill=value))+
  scale_fill_gradientn(
    colours=c("grey90","grey60","darkblue","blue","red"),
    trans="log1p",breaks= log_breaks(n = 5, base = 10)(c(1, 1e5)))+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-41-1.png)<!-- -->


<div class="well">
## Your turn

How many people are likely to be displaced?

Steps:

* Multiply flooded area (`flood2`) **x** population density **x** area
* Summarize with `cellStats()`
* Plot a map of the number of people potentially affected by `flood2`

<button data-toggle="collapse" class="btn btn-primary btn-sm round" data-target="#demo3">Show Solution</button>
<div id="demo3" class="collapse">

For the fine resolution population data

```r
floodpop2=flood2_area*pop_fine
cellStats(floodpop2,sum)
```

```
## [1] 29796929
```


Number of potentially affected people across the region.


```r
gplot(floodpop2,max=1e6)+geom_tile(aes(fill=value))+
  scale_fill_gradientn(
    colours=c("grey90","grey60","darkblue","blue","red"),
    trans="log1p",breaks= log_breaks(n = 5, base = 10)(c(1, 1e4)))+
  coord_equal()
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-43-1.png)<!-- -->

</div>
</div>

Or resample elevation to resolution of population:
1. First aggregate to approximate spatial resolution
2. Resample to align grids perfectly


```r
res(pop)/res(dem)
```

```
## [1] 10 10
```

```r
dem_coarse=dem%>%
  aggregate(fact=10,fun=min,expand=T)%>%
  resample(pop,method="bilinear")
```

For the coarse resolution data

```r
flood_coarse=dem_coarse<=10.97
dem_coarse_area=raster::area(dem_coarse)
flood_coarse_area=flood_coarse*dem_coarse_area
floodpop_coarse=flood_coarse_area*pop
cellStats(floodpop_coarse,sum)
```

```
## [1] 40077457
```


## Raster Distances

`distance()` calculates distances for all cells that are NA to the nearest cell that is not NA.


```r
popcenter=pop>5000
popcenter=mask(popcenter,popcenter,maskvalue=0)
plot(popcenter,col="red",legend=F)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-46-1.png)<!-- -->


In meters if the RasterLayer is not projected (`+proj=longlat`) and in map units (typically also meters) when it is projected.


```r
popcenterdist=distance(popcenter)
plot(popcenterdist)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-47-1.png)<!-- -->


<div class="well">
## Your Turn

Will sea level rise affect any major population centers?

Steps:

* Resample `popcenter` to resolution of `dem` using `method=ngb`
* Identify `popcenter` areas that flood according to `flood2`.


<button data-toggle="collapse" class="btn btn-primary btn-sm round" data-target="#demo4">Show Solution</button>
<div id="demo4" class="collapse">

Will sea level rise affect any major population centers?


```r
popcenter2=raster::resample(popcenter,dem,method="ngb")

floodpop2= flood2==1 & popcenter2
floodpop2=mask(floodpop2,floodpop2,maskval=0)

plot(flood2);plot(floodpop2,add=T,col="red",legend=F);
bgd%>%
  gSimplify(0.01)%>%
  plot(add=T)
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-48-1.png)<!-- -->

</div>
</div>

## Vectorize raster


```r
vpop=rasterToPolygons(popcenter, dissolve=TRUE)

gplot(dem,max=1e5)+geom_tile(aes(fill=value))+
  scale_fill_gradientn(
    colours=c("red","yellow","grey30","grey20","grey10"),
    trans="log1p",breaks= log_breaks(n = 5, base = 10)(c(1, 1e3)))+
  coord_cartesian(ylim=c(21.5,24),xlim=c(90,91))+
  geom_path(data=fortify(bgd),aes(x=long,y=lat,group=group),size=.5)+
  geom_path(data=fortify(vpop),aes(x=long,y=lat,group=group),size=1,col="green")
```

![](06_CreatingWorkflows_files/figure-html/unnamed-chunk-49-1.png)<!-- -->
Warning: very slow on large rasters...

## 3D Visualization
Uses `rgl` library.  


```r
plot3D(dem)
decorate3d()
```

<img src="06_assets/plot3d.png" alt="alt text" width="70%">

50 different styles illustrated [here](https://cran.r-project.org/web/packages/plot3D/vignettes/volcano.pdf).



Overlay population with `drape`


```r
plot3D(dem,drape=pop, zfac=0.2)
decorate3d()
```

## Raster overview

* Perform many GIS operations
* Convenient processing and workflows
* Some functions (e.g. `distance()`) can be slow!
