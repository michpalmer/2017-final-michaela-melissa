California ISO Electric Grid Data Analysis
================
Michaela Palmer & Melissa Ferriter

``` r
knitr::opts_chunk$set(echo = T, warning = F, message = F)
```

``` r
libs <- c("tidyverse", "readxl", "ggpubr", "foreign", "zoo")
sapply(libs, require, character.only=T)
```

Read in data
------------

The data used in this analysis comes from [California Independent System Operator Corporation (CAISO)](http://www.caiso.com/green/renewableswatch.html), which aggregates grid data from electricity producers in California.

``` r
# create and view an object with file names & full paths
days <- c(paste0("0", 1:9), 10:25)
months <- c(paste0("0", 1:9), "10", "11", "12")

urls <- list()
for (i in months) {
  url <- paste0("http://content.caiso.com/green/renewrpt/2017", i, days,"_DailyRenewablesWatch.txt")
  urls[[paste0("month", i)]] <- url
}
```

``` r
# function to import data with proper formatting & columns 
import <- function(data) {
  data.frame(date = as.Date(basename(data), "%Y%m%d"), read_table2( 
    data,
    col_names = c("hour", "geothermal", "biomass", "biogas", "small_hydro", "wind", "solar_pv", "solar_thermal" ),
    skip = 2,
    n_max = 24
  )) 
}

jan <- do.call("rbind", lapply(urls[["month01"]], import))
feb <- do.call("rbind", lapply(urls[["month02"]], import)) 
march <- do.call("rbind", lapply(urls[["month03"]], import)) 
june <- do.call("rbind", lapply(urls[["month06"]], import))
july<- do.call("rbind", lapply(urls[["month07"]], import))
oct <- do.call("rbind", lapply(urls[["month10"]], import))
```

Daily average data plots
========================

The CAISO data are provided at hourly intervals. Plotting the data for the entire 6-year period would have generated a very messy graph, so I calculated daily means and plotted them instead.

``` r
month.list <- list(January=jan, Febuary = feb, June=june, July=july, October=oct)

plot <- function(data, name) {
  data[,-c(2)] %>%
    read.zoo(FUN = identity) %>%
    aggregate(as.Date, mean) %>%
    fortify.zoo() %>%
    select(Index, geothermal, biomass, biogas, small_hydro, wind, solar_pv, solar_thermal) %>%
    gather(Renewables, mean,-Index) %>%
    ggplot() +
    geom_line(mapping = aes(x = Index, y = mean, color = Renewables)) + 
    labs(title = paste(name, "2017 Daily Averages", sep = " "), x = "Date", y = "Production (MW)") + theme_minimal()
}

Map(plot, data = month.list, name = names(month.list))
```

    ## $January

![](final-project_files/figure-markdown_github/unnamed-chunk-4-1.png)

    ## 
    ## $Febuary

![](final-project_files/figure-markdown_github/unnamed-chunk-4-2.png)

    ## 
    ## $June

![](final-project_files/figure-markdown_github/unnamed-chunk-4-3.png)

    ## 
    ## $July

![](final-project_files/figure-markdown_github/unnamed-chunk-4-4.png)

    ## 
    ## $October

![](final-project_files/figure-markdown_github/unnamed-chunk-4-5.png)

``` r
# other tested regex methods
#sapply(strsplit(filelist, "[/_]"), "[", 6)
#sub(".*(\\d{8}).*", "\\1", filelist)
#sub(".*/", "", sub("_.*", "", filelist))
#sub("_.*", "", basename(filelist))
#gsub("(?:.*/){4}([^_]+)_.*", "\\1", filelist)
```