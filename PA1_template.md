---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
html_document:
keep_md: true
---
## Introduction
It is now possible to collect a large amount of data about personal movement
using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone
Up. These type of devices are part of the "quantified self" movement - a group
of enthusiasts who take measurements about themselves regularly to improve
their health, to find patterns in their behavior, or because they are tech
geeks. But these data remain under-utilized both because the raw data are hard
to obtain and there is a lack of statistical methods and software for
processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device.
This device collects data at 5 minute intervals through out the day. The data
consists of two months of data from an anonymous individual collected during
the months of October and November, 2012 and include the number of steps taken
in 5 minute intervals each day.

## Loading and preprocessing the data
The data for this assignment can be downloaded from the course web site:

- Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]   

The variables included in this dataset are:

- **steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA)  
- **date**: The date on which the measurement was taken in YYYY-MM-DD format  
- **interval**: Identifier for the 5-minute interval in which measurement was taken  

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Assignment

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a **single R markdown document** that can be processed by **knitr** and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use **echo = TRUE** so that someone else will be able to read the code. **This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.**

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

## Set global options
echo = TRUE
#library(ggplot2)

## I. Loading and preprocessing the data

```r
if (!file.exists("./activity.csv")) { 
    unzip("./activity.zip") 
}
activity <- read.csv("./activity.csv")
activity$date <- as.POSIXct(x=activity$date, format="%Y-%m-%d")
#summary(activity)
```

## II. What is mean total number of steps taken per day?

```r
activity_tot_steps <- with(activity,
                           aggregate(steps,
                                     by = list(date),
                                     FUN = sum,
                                     na.rm = TRUE))

names(activity_tot_steps) <- c("date", "steps")
hist(activity_tot_steps$steps,
     main = "Mean Total Steps Taken Per Day",
     xlab = "Steps",
     col = "green",
     ylim = c(0,20),
     breaks = seq(0,25000, by=2500))
```

![](PA1_template_files/figure-html/q1-1.png)<!-- -->

```r
mean(activity_tot_steps$steps)
```

```
## [1] 9354.23
```

```r
median(activity_tot_steps$steps)
```

```
## [1] 10395
```

## III. What is the average daily activity pattern?

```r
avg_daily_activity <- aggregate(activity$steps,
                                by = list(activity$interval),
                                FUN = mean,
                                na.rm = TRUE)
names(avg_daily_activity) <- c("interval", "mean")
plot(avg_daily_activity$interval,
     avg_daily_activity$mean,
     type = "l",
     main = "Average Daily Activity Pattern",
     xlab = "Interval",
     ylab = "Average number of steps",
     col = "green")
```

![](PA1_template_files/figure-html/q2a-1.png)<!-- -->

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
x <- max(avg_daily_activity$mean)
avg_daily_activity[avg_daily_activity$mean == x, ]$interval
```

```
## [1] 835
```

## IV. Imputing missing values
Calculate and report the total number of missing values in the dataset (i.e.
the total number of rows with NAs)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The
strategy does not need to be sophisticated. For example, you could use the
mean/median for that day, or the mean for that 5-minute interval, etc.

```r
x <- match(activity$interval,
           avg_daily_activity$interval)
fill_in_steps <- avg_daily_activity$mean[x]
```

Create a new dataset that is equal to the original dataset but with the missing
data filled in.

```r
activity_fill_in <- transform(activity,
                              steps = ifelse(is.na(activity$steps),
                                             yes = fill_in_steps,
                                             no = activity$steps))
total_steps_fill_in <- aggregate(steps ~ date,
                                 activity_fill_in,
                                 sum)
names(total_steps_fill_in) <- c("date", "daily_steps")
```

Make a histogram of the total number of steps taken each day and Calculate and
report the mean and median total number of steps taken per day. Do these values
differ from the estimates from the first part of the assignment? What is the
impact of imputing missing data on the estimates of the total daily number of
steps?

```r
hist(total_steps_fill_in$daily_steps,
     main = "Total number of steps taken each day",
     xlab = "Total steps per day",
     col = "green",
     ylim = c(0,30),
     breaks = seq(0,25000,by=2500))
```

![](PA1_template_files/figure-html/q3d-1.png)<!-- -->

```r
mean(total_steps_fill_in$daily_steps)
```

```
## [1] 10766.19
```

```r
median(total_steps_fill_in$daily_steps)
```

```
## [1] 10766.19
```

## V. Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels - "weekday" and
"weekend" indicating whether a given date is a weekday or weekend day.

```r
weekday <- weekdays(activity$date)
activity <- cbind(activity, weekday)

activity$date <- as.Date(strptime(activity$date, format="%Y-%m-%d"))
activity$datetype <- sapply(activity$date,
                            function(x) {
                                if (weekdays(x) == "Saturday" | weekdays(x) == "Sunday")
                                    {y <- "Weekend"}
                                else
                                    {y <- "Weekday"}
                                y})
```

Make a panel plot containing a time series plot (i.e. type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
library(ggplot2)
activity_by_date <- aggregate(steps ~ interval + datetype,
                              activity,
                              mean,
                              na.rm = TRUE)
plot <- ggplot(activity_by_date,
               aes(x = interval,
                   y = steps,
                   color = datetype)) +
        geom_line() +
        labs(title = "Average daily steps by type of date",
             x = "Interval",
             y = "Average number of steps") +
        facet_wrap(~datetype,
                   ncol = 1,
                   nrow=2)
print(plot)
```

![](PA1_template_files/figure-html/q4b-1.png)<!-- -->
