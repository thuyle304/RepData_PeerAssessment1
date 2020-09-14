---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## 1. Loading and preprocessing the data

```r
mydata <- read.csv("/Users/thuyle/Downloads/activity.csv")
```


## 2. What is mean total number of steps taken per day?
### The firsts step is to group the data by date. Then calculate the total number of steps taken per day

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
groupdata <- group_by(mydata, interval)
sdata <- summarise(groupdata, totalsteps=sum(steps,na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```
### The second step is to make a histogram of the total number of steps taken per day

```r
library("ggplot2")
p1 <- hist(sdata$totalsteps, plot=TRUE)
```

![](PA1_template_files/figure-html/p1-1.png)<!-- -->
### The third step is to caculate the mean and median of the total number of steps taken per day

```r
mean(sdata$totalsteps)
```

```
## [1] 1981.278
```

```r
median(sdata$totalsteps)
```

```
## [1] 1808
```
## 3. What is the average daily activity pattern?
### Calculate the average number of steps taken in 5 minute interval across all days

```r
groupdata2 <- group_by(mydata, interval)
sdata2 <- summarise(groupdata2, ave_step=sum(steps,na.rm = TRUE))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```
### Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
p2 <- ggplot(sdata2, aes(x=interval, y=ave_step)) + geom_line(size = 0.5, linetype = 1, col= "darkblue") + geom_smooth(method = "loess", span = 0.2)
p2 + labs(title="Time series plot of 5 minutes interval and average steps acrossed all days",
       x ="5-minute interval", y = "Average steps across all days")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

![](PA1_template_files/figure-html/p2-1.png)<!-- -->
### The 5-minute interval contains the maximum number of steps in average

```r
sdata2[sdata2$ave_step == max(sdata2$ave_step),1]
```

```
## # A tibble: 1 x 1
##   interval
##      <int>
## 1      835
```
## 4. Imputing missing values
### Total number of missing values in the dataset (i.e.the total number of rows with NAs)

```r
length(mydata[is.na(mydata$steps), 1])
```

```
## [1] 2304
```
### The mean for that 5-minute interval is calculate and used to fill in all of the missing values in the dataset.  

```r
ndata3 <- mutate(groupdata2, meansteps=mean(steps, na.rm = TRUE))
ndata3$steps[is.na(ndata3$steps)] = ndata3$meansteps
```

```
## Warning in ndata3$steps[is.na(ndata3$steps)] = ndata3$meansteps: number of items
## to replace is not a multiple of replacement length
```
### Make a histogram of the total number of steps taken each day and. 

```r
groupdata3 <- group_by(mydata, date)
sdata3 <- summarise(groupdata3, totalsteps=sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
hist(sdata3$totalsteps, plot=TRUE)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
### Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
mean(sdata3$totalsteps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(sdata3$totalsteps, na.rm=TRUE)
```

```
## [1] 10765
```
### The result for the mean and median total number of steps taken per day is the same as the first part where missing data was removed in the first place. This shows that imputing missing data did not have any impact on the estimate of total daily number of steps

## 5. Are there differences in activity patterns between weekdays and weekends?
### Add a variable which shows the weekday correspond to each date in the data, then catergorize them into two types: "weekday" and "weekend"

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```

```r
sdata4 <- mutate(mydata, day=wday(mydata$date))
sdata4$day[sdata4$day <= 6] = "weekday" 
sdata4$day[sdata4$day != "weekday"] =  "weekend"
```
### Calculate average number of steps in 5-minute interval across all days,then make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)


```r
wdata <-  group_by(sdata4, day, interval)
wdata <- summarise(wdata, ave_ws=mean(steps, na.rm=TRUE))
```

```
## `summarise()` regrouping output by 'day' (override with `.groups` argument)
```


```r
p3 <- ggplot(wdata, aes(x=interval, y=ave_ws)) + geom_line(size = 0.5, 
      linetype = 1, col= "darkblue") 
p3 + facet_wrap(~day) + labs(title="Time series plot of the 5-minute interval and average steps acrossed all days", x ="5-minute interval", y = "Number of average steps acrossed all days")
```

![](PA1_template_files/figure-html/p3-1.png)<!-- -->

### The graphs show that there are more activities in the weekend than the weekdays. However similarly, the time when the most activities happened is around 9am.
