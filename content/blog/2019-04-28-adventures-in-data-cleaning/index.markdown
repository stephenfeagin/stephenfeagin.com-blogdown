---
title: Adventures in Data Cleaning
author: Stephen Feagin
date: "2019-04-28"
tags: [r, tidyverse, data cleaning]
---



I’m updating my first R-related blog post, from back in 2016. In it, I used **reshape2** to do some data
cleaning on a messy dataset. In this update, I’m going to use tools from the tidyverse instead. I’m
keeping most of the text the same, but changing the code chunks.<!--more--> 

I’m updating my first R-related blog post, from back in 2016. In it, I used **reshape2** to do some data
cleaning on a messy dataset. In this update, I’m going to use tools from the tidyverse instead. I’m
keeping most of the text the same, but changing the code chunks. As before, this is an R Markdown
file, so you can run the code along with me. You can find the data source 
[here](data/datastream_excerpt.xlsx).

I’ve been working with stock market data, which has proven much less straightforward than I had
expected. Because the data I want is both cross-sectional and time-series — it uses daily data on
multiple values for multiple companies over a length of time — the raw data is an absolute mess. I
got the dataset from Thomson Reuters Datastream, which was a process in and of itself. So in this
post, I’m just going to walk through the steps involved in cleaning the data into a workable data
file.

There are a couple of packages I’ll be using: readxl for reading in the Excel-formatted data, tidyr
and dplyr for data cleaning, and here for dealing with filepaths in the website directoryt tree (you
may not need to deal with that, though). The only one that I want to import into the global
namespace is dplyr, because I only use the others once or twice.


```r
library(dplyr)
```

## Loading Data

Easy enough:


```r
filepath <- "datastream_excerpt.xlsx"
dat <- readxl::read_excel(filepath)
```

Let’s see how that looks in its original format:


```r
dat
```

```
## # A tibble: 23 × 21
##    Name                ABBOTT …¹ ABBOT…² #ERRO…³ #ERRO…⁴ ADAMJ…⁵ ADAMJ…⁶ #ERRO…⁷
##    <dttm>                  <dbl>   <dbl> <chr>   <chr>     <dbl>   <dbl> <chr>  
##  1 1999-04-12 00:00:00      20.5    920. $$ER: … $$ER: …    4.99   2006. $$ER: …
##  2 1999-04-13 00:00:00      20.5    920. <NA>    <NA>       5.12   2059. <NA>   
##  3 1999-04-14 00:00:00      20.5    920. <NA>    <NA>       5.23   2104. <NA>   
##  4 1999-04-15 00:00:00      20.5    920. <NA>    <NA>       5.25   2112. <NA>   
##  5 1999-04-16 00:00:00      20.5    920. <NA>    <NA>       5.32   2143. <NA>   
##  6 1999-04-19 00:00:00      20.5    920. <NA>    <NA>       6.02   2424. <NA>   
##  7 1999-04-20 00:00:00      20.5    920. <NA>    <NA>       6.41   2578. <NA>   
##  8 1999-04-21 00:00:00      20.5    920. <NA>    <NA>       6.25   2514. <NA>   
##  9 1999-04-22 00:00:00      20.5    920. <NA>    <NA>       6.18   2489. <NA>   
## 10 1999-04-23 00:00:00      20.5    920. <NA>    <NA>       6.31   2539. <NA>   
## # … with 13 more rows, 13 more variables: `#ERROR...9` <chr>,
## #   `AGRIAUTO INDUSTRIES` <dbl>, `AGRIAUTO INDUSTRIES - MARKET VALUE` <dbl>,
## #   `#ERROR...12` <chr>, `#ERROR...13` <chr>, `#ERROR...14` <chr>,
## #   `#ERROR...15` <chr>, `AL ABID SILK` <dbl>,
## #   `AL ABID SILK - MARKET VALUE` <dbl>, `#ERROR...18` <chr>,
## #   `#ERROR...19` <chr>, `AL-GHAZI TRACTORS` <dbl>,
## #   `AL-GHAZI TRACTORS - MARKET VALUE` <dbl>, and abbreviated variable names …
```

None of these entries are entirely clear from just looking at the data. The first column, here
called Name, is the numerical representation of the date for that entry. The second column gives us
the adjusted share price for the company Abbott Labs, and the third shows the market value. The
spreadsheet doesn’t actually indicate that the second column is the price, but I know that I
downloaded only the share price and market value for each company. We can also see that there are a
number of columns that Datastream failed to produce data for, showing up as #ERROR.

## Dropping Invalid Columns

The first step is to remove the #ERROR columns. I do this using the dplyr::select_at() function,
which allows me to search for character strings within column names. I redefine my dat object as the
original dataframe, minus any column whose name contains “ERROR.”


```r
dat <- select_at(dat, vars(-starts_with("#ERROR")))
```

While I’m at it, I’m going to rename the columns to all lowercase letters. Just for convenience, not
necessary. I’m also going to change the name of the first column from name to date.
dplyr::rename_all() is similar to dplyr::select_at(), except that I don’t have to specify a rule for
determining which columns to include, and I add the function that I want applied to all of the
column names.


```r
dat <- dat %>% 
  rename_all(tolower) %>% 
  rename(date = name)
```

Let’s take a look at the table now:


```r
dat
```

```
## # A tibble: 23 × 11
##    date                abbott …¹ abbot…² adamj…³ adamj…⁴ agria…⁵ agria…⁶ al ab…⁷
##    <dttm>                  <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1 1999-04-12 00:00:00      20.5    920.    4.99   2006.    1.5     43.2    43.5
##  2 1999-04-13 00:00:00      20.5    920.    5.12   2059.    1.79    51.6    43.5
##  3 1999-04-14 00:00:00      20.5    920.    5.23   2104.    1.58    45.6    43.5
##  4 1999-04-15 00:00:00      20.5    920.    5.25   2112.    1.58    45.6    43.5
##  5 1999-04-16 00:00:00      20.5    920.    5.32   2143.    1.62    46.8    43.5
##  6 1999-04-19 00:00:00      20.5    920.    6.02   2424.    2       57.6    43.5
##  7 1999-04-20 00:00:00      20.5    920.    6.41   2578.    2.08    60      43.5
##  8 1999-04-21 00:00:00      20.5    920.    6.25   2514.    2.04    58.8    43.5
##  9 1999-04-22 00:00:00      20.5    920.    6.18   2489.    2       57.6    43.5
## 10 1999-04-23 00:00:00      20.5    920.    6.31   2539.    2.12    61.2    43.5
## # … with 13 more rows, 3 more variables: `al abid silk - market value` <dbl>,
## #   `al-ghazi tractors` <dbl>, `al-ghazi tractors - market value` <dbl>, and
## #   abbreviated variable names ¹​`abbott labs.(pak.)`,
## #   ²​`abbott labs.(pak.) - market value`, ³​`adamjee insurance`,
## #   ⁴​`adamjee insurance - market value`, ⁵​`agriauto industries`,
## #   ⁶​`agriauto industries - market value`, ⁷​`al abid silk`
```

Much better.

## Reshaping

Now that we’ve cleaned up the column names, we can move onto the real problem: the shape of the
dataset. This is where tidyr comes in. tidyr provides functions to transform long tables into wide
ones, and vice-versa. We currently have a dataset with a row for each day, and seaparate columns for
each variable (share price and market value), grouped into companies. We want to end up with a
dataset with a row for each company-day, and a column for each variable. That is, I want a column
for date, a column for company, a column for price, and a column for market_value. The number of
rows should equal the number of companies multipled by the number of unique days in the dataset.
Here is my general approach:

1. Split the dataset into share price and market value tables
2. Convert both tables into “long” format
3. Join them back together by company name and date

### Spliting the Data

First, we need to split the dataset into two different tables, one for share price and one for
market value.


```r
share_price <- select_at(dat, vars(-ends_with("market value")))
market_value <- select_at(dat, vars(date, ends_with("value")))
```

Here’s the share_price table:


```r
share_price
```

```
## # A tibble: 23 × 6
##    date                `abbott labs.(pak.)` adamjee in…¹ agria…² al ab…³ al-gh…⁴
##    <dttm>                             <dbl>        <dbl>   <dbl>   <dbl>   <dbl>
##  1 1999-04-12 00:00:00                 20.5         4.99    1.5     43.5    16.6
##  2 1999-04-13 00:00:00                 20.5         5.12    1.79    43.5    16.6
##  3 1999-04-14 00:00:00                 20.5         5.23    1.58    43.5    16.6
##  4 1999-04-15 00:00:00                 20.5         5.25    1.58    43.5    16.6
##  5 1999-04-16 00:00:00                 20.5         5.32    1.62    43.5    16.6
##  6 1999-04-19 00:00:00                 20.5         6.02    2       43.5    16.6
##  7 1999-04-20 00:00:00                 20.5         6.41    2.08    43.5    16.6
##  8 1999-04-21 00:00:00                 20.5         6.25    2.04    43.5    16.6
##  9 1999-04-22 00:00:00                 20.5         6.18    2       43.5    16.6
## 10 1999-04-23 00:00:00                 20.5         6.31    2.12    43.5    16.9
## # … with 13 more rows, and abbreviated variable names ¹​`adamjee insurance`,
## #   ²​`agriauto industries`, ³​`al abid silk`, ⁴​`al-ghazi tractors`
```

And the market_value table:


```r
market_value
```

```
## # A tibble: 23 × 6
##    date                abbott labs.(pak.) - ma…¹ adamj…² agria…³ al ab…⁴ al-gh…⁵
##    <dttm>                                  <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
##  1 1999-04-12 00:00:00                      920.   2006.    43.2    310.    959.
##  2 1999-04-13 00:00:00                      920.   2059.    51.6    310.    959.
##  3 1999-04-14 00:00:00                      920.   2104.    45.6    310.    959.
##  4 1999-04-15 00:00:00                      920.   2112.    45.6    310.    959.
##  5 1999-04-16 00:00:00                      920.   2143.    46.8    310.    959.
##  6 1999-04-19 00:00:00                      920.   2424.    57.6    310.    959.
##  7 1999-04-20 00:00:00                      920.   2578.    60      310.    959.
##  8 1999-04-21 00:00:00                      920.   2514.    58.8    310.    959.
##  9 1999-04-22 00:00:00                      920.   2489.    57.6    310.    959.
## 10 1999-04-23 00:00:00                      920.   2539.    61.2    310.    981.
## # … with 13 more rows, and abbreviated variable names
## #   ¹​`abbott labs.(pak.) - market value`, ²​`adamjee insurance - market value`,
## #   ³​`agriauto industries - market value`, ⁴​`al abid silk - market value`,
## #   ⁵​`al-ghazi tractors - market value`
```

### Lengthening the Tables

Next, we use the tidyr::gather() function to turn the very wide tables into long ones.


```r
share_price <- tidyr::gather(share_price, key = "company", value = "share_price", -date)
market_value <- tidyr::gather(market_value, key = "company", value = "market_value", -date)
```

gather() takes a key argument, which is the name of the column in the new table that has all of the
column names from the old table, and a value argument, which is the name of the column in the new
table that actually contains the data. We also include -date to let the function know that we do not
want the date field to be collapsed, we want it excluded from the operation.

Here’s the share_price data now:


```r
share_price
```

```
## # A tibble: 115 × 3
##    date                company            share_price
##    <dttm>              <chr>                    <dbl>
##  1 1999-04-12 00:00:00 abbott labs.(pak.)        20.5
##  2 1999-04-13 00:00:00 abbott labs.(pak.)        20.5
##  3 1999-04-14 00:00:00 abbott labs.(pak.)        20.5
##  4 1999-04-15 00:00:00 abbott labs.(pak.)        20.5
##  5 1999-04-16 00:00:00 abbott labs.(pak.)        20.5
##  6 1999-04-19 00:00:00 abbott labs.(pak.)        20.5
##  7 1999-04-20 00:00:00 abbott labs.(pak.)        20.5
##  8 1999-04-21 00:00:00 abbott labs.(pak.)        20.5
##  9 1999-04-22 00:00:00 abbott labs.(pak.)        20.5
## 10 1999-04-23 00:00:00 abbott labs.(pak.)        20.5
## # … with 105 more rows
```

And market_value:


```r
market_value
```

```
## # A tibble: 115 × 3
##    date                company                           market_value
##    <dttm>              <chr>                                    <dbl>
##  1 1999-04-12 00:00:00 abbott labs.(pak.) - market value         920.
##  2 1999-04-13 00:00:00 abbott labs.(pak.) - market value         920.
##  3 1999-04-14 00:00:00 abbott labs.(pak.) - market value         920.
##  4 1999-04-15 00:00:00 abbott labs.(pak.) - market value         920.
##  5 1999-04-16 00:00:00 abbott labs.(pak.) - market value         920.
##  6 1999-04-19 00:00:00 abbott labs.(pak.) - market value         920.
##  7 1999-04-20 00:00:00 abbott labs.(pak.) - market value         920.
##  8 1999-04-21 00:00:00 abbott labs.(pak.) - market value         920.
##  9 1999-04-22 00:00:00 abbott labs.(pak.) - market value         920.
## 10 1999-04-23 00:00:00 abbott labs.(pak.) - market value         920.
## # … with 105 more rows
```

While I still have the data split, I’m going to clean up the company field in the market_value table
so that it only has the company name.


```r
market_value <- mutate(market_value, company = gsub(" - market value", "", company))
market_value
```

```
## # A tibble: 115 × 3
##    date                company            market_value
##    <dttm>              <chr>                     <dbl>
##  1 1999-04-12 00:00:00 abbott labs.(pak.)         920.
##  2 1999-04-13 00:00:00 abbott labs.(pak.)         920.
##  3 1999-04-14 00:00:00 abbott labs.(pak.)         920.
##  4 1999-04-15 00:00:00 abbott labs.(pak.)         920.
##  5 1999-04-16 00:00:00 abbott labs.(pak.)         920.
##  6 1999-04-19 00:00:00 abbott labs.(pak.)         920.
##  7 1999-04-20 00:00:00 abbott labs.(pak.)         920.
##  8 1999-04-21 00:00:00 abbott labs.(pak.)         920.
##  9 1999-04-22 00:00:00 abbott labs.(pak.)         920.
## 10 1999-04-23 00:00:00 abbott labs.(pak.)         920.
## # … with 105 more rows
```

### Re-Combining

Now all that’s left is to join the tables back together. dplyr provides a number of _join()
functions that mirror SQL joins, so the operation is pretty intuitive if you’ve worked with database
tables.


```r
new_data <- full_join(share_price, market_value, by = c("date", "company"))
new_data
```

```
## # A tibble: 115 × 4
##    date                company            share_price market_value
##    <dttm>              <chr>                    <dbl>        <dbl>
##  1 1999-04-12 00:00:00 abbott labs.(pak.)        20.5         920.
##  2 1999-04-13 00:00:00 abbott labs.(pak.)        20.5         920.
##  3 1999-04-14 00:00:00 abbott labs.(pak.)        20.5         920.
##  4 1999-04-15 00:00:00 abbott labs.(pak.)        20.5         920.
##  5 1999-04-16 00:00:00 abbott labs.(pak.)        20.5         920.
##  6 1999-04-19 00:00:00 abbott labs.(pak.)        20.5         920.
##  7 1999-04-20 00:00:00 abbott labs.(pak.)        20.5         920.
##  8 1999-04-21 00:00:00 abbott labs.(pak.)        20.5         920.
##  9 1999-04-22 00:00:00 abbott labs.(pak.)        20.5         920.
## 10 1999-04-23 00:00:00 abbott labs.(pak.)        20.5         920.
## # … with 105 more rows
```

Beautiful.
