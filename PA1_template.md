# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Unzip the input zip file to the current directory, read the csv file and convert dates to the R date format:

```r
unzip("activity.zip",exdir="./")
activity<-read.csv("activity.csv",stringsAsFactor=FALSE)
activity$date<-as.Date(activity$date,format="%Y-%m-%d")
```



## What is mean total number of steps taken per day?

Group data frame by dates and calculate sum of steps taken on given day:

```r
library(dplyr) # to help manipulate dataset
```

```
## 
## Attaching package: 'dplyr'
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
activity_by_date<-group_by(activity,date)
total_steps<-summarise(activity_by_date,sum(steps))
```

Make a numeric vector of total number of steps taken every day:

```r
steps_per_day<-total_steps[[2]]
```

Create a histogram with distribution of the total number of steps per day and save it to the file:

```r
hist(steps_per_day,xlab="Total number of steps per day",main="",col = "red")
```

![](figure/make_histogram-1.png) 

Now calculate the mean and median:

```r
mean(steps_per_day[!is.na(steps_per_day)])
```

```
## [1] 10766.19
```

```r
median(steps_per_day[!is.na(steps_per_day)])
```

```
## [1] 10765
```

The mean number of steps taken every day is **10766.19** and the median is **10765**.

## What is the average daily activity pattern?

Average the intervals over all days (and exclude the missing values from averaging):

```r
activity_by_interval<-group_by(activity,interval) #group by interval
avg_steps_interval<-summarise(activity_by_interval,mean(steps[!is.na(steps)]))
```

Make a time series plot of average steps taken in a given interval during day. Note that the times are shown without separation between hours and minutes and are in 24h format (e.g. 500 is 5am and 1500 is 3pm).


```r
 plot(avg_steps_interval,type="l",ylab="average number of steps",
      main="Average number of steps in an interval over the observed period",
      xlab="Time during the day")
```

![](figure/make_time_avg-1.png) 

Finding which interval contains on average the highest number of steps:

```r
row_id<-which.max(avg_steps_interval[[2]]) # find on which line there is maximum
avg_steps_interval$interval[row_id] # get the interval
```

```
## [1] 835
```
So the maximum number of steps was on average taken around **8:35** in the morning

## Imputing missing values

First find the amount of missing values:

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

We can also determine the proportion of missing values using

```r
sum(is.na(activity$steps))/length(activity$steps)
```

```
## [1] 0.1311475
```

Replacing the missing values in the original dataset by a mean for a given time interval. Note that the mean is rounded, as number of steps should be always integer.

```r
activity_noNA<-activity
for(i in 1:length(activity_noNA$steps)){
      if(is.na(activity_noNA$steps[i])){
            int<-activity_noNA$interval[i] # get the interval
            id<-which(avg_steps_interval$interval==int) # find on which row is stored average amount of steps
            activity_noNA$steps[i]<-round(avg_steps_interval[[id,2]])# replace the NA by the rounded mean
      }
}
sum(is.na(activity_noNA$steps))
```

```
## [1] 0
```
Now repeating the steps from first part. First grouping by day and making numeric vector with totals:

```r
activity_by_date_noNA<-group_by(activity_noNA,date)
total_steps_noNA<-summarise(activity_by_date_noNA,sum(steps))
steps_per_day_noNA<-total_steps_noNA[[2]]
```
And now the histogram:

```r
hist(steps_per_day_noNA,xlab="Total number of steps per day",main="Distribution after imputing missing values",col = "red")
```

![](figure/make_histogram_noNA-1.png) 

Now calculate the mean and median:

```r
mean(steps_per_day_noNA)
```

```
## [1] 10765.64
```

```r
median(steps_per_day_noNA)
```

```
## [1] 10762
```

These values **differ** from the ones reported in the first part of this assignment. The reason is that NA's are not uniformly distributed (some intervals have more NA's than others). Therefore, after replacing NA's by averages for a given interval, their relative weight is increased as NA's were skipped before. Because there were more missing NA's during the night, where fewer steps were taken, the overall mean and median went down.

## Are there differences in activity patterns between weekdays and weekends?

First add variable that distinguishes between weekday and weekend. It will be called DayType and obtained using the $wday function that returns numeric value of the day of the week. Sunday is first, corresponding to 0 and Saturday corresponds to 6. 

```r
activity_noNA$DayType<-ifelse(as.POSIXlt(activity_noNA$date)$wday==0 | as.POSIXlt(activity_noNA$date)$wday==6,"weekend","weekday" )
```
Then make this variable a factor:

```r
activity_noNA<-transform(activity_noNA,DayType= factor(DayType))
```
Group by daytype and calculate mean for every interval separately for weekdays and weekends

```r
activity_by_interval2<-group_by(activity_noNA,DayType,interval)
avg_steps_interval2<-summarise(activity_by_interval2,steps=mean(steps))
```
Finally, the plot:

```r
library(lattice)
xyplot(steps~interval | DayType, data=avg_steps_interval2,type = "l",layout = c(1, 2),ylab="average number of steps",
      main="Average number of steps in a 5 minute interval over the observed period for weekends and weekdays",
      xlab="Time during the day")
```

![](figure/weekday_weekend_plot-1.png) 
The average number of steps indeed differs between weekdays and weekends. There is much higher amount of average steps in the morning during the weekdays (probably going to work), but the overall activity afterwards is smaller in weekdays than in weekends, indicating that subject might spend weekdays in the office, while he is more active during weekends.
