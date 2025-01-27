---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
## Some Information
The variables included in the Activity.csv dataset are:

- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA) 

- date: The date on which the measurement was taken in YYYY-MM-DD format

- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Loading and preprocessing the data

```r
## Load the data (i.e. read.csv())
data <- read.csv('activity.csv')
cat("Data Dimensions: ", dim(data), "\n\n")
```

```
## Data Dimensions:  17568 3
```

```r
cat("Data Summary: \n", summary(data), "\n\n")
```

```
## Data Summary: 
##  Min.   :  0.00   1st Qu.:  0.00   Median :  0.00   Mean   : 37.38   3rd Qu.: 12.00   Max.   :806.00   NA's   :2304   2012-10-01:  288   2012-10-02:  288   2012-10-03:  288   2012-10-04:  288   2012-10-05:  288   2012-10-06:  288   (Other)   :15840   Min.   :   0.0   1st Qu.: 588.8   Median :1177.5   Mean   :1177.5   3rd Qu.:1766.2   Max.   :2355.0   NA
```

```r
head(data)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
## Process/transform the data (if necessary) into a format suitable for your analysis
data_no_na <- na.omit(data)
```


## What is mean total number of steps taken per day?

```r
## For this part of the assignment, you can ignore the missing values in the dataset.

## 1. Make a histogram of the total number of steps taken each day
### Create table calculating total steps
total_steps <- aggregate(data_no_na$steps, by=list(data_no_na$date), sum)
### Rename columns (makes it easier to make histogram)
colnames(total_steps)[1:2] <- c("date", "total_steps")
### Create histogram
hist(total_steps$total_steps, main ="Histogram of the Total Number of Steps Taken Each Day", xlab = "Total Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
## 2. Calculate and report the mean and median total number of steps taken per day
### Caluclate mean
mean <- mean(total_steps$total_steps)
### Calculate median
median <- median(total_steps$total_steps)
### Print results
cat("Mean: ", mean, "\n\n")
```

```
## Mean:  10766.19
```

```r
cat("Median:", median)
```

```
## Median: 10765
```

## What is the average daily activity pattern?

```r
## 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number 
##    of steps taken, averaged across all days (y-axis)
### Calculate mean of steps
ts <- tapply(data_no_na$steps, data_no_na$interval, mean)
### Create time series plot
plot(row.names(ts), ts, type = "l", xlab = "5 Minute Interval", 
    ylab = "Average Number of Steps", main = "Average Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
## 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
### Calculate which 5min interval contains the maximnum number of steps
max_num_of_steps <- names(which.max(ts))
cat("Interval with Maximum Number of steps:", max_num_of_steps)
```

```
## Interval with Maximum Number of steps: 835
```

## Imputing missing values

```r
## Note that there are a number of days/intervals where there are missing values (coded as NA). 
## The presence of missing days may introduce bias into some calculations or summaries of the data.

## 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
cat("Number of rows missing values before imputing:", sum(is.na(data)), "\n\n")
```

```
## Number of rows missing values before imputing: 2304
```

```r
## 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. 
##    For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
### Reference: https://lgreski.github.io/datasciencedepot/references/missingValueImputation-Gelman.PDF
random.imp <- function (a){
missing <- is.na(a)
n.missing <- sum(missing)
a.obs <- a[!missing]
imputed <- a
imputed[missing] <- sample (a.obs, n.missing, replace=TRUE)
return (imputed)
}

## 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
set.seed(12345)
data_imp <- data
data_imp$steps <- random.imp(data$steps)
cat("Number of rows missing values after imputing:", sum(is.na(data_imp)), "\n\n")
```

```
## Number of rows missing values after imputing: 0
```

```r
## 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean
##    and median total number of steps taken per day. Do these values differ from the estimates 
##    from the first part of the assignment? 
##    What is the impact of imputing missing data on the estimates of the total daily 
##    number of steps?
### Create table calculating total steps
total_steps_imp <- aggregate(data_imp$steps, by=list(data_imp$date), sum)
### Rename columns (makes it easier to make histogram)
colnames(total_steps_imp)[1:2] <- c("date", "total_steps")
### Create histogram
hist(total_steps_imp$total_steps, main ="Histogram of the Total Number of Steps Taken Each Day", xlab = "Total Number of Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
### Caluclate mean
mean_imp <- mean(total_steps_imp$total_steps)
### Calculate median
median_imp <- median(total_steps_imp$total_steps)
### Print results
cat("Mean: ", mean_imp, "\n\n")
```

```
## Mean:  10760.89
```

```r
cat("Median:", median_imp)
```

```
## Median: 10600
```


## Are there differences in activity patterns between weekdays and weekends?

```r
## 1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend"    
##    indicating whether a given date is a weekday or weekend day.
### Add new column to designate day of week
data_imp$day <- weekdays(as.Date(data_imp$date))
### Subset Weekday data
data_imp_weekday <- data_imp[data_imp$day %in% c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday"),]
### Subset Weekend data
data_imp_weekend <- data_imp[data_imp$day %in% c("Saturday", "Sunday"),]

## 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute 
##    interval (x-axis) and the average number of steps taken, averaged across all weekday 
##    days or weekend days (y-axis). The plot should look something like the following,
##    which was created using simulated data:
### Calculate mean of steps for weekday and weekend dataset
ts_weekday <- tapply(data_imp_weekday$steps, data_imp_weekday$interval, mean)
ts_weekend <- tapply(data_imp_weekend$steps, data_imp_weekend$interval, mean)
### Create time series plot for weekday and weekend dataset
plot(row.names(ts_weekday), ts_weekday, type = "l", xlab = "5 Minute Interval", 
    ylab = "Average Number of Steps", main = "Average Number of Steps (Weekday)")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
plot(row.names(ts_weekend), ts_weekend, type = "l", xlab = "5 Minute Interval", 
    ylab = "Average Number of Steps", main = "Average Number of Steps (Weekend)")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-2.png)<!-- -->
