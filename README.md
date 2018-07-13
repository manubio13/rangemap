rangemap vignette
================

-   [Package description](#package-description)
-   [Installing the package](#installing-the-package)
-   [Using the package functions](#using-the-package-functions)
    -   [Setting R up](#setting-r-up)
    -   [Simple graphical exploration of your data.](#simple-graphical-exploration-of-your-data.)
    -   [Species ranges from buffered occurrences](#species-ranges-from-buffered-occurrences)
    -   [Species ranges from boundaries](#species-ranges-from-boundaries)
        -   [Using only occurrences](#using-only-occurrences)
        -   [Using only administrative area names](#using-only-administrative-area-names)
        -   [Using occurrences and administrative areas](#using-occurrences-and-administrative-areas)
    -   [Species ranges from hull polygons](#species-ranges-from-hull-polygons)
        -   [Convex hulls](#convex-hulls)
        -   [Concave hulls](#concave-hulls)
    -   [Species ranges from ecological niche models](#species-ranges-from-ecological-niche-models)
    -   [Species ranges using trend surface analyses](#species-ranges-using-trend-surface-analyses)
    -   [Nice fugures of species ranges](#nice-fugures-of-species-ranges)
        -   [Including the extent of occurrence](#including-the-extent-of-occurrence)
        -   [Including the occurrences](#including-the-occurrences)
        -   [Including the extent of occurrence ond species recods](#including-the-extent-of-occurrence-ond-species-recods)
        -   [Including other components](#including-other-components)
        -   [Saving the figure](#saving-the-figure)

<br>

### Package description

The **rangemap** R package presents various tools to create species range maps based on occurrence data, statistics, and distinct shapefiles. Other tools of this package can be used to analyze environmental characteristics of the species ranges and to create high quality figures of these maps.

<br>

### Installing the package

**rangemap** is in a GitHub repository and can be installed and/or loaded using the following code (make sure to have Internet connection).

``` r
# Installing and loading packages
if(!require(devtools)){
    install.packages("devtools")
}

if(!require(rangemap)){
    devtools::install_github("marlonecobos/rangemap")
}
library(rangemap)
```

<br>

### Using the package functions

#### Setting R up

The following code chunk installs (if needed) and loads the R packages that will be used to perform the example analyses with the **rangemap** package. The working directory will also be defined in this part.

``` r
# pacakges from CRAN
pcakages <- c("rgbif", "maps", "maptools", "raster")
req_packages <- pcakages[!(pcakages %in% installed.packages()[, "Package"])]
if (length(req_packages) > 0) {
  install.packages(req_packages, dependencies = TRUE)
}
sapply(pcakages, require, character.only = TRUE)

# package from github
if(!require(kuenm)){
install_github("marlonecobos/kuenm")
}
library(kuenm)
    
# working directory
setwd("YOUR/WORKING/DIRECTORY")
```

<br>

#### Simple graphical exploration of your data.

The *rangemap\_explore* function generates simple figures to visualize species occurrence data in the geographic space before using other functions of this package. The figure created with this function helps to identify countries involved in the species distribution. Other aspects of the species distribution can also be generally checked here; for instace, disjunt distributions, dimension of the species range, etc.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_explore)
```

An example of the use of this function is written below.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
             c("name", "decimalLongitude", "decimalLatitude")]

# simple figure of the species occurrence data
explore_map <- rangemap_explore(occurrences = occ_g)
```

<br>

#### Species ranges from buffered occurrences

The *rangemap\_buff* function generates a species range for a given species by buffering provided occurrences using a user-defined distance. An approach to the species extent of occurrence (using convex hulls) and the area of occupancy according to the IUCN criteria are also generated. Shape files (occurrences, species range, extent of occurrence, and area of occupancy) can be saved in the working directory if it is needed.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_buff)
```

An example of the use of this function is written below.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Peltophryne empusa",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[1], georeferenced = TRUE) # testing if keys return records

key <- species$key[1] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
            c("name", "decimalLongitude", "decimalLatitude")]

# buffer distance
dist <- 100000
save <- TRUE
name <- "test"

buff_range <- rangemap_buff(occurrences = occ_g, buffer_distance = dist,
                            save_shp = save, name = name)
```

The function *rangemap\_fig* generates customizable figures of species range maps using the objects produced by other function of this package. Let's see hoy the generated range looks like.

<br>

#### Species ranges from boundaries

The *rangemap\_bound* function generates a species range polygon for a given species by considering all the polygons of administrative entities in which the species has been detected. An approach to the species extent of occurrence (using convex hulls) and the area of occupancy according to the IUCN criteria are also generated. Shape files can also be saved in the working directory if needed.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_bound)
```

Examples of the use of this function with most of its variants are written below.

<br>

##### Using only occurrences

Following there is an example in wich administrative areas will be selected using only occurrences. The *rangemap\_explore* function will be used for obtainig a first visualization of the species distributional range.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
            c("name", "decimalLongitude", "decimalLatitude")]

# checking which countries may be involved in the analysis
rangemap_explore(occurrences = occ_g)

level <- 0
dissolve <- FALSE
save <- TRUE
name <- "test1"
countries <- c("PER", "BRA", "COL", "VEN", "ECU", "GUF", "GUY", "SUR", "BOL")

bound_range <- rangemap_bound(occurrences = occ_g, country_code = countries, boundary_level = level, 
                              dissolve = dissolve, save_shp = save, name = name)
```

<br>

##### Using only administrative area names

Following there is an example in wich administrative areas will be selected using only the names of the administrative entities known to be occupied by the species. This approach may be useful in circumstances where goegraphic coordinates or aqurate localitie descriptions do not exist.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
            c("name", "decimalLongitude", "decimalLatitude")]

# checking which countries may be involved in the analysis
rangemap_explore(occurrences = occ_g)

data("country_codes") #list of country names and ISO codes
View(country_codes)

level <- 0
adm <- c("Ecuador", "Peru", "Venezuela", "Colombia", "Brazil") # If we only know the countries in wich the species is
dissolve <- FALSE
save <- TRUE
name <- "test2"
countries <- c("PER", "BRA", "COL", "VEN", "ECU", "GUF", "GUY", "SUR", "BOL")

bound_range <- rangemap_bound(adm_areas = adm, country_code = countries, boundary_level = level,
                              dissolve = dissolve, save_shp = save, name = name)
```

<br>

##### Using occurrences and administrative areas

An example of using both occurrences and administrative areas for creating species ranges with the function *rangemap\_bound* is presented below. This option may be useful when these two types of infromation complement the knowledge of the species distribution.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
            c("name", "decimalLongitude", "decimalLatitude")]

# checking which countries may be involved in the analysis
rangemap_explore(occurrences = occ_g)

level <- 0
adm <- "Ecuador" # Athough no record is on this country, we know it is in Ecuador
dissolve <- FALSE
save <- TRUE
name <- "test3"
countries <- c("PER", "BRA", "COL", "VEN", "ECU", "GUF", "GUY", "SUR", "BOL")

bound_range <- rangemap_bound(occurrences = occ_g, adm_areas = adm, country_code = countries,
                              boundary_level = level, dissolve = dissolve, save_shp = save, name = name)
```

<br>

#### Species ranges from hull polygons

The *rangemap\_hull* function generates a species range polygon for a given species by creating convex or concave hull polygons based on occurrence data. An approach to the species extent of occurrence (using convex hulls) and the area of occupancy according to the IUCN criteria are also generated. Shape files can be saved if needed.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_hull)
```

Examples of the use of this function with most of its variants are written below.

<br>

##### Convex hulls

With the example provided below, a species range will be constructed using convex hulls. This range will be split based on two distinct algorithms of clustering: hierarchical and k-means. Convex hull polygons are commonly used to represent species ranges, however in circunstances where biogeographic barriers for the species dispersal exist, concave hulls may be a better option.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data", limit = 2000) # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
             c("name", "decimalLongitude", "decimalLatitude")]

# unique polygon (non-disjunct distribution)
dist <- 100000
hull <- "convex" 
split <- FALSE
save <- TRUE
name <- "test4"

hull_range <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, save_shp = save, name = name)

# disjunct distributions
## clustering occurrences with the hierarchical method
split <- TRUE
c_method <- "hierarchical"
split_d <- 1500000
name <- "test5"

hull_range1 <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, cluster_method = c_method, split_distance = split_d,
                            save_shp = save, name = name)

## clustering occurrences with the k-means method
c_method <- "k-means"
n_clus <- 3
name <- "test6"

hull_range2 <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, cluster_method = c_method, n_k_means = n_clus,
                            save_shp = save, name = name)
```

<br>

##### Concave hulls

With the following examples, the species range will be constructed using concave hulls. The species range will be calculated as an only area and as disjunt areas by clustering its occurrences using hierarchical and k-means algorithms.

``` r
# unique polygon (non-disjunct distribution)
dist <- 100000
hull <- "concave" 
split <- FALSE
save <- TRUE
name <- "test7"

hull_range3 <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, save_shp = save, name = name)

# disjunct distributions
## clustering occurrences with the hierarchical method
split <- TRUE
c_method <- "hierarchical"
split_d <- 1500000
name <- "test8"

hull_range4 <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, cluster_method = c_method, split_distance = split_d,
                            save_shp = save, name = name)

## clustering occurrences with the k-means method
c_method <- "k-means"
n_clus <- 3
name <- "test9"

hull_range5 <- rangemap_hull(occurrences = occ_g, hull_type = hull, buffer_distance = dist,
                            split = split, cluster_method = c_method, n_k_means = n_clus,
                            save_shp = save, name = name)
```

<br>

#### Species ranges from ecological niche models

The *rangemap\_enm* function.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_buff)
```

An example of the use of this function is written below.

``` r
# parameters
data(sp_mod)
data(sp_train)
occ_sp <- data.frame("A_americanum", sp_train)
thres <- 5
save <- TRUE
name <- "test10"

enm_range <- rangemap_enm(occurrences = occ_sp, model = sp_mod,  threshold = thres,
                          save_shp = save, name = name)
```

<br>

#### Species ranges using trend surface analyses

The *rangemap\_tsa* function generates species range polygons for a given species using a trend surface analysis. An approach to the species extent of occurrence (using convex hulls) and the area of occupancy according to the IUCN criteria are also generated. Shape files can be saved in the working directory if it is needed.

Trend surface analysis is a method based on low-order polynomials of spatial coordinates for estimating a regular grid of points from scattered observations. This method assumes that all cells not occupied by occurrences are absences; hence its use depends on the quality of data and the certainty of having or not a complete sampling of the regiong\_of\_interest.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_tsa)
```

An example of the use of this function is written below.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Peltophryne fustiger",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[5], georeferenced = TRUE) # testing if keys return records

key <- species$key[5] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
             c("name", "decimalLongitude", "decimalLatitude")]

# region of interest
WGS84 <- CRS("+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0")
w_map <- map(database = "world", regions = "Cuba", fill = TRUE, plot = FALSE) # map of the world
w_po <- sapply(strsplit(w_map$names, ":"), function(x) x[1]) # preparing data to create polygon
reg <- map2SpatialPolygons(w_map, IDs = w_po, proj4string = WGS84) # map to polygon

# other data
res <- 1
thr <- 0
save <- TRUE
name <- "test11"

tsa <- rangemap_tsa(occurrences = occ_g, region_of_interest = reg, threshold = thr,
                    resolution = res, save_shp = save, name = name)
```

<br>

#### Nice fugures of species ranges

The *rangemap\_fig* function can be used to plot not only the generated species ranges but also the extent of occurrence and the species records in the same map.

The function's help can be consulted usign the following line of code:

``` r
help(rangemap_fig)
```

Examples of the use of this function are written below.

##### Including the extent of occurrence

``` r
# arguments for the species range figure
extent <- TRUE

# creating the species range figure
range_map <- rangemap_fig(hull_range5, add_extent = extent)

dev.off() # for returning to default par settings
```

<br>

##### Including the occurrences

``` r
# arguments for the species range figure
occ <- TRUE

# creating the species range figure
range_map <- rangemap_fig(hull_range5, add_occurrences = occ)

dev.off() # for returning to default par settings
```

<br>

##### Including the extent of occurrence ond species recods

``` r
# arguments for the species range figure
extent <- TRUE
occ <- TRUE

# creating the species range figure
range_map <- rangemap_fig(hull_range5, add_extent = extent, add_occurrences = occ)

dev.off() # for returning to default par settings
```

<br>

##### Including other components

``` r
# arguments for the species range figure
extent <- TRUE
occ <- TRUE
grid <- TRUE # grid
leggend <- TRUE # leggend of objects included
scale <- TRUE # scale bar
north <- TRUE # north arrow


# creating the species range figure
range_map <- rangemap_fig(hull_range5, add_extent = extent, add_occurrences = occ,
                          grid = grid, sides = sides)

dev.off() # for returning to default par settings
```

##### Saving the figure

``` r
# arguments for the species range figure
extent <- TRUE
occ <- TRUE
grid <- TRUE # grid
leggend <- TRUE # leggend of objects included
scale <- TRUE # scale bar
north <- TRUE # north arrow
save <-  TRUE


# creating the species range figure
range_map <- rangemap_fig(hull_range5, add_extent = extent, add_occurrences = occ,
                          grid = grid, sides = sides, save_fig = save)

dev.off() # for returning to default par settings

#### Species ranges in the environmental space

The *ranges_envcomp* function generates a three dimensional comparison of a species' ranges created using distinct algortihms, to visualize implications of selecting one of them if environmental conditions are considered. 

The function's help can be consulted usign the following line of code:
```

``` r
help(ranges_envcomp)
```

An example of the use of this function is written below.

``` r
# getting the data from GBIF
species <- name_lookup(query = "Dasypus kappleri",
                       rank="species", return = "data") # information about the species

occ_count(taxonKey = species$key[14], georeferenced = TRUE) # testing if keys return records

key <- species$key[14] # using species key that return information

occ <- occ_search(taxonKey = key, return = "data") # using the taxon key

# keeping only georeferenced records
occ_g <- occ[!is.na(occ$decimalLatitude) & !is.na(occ$decimalLongitude),
             c("name", "decimalLongitude", "decimalLatitude")]


# range based on buffers
dist <- 500000

buff <- rangemap_buff(occurrences = occ_g, buffer_distance = dist)


# range based on boundaries
## checking which countries may be involved in the analysis
rangemap_explore(occurrences = occ_g)

level <- 0
adm <- "Ecuador" # Athough no record is on this country, we know it is in Ecuador

countries <- c("PER", "BRA", "COL", "VEN", "ECU", "GUF", "GUY", "SUR", "BOL")

bound <- rangemap_bound(occurrences = occ_g, adm_areas = adm, country_code = countries,
                        boundary_level = level)


# range based on concave hulls
dist1 <- 250000
hull1 <- "concave"

concave <- rangemap_hull(occurrences = occ_g, hull_type = hull1, buffer_distance = dist1)


# ranges comparison in environmental space
## list of ranges
ranges <- list(buff, bound, concave)
names(ranges) <- c("buff", "bound", "concave")

## other data for environmental comparisson
vars <- getData("worldclim", var = "bio", res = 5)

## mask variables to region of interest
WGS84 <- CRS("+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0")
w_map <- map(database = "world", regions = c("Ecuador", "Peru", "Bolivia", "Colombia", "Venezuela",
                                             "Suriname", "Guyana", "French Guyana"), 
             fill = TRUE, plot = FALSE) # map of the world
w_po <- sapply(strsplit(w_map$names, ":"), function(x) x[1]) # preparing data to create polygon
reg <- map2SpatialPolygons(w_map, IDs = w_po, proj4string = WGS84) # map to polygon

e <- extent(reg)
mask <- as(e, 'SpatialPolygons')  

variables <- crop(vars, mask)

## comparison
r_env <- ranges_envcomp(occurrences = occ_g, ranges = ranges, variables = variables, , save_fig = FALSE)
```
