Read Data From Web
================

``` r
#一般画图的操作：预先设定好自己喜欢的theme和color
library(tidyverse)

knitr::opts_chunk$set(
  fig.width = 6,
  fig.asp = .6,
  out.width = "90%"
)

theme_set(theme_minimal()+theme(legend.position = "bottom"))

options(
  ggplot2.continuous.colour = "viridis",
  ggplot2.continuous.fill = "viridis")

scale_colour_discrete = scale_colour_viridis_d
scale_fill_discrete = scale_fill_viridis_d
```

``` r
library(rvest)
library(httr)
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter()         masks stats::filter()
    ## ✖ readr::guess_encoding() masks rvest::guess_encoding()
    ## ✖ dplyr::lag()            masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_html = read_html(url)
```

Get the pieces I actually need

``` r
marj_use_df = drug_use_html |>
  html_table() |>
  dplyr::first() |>
  dplyr::slice(-1)
```

exercise: use the living table of New York website

``` r
cost_of_living = read_html("https://www.bestplaces.net/cost_of_living/city/new_york/new_york") |>
  html_table(header = TRUE) |>
  dplyr::first()
## header = TRUE的意思是让表格的第一行成为标题（如果不用这个代码的话，R会自动补充X1 X2 X3 X4成为标题，在这个例子中  
```

\*\*用nth（x,n,order_by = NULL..)可以取第n个表格

## CSS Selectors

``` r
swm_url = "https://www.imdb.com/list/ls070150896/"
swm_html = read_html(swm_url)
```

``` r
title_vec = swm_html |>
  html_elements(".ipc-title-link-wrapper .ipc-title__text") |>
  html_text()
```

``` r
run_time = swm_html |>
  html_elements(".dli-title-metadata-item:nth-child(2)") |>
  html_text()
```

``` r
score_vec = swm_html |>
  html_elements(".metacritic-score-box") |>
  html_text()
```

``` r
swm_df = 
  tibble(
    title = title_vec,
    runtime = run_time,
    score = score_vec
  )
```

Let us import some books

``` r
book_html = read_html("https://books.toscrape.com/")
book_titles = book_html |>
  html_elements("h3") |>
  html_text()
star_vec = book_html |>
  html_elements(".star-rating") |>
  html_text()
price_vec = book_html |>
  html_elements(".price_color") |>
  html_text()
books = tibble(
  title = book_titles,
  stars = star_vec,
  price = price_vec
)
```

## Use API

Get water data 如果api可以导出csv的话

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/ia2d-e54m.csv") |>
  content()
```

    ## Rows: 45 Columns: 4
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (4): year, new_york_city_population, nyc_consumption_million_gallons_per...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

如果api只能导出json的话。。

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/ia2d-e54m.json") |>
  content("text") |>
  jsonlite::fromJSON() |>
  as_tibble()
```

another example using the BRFSS data

``` r
brfss_df = GET("https://chronicdata.cdc.gov/resource/acme-vg9e.csv",
               query = list("$limit" = 5000)) |>
  content()
```

    ## Rows: 5000 Columns: 23
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (16): locationabbr, locationdesc, class, topic, question, response, data...
    ## dbl  (6): year, sample_size, data_value, confidence_limit_low, confidence_li...
    ## lgl  (1): locationid
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Pokemon API

``` r
pokemon = GET("https://pokeapi.co/api/v2/pokemon/ditto") |>
  content()
```
