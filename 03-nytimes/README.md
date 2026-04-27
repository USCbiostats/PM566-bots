
# NYTimes health news

The following is an example of how to use the [NYTimes
API](https://developer.nytimes.com/) to get a list of articles related
to a particular term. Here we are searching articles that include the
term `"health"`, and that were published up to seven days ago since
2026-04-27.

``` r
# Preparing a function to use the GET method with the NYTimes API
library(httr)
f <- function(offset = 0) {
  GET(
    url = "https://api.nytimes.com/",
    path = "svc/search/v2/articlesearch.json",
    query = list(
      fq           = "health",
      facet        = "true",
      "begin_date" = gsub("-", "", Sys.Date() - 7),
      "api-key"    = Sys.getenv("NYT_APIKEY"),
      offset       = offset
      )
    )
}

# Retrieving 40 articles
keywords <- NULL
maxtries <- 5
niters   <- 10
i        <- 1
ntries   <- 1
while ((ntries < maxtries) && (i <= niters)) {
  
  res <- tryCatch(f(10 * (i-1)), error = function(e) e)
  
  # If it returns error
  if (inherits(res, "error")) {
    ntries <- ntries + 1
    next
  }
  
  # If the return code is not 200
  if (httr::status_code(res) != 200) {
    ntries <- ntries + 1
    next
  }
  
  # Parsing the data
  ans <- httr::content(res)
  keywords <- c(
    keywords,
    sapply(ans$response$docs, function(x) sapply(x$keywords, "[[", "value"))
    )
  
  # Incrementing and restarting
  i      <- i + 1
  ntries <- 1L
  
}
```

Now that we got the data, we can proceed to do some visualization

``` r
library(ggplot2)
library(ggwordcloud)

keywords <- as.data.frame(table(unlist(keywords)))

# Just to make sure that we were able to download info!
stopifnot(nrow(keywords) > 1)
ggwordcloud(keywords[, 1], keywords[, 2])
```

![](README_files/figure-gfm/preparing-data-1.png)<!-- -->

Finally, a table with the top 20 articles

``` r
tab <- keywords[order(-keywords[,2]),][1:20,]
colnames(tab) <- c("Keyword", "# Articles")
knitr::kable(tab, row.names = FALSE)
```

| Keyword                                                      | \# Articles |
|:-------------------------------------------------------------|------------:|
| Trump, Donald J                                              |          20 |
| Washington (DC)                                              |          20 |
| White House Correspondents’ Dinner Shooting (April 25, 2026) |          20 |
| Assassinations and Attempted Assassinations                  |          15 |
| United States Politics and Government                        |          15 |
| Allen, Cole Tomas                                            |          10 |
| News and News Media                                          |          10 |
| White House Correspondents Assn                              |          10 |
| Agriculture and Farming                                      |           5 |
| Artificial Intelligence                                      |           5 |
| Bayer AG                                                     |           5 |
| Blanche, Todd (Attorney)                                     |           5 |
| Blankfein, Lloyd C                                           |           5 |
| Brain                                                        |           5 |
| Brooklyn (NYC)                                               |           5 |
| Budgets and Budgeting                                        |           5 |
| CBS News                                                     |           5 |
| Chemicals                                                    |           5 |
| Child Abuse and Neglect                                      |           5 |
| Child Care                                                   |           5 |
