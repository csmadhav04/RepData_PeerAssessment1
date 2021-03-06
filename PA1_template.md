---
title: "PA1_template"
output: html_document
keep_md: true
---


-----------------------------------------
Course project 1: Reproducible Research
-----------------------------------------

#Loading and preprocessing the data

## Load data


```r
data<-read.csv("activity.csv")
```

#What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

##Calculate the total number of steps taken per day

```r
daily_steps <- aggregate(steps ~ date, data, FUN=sum,na.rm=TRUE)
```

##Make a histogram of the total number of steps taken each day

```r
hist(daily_steps$steps, main = "Histogram of total number of steps taken each day", xlab = "Steps",ylab="# days")
```

![plot of chunk histogram_steps_per_day](figure/histogram_steps_per_day-1.png)





##Calculate and report the mean and median of the total number of steps taken per day


###Mean of the total number of steps taken per day

```r
mean(daily_steps$steps)
```

```
## [1] 10766.19
```
###Median of the total number of steps taken per day

```r
median(daily_steps$steps)
```

```
## [1] 10765
```

#What is the average daily activity pattern?

##Make a time series plot (i.e.type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

###Calculate average steps per interval

```r
steps_per_interval <- aggregate(steps ~ interval, data, FUN=mean,na.rm=TRUE)
interval <- levels(as.factor(data$interval))
```
###Time series plot

```r
plot(interval, steps_per_interval$steps, type = "l",
     main = "Plot of the average number of steps taken per interval",
     xlab = "Interval", ylab = "Average number of steps taken")
```

![plot of chunk plot_steps_per_interval](figure/plot_steps_per_interval-1.png)



##Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
steps_per_interval[which.max(steps_per_interval$steps), ]$interval
```

```
## [1] 835
```


#Imputing missing values
##Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

##Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
nrow(data[(!complete.cases(data)),])
```

```
## [1] 2304
```

##Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
###Missing values would be imputed based on the average of steps for corresponding 5-minute intervals

##Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
adj<- rep(steps_per_interval$steps,length(unique(data$date)))
data_new<-data

for (i in 1:length(data_new[,1])){  
  
  if(is.na(data_new[i,1])==TRUE){
    data_new[i,1]= adj[i]
  }}
```

##Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

###Calculate the total number of steps taken per day for the updated data

```r
steps_per_day_new <- aggregate(steps ~ date, data_new, FUN=sum,na.rm=TRUE)
```

###Plot the histogram of the total number of steps taken each day 

```r
hist(steps_per_day_new$steps, main = "Histogram of total number of steps taken each day", 
     xlab = "Steps",ylab="# days")
```

![plot of chunk hist_steps_per_day_new](figure/hist_steps_per_day_new-1.png)

###Mean of the total number of steps taken per day

```r
mean(steps_per_day_new$steps)
```

```
## [1] 10766.19
```

###Median of the total number of steps taken per day

```r
median(steps_per_day_new$steps)
```

```
## [1] 10766.19
```

###**Mean is the same but median has changed slightly due to imputation**


#Are there differences in activity patterns between weekdays and weekends?
##For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

##Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
data_new$date<-as.Date(data_new$date) #Convert factor to date

data_new$day<-weekdays(data_new$date)

weekend<-data_new[(weekdays(data_new$date) %in% c("Saturday","Sunday")),]   
weekday<-data_new[(!weekdays(data_new$date) %in% c("Saturday","Sunday")),]

weekend$group <- "weekend"
weekday$group<- "weekday"
```

##Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
library(ggplot2)
avg_steps <- aggregate(steps~ interval+group, rbind(weekend,weekday), FUN=mean,na.rm=TRUE)
qplot(interval, steps, data = avg_steps, facets = group~.) + 
geom_line(size = 1) + ylab("Mean steps") + 
ggtitle("Average number of steps taken")
```

![plot of chunk panel_plot_steps_weekday](figure/panel_plot_steps_weekday-1.png)



