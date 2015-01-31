---
title: "promisedat"
author: "David Holstius"
date: "January 30, 2015"
output: html_document
---


```r
getwd() # if clicked "Knit HTML" then this is "data-raw" (why?)
```

```
## [1] "/Users/dholstius/GitHub/promisedat/data-raw"
```


```r
library(devtools)
inst("promisedat") # if not yet installed then this is NULL
```

```
## NULL
```

```r
system.file(package = "promisedat") # if not yet installed then this is ""
```

```
## [1] ""
```

## HACK so both "Run Chunk" and "Knit HTML" will work


```r
if (file.exists("data")) {
  pkg_root <- "." 
} else if (file.exists(file.path("..", "data"))) {
  pkg_root <- ".." # we're in data-raw/
} else {
  stop("We're lost")
}

rel_path <- function (...) {
  message("rel_path(): ", file.path(pkg_root))
  file.path(pkg_root, ...)
}

rel_path("data")
```

```
## rel_path(): ..
```

```
## [1] "../data"
```

## To find `inst/` whether installed or not (yet) 


```r
# Prefers installed version but falls back to dev if not yet installed
inst_path <- function (...) {
  path <- system.file(rel_path("inst"), package = "promisedat")
  normalizePath(file.path(path, ...))
}

inst_path("extdata")
```

```
## Warning in normalizePath(file.path(path, ...)): path[1]="/extdata": No
## such file or directory
```

```
## [1] "/extdata"
```

## The magic


```r
# in real world, these are DB table names
datasets <- c(my_cars = "cars", my_iris = "iris")

my_env <- new.env()
assign("dev_path", dev_path, envir = my_env)
```

```
## Error in assign("dev_path", dev_path, envir = my_env): object 'dev_path' not found
```

```r
assign("inst_path", inst_path, envir = my_env)

for (.name in names(datasets)) {
  
  # in real world, we'd fetch a table here
  dat <- get(datasets[[.name]])                      
  
  # write to CSV file (in pkg extdata/)
  # NOTE: use of file.path() means you have to knit() me from pkg root
  csv_file <- force(paste0(.name, ".csv"))
  write.csv(dat, file = inst_path("extdata", csv_file))
  
  # create promise to read it
  promise_env <- new.env(parent = my_env)               
  assign("csv_file", csv_file, envir = promise_env)
  delayedAssign(
    .name, 
    {
      fn <- inst_path("extdata", csv_file)
      if (!file.exists(fn)) warning(fn, " doesn't exist") else read.csv(fn)
    },
    assign.env = my_env, 
    eval.env = promise_env)

}
```

```
## Warning in normalizePath(file.path(path, ...)):
## path[1]="/extdata/my_cars.csv": No such file or directory
```

```
## Warning in file(file, ifelse(append, "a", "w")): cannot open file
## '/extdata/my_cars.csv': No such file or directory
```

```
## Error in file(file, ifelse(append, "a", "w")): cannot open the connection
```


```r
save(
  list = names(datasets), 
  envir = my_env,
  file = dev_path("data", "promisedat.rda"), 
  eval.promises = FALSE)
```

```
## Error in save(list = names(datasets), envir = my_env, file = dev_path("data", : objects 'my_cars', 'my_iris' not found
```
