# Reproducible Research: Peer Assessment 1

In _Peer Assessment 1_, we will make use of the following packages:


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(tidyr)
library(data.table)
```

```
## 
## Attaching package: 'data.table'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     between, last
```

```r
library(dygraphs)
library(ggplot2)
library(ggthemr)
library(knitr)
```


## Loading and preprocessing the data

We begin the data loading and preprocessing section by verifying the existence of the activity.zip file within the directory.  The zip file is extracted if activity.csv is not already in the directory.



```r
## Check if data is available.  Download and unzip if it is not.
fname <- "activity.csv"
zipname <- "activity.zip"
zipurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"

if(!file.exists(fname))
{
        if(!file.exists(zipname))  
        {
                download.file(zipurl,zipname)
        }
        unzip(zipname)
}
```

Now we load the data using `fread()`.


```r
activity <- fread("activity.csv")
```


## What is mean total number of steps taken per day?

We begin by using the summarize and group by facilities of the `dplyr` package to calculate daily totals:


```r
activity.total <- activity %>%
                  group_by(date) %>%
                  summarize(total.steps=sum(steps,na.rm=TRUE)) 
```

Next we plot the histogram of daily totals using `gplot2`.  The `ggthemr` package is used to make a graph with a nicely designed palette:


```r
ggthemr('fresh')

activity.total %>% ggplot() +
                  geom_histogram(aes(total.steps), 
                                 binwidth = 2000,
                                 alpha=0.75,
                                 color=ggthemr:::get_themr()$palette$background) +
                  ggtitle("Histogram of Total Daily Steps") +
                  xlab("Total Steps") +
                  ylab("Count of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

We again use the `dplyr` package to find the mean and median steps per day--note that we ungroup prior to summarizing.


```r
activity.mean <- activity.total %>%
                  ungroup() %>%
                  summarize(mean.steps=round(mean(total.steps),digits=2),
                            median.steps=median(total.steps))
```

We find that the mean total number of steps per day is 9354.23 and the median is 10395.

## What is the average daily activity pattern?

We begin by calculating averages by `interval`.


```r
activity.pattern <- activity %>%
                  group_by(interval) %>%
                  summarize(mean.steps=mean(steps,na.rm=TRUE)) 
```

We now utilize the `dygraphs` package to create an interactive plot of activity by interval.  We have added in a `Roller` in the lower left hand corner which can be used to smooth the series using a backward-looking moving average.  The default value is set to `1`, corresponding to the interval-lvel observation.  A setting of `12` will match to an hourl moving average, given the interval size of 5 minutes.


```r
# Get color to match with histogram:
swtch <- ggthemr:::get_themr()$palette$swatch

dygraph(activity.pattern, main = "Daily Step Pattern") %>% 
      dyOptions(colors=swtch[2]) %>%
      dyRoller(rollPeriod=1) %>%
      dyRangeSelector()
```

<!--html_preserve--><div id="htmlwidget-6871" style="width:672px;height:480px;" class="dygraphs html-widget"></div>
<script type="application/json" data-for="htmlwidget-6871">{"x":{"attrs":{"title":"Daily Step Pattern","labels":["interval","mean.steps"],"legend":"auto","retainDateWindow":false,"axes":{"x":{"pixelsPerLabel":60,"drawAxis":true},"y":{"drawAxis":true}},"stackedGraph":false,"fillGraph":false,"fillAlpha":0.15,"stepPlot":false,"drawPoints":false,"pointSize":1,"drawGapEdgePoints":false,"connectSeparatedPoints":false,"strokeWidth":1,"strokeBorderColor":"white","colors":["#65ADC2"],"colorValue":0.5,"colorSaturation":1,"includeZero":false,"drawAxesAtZero":false,"logscale":false,"axisTickSize":3,"axisLineColor":"black","axisLineWidth":0.3,"axisLabelColor":"black","axisLabelFontSize":14,"axisLabelWidth":60,"drawGrid":true,"gridLineWidth":0.3,"rightGap":5,"digitsAfterDecimal":2,"labelsKMB":false,"labelsKMG2":false,"labelsUTC":false,"maxNumberWidth":6,"animatedZooms":false,"mobileDisableYTouch":true,"showRoller":true,"rollPeriod":1,"showRangeSelector":true,"rangeSelectorHeight":40,"rangeSelectorPlotFillColor":" #A7B1C4","rangeSelectorPlotStrokeColor":"#808FAB","interactionModel":"Dygraph.Interaction.defaultModel"},"annotations":[],"shadings":[],"events":[],"format":"numeric","data":[[0,5,10,15,20,25,30,35,40,45,50,55,100,105,110,115,120,125,130,135,140,145,150,155,200,205,210,215,220,225,230,235,240,245,250,255,300,305,310,315,320,325,330,335,340,345,350,355,400,405,410,415,420,425,430,435,440,445,450,455,500,505,510,515,520,525,530,535,540,545,550,555,600,605,610,615,620,625,630,635,640,645,650,655,700,705,710,715,720,725,730,735,740,745,750,755,800,805,810,815,820,825,830,835,840,845,850,855,900,905,910,915,920,925,930,935,940,945,950,955,1000,1005,1010,1015,1020,1025,1030,1035,1040,1045,1050,1055,1100,1105,1110,1115,1120,1125,1130,1135,1140,1145,1150,1155,1200,1205,1210,1215,1220,1225,1230,1235,1240,1245,1250,1255,1300,1305,1310,1315,1320,1325,1330,1335,1340,1345,1350,1355,1400,1405,1410,1415,1420,1425,1430,1435,1440,1445,1450,1455,1500,1505,1510,1515,1520,1525,1530,1535,1540,1545,1550,1555,1600,1605,1610,1615,1620,1625,1630,1635,1640,1645,1650,1655,1700,1705,1710,1715,1720,1725,1730,1735,1740,1745,1750,1755,1800,1805,1810,1815,1820,1825,1830,1835,1840,1845,1850,1855,1900,1905,1910,1915,1920,1925,1930,1935,1940,1945,1950,1955,2000,2005,2010,2015,2020,2025,2030,2035,2040,2045,2050,2055,2100,2105,2110,2115,2120,2125,2130,2135,2140,2145,2150,2155,2200,2205,2210,2215,2220,2225,2230,2235,2240,2245,2250,2255,2300,2305,2310,2315,2320,2325,2330,2335,2340,2345,2350,2355],[1.71698113207547,0.339622641509434,0.132075471698113,0.150943396226415,0.0754716981132075,2.09433962264151,0.528301886792453,0.867924528301887,0,1.47169811320755,0.30188679245283,0.132075471698113,0.320754716981132,0.679245283018868,0.150943396226415,0.339622641509434,0,1.11320754716981,1.83018867924528,0.169811320754717,0.169811320754717,0.377358490566038,0.264150943396226,0,0,0,1.13207547169811,0,0,0.132075471698113,0,0.226415094339623,0,0,1.54716981132075,0.943396226415094,0,0,0,0,0.207547169811321,0.622641509433962,1.62264150943396,0.584905660377358,0.490566037735849,0.0754716981132075,0,0,1.18867924528302,0.943396226415094,2.56603773584906,0,0.339622641509434,0.358490566037736,4.11320754716981,0.660377358490566,3.49056603773585,0.830188679245283,3.11320754716981,1.11320754716981,0,1.56603773584906,3,2.24528301886792,3.32075471698113,2.9622641509434,2.09433962264151,6.05660377358491,16.0188679245283,18.3396226415094,39.4528301886792,44.4905660377358,31.4905660377358,49.2641509433962,53.7735849056604,63.4528301886792,49.9622641509434,47.0754716981132,52.1509433962264,39.3396226415094,44.0188679245283,44.1698113207547,37.3584905660377,49.0377358490566,43.811320754717,44.377358490566,50.5094339622642,54.5094339622642,49.9245283018868,50.9811320754717,55.6792452830189,44.3207547169811,52.2641509433962,69.5471698113208,57.8490566037736,56.1509433962264,73.377358490566,68.2075471698113,129.433962264151,157.528301886792,171.150943396226,155.396226415094,177.301886792453,206.169811320755,195.924528301887,179.566037735849,183.396226415094,167.018867924528,143.452830188679,124.037735849057,109.11320754717,108.11320754717,103.716981132075,95.9622641509434,66.2075471698113,45.2264150943396,24.7924528301887,38.7547169811321,34.9811320754717,21.0566037735849,40.5660377358491,26.9811320754717,42.4150943396226,52.6603773584906,38.9245283018868,50.7924528301887,44.2830188679245,37.4150943396226,34.6981132075472,28.3396226415094,25.0943396226415,31.9433962264151,31.3584905660377,29.6792452830189,21.3207547169811,25.5471698113208,28.377358490566,26.4716981132075,33.4339622641509,49.9811320754717,42.0377358490566,44.6037735849057,46.0377358490566,59.188679245283,63.8679245283019,87.6981132075472,94.8490566037736,92.7735849056604,63.3962264150943,50.1698113207547,54.4716981132075,32.4150943396226,26.5283018867925,37.7358490566038,45.0566037735849,67.2830188679245,42.3396226415094,39.8867924528302,43.2641509433962,40.9811320754717,46.2452830188679,56.4339622641509,42.7547169811321,25.1320754716981,39.9622641509434,53.5471698113208,47.3207547169811,60.811320754717,55.7547169811321,51.9622641509434,43.5849056603774,48.6981132075472,35.4716981132075,37.5471698113208,41.8490566037736,27.5094339622642,17.1132075471698,26.0754716981132,43.622641509434,43.7735849056604,30.0188679245283,36.0754716981132,35.4905660377358,38.8490566037736,45.9622641509434,47.7547169811321,48.1320754716981,65.3207547169811,82.9056603773585,98.6603773584906,102.11320754717,83.9622641509434,62.1320754716981,64.1320754716981,74.5471698113208,63.1698113207547,56.9056603773585,59.7735849056604,43.8679245283019,38.5660377358491,44.6603773584906,45.4528301886792,46.2075471698113,43.6792452830189,46.622641509434,56.3018867924528,50.7169811320755,61.2264150943396,72.7169811320755,78.9433962264151,68.9433962264151,59.6603773584906,75.0943396226415,56.5094339622642,34.7735849056604,37.4528301886792,40.6792452830189,58.0188679245283,74.6981132075472,85.3207547169811,59.2641509433962,67.7735849056604,77.6981132075472,74.2452830188679,85.3396226415094,99.4528301886792,86.5849056603774,85.6037735849057,84.8679245283019,77.8301886792453,58.0377358490566,53.3584905660377,36.3207547169811,20.7169811320755,27.3962264150943,40.0188679245283,30.2075471698113,25.5471698113208,45.6603773584906,33.5283018867925,19.622641509434,19.0188679245283,19.3396226415094,33.3396226415094,26.811320754717,21.1698113207547,27.3018867924528,21.3396226415094,19.5471698113208,21.3207547169811,32.3018867924528,20.1509433962264,15.9433962264151,17.2264150943396,23.4528301886792,19.2452830188679,12.4528301886792,8.0188679245283,14.6603773584906,16.3018867924528,8.67924528301887,7.79245283018868,8.13207547169811,2.62264150943396,1.45283018867925,3.67924528301887,4.81132075471698,8.50943396226415,7.07547169811321,8.69811320754717,9.75471698113208,2.20754716981132,0.320754716981132,0.113207547169811,1.60377358490566,4.60377358490566,3.30188679245283,2.84905660377358,0,0.830188679245283,0.962264150943396,1.58490566037736,2.60377358490566,4.69811320754717,3.30188679245283,0.641509433962264,0.226415094339623,1.07547169811321]],"fixedtz":false,"tzone":""},"evals":["attrs.interactionModel"],"jsHooks":[]}</script><!--/html_preserve-->

Analysis of the graph suggests that average step activity is effectively `0` between the hours of `12 am` and `4 am`.  Activity increases to a rough peak in the `8-9 am` hour and lower localized peaks around `12 pm`, `4 pm`, and `7 pm`.  Activity then decreases through the evening and night.   

**Peak average step activity of `206` steps occurs in the `8:35 am` interval.**


## Imputing missing values

THe assignment reccommends a simple method, such as mean or median value imputation be used.  In particular both per-day and per-interval measures are suggested.  A quick examination of the main `activity` reveals that daily will not work:


```r
# Table of activity:
kable(head(activity),format="markdown")
```



| steps|date       | interval|
|-----:|:----------|--------:|
|    NA|2012-10-01 |        0|
|    NA|2012-10-01 |        5|
|    NA|2012-10-01 |       10|
|    NA|2012-10-01 |       15|
|    NA|2012-10-01 |       20|
|    NA|2012-10-01 |       25|

```r
# All values are missing!
sum(!is.na(activity[date=="2012-10-01"]$steps))
```

```
## [1] 0
```

Given the above, we have elected to use a per-interval measure for our imputation. We utilize `dplyr` to remove the `NA` records from activity, then join them with the means calculated from the daily pattern identified in the last segment using an `inner join`.  THese are then matched back against `activity` using a `row bind`. 


```r
activity.full <- activity %>%
                  filter(!is.na(steps))

activity.imputed <- activity %>% 
                  filter(is.na(steps)) %>%
                  select(date,interval) %>%
                  inner_join(activity.pattern,by="interval") %>%
                  rename(steps=mean.steps) %>%
                  select(steps,date,interval) %>%
                  bind_rows(activity.full) %>%
                  arrange(date,interval)
```

We now proceed to reproduce the histogram, mean, and median:


```r
ggthemr('fresh')
activity.imputed.tot <- activity.imputed %>% 
                        group_by(date) %>%
                        summarize(total.steps=sum(steps,na.rm=TRUE))
      
      
activity.imputed.tot %>%ggplot() +
                        geom_histogram(aes(total.steps), 
                                       binwidth = 2000,
                                       alpha=0.75,
                                       color="white") +
                        ggtitle("Histogram of Total Daily Steps (Imputed)") +
                        xlab("Total Steps") +
                        ylab("Count of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

We again use the `dplyr` package to find the mean and median steps per day--note that we ungroup prior to summarizing.


```r
activity.imputed.mn <- activity.imputed.tot %>%
                        ungroup() %>%
                        summarize(mean.steps.imputed=mean(total.steps),
                                  median.steps.imputed=median(total.steps)) %>%
                        bind_cols(activity.mean) %>%
                        mutate(mean.diff=mean.steps.imputed-mean.steps,
                               median.diff=median.steps-median.steps.imputed) %>%
                        select(mean.steps,
                               mean.steps.imputed,
                               mean.diff,
                               median.steps,
                               median.steps.imputed,
                               median.diff) 
                        
kable(activity.imputed.mn,format="markdown")
```



| mean.steps| mean.steps.imputed| mean.diff| median.steps| median.steps.imputed| median.diff|
|----------:|------------------:|---------:|------------:|--------------------:|-----------:|
|    9354.23|           10766.19|  1411.959|        10395|             10766.19|   -371.1887|

We can see that the mean value changed by 1411.9586792 and the median changed by -371.1886792 due to the imputation process.


## Are there differences in activity patterns between weekdays and weekends?

First we use `weekdays()` to identify which dates are Saturdays or Sundays.


```r
activity.wknd<- activity %>%
                  mutate(wkdy=weekdays(as.Date(date))) 

activity.wknd[wkdy %in% c("Saturday","Sunday")]$wkdy <- "weekend"
activity.wknd[wkdy != "weekend"]$wkdy <- "weekday"
```

We use `dplyr` to calculate the patterns.


```r
activity.pattern.wknd <- activity.wknd %>%
                  group_by(interval,wkdy) %>%
                  summarize(mean.steps=mean(steps,na.rm=TRUE)) 
```

We now utilize `ggplot2` to plot a panel comparison of the two patterns.


```r
activity.pattern.wknd %>% ggplot(aes(x=interval, 
                                     y=mean.steps)) +
                              facet_wrap(~wkdy,
                                         nrow = 2,
                                         ncol = 1) +
                              geom_line() +
                              xlab("Time of Day (5-Minute Interval)") +
                              ylab("Mean Steps") +
                              geom_smooth(size=.5,
                                          linetype=2, 
                                          se=FALSE) +
                              no_legend()
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

We can see from above, that activity shifts on the weekend. There is higher activity in the afternoon-to-early-evening when compared to weekdays.  The weekdays tend to have an early-morning peak, which is not reached on the weekends.  THis could be due to concentrated pre-work exercise occuring during the week, while the weekend offers more freedom in terms of time available for activity.
