---
syncID: 5f9c4048a27749c19ee8ecfc78806363
title: "Download and Explore NEON Data"
description: "Tutorial for downloading data from the Data Portal and the neonUtilities package, then exploring and understanding the downloaded data"
dateCreated:  2018-11-07
authors: [Claire K. Lunch]
contributors: [Christine Laney, Megan A. Jones]
estimatedTime: 1 - 2 hours
packagesLibraries: [devtools, geoNEON, neonUtilities, rhdf5, raster]
topics: data-management, rep-sci
languagesTool: R, API
dataProduct:
code1: R/download-explore/NEON-download-explore
tutorialSeries: 
urlTitle: download-explore-neon-data
---


This tutorial covers downloading NEON data, using the Data Portal and 
the neonUtilities R package, as well as basic instruction in beginning to 
explore and work with the downloaded data, including guidance in 
navigating data documentation.

## NEON data
There are 3 basic categories of NEON data:

1. Remote sensing (AOP) - Data collected by the airborne observation platform, 
e.g. LIDAR, surface reflectance
1. Observational (OS) - Data collected by a human in the field, or in an 
analytical laboratory, e.g. beetle identification, foliar isotopes
1. Instrumentation (IS) - Data collected by an automated, streaming sensor, e.g. 
net radiation, soil carbon dioxide. This category also includes the eddy 
covariance (EC) data, which are processed and structured in a unique way, distinct 
from other instrumentation data.

This lesson covers all four (sub)types of data. The download procedures are 
similar for all types, but data navigation differs significantly by type.


<div id="ds-objectives" markdown="1">

## Objectives

After completing this activity, you will be able to:

* Download NEON data using the neonUtilities package.
* Understand downloaded data sets and load them into R for analyses.

## Things You’ll Need To Complete This Tutorial
To complete this tutorial you will need the most current version of R and, 
preferably, RStudio loaded on your computer.

### Install R Packages

* **devtools**: Needed to install packages from GitHub
* **neonUtilities**: Basic functions for accessing NEON data
* **raster**: Raster package; needed for remote sensing data
* **geoNEON**: For working with NEON spatial data
* **rhdf5**: HDF5 package; needed for eddy covariance data

Some of these packages are on CRAN and can be installed by 
`install.packages()`, others need to be installed from 
other repositories:


    install.packages("devtools")
    install.packages("neonUtilities")
    install.packages("raster")
    devtools::install_github("NEONScience/NEON-geolocation/geoNEON")
    install.packages("BiocManager")
    BiocManager::install("rhdf5")


### Additional Resources

* <a href="https://www.neonscience.org/neonDataStackR" target="_blank">Tutorial for neonUtilities.</a> Some overlap with this tutorial but goes into more detail about the neonUtilities package.
* <a href="https://www.neonscience.org/neon-utilities-python" target="_blank">Tutorial for using neonUtilities from a Python environment.</a>
* <a href="https://github.com/NEONScience/NEON-Utilities/neonUtilities" target="_blank">GitHub repository for neonUtilities</a>
* <a href="https://github.com/NEONScience/NEON-geolocation/geoNEON" target="_blank">GitHub repository for geoNEON</a>

</div>

## Getting started: Download data from the Portal and load packages

Go to the 
<a href="http://data.neonscience.org" target="_blank">NEON Data Portal</a> 
and download some data! Almost any IS or OS data product can be used for this 
section of the tutorial, but we will proceed assuming you've downloaded 
Photosynthetically Active Radiation (PAR) (DP1.00024.001) data. For optimal 
results, download three months of data from two sites. The downloaded file 
should be a zip file named NEON_par.zip

Now switch over to R and load all the packages installed above.


    # load packages
    library(neonUtilities)
    library(geoNEON)
    library(raster)
    library(rhdf5)
    
    # Set global option to NOT convert all character variables to factors
    options(stringsAsFactors=F)


## Stack the downloaded data files: stackByTable()

The `stackByTable()` function will unzip and join the files in the 
downloaded zip file.


    # Modify the file path to match the path to your zip file
    stackByTable("~/Downloads/NEON_par.zip")

In the same directory as the zipped file, you should now have an unzipped 
folder of the same name. When you open this you will see a new folder 
called **stackedFiles**, which should contain three files: 
**PARPAR_30min.csv**, **PARPAR_1min.csv**, and **variables.csv**.

We'll look at these files in more detail below.

## Download data using neonUtilities: zipsByProduct()

The function `zipsByProduct()` is a wrapper for the NEON API, it 
downloads zip files for the data product specified and stores them in 
a format that can then be passed on to `stackByTable()`.

One of the inputs to `zipsByProduct()` is the data product ID, or 
DPID, of the data you want to download. The DPID can be found in the 
<a href="http://data.neonscience.org/data-product-catalog" target="_blank">data product catalog</a>. 
It will be in the form DP#.#####.###; e.g., the DPID of PAR, downloaded  
above, is DP1.00024.001.

Input options for `zipsByProduct()` are:

* `dpID`: the data product ID, e.g. DP1.00002.001
* `site`: either the 4-letter code of a single site, e.g. HARV, or "all", 
indicating all sites with data available
* `package`: either basic or expanded data package
* `avg`: either "all", to download all data (the default), or the 
number of minutes in the averaging interval. See example below; 
only applicable to IS data
* `savepath`: the file path you want to download to; defaults to the 
working directory
* `check.size`: T or F: should the function pause before downloading 
data and warn you about the size of your download? Defaults to T; if 
you are using this function within a script or batch process you 
will want to set it to F.

Here, we'll download woody vegetation structure data from 
Wind River Experimental Forest (WREF).


    zipsByProduct(dpID="DP1.10098.001", site="WREF", 
                  package="expanded", check.size=T,
                  savepath="~/Downloads")

In the file location for your download, you should now have a 
folder named filesToStack10098. Use `stackByTable()` to stack 
the files in this folder, as above, with the additional 
input `folder=T`:


    stackByTable(filepath="~/Downloads/filesToStack10098", 
                 folder=T)

The **filesToStack10098** folder should now contain a **stackedFiles** 
folder with six files:

* vst_shrubgroup.csv
* vst_perplotperyear.csv
* vst_mappingandtagging.csv
* vst_apparentindividual.csv
* variables.csv
* validation.csv

We'll look at these files in more detail below.

## Download remote sensing data: byFileAOP() and byTileAOP()

Remote sensing data files are very large, so downloading them 
can take a long time. `byFileAOP()` and `byTileAOP()` enable 
easier programmatic downloads, but be aware it can take a very 
long time to download large amounts of data.

Input options for the AOP functions are:

* `dpID`: the data product ID, e.g. DP1.00002.001
* `site`: the 4-letter code of a single site, e.g. HARV
* `year`: the 4-digit year
* `savepath`: the file path you want to download to; defaults to the 
working directory
* `check.size`: T or F: should the function pause before downloading 
data and warn you about the size of your download? Defaults to T; if 
you are using this function within a script or batch process you 
will want to set it to F.
* `easting`: `byTileAOP()` only. Vector of easting UTM coordinates whose 
corresponding tiles you want to download
* `northing`: `byTileAOP()` only. Vector of northing UTM coordinates 
whose corresponding tiles you want to download
* `buffer`: `byTileAOP()` only. Size in meters of buffer to include 
around coordinates when deciding which tiles to download

Here, we'll download one tile of Ecosystem structure (Canopy Height 
Model) (DP3.30015.001) from WREF in 2017.


    byTileAOP("DP3.30015.001", site="WREF", year="2017",
              easting=580000, northing=5075000, savepath="~/Downloads")

In the directory indicated in `savepath`, you should now have a folder 
named `DP3.30015.001` with several nested subfolders, leading to a tif 
file of a canopy height model tile. We'll look at this in more detail 
below.

## Navigate data downloads: IS

Let's take a look at the PAR data we downloaded earlier. Start by 
reading in the 30-minute file:


    par30 <- read.delim("~/Downloads/NEON_par/stackedFiles/PARPAR_30min.csv", 
                        sep=",")
    View(par30)

The first four columns are added by `stackByTable()` when it merges 
files across sites, months, and tower heights. The remaining columns 
are described by the variables file:


    parvar <- read.delim("~/Downloads/NEON_par/stackedFiles/variables.csv", 
                        sep=",")
    View(parvar)

The variables file shows you the definition and units for each column 
of data.

Now that we know what we're looking at, let's convert the time stamp 
to a format R understands and then plot PAR from the top tower level:


    par30$startDateTime <- as.POSIXct(par30$startDateTime, 
                                      format="%Y-%m-%d T %H:%M:%S Z", 
                                      tz="GMT")
    
    plot(PARMean~startDateTime, 
         data=par30[which(par30$verticalPosition==80),],
         type="l")

Looks good! The sun comes up and goes down every day, and some days 
are cloudy. If you want to dig in a little deeper, try plotting PAR 
from lower tower levels on the same axes to see light attenuation 
through the canopy.

## Navigate data downloads: OS

Let's take a look at the vegetation structure data. OS data products 
are simple in that the data generally tabular, and data volumes are 
lower than the other NEON data types, but they are complex in that 
almost all consist of multiple tables containing information collected 
at different times in different ways. Complexity in working with OS 
data involves bringing those data together.

We'll read in the vst_mappingandtagging and vst_apparentindividual 
files:


    vegmap <- read.delim("~/Downloads/filesToStack10098/stackedFiles/vst_mappingandtagging.csv",
                         sep=",")
    View(vegmap)
    vegind <- read.delim("~/Downloads/filesToStack10098/stackedFiles/vst_apparentindividual.csv",
                         sep=",")
    View(vegind)

As with the IS data, the variables file can tell you more about 
the data. OS data also come with a validation file, which contains 
information about the validation and controlled data entry that 
were applied to the data:


    vstvar <- read.delim("~/filesToStack10098/stackedFiles/variables.csv", 
                        sep=",")
    View(vstvar)
    
    vstval <- read.delim("~/filesToStack10098/stackedFiles/validation.csv", 
                        sep=",")
    View(vstval)

OS data products each come with a Data Product User Guide, 
which can be downloaded with the data or accessed from the 
document library on the Data Portal. Here, we'll use 
information that can be found in the User Guide about 
how to (1) calculate stem locations for each tree and (2) how 
to join the mapping and individual data.

First, use the `geoNEON` package to calculate stem locations:


    names(vegmap)
    vegmap <- geoNEON::def.calc.geo.os(vegmap, "vst_mappingandtagging")
    names(vegmap)

And now merge the mapping data with the individual measurements. 
`individualID` is the linking variable, the others are included 
to avoid having duplicate columns.


    veg <- merge(vegind, vegmap, by=c("individualID","namedLocation",
                                      "domainID","siteID","plotID"))

Using the merged data, now we can map the stems in plot 85 
(plot chosen at random). Note that the coordinates are in 
meters but stem diameters are in cm.


    symbols(veg$adjEasting[which(veg$plotID=="WREF_085")], 
            veg$adjNorthing[which(veg$plotID=="WREF_085")], 
            circles=veg$stemDiameter[which(veg$plotID=="WREF_085")]/100, 
            xlab="Easting", ylab="Northing", inches=F)

## Navigate data downloads: AOP

To work with AOP data, the best bet is the `raster` package. 
It has functionality for most analyses you might want to do.

We'll use it to read in the tile we downloaded:


    chm <- raster("~/Downloads/DP3.30015.001/2017/FullSite/D16/2017_WREF_1/L3/DiscreteLidar/CanopyHeightModelGtif/NEON_D16_WREF_DP3_580000_5075000_CHM.tif")

The `raster` package includes plotting functions:


    plot(chm, col=topo.colors(6))



