---
title: "data-raw"
author: "David Holstius"
date: "January 30, 2015"
output: html_document
---


```r
if (file.exists("data")) pkg_root <- "." else pkg_root <- ".." # HACK 
message("pkg_root is ", pkg_root)
```

```
## pkg_root is ..
```


```r
# in real world, these are DB table names
datasets <- c(my_cars = "cars", my_iris = "iris")

dataset_env <- new.env()

for (.name in names(datasets)) {
  
  # in real world, we'd fetch a table here
  dat <- get(datasets[[.name]])                      
  
  # write to CSV file (in pkg extdata/)
  # NOTE: use of file.path() means you have to knit() me from pkg root
  csv_file <- force(paste0(.name, ".csv"))
  write.csv(dat, file = file.path(pkg_root, "inst", "extdata", csv_file))
  
  # create promise to read it
  promise_env <- new.env()               
  assign("csv_file", csv_file, envir = promise_env)
  delayedAssign(
    .name, 
    {
      # fn <- system.file("extdata", csv_file, package = "promisedat")
      print(devtools::inst("promisedat"))
      fn <- file.path(devtools::inst("promisedat"), "extdata", csv_file)
      if (!file.exists(fn)) warning(fn, " doesn't exist") else read.csv(fn)
    },
    assign.env = dataset_env, 
    eval.env = promise_env)

}
```


```r
save(
  list = names(datasets), 
  envir = dataset_env,
  file = file.path(pkg_root, "data", "promisedat.rda"), 
  eval.promises = FALSE)
```
