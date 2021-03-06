---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

1. Load the data (i.e. \color{red}{\verb|read.csv()|}read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis

I will analyse Activity monitoring data. 
This dataset is including below variables

- steps: Number of steps taking in a  minute interval. This has missing values.
- date: the date on which the measurement was taken in YYYY-MM-DD format.
- interval: Identifier for 5-minute interval in which measurement was taken.

First, I will see this varibles.


```r
dat = read.csv('activity.csv', header = T)
names(dat)
```

```
## [1] "steps"    "date"     "interval"
```

```r
str(dat)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(dat)
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

Second, transform this data into a format suitable for analysis

```r
library(data.table)
dat_tbl = data.table(dat)
dat_tbl_summary = dat_tbl[, list(total_steps = sum(steps, na.rm = T)), 
                          by = date]
```

## What is mean total number of steps taken per day?
1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

Before plotting graph, checking mean and median of total step.


```r
round(mean(dat_tbl_summary$total_steps), 1)
```

```
## [1] 9354.2
```

```r
round(median(dat_tbl_summary$total_steps), 1)
```

```
## [1] 10395
```

Plotting histgtram to see total number of steps tanken per day.

Here, ignore missing values.

```r
gen_hist = function(x, title){
        hist(x, 
             breaks = 20,
             main = title,
             xlab = 'Total Number of Steps', col = 'grey',
            
             cex.main = .9)
        
        #caluclate mean and median
        mean_value = round(mean(x), 1)
        median_value = round(median(x), 1)
        
        #place lines for mean and median on histogram
        abline(v=mean_value, lwd = 3, col = 'blue')
        abline(v=median_value, lwd = 3, col = 'red')
        
        #create legend
        legend('topright', lty = 1, lwd = 3, col = c("blue", "red"),
               cex = .8, 
               legend = c(paste('Mean: ', mean_value),
               paste('Median: ', median_value))
               )
}
gen_hist(dat_tbl_summary$total_steps, 'Number of Steps Taken Per Day')
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## What is the average daily activity pattern?
1. Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

First, summarize dataset by 5-minute interval

```r
dat_tbl_summary_intv = dat_tbl[, list(avg_steps = mean(steps, na.rm = T)), 
                          by = interval]
```

Second, plotting line graph to see time series with above interval

```r
with(dat_tbl_summary_intv, {
        plot(interval, avg_steps, type = 'l',
             main = 'Average Steps by Time Interval',
             xlab = '5 Minute Time Interval',
             ylab = 'Average Number of Steps')
        })
max_steps = dat_tbl_summary_intv[which.max(avg_steps), ]
max_lab = paste('Maximum Of ', round(max_steps$avg_steps, 1), ' Steps \n On ', max_steps$interval, 'th Time Interval', sep = '')

points(max_steps$interval,  max_steps$avg_steps, col = 'red', lwd = 3, pch = 19)

legend("topright",
       legend = max_lab,
       text.col = 'red',
       bty = 'n'
       )
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with \color{red}{\verb|NA|}NAs)
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

First, calculate the number of missing values

```r
sum(is.na(dat$steps))
```

```
## [1] 2304
```

Second, filling missing vaues in the dataset. I use mean for that 5-minute interval

```r
setkey(dat_tbl, interval)
setkey(dat_tbl_summary_intv, interval)

NA_replace = function(x,y){
        if(is.na(x)){
                
                return(y)
        }
        return(x)
}

dat_tbl_miss = dat_tbl[dat_tbl_summary_intv]
dat_tbl_miss$new_steps = mapply(NA_replace,dat_tbl_miss$steps, dat_tbl_miss$avg_steps)

dat_tbl_summary_miss = dat_tbl_miss[, list(new_steps = sum(new_steps, na.rm = T)), 
                          by = date]

head(dat_tbl_summary_miss)
```

```
##          date new_steps
## 1: 2012-10-01  10766.19
## 2: 2012-10-02    126.00
## 3: 2012-10-03  11352.00
## 4: 2012-10-04  12116.00
## 5: 2012-10-05  13294.00
## 6: 2012-10-06  15420.00
```

Finally, plotting histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day.

```r
gen_hist(dat_tbl_summary$total_steps, 'Missing Values Removed')
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

```r
gen_hist(dat_tbl_summary_miss$new_steps, 'Missing Values Replaced With \n Mean For Interval')
```

![](PA1_template_files/figure-html/unnamed-chunk-9-2.png)<!-- -->

## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
2. Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

First, create new variables, “weekday” and “weekend”.

```r
# In Japan, '土曜日' means Saturday, '日曜日' means Sunday.
weekpart = function(x){
        if(x %in% c('土曜日', '日曜日')){
                return('Weekend')
        }
        
        return('Weekday')
}

dat_tbl_miss$dayname = weekdays(as.Date(dat_tbl_miss$date))

dat_tbl_miss$daytype = as.factor(apply(as.matrix(dat_tbl_miss$dayname), 1, weekpart))

dat_tbl_summary_miss = dat_tbl_miss[, list(avg_steps = mean(new_steps, na.rm = T)), 
                          by = list(interval, daytype)]

head(dat_tbl_summary_miss)
```

```
##    interval daytype  avg_steps
## 1:        0 Weekday 2.25115304
## 2:        0 Weekend 0.21462264
## 3:        5 Weekday 0.44528302
## 4:        5 Weekend 0.04245283
## 5:       10 Weekday 0.17316562
## 6:       10 Weekend 0.01650943
```

Finally, plotting panel plot with time series.

```r
library(lattice)
xyplot(avg_steps~interval | daytype, data = dat_tbl_summary_miss,
      type = 'l',
      xlab = 'Interval',
      ylab = 'Number of Steps',
      layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->
