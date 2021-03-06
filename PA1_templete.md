Reproducible Research Peer asginment 1(Couresera_Datascience)
=============================================================

# Loading and preprocessing the data

1. Load the data

```r
library(ggplot2)
library(plyr)
setwd("~/Desktop/datascience")
activity <- read.csv("activity.csv", colClasses = c("numeric", "Date", "numeric"),
                     header=TRUE, na.strings="NA")
```

2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
perday <- aggregate(steps ~ date, activity, sum)
perday <- cbind(perday, label=rep("w/ NA", nrow(perday)))
```

# What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

```r
ggplot(perday, aes(x=steps)) +geom_histogram(binwidth=1000, color="white") + labs(x="Steps per day", y="Frequency")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

2. Calculate and report the mean and median total number of steps taken per day

```r
activity_mean <- mean(perday$steps, na.rm=TRUE)
activity_median <- median(perday$steps, na.rm=TRUE)
```
The mean is 1.0766189 &times; 10<sup>4</sup> and the median is 1.0765 &times; 10<sup>4</sup>.

# What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
perinterval <- aggregate(steps ~ interval, activity, mean, na.rm=TRUE)
ggplot(perinterval, aes(x = interval, y = steps)) + geom_line() + xlab("5-minute interval") + 
  ylab("steps")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
perinterval[which.max(perinterval$steps),]
```

```
##     interval    steps
## 104      835 206.1698
```

# Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(as.numeric(is.na(activity$steps)))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
temp_activity <-  adply(activity, 1, function(x) if (is.na(x$steps)) {
  x$steps = round(perinterval[perinterval$interval == x$interval, 2])
  x} else {x})
```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
new_perday <- aggregate(steps ~ date, temp_activity, sum)
new_perday <- cbind(new_perday, label=rep("w/o NA", nrow(new_perday)))
```

4. Make a histogram of the total number of steps taken each day 


```r
ggplot(new_perday, aes(x=steps)) + geom_histogram(binwidth=1000, color="white") +
  labs(x="Steps", y="Frequency")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

Calculate and report the mean and median total number of steps taken per day. 


```r
new_mean <- mean(new_perday$steps, na.rm=TRUE)
new_median <- median(new_perday$steps, na.rm=TRUE)
```
The new mean is 1.0765639 &times; 10<sup>4</sup> and the new median is 1.0762 &times; 10<sup>4</sup>

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
total_perday <- rbind(perday, new_perday)
levels(total_perday$label) <- c("w/ NA", "w/o NA")
ggplot(total_perday, aes(x=steps, fill=label)) + geom_histogram(binwidth=1000, color="white") +
  labs(x="Steps", y="Frequency")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

The impact shows the graph and it is not neglectable.

# Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
we_data <- subset(temp_activity, weekdays(date) %in% c("Saturday", "Sunday"))
wd_data <- subset(temp_activity, !weekdays(date) %in% c("Saturday", "Sunday"))

we_data <- aggregate(steps ~ interval, we_data, mean)
wd_data <- aggregate(steps ~ interval, wd_data, mean)

we_data <- cbind(we_data, day=rep("weekend"))
wd_data <- cbind(wd_data, day=rep("weekday"))

week_data <- rbind(we_data, wd_data)
levels(week_data$day) <- c("weekend", "weekday")
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
ggplot(week_data, aes(x=interval, y=steps)) + geom_line() +
  facet_grid(day ~.) + labs(x="interval", y="Steps")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png) 

That's all of my assignment.
