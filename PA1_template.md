---
title: 'Reproducible Research: Peer Assessment 1'
output: 
  html_document:
    keep_md: true
---
## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


## Loading and preprocessing the data

The data can be downloaded here [Activity monitoring data.](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

- **steps:** Number of steps taking in a 5-minute interval (missing values are coded as *NA*).

- **date:** The date on which the measurement was taken in *YYYY-MM-DD* format.

- **interval:** Identifier for the 5-minute interval in which measurement was taken.

The dataset is stored in a comma separated value (*CSV*) file and there are a total of 17,568 observations.


```r
library(curl)
```

```
## Using libcurl 7.64.1 with Schannel
```

```r
## DOWNLOAD THE DATA SET

nombreArch <- "repdata_data_activity.zip"

## Checks if the archive already exists
if(!file.exists(nombreArch)) {
    archURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(archURL, nombreArch, method = "curl")
}

## Check if the folder exists
if (!file.exists("repdata_data_activity")) {
    unzip(nombreArch)
}
```


```r
library(ggplot2)
actividad <- read.csv("repdata_data_activity/activity.csv")

actividad$date <- as.Date.factor(actividad$date)

diaSemana <- weekdays(actividad$date)
actividad <- cbind(actividad, diaSemana)
summary(actividad)
```

```
##      steps             date               interval       diaSemana        
##  Min.   :  0.00   Min.   :2012-10-01   Min.   :   0.0   Length:17568      
##  1st Qu.:  0.00   1st Qu.:2012-10-16   1st Qu.: 588.8   Class :character  
##  Median :  0.00   Median :2012-10-31   Median :1177.5   Mode  :character  
##  Mean   : 37.38   Mean   :2012-10-31   Mean   :1177.5                     
##  3rd Qu.: 12.00   3rd Qu.:2012-11-15   3rd Qu.:1766.2                     
##  Max.   :806.00   Max.   :2012-11-30   Max.   :2355.0                     
##  NA's   :2304
```

## What is mean total number of steps taken per day?


```r
actividad_pasos <- with(actividad, aggregate(steps, by = list(date), FUN = sum, na.rm = T))

names(actividad_pasos) <- c("dates", "steps")

hist(actividad_pasos$steps, main = "Total number of steps taken per day", xlab = "Total steps taken per day", col = "red", ylim = c(0, 20), breaks = seq(0, 25000, by = 2500))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Mean of steps taken per day.


```r
mean(actividad_pasos$steps)
```

```
## [1] 9354.23
```
Median of steps taken per day.

```r
median(actividad_pasos$steps)
```

```
## [1] 10395
```
## What is the average daily activity pattern?

1. Time series plot (*type=“l”*) of the 5-minute interval (*x-axis*) and the average number of steps taken, averaged across all days (*y-axis*).


```r
actividad.diaria.media <- aggregate(actividad$steps, by = list(actividad$interval), FUN = mean, na.rm = T)

names(actividad.diaria.media) <- c("interval", "mean")

plot(actividad.diaria.media$interval, actividad.diaria.media$mean, type = "l", xlab = "Interval", ylab = "Average number of steps", main = "Average number of steps per interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

2. 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps.


```r
actividad.diaria.media[which.max(actividad.diaria.media$mean),]$interval
```

```
## [1] 835
```

## Imputing missing values

There are a number of days/intervals where there are missing values (coded as *NA*). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (*i.e*. the total number of rows with *NAs*).


```r
sum(is.na(actividad$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
pasos.limpios <- actividad.diaria.media$mean[match(actividad$interval, actividad.diaria.media$interval)]
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
actividad.limpia <- transform(actividad, steps = ifelse(is.na(actividad$steps), yes = pasos.limpios, no = actividad$steps))

pasos.limpios.totales <- aggregate(steps ~ date, actividad.limpia, sum)

names(pasos.limpios.totales) <- c("date", "daily.steps")
```

4. Make a histogram of the total number of steps taken each day, calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
hist(pasos.limpios.totales$daily.steps, col = "purple", xlab = "Total steps per day", ylim = c(0, 30), main = "Total number of steps taken each day", breaks = seq(0, 25000, by = 2500))
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Mean of the total number of steps taken per day.


```r
mean(pasos.limpios.totales$daily.steps)
```

```
## [1] 10766.19
```
Median of the total number of steps taken per day.

```r
median(pasos.limpios.totales$daily.steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
actividad$fecha <- sapply(as.Date.factor(actividad$date), function(x) {
    if (weekdays(x) == "sabado" | weekdays(x) == "domingo") {
        y <- "Weekend"
    } else {
        y <- "Weekday"
    }
    y
})
```

2. A panel plot containing a time series plot (*type=“l”*) of the 5-minute interval (*x-axis*) and the average number of steps taken, averaged across all weekday days or weekend days (*y-axis*).


```r
actividad.fecha <- aggregate(steps ~ interval + fecha, actividad, mean, na.rm = T)

ggplot(actividad.fecha, aes(x = interval, y = steps, color = fecha)) + geom_line() + labs(title = "Average daily stpes by date type", x = "Interval", y = "Average number of steps") + facet_wrap(~ fecha, ncol= 1, nrow = 2)
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

