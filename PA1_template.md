Exploratory analysis on Activity monitoring data
================
Kishibe Rohan
14th July,2020

## Overview

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up. These type of devices are part of the
“quantified self” movement – a group of enthusiasts who take
measurements about themselves regularly to improve their health, to find
patterns in their behavior, or because they are tech geeks. But these
data remain under-utilized both because the raw data are hard to obtain
and there is a lack of statistical methods and software for processing
and interpreting the data.

This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012 and
include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

  - Dataset: [Activity monitoring
    data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval (missing values are
coded as 𝙽𝙰) date: The date on which the measurement was taken in
YYYY-MM-DD format interval: Identifier for the 5-minute interval in
which measurement was taken The dataset is stored in a
comma-separated-value (CSV) file and there are a total of 17,568
observations in this dataset.

## Loading and exploring the dataset

Make sure the .csv file is in the same directory

``` r
library(ggplot2)
activity <- read.csv("activity.csv")
```

``` r
dim(activity)
names(activity)
head(activity)
str(activity)
```

\#\#Histogram of the total number of steps taken each day

``` r
#Fetching the required data 
dailySteps <- aggregate(steps ~ date,activity,sum,na.rm=TRUE)

#Plot the histogram

ggplot(dailySteps, aes(x =
                         steps)) +
    geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

![](PA1_template_files/figure-gfm/unnamed-chunk-3-1.png)<!-- --> \#\#
Mean and median of daily total steps

``` r
meanDailySteps <- mean(dailySteps$steps,na.rm = TRUE)
meanDailySteps
```

    ## [1] 10766.19

``` r
medianDailySteps <- median(dailySteps$steps,na.rm=TRUE)
medianDailySteps
```

    ## [1] 10765

## Average daily activity pattern

1.  Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = “𝚕”) of the 5-minute interval
    (x-axis) and the average number of steps taken, averaged across all
    days (y-axis)

<!-- end list -->

``` r
stepsPerInterval <- aggregate(steps~interval,data=activity,mean,na.rm=TRUE)

ggplot(stepsPerInterval,aes(x=interval,y=steps))+geom_line(color="blue",size = 1) + labs(title = "Average Daily Steps", x = "Interval", y = "Steps")
```

![](PA1_template_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

2.  Which 5-minute interval, on average across all the days in the
    dataset, contains the maximum number of steps?

<!-- end list -->

``` r
maxStepsInterval <- stepsPerInterval[which.max(stepsPerInterval$steps),]$interval

maxStepsInterval
```

    ## [1] 835

## Imputing missing values

1.  Calculate and report the total number of missing values in the
    dataset (i.e. the total number of rows with 𝙽𝙰s)

<!-- end list -->

``` r
totalMissing <- sum(is.na(activity$steps))
totalMissing
```

    ## [1] 2304

2.Devise a strategy for filling in all of the missing values in the
dataset. The strategy does not need to be sophisticated.

Strategy used : Fill in all the missing values in the dataset with the
mean per interval.

``` r
 meanStepsPerInterval <- function(interval)
 {
     stepsPerInterval[stepsPerInterval$interval==interval,]$steps
 }
```

3.  Create a new dataset that is equal to the original dataset but with
    the missing data filled in.

<!-- end list -->

``` r
 tidyData <- activity
 for(i in 1:nrow(tidyData))
 {
 if(is.na(tidyData[i,]$steps))
 {
   tidyData[i,]$steps <- meanStepsPerInterval(tidyData[i,]$interval)
 }
 }
```

4.  Make a histogram of the total number of steps taken each day and
    Calculate and report the mean and median total number of steps taken
    per day

<!-- end list -->

``` r
stepsPerDayNoNA <- aggregate(steps ~ date, tidyData, sum)



 ggplot(stepsPerDayNoNA, aes(x =
                         steps)) +
    geom_histogram(fill = "blue", binwidth = 1000) +
    labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

![](PA1_template_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

The mean didn’t change after the replacements of NAs, the median changed
about 0.1% of the original value.

## Are there differences in activity patterns between weekdays and weekends?

1.  Create a new factor variable in the dataset with two levels –
    “weekday” and “weekend” indicating whether a given date is a
    weekday or weekend day.

<!-- end list -->

``` r
tidyData$date <- as.Date(strptime(tidyData$date,format ="%Y-%m-%d" ))
tidyData$day <- weekdays(tidyData$date)

for (i in 1:nrow(tidyData)) {
    if (tidyData[i,]$day %in%
        c("Saturday","Sunday")) {
        tidyData[i,]$day<-"weekend"
    }
    else{
        tidyData[i,]$day<-"weekday"
    }
}

stepsByDay <- aggregate(tidyData$steps ~ tidyData$interval + tidyData$day,tidyData,mean)
```

2.  Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = “𝚕”) of
    the 5-minute interval (x-axis) and the average number of steps
    taken, averaged across all weekday days or weekend days (y-axis).

<!-- end list -->

``` r
library(lattice)
names(stepsByDay) <- c("interval","day","steps")
xyplot(steps~interval|day,stepsByDay,type="l",layout=c(1,2),xlab="Interval",ylab="Steps")
```

![](PA1_template_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->
