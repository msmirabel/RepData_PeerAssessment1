# Reproducible Research: Peer Assessment 1


```r
load.packages <- function(pkgs)
{
	if(class(pkgs) == "character")
		pkgs = c(pkgs)
	for(pkg in pkgs){
		if(!(pkg %in% installed.packages()[,"Package"])){
			install.packages(pkg, repos="http://cran.us.r-project.org")
		}
		library(pkg,character.only=TRUE)
	}
}
load.packages(c("dplyr", "ggplot2", "plyr", "tidyr", "knitr", "lattice"))
```

## Loading and preprocessing the data


```r
##if repdata-data-activity.zip file doesn't exist in the working folder, download it and unzip it. 
if(!file.exists("./repdata-data-activity.zip"))
{ 
  fileurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
  download.file(fileurl, "./repdata-data-activity.zip", mode="wb")
  unzip("repdata-data-activity.zip")
}

## read the activity.csv file into activitydata
activitydata <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?


```r
##Aggregate the steps by data after removing the NA values and create a histogram
totaldaily <- aggregate(activitydata["steps"], by=activitydata["date"], FUN=sum, na.rm=TRUE)
hist(totaldaily$steps, main="Histogram of the total number of steps taken each day", xlab="Steps", ylab="Frequency")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
##Calculate and report the mean and median total number of steps taken per day.
print(paste("Mean of total number of steps taken each day is", mean(totaldaily$steps)))
```

```
## [1] "Mean of total number of steps taken each day is 9354.22950819672"
```

```r
print(paste("Median of total number of steps taken each day is", median(totaldaily$steps)))
```

```
## [1] "Median of total number of steps taken each day is 10395"
```

## What is the average daily activity pattern?


```r
##average the number of steps per interval without NA values
avgsteps<-aggregate(activitydata["steps"], by=activitydata["interval"], FUN="mean", na.rm=TRUE)

##plot the average number of steps taken, averaged across all days per interval
plot(avgsteps$interval, avgsteps$steps, type="l", main="5-minute interva- average number of steps taken, averaged across all days", xlab="5-minute interval", ylab="average number of steps taken")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
##interval that contains the maximum number of steps
print(paste(avgsteps[avgsteps$steps== max(avgsteps$steps),"interval"], "interval contains the maximum number of steps"))
```

```
## [1] "835 interval contains the maximum number of steps"
```


## Imputing missing values

```r
##calculate the number of missing values (NA values) in the dataset 
print(paste("Total number of missing values in the activity dataset (coded as NA) is",  nrow(activitydata[is.na(activitydata$steps),])))
```

```
## [1] "Total number of missing values in the activity dataset (coded as NA) is 2304"
```

```r
##My strategy is to fill in the NA values with each interval median value
print(paste("Fill in the missing values in the dataset with median for that 5-minute interval"))
```

```
## [1] "Fill in the missing values in the dataset with median for that 5-minute interval"
```

```r
##create a table that shows the median steps by interval. 
mediandata<-aggregate(activitydata["steps"], by=activitydata["interval"], FUN="median", na.rm=TRUE)

##merge the original data and median data by using interval and fill in the missing values with the median value for the interval
mergedata<-merge(activitydata, mediandata, by.x="interval", by.y="interval")
mergedata$steps <- ifelse(is.na(mergedata$steps.x), mergedata$steps.y, mergedata$steps.x)

##calculate the total number of steps each day and make a histogram of the total number of steps taken each da
mergetotaldaily <- aggregate(mergedata["steps"], by=mergedata["date"], FUN=sum)
hist(mergetotaldaily$steps, main="Histogram of the total number of steps taken each day", xlab="Date", ylab="Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
##Calculate and report the mean and median total number of steps taken per day.
print(paste("Mean of total number of steps taken each day", mean(mergetotaldaily$steps)))
```

```
## [1] "Mean of total number of steps taken each day 9503.86885245902"
```

```r
print(paste("Median of total number of steps taken each day", median(mergetotaldaily$steps)))
```

```
## [1] "Median of total number of steps taken each day 10395"
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

## Are there differences in activity patterns between weekdays and weekends?

```r
## conver the data column to date and create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
mergedata$date <- as.Date(mergedata$date)
mergedata$week <- ifelse(weekdays(mergedata$date) %in% c("Saturday", "Sunday"), "weekend", "weekday")

##subset the mergedata with only the 4 columns
collist <- c("interval", "date", "steps", "week")
mergedata <-mergedata[collist]

##group by week and interval and then summarize by the mean of steps
mergeaggdata = mergedata %>% 
  dplyr::group_by(week, interval) %>% 
  dplyr::summarise(avgSteps = mean(steps))

##draw xyplot using avg steps as y axis and interval as x-axis for weekend and weekdays. 
library(lattice)
xyplot(avgSteps ~ interval|week, data=mergeaggdata, type="l", layout=c(1,2), main="interval and average number of steps taken, across all weekdays or weekends", ylab="Number of steps", xlab="Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
