---
title: "Advanced Filters"
author: "Johann de Boer"
date: "2018-09-09"
output:
  html_vignette:
    keep_md: yes
  md_document:
    variant:
      markdown_github
vignette: >
  %\VignetteIndexEntry{ADVANCED FILTERS}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

`ganalytics` provides functions that makes it easy to define filters using natural R language operators. This example shows how to use `ganalytics` to define dimension or metric filters that can be used by the `googleAnalyticsR` package. The current development version of `googleAnalyticsR` supports filters defined with `ganalytics`.

## Setup/Config

Note that this example requires the current development versions of the `googleAnalyticsR` (>=0.5.0.9000) and `ganalytics` (>=0.10.4.9000) R packages available from GitHub. To install these, run the following code in R:

```r
devtools::install_github("MarkEdmondson1234/googleAnalyticsR")
devtools::install_github("jdeboer/ganalytics")
```

Once installed, load these packages. Please refer to the `googleAnalyticsR` package documentation on configuration steps you may need to complete in order to use the Google Analytics APIs.


```r
library(googleAnalyticsR)
library(ganalytics)
library(dplyr)
library(tidyr)
library(ggplot2)
library(purrr)
library(knitr)

ga_auth(file.path("~", "ga.oauth"))
```

```
## Token cache file: ~/ga.oauth
```

```r
view_id <- "117987738"
start_date <- "2018-05-01"
end_date <- "2018-06-30"
```

## Pull the Data

In this example, we'll define the following filters:
* Device category is desktop or tablet - a dimension filter using an OR condition.
* New visitors using either a desktop or tablet device - a dimension filter involving both an AND and an OR condition.
* At least one goal completion or transaction - a metric filter using an OR condition.

The above list of filters will be defined using `ganalytics` expressions as follows:


```r
# Device category is desktop or tablet - a dimension filter using an OR condition.
desktop_or_mobile <- Expr(~deviceCategory == "desktop") | Expr(~deviceCategory == "tablet")

# New visitors using either a desktop or tablet device - a dimension filter involving both an AND and an OR condition.
new_desktop_and_mobile_visitors <- Expr(~userType == "new") & desktop_or_mobile

# At least one goal completion or transaction - a metric filter using an OR condition.
at_least_one_conversion <- Expr(~goalCompletionsAll > 0) | Expr(~transactions > 0)
```

We can now use `googleAnalyticsR` to 


```r
results <- google_analytics(
  viewId = view_id,
  date_range = c(start_date, end_date),
  metrics = c("users", "sessions", "goalCompletionsAll", "transactions"),
  dimensions = c("deviceCategory", "userType"),
  dim_filters = new_desktop_and_mobile_visitors,
  met_filters = at_least_one_conversion
)
```


```r
kable(results)
```



deviceCategory   userType       users   sessions   goalCompletionsAll   transactions
---------------  ------------  ------  ---------  -------------------  -------------
desktop          New Visitor      962        933                  777              0
tablet           New Visitor       39         39                   38              0

