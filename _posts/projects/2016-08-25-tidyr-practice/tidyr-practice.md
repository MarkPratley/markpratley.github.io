---
layout: page-fullwidth
title: "TidyR Cheatsheet"
meta_teaser: "TidyR Cheatsheet"
teaser: "TidyR Cheatsheet"
breadcrumb: true
comments: true
meta: true
header:
    title: ""
    image_fullwidth: tidyr-hexbin-sticker-from-rstudio.png
    background-color: "#262930"
    caption: tidyr
image:
    thumb:  tidyr-hexbin-sticker-from-rstudio.png
    homepage: tidyr-hexbin-sticker-from-rstudio.png
    caption: tidyr
categories:
    - projects
tags:
    - r
    - tidyr
---

## f(x) definitions and Quick Links

[gather](#Gather)(data, key, value, ..., na.rm = FALSE, convert = FALSE, factor_key = FALSE)

gather() takes multiple columns, and gathers them into key-value pairs: it makes “wide” data longer.

[spread](#Spread)(data, key, value, fill = NA, convert = FALSE, drop = TRUE)

Spread(), takes two columns (a key-value pair) and spreads them in to multiple columns, making “long” data wider.

[separate](#Separate)(data, col, into, sep = "[^[:alnum:]]+", remove = TRUE,
  convert = FALSE, extra = "warn", fill = "warn", ...)

Sometimes two variables are clumped together in one column. separate() allows you to tease them apart.

## Intro

I'm always forgetting how to use the main verbs from [tidyr](https://cran.r-project.org/web/packages/tidyr/index.html) so this is a brief foray into practicing and remembering them.

The main verbs are [gather](#Gather), [spread](#Spread) and [separate](#Separate).

Some of the explanatory text is from [here](https://blog.rstudio.org/2014/07/22/introducing-tidyr/)

Some data to practice on is taken from [here](http://garrettgman.github.io/tidying/) and [here](http://www.jvcasillas.com/tidyr_tutorial/)



{% highlight r %}
library(tidyr)
library(dplyr)
library(stringr)
library(reshape)
library(DSR)
{% endhighlight %}

## tidyr

tidyr is new package that makes it easy to "tidy" your data. Tidy data is data that's easy to work with: it's easy to munge (with dplyr), visualise (with ggplot2 or ggvis) and model (with R's hundreds of modelling packages). The two most important properties of tidy data are:

- Each column is a variable.
- Each row is an observation.

Arranging your data in this way makes it easier to work with because you have a consistent way of referring to variables (as column names) and observations (as row indices). When use tidy data and tidy tools, you spend less time worrying about how to feed the output from one function into the input of another, and more time answering your questions about the data.

To tidy messy data, you first identify the variables in your dataset, then use the tools provided by tidyr to move them into columns. tidyr provides three main functions for tidying your messy data: gather(), separate() and spread().

## <a id="Gather"></a>Gather

gather() takes multiple columns, and gathers them into key-value pairs: it makes "wide" data longer. Other names for gather include melt (reshape2), pivot (spreadsheets) and fold (databases). 

### Examples

table4 %>% gather(Year, Cases, 2:3)

tidyr.ex %>% gather(Day, Score, c(day1score, day2score))

airquality %>% gather(key, value, -Month, -Day)

messy %>% gather(key, value, -id, -trt)

french_fries %>% gather(flavour, value, -c(time:rep))

## Gather 1


{% highlight r %}
table4 %>% 
    print()
{% endhighlight %}



{% highlight text %}
## # A tibble: 3 x 3
##       country `1999` `2000`
##        <fctr>  <int>  <int>
## 1 Afghanistan    745   2666
## 2      Brazil  37737  80488
## 3       China 212258 213766
{% endhighlight %}

Convert table4 to tidy format


{% highlight r %}
table4 %>% 
    gather(Year, Cases, 2:3) %>% 
    print()
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 3
##       country  Year  Cases
##        <fctr> <chr>  <int>
## 1 Afghanistan  1999    745
## 2      Brazil  1999  37737
## 3       China  1999 212258
## 4 Afghanistan  2000   2666
## 5      Brazil  2000  80488
## 6       China  2000 213766
{% endhighlight %}

## Gather 2


{% highlight r %}
tidyr.ex <- data.frame(
    participant = c("p1", "p2", "p3", "p4", "p5", "p6"), 
    info        = c("g1m", "g1m", "g1f", "g2m", "g2m", "g2m"),
    day1score   = rnorm(n = 6, mean = 80, sd = 15), 
    day2score   = rnorm(n = 6, mean = 88, sd = 8)
)

tidyr.ex %>% 
    print()
{% endhighlight %}



{% highlight text %}
##   participant info day1score day2score
## 1          p1  g1m  74.60504  98.31054
## 2          p2  g1m  91.37222  85.61189
## 3          p3  g1f  69.64784  94.38421
## 4          p4  g2m  73.07411  89.32712
## 5          p5  g2m  88.17903  61.15229
## 6          p6  g2m  74.70297  89.23591
{% endhighlight %}

Convert tidyr.ex to tidy format


{% highlight r %}
tidyr.ex %>% 
    gather(Day, Score, c(day1score, day2score)) %>% 
    mutate(Day=str_extract(Day, "\\d")) %>% 
    print()
{% endhighlight %}



{% highlight text %}
##    participant info Day    Score
## 1           p1  g1m   1 74.60504
## 2           p2  g1m   1 91.37222
## 3           p3  g1f   1 69.64784
## 4           p4  g2m   1 73.07411
## 5           p5  g2m   1 88.17903
## 6           p6  g2m   1 74.70297
## 7           p1  g1m   2 98.31054
## 8           p2  g1m   2 85.61189
## 9           p3  g1f   2 94.38421
## 10          p4  g2m   2 89.32712
## 11          p5  g2m   2 61.15229
## 12          p6  g2m   2 89.23591
{% endhighlight %}

# Gather 3

Using airquality dataset


{% highlight r %}
airquality %>% 
    head() %>% 
    print()
{% endhighlight %}



{% highlight text %}
##   Ozone Solar.R Wind Temp Month Day
## 1    41     190  7.4   67     5   1
## 2    36     118  8.0   72     5   2
## 3    12     149 12.6   74     5   3
## 4    18     313 11.5   62     5   4
## 5    NA      NA 14.3   56     5   5
## 6    28      NA 14.9   66     5   6
{% endhighlight %}

Convert airquality to tidy format, show first 3 values for each key


{% highlight r %}
airquality.tidy <- 
    airquality %>% 
    gather(key, value, -Month, -Day)

airquality.tidy %>% 
    group_by(key) %>% 
    slice(1:3) %>% 
    print()
{% endhighlight %}



{% highlight text %}
## Source: local data frame [12 x 4]
## Groups: key [4]
## 
##    Month   Day     key value
##    <int> <int>   <chr> <dbl>
## 1      5     1   Ozone  41.0
## 2      5     2   Ozone  36.0
## 3      5     3   Ozone  12.0
## 4      5     1 Solar.R 190.0
## 5      5     2 Solar.R 118.0
## 6      5     3 Solar.R 149.0
## 7      5     1    Temp  67.0
## 8      5     2    Temp  72.0
## 9      5     3    Temp  74.0
## 10     5     1    Wind   7.4
## 11     5     2    Wind   8.0
## 12     5     3    Wind  12.6
{% endhighlight %}

# gather #4


{% highlight r %}
messy <- data.frame(
  id = 1:4,
  trt = sample(rep(c('control', 'treatment'), each = 2)),
  work.T1 = runif(4),
  home.T1 = runif(4),
  work.T2 = runif(4),
  home.T2 = runif(4)
)
messy %>% 
    print()
{% endhighlight %}



{% highlight text %}
##   id       trt   work.T1    home.T1   work.T2   home.T2
## 1  1   control 0.6468060 0.39864924 0.6708680 0.6786486
## 2  2 treatment 0.9892446 0.87098142 0.3518354 0.7842983
## 3  3   control 0.2392983 0.29302979 0.8332908 0.7867416
## 4  4 treatment 0.2089560 0.05470081 0.7339074 0.5762343
{% endhighlight %}


{% highlight r %}
tidier <- 
    messy %>% 
    gather(key, value, -id, -trt)

tidier %>% 
    head(5)
{% endhighlight %}



{% highlight text %}
##   id       trt     key     value
## 1  1   control work.T1 0.6468060
## 2  2 treatment work.T1 0.9892446
## 3  3   control work.T1 0.2392983
## 4  4 treatment work.T1 0.2089560
## 5  1   control home.T1 0.3986492
{% endhighlight %}

# gather #5


{% highlight r %}
french_fries %>% 
    head()
{% endhighlight %}



{% highlight text %}
##    time treatment subject rep potato buttery grassy rancid painty
## 61    1         1       3   1    2.9     0.0    0.0    0.0    5.5
## 25    1         1       3   2   14.0     0.0    0.0    1.1    0.0
## 62    1         1      10   1   11.0     6.4    0.0    0.0    0.0
## 26    1         1      10   2    9.9     5.9    2.9    2.2    0.0
## 63    1         1      15   1    1.2     0.1    0.0    1.1    5.1
## 27    1         1      15   2    8.8     3.0    3.6    1.5    2.3
{% endhighlight %}
messy %>% 
    gather(key, value, -id, -trt)
    

{% highlight r %}
ff.tidy <- 
    french_fries %>% 
    gather(flavour, value, -c(time:rep))

ff.tidy %>% 
    group_by(treatment, subject, flavour) %>%
    summarise(value = mean(value)) %>%
    spread(flavour, value) %>%
    head(5)
{% endhighlight %}



{% highlight text %}
## Source: local data frame [5 x 7]
## Groups: treatment, subject [5]
## 
##   treatment subject   buttery    grassy   painty   potato   rancid
##      <fctr>  <fctr>     <dbl>     <dbl>    <dbl>    <dbl>    <dbl>
## 1         1       3 0.3722222 0.1888889 3.111111 6.216667 2.105556
## 2         1      10 6.7500000 0.5850000 1.375000 9.955000 4.020000
## 3         1      15 0.7200000 0.4200000 3.260000 3.360000 3.965000
## 4         1      16 3.2600000 0.7550000 1.230000 6.495000 4.120000
## 5         1      19 3.0550000 2.0200000 2.775000 9.385000 5.360000
{% endhighlight %}

# <a id="Separate"></a>Separate

Sometimes two variables are clumped together in one column. separate() allows you to tease them apart (extract() works similarly but uses regexp groups instead of a splitting pattern or position).

### Examples

table3 %>% separate(rate, c("thingy1", "thingy2"), sep="/")

tidier %>% separate(key, into = c("loc", "time"), sep="\\.")

astrag %>% separate(Taxon, c("genus", "species"), sep="_")

# Separate #1

table2


{% highlight r %}
table3 %>% 
    print()
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 3
##       country  year              rate
##        <fctr> <int>             <chr>
## 1 Afghanistan  1999      745/19987071
## 2 Afghanistan  2000     2666/20595360
## 3      Brazil  1999   37737/172006362
## 4      Brazil  2000   80488/174504898
## 5       China  1999 212258/1272915272
## 6       China  2000 213766/1280428583
{% endhighlight %}



{% highlight r %}
table3 %>% 
    separate(rate, c("thingy1", "thingy2"), sep="/")
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 4
##       country  year thingy1    thingy2
## *      <fctr> <int>   <chr>      <chr>
## 1 Afghanistan  1999     745   19987071
## 2 Afghanistan  2000    2666   20595360
## 3      Brazil  1999   37737  172006362
## 4      Brazil  2000   80488  174504898
## 5       China  1999  212258 1272915272
## 6       China  2000  213766 1280428583
{% endhighlight %}

# Separate #2

Using tidier from above


{% highlight r %}
tidier %>% 
    head(5)
{% endhighlight %}



{% highlight text %}
##   id       trt     key     value
## 1  1   control work.T1 0.6468060
## 2  2 treatment work.T1 0.9892446
## 3  3   control work.T1 0.2392983
## 4  4 treatment work.T1 0.2089560
## 5  1   control home.T1 0.3986492
{% endhighlight %}



{% highlight r %}
tidy <- 
    tidier %>% 
    separate(key, into = c("loc", "time"), sep="\\.")

tidy %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   id       trt  loc time     value
## 1  1   control work   T1 0.6468060
## 2  2 treatment work   T1 0.9892446
## 3  3   control work   T1 0.2392983
## 4  4 treatment work   T1 0.2089560
## 5  1   control home   T1 0.3986492
## 6  2 treatment home   T1 0.8709814
{% endhighlight %}


# Separate #3



{% highlight r %}
astrag <- read.table("http://hompal-stats.wabarr.com/datasets/barr_astrag_2014.txt", header=TRUE, sep="\t")

astrag %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   individual                 Taxon Habitat    ACF  APD     B DistRad DMTD
## 1  AMNH81690    Aepyceros_melampus      LC 384.72 6.89 15.16    9.22 1.78
## 2  AMNH82050    Aepyceros_melampus      LC     NA 6.09 14.00    8.73 1.64
## 3  AMNH83534    Aepyceros_melampus      LC     NA 6.14 17.22    9.19 1.70
## 4  AMNH85150    Aepyceros_melampus      LC     NA 5.80 15.27    8.85 1.72
## 5 AMNH233038 Alcelaphus_buselaphus       O 607.83 7.46 18.47   11.33 2.26
## 6  AMNH34717 Alcelaphus_buselaphus       O 465.91 6.56 15.47    9.79 1.78
##   DTArea   LML   MIN   MML PMTD ProxRad  PTArea   WAF   WAT
## 1 553.14 35.78 28.04 34.39 6.33   10.00  915.50 21.39 21.83
## 2 558.17 35.46 27.28 33.18 6.54   10.50  841.79 22.47 21.11
## 3 554.00 39.15 29.99 36.94 7.63   10.65  976.25 22.07 21.54
## 4 498.67 36.61 28.02 34.09 6.97   10.00  841.71 22.55 21.03
## 5 900.76 45.25 36.03 43.55 6.83   13.85 1557.05 28.98 27.07
## 6 649.47 36.81 30.80 35.95 4.28   10.91 1139.50 24.04 25.84
{% endhighlight %}



{% highlight r %}
astrag %>% 
    select(individual, Taxon, Habitat) %>% 
    separate(Taxon, c("genus", "species"), sep="_") %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   individual      genus    species Habitat
## 1  AMNH81690  Aepyceros   melampus      LC
## 2  AMNH82050  Aepyceros   melampus      LC
## 3  AMNH83534  Aepyceros   melampus      LC
## 4  AMNH85150  Aepyceros   melampus      LC
## 5 AMNH233038 Alcelaphus buselaphus       O
## 6  AMNH34717 Alcelaphus buselaphus       O
{% endhighlight %}


# <a id="Spread"></a>Spread

spread(), takes two columns (a key-value pair) and spreads them in to multiple columns, making "long" data wider. Spread is known by other names in other places: it's cast in reshape2, unpivot in spreadsheets and unfold in databases. spread() is used when you have variables that form rows instead of columns. You need spread() less frequently than gather() or separate()

### Examples

table2 %>% spread(key, value)

ff.tidy %>% spread(flavour, value)

airquality.tidy %>% spread(key, value)

# Spread #1

table2


{% highlight r %}
table2 %>% 
    print()
{% endhighlight %}



{% highlight text %}
## # A tibble: 12 x 4
##        country  year        key      value
##         <fctr> <int>     <fctr>      <int>
## 1  Afghanistan  1999      cases        745
## 2  Afghanistan  1999 population   19987071
## 3  Afghanistan  2000      cases       2666
## 4  Afghanistan  2000 population   20595360
## 5       Brazil  1999      cases      37737
## 6       Brazil  1999 population  172006362
## 7       Brazil  2000      cases      80488
## 8       Brazil  2000 population  174504898
## 9        China  1999      cases     212258
## 10       China  1999 population 1272915272
## 11       China  2000      cases     213766
## 12       China  2000 population 1280428583
{% endhighlight %}



{% highlight r %}
table2 %>% 
    spread(key, value)
{% endhighlight %}



{% highlight text %}
## # A tibble: 6 x 4
##       country  year  cases population
## *      <fctr> <int>  <int>      <int>
## 1 Afghanistan  1999    745   19987071
## 2 Afghanistan  2000   2666   20595360
## 3      Brazil  1999  37737  172006362
## 4      Brazil  2000  80488  174504898
## 5       China  1999 212258 1272915272
## 6       China  2000 213766 1280428583
{% endhighlight %}

# Spread #2

From french fries above


{% highlight r %}
ff.tidy %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   time treatment subject rep flavour value
## 1    1         1       3   1  potato   2.9
## 2    1         1       3   2  potato  14.0
## 3    1         1      10   1  potato  11.0
## 4    1         1      10   2  potato   9.9
## 5    1         1      15   1  potato   1.2
## 6    1         1      15   2  potato   8.8
{% endhighlight %}



{% highlight r %}
ff.tidy %>% 
    spread(flavour, value) %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   time treatment subject rep buttery grassy painty potato rancid
## 1    1         1       3   1     0.0    0.0    5.5    2.9    0.0
## 2    1         1       3   2     0.0    0.0    0.0   14.0    1.1
## 3    1         1      10   1     6.4    0.0    0.0   11.0    0.0
## 4    1         1      10   2     5.9    2.9    0.0    9.9    2.2
## 5    1         1      15   1     0.1    0.0    5.1    1.2    1.1
## 6    1         1      15   2     3.0    3.6    2.3    8.8    1.5
{% endhighlight %}


# Spread #3

Using airquality.tidy from above


{% highlight r %}
airquality.tidy %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   Month Day   key value
## 1     5   1 Ozone    41
## 2     5   2 Ozone    36
## 3     5   3 Ozone    12
## 4     5   4 Ozone    18
## 5     5   5 Ozone    NA
## 6     5   6 Ozone    28
{% endhighlight %}



{% highlight r %}
airquality.tidy %>% 
    spread(key, value) %>% 
    head()
{% endhighlight %}



{% highlight text %}
##   Month Day Ozone Solar.R Temp Wind
## 1     5   1    41     190   67  7.4
## 2     5   2    36     118   72  8.0
## 3     5   3    12     149   74 12.6
## 4     5   4    18     313   62 11.5
## 5     5   5    NA      NA   56 14.3
## 6     5   6    28      NA   66 14.9
{% endhighlight %}

The github repository for this project can be viewed [here](https://github.com/MarkPratley/tidyr-practice).




