---
title: "project1"
author: "Laukik Patil"
date: "January 8, 2018"
output:
  html_document:
    keep_md: TRUE
---


##Current Directory: 
###C:/Users/lpatil/Documents/R/reproducible_research/project1

#Loading and preprocessing the data
## Download Data File


```r
#Download file URL
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
# download into the placeholder file
download.file(url, "activity.zip")
```

## Unzip and Read Data File

```r
# unzip the file to the temporary directory and overwrite
unzip("activity.zip", overwrite = TRUE)
#Read the data file
data <- read.csv("activity.csv")
```
#What is mean total number of steps taken per day?
## Group data by day

```r
# load dplyr package
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
# Group data by day

steps_by_day <- data %>% group_by( date) %>% 
  summarise(daily_total=sum(steps, na.rm=TRUE) )


hist(steps_by_day$daily_total, xlab="Steps", main="Daily Step Histogram")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->


### The mean of daily steps is 9354.2295082
### The Median of daily steps is 10395

#What is the average daily activity pattern?
##Group data by interval


```r
# load ggplot2 package
library(ggplot2)

# Group data by day

steps_by_interval <- data %>% group_by( interval) %>% 
  summarise(interval_average=mean(steps, na.rm=TRUE) )

ggplot(steps_by_interval, aes(interval, interval_average)) + geom_line() +
   xlab("Time Interval") + ylab("Avarage Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
max_steps_interval <- steps_by_interval %>% filter(interval_average == max(interval_average)) %>% select(interval)
```

###The interval with maximum average steps is 835

###Total missing values: 2304

#Imputing Missing Values


```r
#Merge original data with interval average data frame
data_impute<-merge(data, steps_by_interval)

#Replace missing values by interval average
data_impute$steps <- ifelse(is.na(data_impute$steps), data_impute$interval_average, data_impute$steps)

# New Histogram
hist(data_impute$steps, xlab="Steps", main="Daily Step Histogram")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Imputing the values for missing data has canged the original histogram. Now all the highest frequency of steps in around 0.

#Are there differences in activity patterns between weekdays and weekends?


```r
#Extract he day of week from date
data_impute$date_as_date <- as.Date(data_impute$date, '%Y-%m-%d')
data_impute$day_of_week <- weekdays(data_impute$date_as_date)

# Add variable for weekday/weekend
data_impute$is_weekend <- ifelse(data_impute$day_of_week =="Saturday" |data_impute$day_of_week == "Sunday", "weekend", "weekday")

data_impute$is_weekend <- as.factor(data_impute$is_weekend)

data_impute_plot <- data_impute %>% group_by( is_weekend, interval) %>% 
  summarise(interval_average=mean(steps, na.rm=TRUE) )

#Plot
ggplot(data_impute_plot, aes(interval, interval_average)) + geom_line() +
   xlab("Time Interval") + ylab("Avarage Steps") + facet_grid( is_weekend ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
