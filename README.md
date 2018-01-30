Fitting Linear and Quadratic Models of Nematode Population Responses to
Time and Temperature
================

AH Sparks<sup>1</sup> and JP Thompson<sup>1</sup>

  - <sup>1</sup> University of Southern Queensland, Centre for Crop
    Health, West St., Toowoomba, Queensland, Australia

> Thompson, JP, 2015. Modelling population densities of root-lesion
> nematode *Pratylenchus thornei* from soil profile temperatures to
> choose an optimum sowing date for wheat in a subtropical region.
> *Field Crops Research* 183:50-55 DOI: 10.1016/j.fcr.2015.07.005. URL:
> <http://www.sciencedirect.com/science/article/pii/S0378429015300083>

# Introduction

*Pratylenchus thornei*, the root-lesion nematode is widely distributed
in wheat (*Triticum aestivum*) growing areas of many countries and is of
particular concern in sub-tropical environments (Thompson 2015). These
nematodes penetrate roots to feed and reproduce in the root cortex
leading to loss of root function, which affects nutrient and water
uptake of nutrients and water causing nutrient deficiency and water
stress (Thompson 2015).

In the original paper the population of *P. thornei* in wheat in
Queensland, Australia is modelled using a linear and quadratic
equations. The study aimed to investigate the effects of soil profile
temperatures after different sowing dates on reproduction of the
nematodes in susceptible and moderately resistant wheat cultivars in the
subtropical grain region of eastern Australia. This document recreates
the models for population densities of *P. thornei* as described in
*Modelling population densities of root-lesion nematode* (P. thornei)
*from soil profile temperatures to choose an optimum sowing date for
wheat* (Thompson 2015).

There are two types of models described in the paper, the first model is
a linear model used to describe the unplanted control and two quadratic
models fit Gatcher (Susceptible) and GS50a (Moderately Resistant) wheat
cultivars.

## Session Setup

Using the *tidyverse* package simplifies the libraries. It loads,
*readr*, used to import the data, *tidyr* used to format the data and
*ggplot2* used for visualising the data.

``` r
library(tidyverse)
```

We will use the `set.seed` function for reproducibility.

    set.seed(52)

## Data Import and Inspection

Import the data using `read.csv` from *readr*, which is a part of the
*tidyverse*.

``` r
nema <- read_csv("data/Degree Days Relationships.csv")

nema
```

    ## # A tibble: 24 x 9
    ##    Weeks  Days Temperature Degree_days Unplanted Gatcher GS50a Potam
    ##    <int> <int>       <dbl>       <int>     <dbl>   <dbl> <dbl> <dbl>
    ##  1     8    56        15.0         280      5.75    6.77  6.69  7.61
    ##  2     8    56        20.0         560      5.92    9.51  7.42  9.28
    ##  3     8    56        22.5         700      6.38    9.96  8.21  9.02
    ##  4     8    56        25.0         840      6.51    9.35  8.25  9.73
    ##  5    10    70        15.0         350      5.85    7.44  6.04  5.97
    ##  6    10    70        20.0         700      6.16   10.3   8.91 10.3 
    ##  7    10    70        22.5         875      6.19   10.4   9.18 10.7 
    ##  8    10    70        25.0        1050      6.36   10.6   9.04 10.5 
    ##  9    12    84        15.0         420      5.76    9.93  8.19  8.74
    ## 10    12    84        20.0         840      6.98   11.7   9.85 11.3 
    ## # ... with 14 more rows, and 1 more variable: Suneca <dbl>

### Description of Fields in the Data

  - **Weeks** Number of weeks after planting

  - **Days** Number of days after planting

  - **Temperature** Temperature (˚C) Treatment

  - **Degree\_days** Degree days above 10 ˚C

  - **Unplanted** Log nematode population in the control treatment with
    no wheat planted

  - **Gatcher** Log nematode population in a susceptible wheat cultivar

  - **GS50a** Log nematode population moderately resistant wheat
    cultivar

  - **Potam** Log nematode population susceptible wheat cultivar

  - **Suneca** Log nematode population susceptible wheat cultivar

### Wide to Long Data

You can see that each of the varieties have their own column in the
original data format (wide). Using `gather()` from the *tidyr* package
(part of the *tidyverse*), convert from to long format where the
varieties are all listed in a single column, “Variety”. The `data`
paramter tells R which data frame to gather. The `key` parameter is the
name of the new column to be created called “Variety”, `value` specifies
the column that will contain the values that go with the varieties,
“Population”. The last portion tells `gather()` which columns are to
be gathered. Using the operator `:` means take the columns from
“Unplanted” to “Suneca” and gather them without needing to type all
the column names.

``` r
nema_long <-
  gather(data = nema,
         key = Variety,
         value = Population,
         Unplanted:Suneca)

nema_long
```

    ## # A tibble: 120 x 6
    ##    Weeks  Days Temperature Degree_days Variety   Population
    ##    <int> <int>       <dbl>       <int> <chr>          <dbl>
    ##  1     8    56        15.0         280 Unplanted       5.75
    ##  2     8    56        20.0         560 Unplanted       5.92
    ##  3     8    56        22.5         700 Unplanted       6.38
    ##  4     8    56        25.0         840 Unplanted       6.51
    ##  5    10    70        15.0         350 Unplanted       5.85
    ##  6    10    70        20.0         700 Unplanted       6.16
    ##  7    10    70        22.5         875 Unplanted       6.19
    ##  8    10    70        25.0        1050 Unplanted       6.36
    ##  9    12    84        15.0         420 Unplanted       5.76
    ## 10    12    84        20.0         840 Unplanted       6.98
    ## # ... with 110 more rows

Now that the data are in the format that *ggplot2* prefers, take a look
at the data first to see what it looks like. Fit a smoothed line for
each Variety to the raw data. The individual temperatures are shown here
by shape, the variety by colour.

``` r
ggplot(
  nema_long,
  aes(
    x = Degree_days,
    y = Population,
    colour = Variety,
    group = Variety
  )
) +
  geom_point(aes(shape = as.factor(Temperature))) +
  geom_smooth() +
  ylab(expression(paste("ln(",
                        italic("P. thornei"),
                        "/kg soil) + 1"),
                  sep = "")) +
  xlab("Thermal Time (˚C Days Above 10˚C)") +
  theme_minimal() +
  scale_shape_discrete("Temperature") +
  theme(axis.text.x  = element_text(angle = 90,
                                    vjust = 0.5))
```

    ## `geom_smooth()` using method = 'loess' and formula 'y ~ x'

![](README_files/figure-gfm/raw_data_scatterplots-1.png)<!-- -->

# Model Fitting

## Unplanted Model

The paper uses a linear model for the unplanted control. Here write a
function to use in modelling the unplanted population data.

### Linear Model

``` r
linear_model <- function(df) {
  lm(Population ~ Degree_days,
     data = df)
}
```

Now check the model fit, using `filter()` from *dplyr* to select only
Unplanted data from the data set for the model and fit the linear model
to the data.

Lastly we can show the unplanted data alone as a scatterplot with the
model line fitted using `geom_smooth()` from *ggplot2*.

``` r
unplanted_model <- nema_long %>%
  filter(Variety == "Unplanted") %>%
  linear_model()

par(mfrow = c(2, 2))
plot(unplanted_model)
```

![](README_files/figure-gfm/check_model-1.png)<!-- -->

``` r
summary(unplanted_model)
```

    ## 
    ## Call:
    ## lm(formula = Population ~ Degree_days, data = df)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -0.66053 -0.25811 -0.05683  0.21123  0.98511 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 5.4150643  0.1929731  28.061  < 2e-16 ***
    ## Degree_days 0.0012950  0.0001823   7.103 4.01e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.3847 on 22 degrees of freedom
    ## Multiple R-squared:  0.6964, Adjusted R-squared:  0.6826 
    ## F-statistic: 50.45 on 1 and 22 DF,  p-value: 4.006e-07

``` r
nema_long %>%
  group_by(Variety) %>%
  filter(Variety == "Unplanted") %>%
  ggplot(aes(
    x = Degree_days,
    y = Population,
    colour = Variety,
    group = Variety
  )) +
  geom_point(aes(shape = as.factor(Temperature))) +
  geom_smooth(method = "lm",
              formula = y ~ x,
              size = 1) +
  ylab(expression(paste("ln(",
                        italic("P. thornei"),
                        "/kg soil) + 1"),
                  sep = "")) +
  xlab("Thermal Time (˚C Days Above 10˚C)") +
  theme_minimal() +
  scale_shape_discrete("Temperature") +
  scale_colour_discrete("Variety") +
  theme(axis.text.x  = element_text(angle = 90,
                                    vjust = 0.5)) +
  ggtitle("Unplanted Linear Model")
```

![](README_files/figure-gfm/check_model-2.png)<!-- -->

## Quadratic Models

In the original paper, Gatcher and GS50a best fit quadratic models,
which are fit here.

``` r
quadratic_model <- function(df) {
  lm(Population ~ Degree_days + I(Degree_days^2),
      data = df)
}
```

### Susceptible Varieties

Gatcher, Potam and Suneca all have very similar curves, here Gatcher is
used to fit a quadratic model as in the original paper following the
same methods as above for the linear model.

``` r
s_model <- nema_long %>%
  filter(Variety == "Gatcher") %>% 
  quadratic_model()

par(mfrow = c(2, 2))
plot(s_model)
```

![](README_files/figure-gfm/susceptible_model-1.png)<!-- -->

``` r
summary(s_model)
```

    ## 
    ## Call:
    ## lm(formula = Population ~ Degree_days + I(Degree_days^2), data = df)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.80668 -0.58936  0.07297  0.58228  1.14866 
    ## 
    ## Coefficients:
    ##                    Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)       5.476e+00  9.043e-01   6.055 5.21e-06 ***
    ## Degree_days       8.961e-03  1.909e-03   4.693 0.000124 ***
    ## I(Degree_days^2) -2.612e-06  9.008e-07  -2.899 0.008579 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.8631 on 21 degrees of freedom
    ## Multiple R-squared:  0.7998, Adjusted R-squared:  0.7808 
    ## F-statistic: 41.96 on 2 and 21 DF,  p-value: 4.621e-08

``` r
nema_long %>%
  group_by(Variety) %>%
  filter(Variety == "Gatcher") %>%
  ggplot(aes(
    x = Degree_days,
    y = Population,
    colour = Variety,
    group = Variety
  )) +
  geom_point(aes(shape = as.factor(Temperature))) +
  geom_smooth(method = "lm",
              formula = y ~ x + I(x^2),
              size = 1) +
  ylab(expression(paste("ln(",
                        italic("P. thornei"),
                        "/kg soil) + 1"),
                  sep = "")) +
  xlab("Thermal Time (˚C Days Above 10˚C)") +
  theme_minimal() +
  scale_shape_discrete("Temperature") +
  scale_colour_discrete("Variety") +
  theme(axis.text.x  = element_text(angle = 90,
                                    vjust = 0.5)) +
  ggtitle("Gatcher Quadratic Model")
```

![](README_files/figure-gfm/susceptible_model-2.png)<!-- -->

### Moderately Resistant Cultiver

GS50a, moderately resistant to *P. thornei* also fit a quadratic model
but the coefficients were slightly different due to different responses
to the variety and temperature.

``` r
mr_model <- nema_long %>%
  filter(Variety == "GS50a") %>%
  quadratic_model()

par(mfrow = c(2, 2))
plot(mr_model)
```

![](README_files/figure-gfm/moderately_resistant_model-1.png)<!-- -->

``` r
summary(mr_model)
```

    ## 
    ## Call:
    ## lm(formula = Population ~ Degree_days + I(Degree_days^2), data = df)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.11285 -0.39845  0.02889  0.45494  1.18598 
    ## 
    ## Coefficients:
    ##                    Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)       5.157e+00  6.779e-01   7.607 1.83e-07 ***
    ## Degree_days       6.274e-03  1.431e-03   4.384  0.00026 ***
    ## I(Degree_days^2) -1.609e-06  6.753e-07  -2.383  0.02672 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.647 on 21 degrees of freedom
    ## Multiple R-squared:  0.8233, Adjusted R-squared:  0.8065 
    ## F-statistic: 48.92 on 2 and 21 DF,  p-value: 1.248e-08

``` r
nema_long %>%
  group_by(Variety) %>%
  filter(Variety == "GS50a") %>%
  ggplot(aes(
    x = Degree_days,
    y = Population,
    colour = Variety,
    group = Variety
  )) +
  geom_point(aes(shape = as.factor(Temperature))) +
  geom_smooth(method = "lm",
              formula = y ~ x + I(x^2),
              size = 1) +
  ylab(expression(paste("ln(",
                        italic("P. thornei"),
                        "/kg soil) + 1"),
                  sep = "")) +
  xlab("Thermal Time (˚C Days Above 10˚C)") +
  theme_minimal() +
  scale_shape_discrete("Temperature") +
  scale_colour_discrete("Variety") +
  theme(axis.text.x  = element_text(angle = 90,
                                    vjust = 0.5)) +
  ggtitle("GS50a Quadratic Model")
```

![](README_files/figure-gfm/moderately_resistant_model-2.png)<!-- -->

As in the original paper, the model equations can be derived from these
models as
    well.

# Reproducibility

    ## Session info -------------------------------------------------------------

    ##  setting  value                       
    ##  version  R version 3.4.3 (2017-11-30)
    ##  system   x86_64, darwin16.7.0        
    ##  ui       unknown                     
    ##  language (EN)                        
    ##  collate  en_AU.UTF-8                 
    ##  tz       Australia/Brisbane          
    ##  date     2018-01-30

    ## Packages -----------------------------------------------------------------

    ##  package    * version    date       source                          
    ##  assertthat   0.2.0      2017-04-11 CRAN (R 3.4.3)                  
    ##  backports    1.1.2      2017-12-13 CRAN (R 3.4.3)                  
    ##  base       * 3.4.3      2018-01-15 local                           
    ##  bindr        0.1        2016-11-13 CRAN (R 3.4.3)                  
    ##  bindrcpp   * 0.2        2017-06-17 CRAN (R 3.4.3)                  
    ##  broom        0.4.3      2017-11-20 CRAN (R 3.4.3)                  
    ##  cellranger   1.1.0      2016-07-27 CRAN (R 3.4.3)                  
    ##  cli          1.0.0      2017-11-05 CRAN (R 3.4.3)                  
    ##  colorspace   1.3-2      2016-12-14 CRAN (R 3.4.3)                  
    ##  compiler     3.4.3      2018-01-15 local                           
    ##  crayon       1.3.4      2017-09-16 CRAN (R 3.4.3)                  
    ##  datasets   * 3.4.3      2018-01-15 local                           
    ##  devtools     1.13.4     2017-11-09 CRAN (R 3.4.3)                  
    ##  digest       0.6.15     2018-01-28 cran (@0.6.15)                  
    ##  dplyr      * 0.7.4      2017-09-28 CRAN (R 3.4.3)                  
    ##  evaluate     0.10.1     2017-06-24 CRAN (R 3.4.3)                  
    ##  forcats    * 0.2.0      2017-01-23 CRAN (R 3.4.3)                  
    ##  foreign      0.8-69     2017-06-22 CRAN (R 3.4.3)                  
    ##  ggplot2    * 2.2.1.9000 2018-01-22 Github (hadley/ggplot2@401511e) 
    ##  glue         1.2.0      2017-10-29 CRAN (R 3.4.3)                  
    ##  graphics   * 3.4.3      2018-01-15 local                           
    ##  grDevices  * 3.4.3      2018-01-15 local                           
    ##  grid         3.4.3      2018-01-15 local                           
    ##  gtable       0.2.0      2016-02-26 CRAN (R 3.4.3)                  
    ##  haven        1.1.1      2018-01-18 cran (@1.1.1)                   
    ##  hms          0.4.1      2018-01-24 cran (@0.4.1)                   
    ##  htmltools    0.3.6      2017-04-28 CRAN (R 3.4.3)                  
    ##  httr         1.3.1      2017-08-20 CRAN (R 3.4.3)                  
    ##  jsonlite     1.5        2017-06-01 CRAN (R 3.4.3)                  
    ##  knitr        1.19       2018-01-29 cran (@1.19)                    
    ##  labeling     0.3        2014-08-23 CRAN (R 3.4.3)                  
    ##  lattice      0.20-35    2017-03-25 CRAN (R 3.4.3)                  
    ##  lazyeval     0.2.1      2017-10-29 CRAN (R 3.4.3)                  
    ##  lubridate    1.7.1      2017-11-03 CRAN (R 3.4.3)                  
    ##  magrittr     1.5        2014-11-22 CRAN (R 3.4.3)                  
    ##  memoise      1.1.0      2017-04-21 CRAN (R 3.4.3)                  
    ##  methods    * 3.4.3      2018-01-15 local                           
    ##  mnormt       1.5-5      2016-10-15 CRAN (R 3.4.3)                  
    ##  modelr       0.1.1      2017-07-24 CRAN (R 3.4.3)                  
    ##  munsell      0.4.3      2016-02-13 CRAN (R 3.4.3)                  
    ##  nlme         3.1-131    2017-02-06 CRAN (R 3.4.3)                  
    ##  parallel     3.4.3      2018-01-15 local                           
    ##  pillar       1.1.0      2018-01-14 CRAN (R 3.4.3)                  
    ##  pkgconfig    2.0.1      2017-03-21 CRAN (R 3.4.3)                  
    ##  plyr         1.8.4      2016-06-08 CRAN (R 3.4.3)                  
    ##  psych        1.7.8      2017-09-09 CRAN (R 3.4.3)                  
    ##  purrr      * 0.2.4      2017-10-18 CRAN (R 3.4.3)                  
    ##  R6           2.2.2      2017-06-17 CRAN (R 3.4.3)                  
    ##  Rcpp         0.12.15    2018-01-20 cran (@0.12.15)                 
    ##  readr      * 1.1.1      2017-05-16 CRAN (R 3.4.3)                  
    ##  readxl       1.0.0      2017-04-18 CRAN (R 3.4.3)                  
    ##  reshape2     1.4.3      2017-12-11 CRAN (R 3.4.3)                  
    ##  rlang        0.1.6.9003 2018-01-30 Github (tidyverse/rlang@a8c15c6)
    ##  rmarkdown    1.8        2017-11-17 CRAN (R 3.4.3)                  
    ##  rprojroot    1.3-2      2018-01-03 CRAN (R 3.4.3)                  
    ##  rstudioapi   0.7        2017-09-07 CRAN (R 3.4.3)                  
    ##  rvest        0.3.2      2016-06-17 CRAN (R 3.4.1)                  
    ##  scales       0.5.0.9000 2018-01-16 Github (hadley/scales@d767915)  
    ##  stats      * 3.4.3      2018-01-15 local                           
    ##  stringi      1.1.6      2017-11-17 CRAN (R 3.4.3)                  
    ##  stringr    * 1.2.0      2017-02-18 CRAN (R 3.4.3)                  
    ##  tibble     * 1.4.2      2018-01-22 cran (@1.4.2)                   
    ##  tidyr      * 0.8.0      2018-01-29 cran (@0.8.0)                   
    ##  tidyselect   0.2.3      2017-11-06 CRAN (R 3.4.3)                  
    ##  tidyverse  * 1.2.1      2017-11-14 cran (@1.2.1)                   
    ##  tools        3.4.3      2018-01-15 local                           
    ##  utf8         1.1.3      2018-01-03 CRAN (R 3.4.3)                  
    ##  utils      * 3.4.3      2018-01-15 local                           
    ##  withr        2.1.1.9000 2018-01-16 Github (jimhester/withr@df18523)
    ##  xml2         1.2.0      2018-01-24 cran (@1.2.0)                   
    ##  yaml         2.1.16     2017-12-12 CRAN (R 3.4.3)