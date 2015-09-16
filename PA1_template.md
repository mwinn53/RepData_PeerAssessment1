# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

### 1. Load the data (i.e. ```read.csv()```)


```r
    rm(list=ls())
    
    csv <- read.csv("activity.csv")
```

### 2. Process/transform the data (if necessary) into a format suitable for your analysis

In addition to setting global options, the time field is padded with zeros for a 
uniform 4-character field width.
    

```r
    knitr::opts_chunk$set(fig.width=10, 
                   fig.height=6,
                   echo=TRUE)

    require(stringr)    
```

```
## Loading required package: stringr
```

```r
    csv$interval <- str_pad(csv$interval, 4, pad = "0") 
```



## What is mean total number of steps taken per day?

*For this part of the assignment, you can ignore the missing values in the dataset.*

### 1. Make a histogram of the total number of steps taken each day

The ```tapply``` function is used to sum the total number of steps measured at 
each of the 5-minute intervals and stores the daily totals in an array called 
```stepsByDay```.  

A histogram (left) summarizes frequencies of the total steps per day, however, 
a bar plot (right) is more appropriate for visualizing the total steps
(*y-axis*) over time (*x-axis*). Both plots of ```stepsByDay``` are included 
below to illustrate the difference.


```r
    stepsByDay <- tapply(csv$steps, as.Date(csv$date), sum)
    
    par(mfrow = c(1, 2))    

    hist(stepsByDay,
         main = "Frequency of Daily Steps",
         xlab = "Total Number of Steps per Day",
         ylab = "Frequency"
         )

    barplot(stepsByDay, 
         main = "Sum of Steps Taken per Day",
         xlab = "Date",
         ylab = "Sum of Steps"
         )
```

![](PA1_template_files/figure-html/ques1-1-1.png) 

### 2. Calculate and report the mean and median total number of steps taken per day


```r
    meansteps <- mean(stepsByDay, na.rm = TRUE)

    mediansteps <- median(stepsByDay, na.rm = TRUE)
    
    c(meansteps, mediansteps)
```

```
## [1] 10766.19 10765.00
```

The mean and median total number of steps taken per day are 
**10766.19** and **10765**, 
respectively. The summarized data is already available in the ```stepsByDay```
array, so the ```mean``` and ```median``` functions are used to calculate those
values.


## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. ```type = "l"```) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

The line plot below shows the average number of steps for each of the five 
minute intervals.


```r
    stepsByTime <- aggregate(csv$steps, list(csv$interval), mean, na.rm = TRUE)    
    names(stepsByTime) <- paste(c("interval", "steps"))
    
    plot(stepsByTime, type = 'l',
         main = "Average Daily Activity Pattern",
         xlab = "Time Interval",
         ylab = "Total Number of Steps"
    )
```

![](PA1_template_files/figure-html/ques2-1-1.png) 


### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
    maxSteps <- max(stepsByTime$steps)

    peakSteps <- stepsByTime[stepsByTime$steps == maxSteps,]

    peakSteps
```

```
##     interval    steps
## 104     0835 206.1698
```

The time interval with the maximum average number of steps is 
****, with an average of **206.1698113** steps.


## Imputing missing values

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with ```NA```s)

```r
    numNAs <- sum(is.na(csv$steps))
    numNAs
```

```
## [1] 2304
```

There are ``2304`` ```NA``` values in the dataset.



### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

The strategy to handle ```NA``` values is to insert the mean value for that 
particular five minute interval. For the sake of clarity, the steps are 
rounded to the nearest whole number.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
    newcsv <- csv

    newcsv$steps <- ifelse(is.na(newcsv$steps),
                        stepsByTime$steps[
                            match(newcsv$interval, stepsByTime$interval)
                            ],
                        newcsv$steps)
    
    newcsv$steps <- round(newcsv$steps, digits = 0)
```


### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

For ease of comparison, the two plots are shown side-by-side below.


```r
    par(mfrow = c(1, 2))    

    barplot(stepsByDay, 
         main = "Sum of Steps Taken per Day",
         sub = "(NAs excluded)",
         xlab = "Date",
         ylab = "Sum of Steps"
         )

    stepsByDay <- tapply(newcsv$steps, as.Date(newcsv$date), sum)

    barplot(stepsByDay, 
         main = "Sum of Steps Taken per Day",
         sub = "(NAs replaced)",
         xlab = "Date",
         ylab = "Sum of Steps"
         )        
```

![](PA1_template_files/figure-html/ques3-4-1.png) 



```r
    newmeansteps <- mean(stepsByDay, na.rm = TRUE)

    newmediansteps <- median(stepsByDay, na.rm = TRUE)

    c(meansteps, newmeansteps, '\t',
      mediansteps, newmediansteps)
```

```
## [1] "10766.1886792453" "10765.6393442623" "\t"               
## [4] "10765"            "10762"
```

####    - Do these values differ from the estimates from the first part of the assignment? 

####    - What is the impact of imputing missing data on the estimates of the total daily number of steps?

There are slight differences in the plot and the total mean/median estimates 
that are visible upon close examination. Substituting the five-minute interval
means for the ```NA``` values--instead of omitting them--changes the total daily
mean from **10766.19** to 
**10765.64**, and the median from 
**10765** to 
**10762**.

## Are there differences in activity patterns between weekdays and weekends?

*For this part the ```weekdays()``` function may be of some help here. Use the dataset with the filled-in missing values for this part.*

###     1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

###     2. Make a panel plot containing a time series plot (i.e. ```type = "l"```) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).




