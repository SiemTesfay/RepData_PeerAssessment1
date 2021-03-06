---
title: "Reproducible Research: Peer-graded Assignment- Course Project 1"
output: 
  html_document:
    keep_md: true
---



### Loading and preprocessing the data  
Goal :  
1. Loading the data   
2. Preprocessing   


```r
# Downloading
dir.create("./RepData_Assign_1")
```

```
## Warning in dir.create("./RepData_Assign_1"): './RepData_Assign_1' already exists
```

```r
urlZip <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(url = urlZip, destfile = "./RepData_Assign_1/repdata_data_activity.zip")
unzip("./RepData_Assign_1/repdata_data_activity.zip", exdir = "./RepData_Assign_1")

# Loading 
data <- read.csv("./RepData_Assign_1/activity.csv", sep = ",")
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```
Transforming the data

```r
# Transform the date attribute to an actual date format
data$date <- as.POSIXct(data$date, format="%Y-%m-%d")

# Compute the weekdays from the date attribute
data_raw <- data.frame(date=data$date, 
                           weekday=tolower(weekdays(data$date)), 
                           steps=data$steps, 
                           interval=data$interval)

# Compute the day type (weekend or weekday)
data_raw <- cbind(data_raw, 
                      daytype=ifelse(data_raw$weekday == "saturday" | 
                                     data_raw$weekday == "sunday", "weekend", 
                                     "weekday"))

# Create the final data.frame
data <- data.frame(date=data_raw$date, 
                       weekday=data_raw$weekday, 
                       daytype=data_raw$daytype, 
                       interval=data_raw$interval,
                       steps=data_raw$steps)
```


Displaying the data

```r
head(data)
```

```
##         date weekday daytype interval steps
## 1 2012-10-01  monday weekday        0    NA
## 2 2012-10-01  monday weekday        5    NA
## 3 2012-10-01  monday weekday       10    NA
## 4 2012-10-01  monday weekday       15    NA
## 5 2012-10-01  monday weekday       20    NA
## 6 2012-10-01  monday weekday       25    NA
```
### What is mean total number of steps taken per day?  
NB : NA values are ignored.

0. Checking if there are NA values in column 'steps'

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```
There are 2304 fields that are NA. 

1. Calculating total number of steps taken each day

```r
totalSteps <- aggregate(steps ~ date, data, FUN=sum) #, na.rm = TRUE)
#Displaying the data
head(totalSteps)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```
2. Make a *histogram* of the total number of steps taken each day

```r
#Ploting total steps taken - Histogram
hist(totalSteps$steps, main = "Total Steps Per Day (NA Excluded)", xlab = 'Number of Steps')
```

![](PA1_template.Rmd-_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day

```r
#Calculatin mean
tot_mean <- mean(totalSteps$steps)
tot_mean
```

```
## [1] 10766.19
```

```r
#Calculating median
tot_median <- median(totalSteps$steps)
tot_median
```

```
## [1] 10765
```
For the total steps taken per day the mean is 1.0766189\times 10^{4} and median is 10765.

### What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l"\color{red}{\verb|type = "l"|}type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
#computing mean by interval accross eaach day
mean_by_interval <- aggregate(data$steps,
                              by = list(data$interval),
                              FUN = "mean",
                              na.rm = TRUE)
#Displaying the data
head(mean_by_interval)
```

```
##   Group.1         x
## 1       0 1.7169811
## 2       5 0.3396226
## 3      10 0.1320755
## 4      15 0.1509434
## 5      20 0.0754717
## 6      25 2.0943396
```
As shown above the colum names are given by default. Below the columns will be renamed.

```r
#renaming the column names
names(mean_by_interval) <- c("Intervals", "Mean")
#Displaying the data
head(mean_by_interval)
```

```
##   Intervals      Mean
## 1         0 1.7169811
## 2         5 0.3396226
## 3        10 0.1320755
## 4        15 0.1509434
## 5        20 0.0754717
## 6        25 2.0943396
```


```r
plot(x = mean_by_interval$Intervals, 
     y = mean_by_interval$Mean, 
     type = "l",
     col = "red",
     ylab = "Number of Steps",
     xlab = "Interval")
```

![](PA1_template.Rmd-_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_pos <- mean_by_interval[which.max(mean_by_interval$Mean),]
max_pos
```

```
##     Intervals     Mean
## 104       835 206.1698
```
Thus Interval #835 has the max number of steps.  

### Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NA\color{red}{\verb|NA|}NAs)


```r
#counting the number of NA
NA_sum <- sum(is.na(data$steps))
NA_sum
```

```
## [1] 2304
```
Total of 2304 NA are presnt in the data.

2. Devise a strategy for filling in all of the missing values in the dataset.
Strategy : filling NA with median.


```r
# finding the positions of NA
NA_pos <- which(is.na(data$steps))
head(NA_pos)
```

```
## [1] 1 2 3 4 5 6
```

```r
tail(NA_pos)
```

```
## [1] 17563 17564 17565 17566 17567 17568
```

Now, we will create vector of mean of the intervals.


```r
mean_vec <- rep(mean(data$steps, na.rm = T), times = length(NA_pos))
head(mean_vec)
```

```
## [1] 37.3826 37.3826 37.3826 37.3826 37.3826 37.3826
```


3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

We will mearge 'mean_vec' according to 'NA_pos' to create new database free of NA


```r
#Duplicating the original data
newData <- data
newData [NA_pos, "steps"] <- mean_vec
head(newData)
```

```
##         date weekday daytype interval   steps
## 1 2012-10-01  monday weekday        0 37.3826
## 2 2012-10-01  monday weekday        5 37.3826
## 3 2012-10-01  monday weekday       10 37.3826
## 4 2012-10-01  monday weekday       15 37.3826
## 5 2012-10-01  monday weekday       20 37.3826
## 6 2012-10-01  monday weekday       25 37.3826
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Total Steps taken each day according to the new data.

```r
totalSteps_New <- aggregate(steps ~ date, newData, FUN=sum)
head(totalSteps_New)
```

```
##         date    steps
## 1 2012-10-01 10766.19
## 2 2012-10-02   126.00
## 3 2012-10-03 11352.00
## 4 2012-10-04 12116.00
## 5 2012-10-05 13294.00
## 6 2012-10-06 15420.00
```
Ploting Histogram

```r
hist(totalSteps_New$steps, 
     breaks=seq(from=0, to=25000, by=2500),
     col="blue", 
     xlab="Total number of steps", 
     ylim=c(0, 30), 
     main="Histogram of the total number of steps taken each day\n(NA replaced by mean value)")
```

![](PA1_template.Rmd-_files/figure-html/unnamed-chunk-17-1.png)<!-- -->

Calculating and reporting the mean and median of the total number of steps taken per day after NA replcaed

```r
#Calculatin mean
tot_mean_New <- mean(totalSteps_New$steps)
tot_mean_New
```

```
## [1] 10766.19
```

```r
#Calculating median
tot_median_New <- median(totalSteps_New$steps)
tot_median_New
```

```
## [1] 10766.19
```

### Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - “weekdays” and “weekend” indicating whether a given date is a weekday or weekend day.

**This part was done in the above steps - preprosessing**

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5- minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
# Load the lattice graphical library
library(lattice)

# Compute the average number of steps taken, averaged across all daytype variable
mean_data <- aggregate(newData$steps, 
                       by=list(newData$daytype, newData$weekday, newData$interval),
                       mean)

# Rename the attributes
names(mean_data) <- c("daytype", "weekday", "interval", "mean")

## Displaying mean_data
head(mean_data)
```

```
##   daytype  weekday interval     mean
## 1 weekday   friday        0 8.307244
## 2 weekday   monday        0 9.418355
## 3 weekend saturday        0 4.672825
## 4 weekend   sunday        0 4.672825
## 5 weekday thursday        0 9.375844
## 6 weekday  tuesday        0 0.000000
```

The time series plot take the following form


```r
xyplot(mean ~ interval | daytype, mean_data, 
       type="l", 
       lwd=1, 
       xlab="Interval", 
       ylab="Number of steps", 
       layout=c(1,2))
```

![](PA1_template.Rmd-_files/figure-html/unnamed-chunk-20-1.png)<!-- -->

Clearing the memory.

```r
rm(list = ls())
```

















































