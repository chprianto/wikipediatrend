# Introducing Wikipediatrend -- Easy Analyses of Public Attention, Anxiety and Information Seeking
Peter Meißner  

2015-02-19

## Current version auto-build status
<img src="https://api.travis-ci.org/petermeissner/wikipediatrend.svg?branch=master"></img>


## Introduction

Wikipedia provides a lot of valuable meta data. 
One type of information are page access statistics -- e.g. http://stats.grok.se/en/201409/Peter_Principle. Another type are the info pages -- e.g. https://en.wikipedia.org/w/index.php?title=Peter_Principle&action=info. While the latter falls into the jurisdiction of the [MediaWiki](http://cran.r-project.org/web/packages/WikipediR/index.html)-package , this package allows loading page view statistics into R. 


## Stats.grok.se API and the wikipediatrend package

`http://stats.grok.se` retrieves Wikipedia page access statistics on a daily 
basis. The information is either presented in HTML or provided as JSON data.


http://stats.grok.se/en/201409/Peter_Principle

versus

http://stats.grok.se/json/en/201409/Peter_Principle



A single request results in data for a specific entry, from one of Wikipedia's different language subdomains and for all days of a given month. The `wikipediatrend` package draws on this Web API and provides a consistent and convenient way to use those data within R. Furthermore the package not only takes care of the communication between the Web API at `stats.grok.se` and your local R session but also provides means to minimize traffic and workload for `stats.grok.se`-server  -- data is (if user decides so) stored locally in CSV format and reused for subsequent requests. 



## A first tutorial


```r
require(wikipediatrend)
```

```
## Loading required package: wikipediatrend
```

The workhorse of the package is the `wp_trend()` function with several arguments:

- **page**        [ `"Peter_principle"` ]: <br>
... here goes the name of the page (might also be a vector of page names, e.g.: `c("cat","dog")`)

- **from**        [ `Sys.Date()-30` ]: <br>
... starting date of the timespan to be considered

- **to**          [ `Sys.Date()` ]: <br>
... end date of the timespan to be considered

- **lang**        [ `"en"` ]: <br>
... language of the page (might also be a vector of languages, e.g.: `c("de","en")`)

- **file**        [ `.wp_trend_cache` ]: <br>
... file to be used as cache / data storage (By default all data retreived will be stored in a temporary file. If a file name is provided, date will be stored there as CSV. Data will always be appended to an already existing data storage file.)


<!-- thrown out since version 0.3.xxx
  - **friendly**    [ `F` ]: <br>
  ... should `wp_trend()` try minimize workload on behalf of `stats.grok.se`
  - **requestFrom** [ `"anonymous"` ]: <br>
  ... do you care to identify yourself towards `stats.grok.se`
  - **userAgent** [ `FALSE` ] <br>
  ... do you care to send information on your plattform, R version and the package used to make server requests
-->

Let's have a first run using the defaults:

```r
peter_principle <- wp_trend()
```

```
## http://stats.grok.se/json/en/201501/Peter_principle
## http://stats.grok.se/json/en/201502/Peter_principle
## 
## Results written to:
## D:\Users\Peter\AppData\Local\Temp\Rtmpi4DSLf\wp_trend_cache_21dc1ea2367d.csv
```

The function informs us that using the friendly option might be a good idea and shows us which URLs it used to retrieve the information we were asking for. 

The function's return is a data frame with two variables *date* and *count*:


```r
dim(peter_principle)
```

```
## [1] 29  6
```

```r
class(peter_principle)
```

```
## [1] "data.frame"
```

```r
head(peter_principle)
```

```
##         date count project           title rank  month
## 1 2015-01-20  1290      en Peter_principle   -1 201501
## 2 2015-01-21  1481      en Peter_principle   -1 201501
## 3 2015-01-22  1285      en Peter_principle   -1 201501
## 4 2015-01-23  1322      en Peter_principle   -1 201501
## 5 2015-01-24   548      en Peter_principle   -1 201501
## 6 2015-01-25   573      en Peter_principle   -1 201501
```

We can use this information to visualize the page view trend. Using `wp_wday()` we can furthermore discriminate weekdays <span style="color:black">(black)</span> from weekends <span style="color:red">(red)</span>. 


```r
plot( peter_principle, 
      col=ifelse( wp_wday(peter_principle$date) > 5 , "red", "black") ,
      ylim=c(0, max(peter_principle$count)),
      main="Peter Principle's Wikipedia Attention",
      ylab="views per day", xlab="time")
```

```
## Warning in data.matrix(x): NAs introduced by coercion
```

```
## Warning in data.matrix(x): NAs introduced by coercion
```

```
## Error in UseMethod("wp_wday"): no applicable method for 'wp_wday' applied to an object of class "Date"
```

```r
lines(peter_principle)
```

```
## Warning in data.matrix(x): NAs introduced by coercion
```

```
## Warning in data.matrix(x): NAs introduced by coercion
```

```
## Error in plot.xy(xy.coords(x, y), type = type, ...): plot.new has not been called yet
```

Looking at the graph we can conclude that the *Peter Principle* as a work related phenomenon obviously is something that is most pressing on workdays -- or maybe people in general just tend to use their computers less on weekends.







## Being friendly

One of the most important features of the package is its `friendly` option. On the one hand, it saves us time when making subsequent requests of the same page because less pages have to be loaded. On the other hand, it serves to minimize workload on behalf of the `stats.grok.se`-server that kindly provides the information we are using. 

The option can be set to different values: 

- **FALSE**, the default, deactivates `wp_trend()`'s friendly behavior altogether
- **TRUE**, activates `wp_trend()`'s friendly behavior and retreieved access statistics are stored on disk in CSV format via `write.csv()`
- **1** is the same as **TRUE**
- **2**, is the same as **TRUE** but storage takes place via `write.csv2()`

Let's try it out by making two subsequent requests to get access statistics for for information on ISIS. 


```r
file.remove("wp__Islamic_State_of_Iraq_and_the_Levant__en.csv")
```

While for the first request the server has to provide information many times, the second request only asks for those months for which we do not have complete data already. Furthermore, `wp_trend()` informs us that the data has been stored in a CSV-file.




```r
isis <- wp_trend("Islamic_State_of_Iraq_and_the_Levant", from="2013-01-01", friendly=T)
```

```
## Error in wp_trend("Islamic_State_of_Iraq_and_the_Levant", from = "2013-01-01", : unused argument (friendly = T)
```

The second request uses this previous saved information to minimize traffic and function execution time. If it downloads new data, it updates the data already stored on disk.



```r
isis <- wp_trend("Islamic_State_of_Iraq_and_the_Levant", from="2012-12-01", friendly=T)
```

```
## Error in wp_trend("Islamic_State_of_Iraq_and_the_Levant", from = "2012-12-01", : unused argument (friendly = T)
```

Last but not least, let's have a look at the data ... 


```r
plot( isis, 
      ylim=c(0, max(isis$count)),
      main="ISIS' Wikipedia Attention",
      ylab="views per day", xlab="time",
      type="l")
```

```
## Error in plot(isis, ylim = c(0, max(isis$count)), main = "ISIS' Wikipedia Attention", : object 'isis' not found
```

... revealing what most might have already suspected: ISIS is quite a new phenomenon. 


## So what? 

### Cats

First of all we are now able to study cats:


```r
cats <- wp_trend("Cat", from="2007-01-01", friendly=T)
```

```
## Error in wp_trend("Cat", from = "2007-01-01", friendly = T): unused argument (friendly = T)
```

```r
  # throw out extreme values
  no_outlier <- 
  cats$count < 
    quantile(cats$count, na.rm=T, 0.99) & 
  cats$count > 
    quantile(cats$count, na.rm=T, 0.01)  
```

```
## Error in eval(expr, envir, enclos): object 'cats' not found
```

```r
plot( cats[no_outlier,], 
      col="black",
      ylim=c(0, max(cats[no_outlier,]$count)),
      main="Cats' Wikipedia Attention",
      ylab="views per day", xlab="time", type="h")
```

```
## Error in plot(cats[no_outlier, ], col = "black", ylim = c(0, max(cats[no_outlier, : object 'cats' not found
```

```r
soo_2012_13 <- wp_year(cats$date)== 2012 | wp_year(cats$date)== 2013 
```

```
## Error in wp_year(cats$date): object 'cats' not found
```

```r
cats_model  <- lm(count ~ date + date^2 + date^3 + soo_2012_13 ,data=cats)
```

```
## Error in is.data.frame(data): object 'cats' not found
```

```r
cats_smooth <- data.frame(date=cats$date, count_smooth=predict(cats_model))
```

```
## Error in data.frame(date = cats$date, count_smooth = predict(cats_model)): object 'cats' not found
```

```r
lines(cats_smooth,col=rgb(1,0,0,0.5),lwd=5)
```

```
## Error in lines(cats_smooth, col = rgb(1, 0, 0, 0.5), lwd = 5): object 'cats_smooth' not found
```

... and triumphantly can conclude: 

**Cats' popularity is in decline overal and more so cats are soooo old fashioned 2012 and 2013.**.

### Ebola

Or we can study how the desire to inform oneself about Ebola varies over time:


```r
ebola_en <- wp_trend("Ebola", from="2008-01-01", friendly=T)
```

```
## Error in wp_trend("Ebola", from = "2008-01-01", friendly = T): unused argument (friendly = T)
```

```r
plot( ebola_en, 
      ylim=c(0, max(ebola_en$count)),
      main="Ebola's Wikipedia Attention",
      ylab="views per day", xlab="time",
      type="l")
```

```
## Error in plot(ebola_en, ylim = c(0, max(ebola_en$count)), main = "Ebola's Wikipedia Attention", : object 'ebola_en' not found
```

```r
lines(ebola_en)
```

```
## Error in lines(ebola_en): object 'ebola_en' not found
```

Which unsurprisingly peaks in 2014 with the Ebola outbreak in Western Africa. 

Using the language option we can also study if media attentions differ between languages / cultures by comparing the attention patterns for the English Wikipedia with those for the German Wikipedia:



```r
ebola_de <- wp_trend("Ebola", lang="de", from="2008-01-01", friendly=T)
```

```
## Error in wp_trend("Ebola", lang = "de", from = "2008-01-01", friendly = T): unused argument (friendly = T)
```


```r
plot( ebola_en, 
      ylim=c(0, max(ebola_en$count)),
      main="Ebola's Wikipedia Attention",
      ylab="views per day", xlab="time",
      type="n")
```

```
## Error in plot(ebola_en, ylim = c(0, max(ebola_en$count)), main = "Ebola's Wikipedia Attention", : object 'ebola_en' not found
```

```r
lines(ebola_en, col="red")
```

```
## Error in lines(ebola_en, col = "red"): object 'ebola_en' not found
```

```r
lines(ebola_de, col=rgb(0,0,0,0.7))
```

```
## Error in lines(ebola_de, col = rgb(0, 0, 0, 0.7)): object 'ebola_de' not found
```

```r
legend("topleft", 
       c("en", "de"), 
       col=c("red", rgb(0,0,0,0.7)),
       lty=1
       )
```

```
## Error in strwidth(legend, units = "user", cex = cex, font = text.font): plot.new has not been called yet
```

The similarities are striking. 


## Package specific time functions

Because data received from stad.grok.se is not always clean -- one might e.g. get dates like `2008-02-30` which is impossible -- the package has its own error robust date functions.

Furthermore, these functions work on all kinds of date formats like Date, numeric, character, POSIXlt, and POSIXct without having to make transformations all the time. The downside of this implementation is that edecuted guesses have to be made: 

  - character data is assumed to be given in format "yyyy-mm-dd" like in 2015-02-19
  - numerics are assumed to be days since `1970-01-01` (which is R's default anyways)
  
To conclude, wikipediatrend time functions are easy to use efficient little helpers to work with the data provided by the package but are to be used with caution outside the package due the fact that convenience is based on educated guesses that can go wrong. 

1. `wp_date()`
  - is equivalent to `as.Date()` from the base package:
    - except that it will return `NA` in response to rediculous dates instead of an error
    - it will always assume `1970-01-01` to be the origin whenever numerics have to be tranfered to date but no origin is supplied 
2. `wp_day()`
  - extracts the day from a data
3. `wp_month()`
  - extracts the month from a date
4. `wp_month()`
  - extracts the year from a date
5. `wp_wday()`
  - extracts week day from a data (1 for Mondays, 2 for Tuesdays, ...)
6. `wp_yearmonth()`
  - extracts the year and month of a date and glues them together -- e.g. `2014-03-05` gets transformed to `"201403"`



# Credits

- Parts of the package's code have been shamelessly copied and modified from R base package written by R core team. This concerns the `wp_date()` generic and its methods and is detailed in the help files. 



<!-- http://www.tandfonline.com/doi/pdf/10.1080/10410236.2011.571759 -->



