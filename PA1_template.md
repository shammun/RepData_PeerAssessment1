# Peer Assessment 1

## Get the data

```r
setwd("D:/Downloads/Downloads Old/Downloads/Data Science Specialization")
data <- read.csv("activity.csv")
```

## Calculation of mean total number of steps taken per day.

```r
library(ggplot2)
steps.total <- tapply(data$steps, data$date, sum, na.rm=TRUE)
qplot(steps.total, binwidth=500, xlab="Total number of steps on each day",ylab="COunt or Frequency")
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1.png) 

```r
mean(steps.total, na.rm=TRUE)
```

```
## [1] 9354
```

```r
median(steps.total, na.rm=TRUE)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

```r
library(ggplot2)
summary.by.average <- aggregate(x=list(steps=data$steps), by=list(interval=
  data$interval), mean, na.rm=TRUE)
ggplot(summary.by.average, aes(interval, steps)) +
  geom_line() +
  labs(x="Interval of 5-minutes") +
  labs(title="Average number of steps taken")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

On average across all the days in the dataset, the 5-minute interval contains
the maximum number of steps?

```r
summary.by.average[which.max(summary.by.average$steps),]
```

```
##     interval steps
## 104      835 206.2
```

## Imputing missing values

Note that there are many days/intervals where there are missing values (coded as `NA`). The presence of missing days may introduce bias into some calculations or summaries of the data.


```r
missing <- is.na(data$steps)
table(missing)
```

```
## missing
## FALSE  TRUE 
## 15264  2304
```

All of the missing values will be replaced with mean value for the corresponding 5-minute interval.


```r
# Replace each missing value with the mean value of its 5-minute interval
replaced.value <- function(steps, interval) {
  replaced <- NA
  if (!is.na(steps))
    replaced <- c(steps)
  else
    replaced <- (summary.by.average[summary.by.average$interval==interval, "steps"])
  return(replaced)
}
modified.data <- data
modified.data$steps <- mapply(replaced.value, modified.data$steps, modified.data$interval)
```
Now, make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.


```r
total.steps <- tapply(modified.data$steps, modified.data$date, sum)
qplot(total.steps, binwidth=500, xlab="Total number of steps taken each day")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

```r
mean(total.steps)
```

```
## [1] 10766
```

```r
median(total.steps)
```

```
## [1] 10766
```

We now have higher mean and median values after imputing missing data. This is due to the replacement of 'NA' values with mean steps which was previously replaced by 0. So we now have higher values.

## Are there differences in activity patterns between weekdays and weekends?
We create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
indicator<- function(date) {
  day <- weekdays(date)
  if (day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"))
    return("weekday")
  else if (day %in% c("Saturday", "Sunday"))
    return("weekend")
  else
    stop("Invalid!! Stop!!")
}
```

Now, let's make a panel plot containing plots of average number of steps taken
on weekdays and weekends.

```r
modified.data$date <- as.Date(modified.data$date)
modified.data$day <- sapply(modified.data$date, indicator)
summary.mean <- aggregate(steps ~ interval + day, data=modified.data, mean)
ggplot(summary.mean, aes(interval, steps)) + geom_line() + facet_grid(day ~ .) +
    labs(x="Interval of 5-minutes") +
    labs(title="Number of steps taken")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 
