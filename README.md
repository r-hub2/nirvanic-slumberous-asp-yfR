
<!-- README.md is generated from README.Rmd. Please edit that file -->
<!-- badges: start -->

[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![Codecov test
coverage](https://codecov.io/gh/ropensci/yfR/branch/main/graph/badge.svg)](https://app.codecov.io/gh/ropensci/yfR?branch=main)
[![R build
(rcmdcheck)](https://github.com/ropensci/yfR/workflows/R-CMD-check/badge.svg)](https://github.com/ropensci/yfR/actions)
[![Status at rOpenSci Software Peer
Review](https://badges.ropensci.org/523_status.svg)](https://github.com/ropensci/software-review/issues/523)
[![R-CMD-check](https://github.com/ropensci/yfR/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/ropensci/yfR/actions/workflows/R-CMD-check.yaml)
<!-- badges: end -->

# Motivation

`yfR` facilitates importing stock prices from Yahoo finance, organizing
the data in the `tidy` format and speeding up the process using a cache
system and parallel computing. `yfR` is the second and
backwards-incompatible version of
[BatchGetSymbols](https://CRAN.R-project.org/package=BatchGetSymbols),
released in 2016 (see vignette [yfR and
BatchGetSymbols](https://docs.ropensci.org/yfR/articles/diff-batchgetsymbols.html)
for details).

In a nutshell, [Yahoo Finance (YF)](https://finance.yahoo.com/) provides
a vast repository of stock price data around the globe. It covers a
significant number of markets and assets, being used extensively in
academic research and teaching. In order to import the financial data
from YF, all you need is a ticker (id of a stock, e.g. “GM” for [General
Motors](https://finance.yahoo.com/quote/GM?p=GM&.tsrc=fin-srch)) and a
time period – first and last date.

# The Data

The main function of the package, `yfR::yf_get`, returns a dataframe
with the financial data. All price data is measured at the unit of the
financial exchange. For example, price data for GM (NASDAQ/US) is
measured in dollars, while price data for PETR3.SA (B3/BR) is measured
in Reais (Brazilian currency).

The returned data contains the following columns:

**ticker**: The requested tickers (ids of stocks);

**ref_date**: The reference day (this can also be year/month/week when
using argument freq_data);

**price_open**: The opening price of the day/period;

**price_high**: The highest price of the day/period;

**price_close**: The close/last price of the day/period;

**volume**: The financial volume of the day/period, in the unit of the
exchange;

**price_adjusted**: The stock price adjusted for corporate events such
as splits, dividends and others – this is usually what you want/need for
studying stocks as it represents the real financial performance of
stockholders;

**ret_adjusted_prices**: The arithmetic or log return (see input
type_return) for the adjusted stock prices;

**ret_adjusted_prices**: The arithmetic or log return (see input
type_return) for the closing stock prices;

**cumret_adjusted_prices**: The accumulated arithmetic/log return for
the period (starts at 100%).

# Finding tickers

The easiest way to find the tickers of a company stock is to search for
it in [Yahoo Finance’s](https://finance.yahoo.com/) website. At the top
page you’ll find a search bar:

<figure>
<img src="/inst/figures/search-yf.png?raw=true"
title="Example of search in YF" alt="YF Search" />
<figcaption aria-hidden="true">YF Search</figcaption>
</figure>

A company can have many different stocks traded at different markets
(see picture above). As the example shows, Petrobras is traded at NYQ
(New York Exchange), SAO (Sao Paulo/Brazil - B3 exchange) and BUE
(Buenos Aires/Argentina Exchange), all with different symbols (tickers).
For market indices, a list of tickers is available
[here](https://finance.yahoo.com/world-indices).

## Features of `yfR`

- Fetches daily/weekly/monthly/annual stock prices/returns from yahoo
  finance and outputs a dataframe (tibble) in the long format (stacked
  data);

- A new feature called **collections** facilitates download of multiple
  tickers from a particular market/index. You can, for example, download
  data for all stocks in the SP500 index with a simple call to
  `yf_collection_get("SP500")`;

- A session-persistent smart cache system is available by default. This
  means that the data is saved locally and only missing portions are
  downloaded, if needed.

- All dates are compared to a benchmark ticker such as SP500 and,
  whenever an individual asset does not have a sufficient number of
  dates, the software drops it from the output. This means you can
  choose to ignore tickers with a high proportion of missing dates.

- A customized function called `yf_convert_to_wide()` can transform the
  long dataframe into a wide format (tickers as columns), much used in
  portfolio optimization. The output is a list where each element is a
  different target variable (prices, returns, volumes).

- Parallel computing with package `furrr` is available, speeding up the
  data importation process.

## Warnings

- Yahoo finance data is far from perfect or reliable, specially for
  individual stocks. In my experience, using it for research code with
  stock **indices** is fine and I can match it with other data sources.
  But, adjusted stock prices for **individual assets** is messy as stock
  events such as splits or dividends are not properly registered. I was
  never able to match it with other data sources, specially for long
  time periods with lots of corporate events. My advice is to **never
  use the yahoo finance data of individual stocks in production**
  (research papers or academic documents – thesis and dissertations). If
  adjusted price data of individual stocks is important for your
  research, **use other data sources** such as
  [EOD](https://eodhistoricaldata.com/), [SimFin](https://simfin.com/)
  or [Economática](https://economatica.com/).

## Installation

    # CRAN (stable)
    install.packages('yfR')

    # Github (dev version)
    devtools::install_github('ropensci/yfR')

    # ropensci
    install.packages("yfR", repos = "https://ropensci.r-universe.dev")

## A simple example of usage

``` r
library(yfR)

# set options for algorithm
my_ticker <- 'META'
first_date <- Sys.Date() - 30
last_date <- Sys.Date()

# fetch data
df_yf <- yf_get(tickers = my_ticker, 
                     first_date = first_date,
                     last_date = last_date)
#> 
#> ── Running yfR for 1 stocks | 2024-05-25 --> 2024-06-24 (30 days) ──
#> 
#> ℹ Downloading data for benchmark ticker ^GSPC
#> ℹ (1/1) Fetching data for META
#> !    - not cached
#> ✔    - cache saved successfully
#> ✔    - got 18 valid rows (2024-05-28 --> 2024-06-21)
#> ✔    - got 100% of valid prices -- Time for some tea?
#> ℹ Binding price data
#> 
#> ── Diagnostics ─────────────────────────────────────────────────────────────────
#> ✔ Returned dataframe with 18 rows -- You got it msperlin!
#> ℹ Using 5.5 kB at /tmp/Rtmp8A98vn/yf_cache for 2 cache files
#> ℹ Out of 1 requested tickers, you got 1 (100%)
```

``` r

# output is a tibble with data
head(df_yf)
#> # A tibble: 6 × 11
#>   ticker ref_date   price_open price_high price_low price_close   volume
#>   <chr>  <date>          <dbl>      <dbl>     <dbl>       <dbl>    <dbl>
#> 1 META   2024-05-28       477.       481.      475.        480. 10175800
#> 2 META   2024-05-29       475.       480.      474.        474.  9226200
#> 3 META   2024-05-30       472.       472.      465.        467. 10735200
#> 4 META   2024-05-31       466.       469.      454.        467. 16919800
#> 5 META   2024-06-03       471.       480.      468.        477. 11279400
#> 6 META   2024-06-04       477        479.      473.        477.  7088700
#> # ℹ 4 more variables: price_adjusted <dbl>, ret_adjusted_prices <dbl>,
#> #   ret_closing_prices <dbl>, cumret_adjusted_prices <dbl>
```

# Acknowledgements

Package `yfR` is based on [quantmod](https://www.quantmod.com/)
(@joshuaulrich) and uses one of its functions (`quantmod::getSymbols`)
for fetching raw data from Yahoo Finance. As with any API, there is
significant work in maintaining the code. Joshua was always fast and
openminded in implemented required changes, and I’m very grateful for
it.