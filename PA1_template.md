Downloading the Data and Reading into R
---------------------------------------

Firstly, we have to create a folder for data that we are going to
download. If "data" folder doesn't exist, we will create it. Then we
will download the data with download.file() function and name it
"repres.zip". After that, we we will decompress the zip file and read
the "activity.csv" data set into R.

    if(!file.exists("./data")){
      dir.create("./data")
    }
    fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(fileURL, destfile = "./data/repres.zip")
    unzip("./data/repres.zip", exdir = "./data")
    data <- read.csv("./data/activity.csv")

What is mean total number of steps taken per day?
-------------------------------------------------

We are going to construct histogram of the total number of steps taken
each day. Mean and median values of total number of steps taken each day
are calcutated.

    aggregate_step_by_day <- aggregate(steps ~ date, data, sum, na.rm = T)
    hist(aggregate_step_by_day$steps, xlab = "Total Steps", main = "Histogram of Total Steps")

<img src="proj1_files/figure-markdown_strict/unnamed-chunk-2-1.png" style="display: block; margin: auto;" />

    mean(aggregate_step_by_day$steps)

    ## [1] 10766.19

    median(aggregate_step_by_day$steps)

    ## [1] 10765

What is the average daily activity pattern?
-------------------------------------------

Time series of the 5-minute intervals and the average number of steps
taken averaged across all days is plotted. 5-minute interval that
contains the maximum number of steps is found.

    aggregate_maxstep_by_interval <- aggregate(steps ~ interval, data, mean, na.rm = T)
    plot(aggregate_maxstep_by_interval$interval, aggregate_maxstep_by_interval$steps,
         type = "l", ylab = "Average Number of Steps", xlab = "Interval")

<img src="proj1_files/figure-markdown_strict/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

    maxstep_by_interval <- aggregate_maxstep_by_interval[which.max(aggregate_maxstep_by_interval$steps), 1]
    maxstep_by_interval

    ## [1] 835

Imputing missing values
-----------------------

Total number of NA values in steps column is found. Missing values are
imputed using the 5-minute interval averages. New data set called
"data\_filled" is created with imputed NA values. Histogram of the total
number of steps taken each day, mean, and median are found. The mean
stays approximately the same because we are imputed the missing values
with the 5-minute interval means. Median value is shifted a little bit,
but it is mainly caused by the positions of missing values.

    sum(is.na(data$steps))

    ## [1] 2304

    meanInterval <- function(x){
      aggregate_maxstep_by_interval[aggregate_maxstep_by_interval$interval == x,]$steps
    }
    data_filled <- data
    for(i in 1:nrow(data_filled)){
      if(is.na(data_filled[i, ]$steps)){
        data_filled[i, ]$steps <- meanInterval(data_filled[i, ]$interval)
      }
    }
    sum(is.na(data_filled$steps))

    ## [1] 0

    aggregate_step_by_day_filled <- aggregate(steps ~ date, data_filled, sum)
    hist(aggregate_step_by_day_filled$steps, main = "Total Number of Steps after Imputing",
         xlab = "Total Number of Steps")

<img src="proj1_files/figure-markdown_strict/unnamed-chunk-4-1.png" style="display: block; margin: auto;" />

    mean(aggregate_step_by_day_filled$steps)

    ## [1] 10766.19

    median(aggregate_step_by_day_filled$steps)

    ## [1] 10766.19

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

For this part of the project, "lubridate" package is used for
identifying the weekends and weekdays. "day" column is consisted of 2
level factor with levels "weekday" and "weekend". "lattice" package is
used for plotting the time series plot of 5-minute interval and the
average number of steps taken, averaged across all weekdays or weekend
days.

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

    data_filled$date <- ymd(data_filled$date)
    for(i in 1:nrow(data_filled)){
      if(wday(data_filled$date[i]) == 1 | wday(data_filled$date[i]) == 7){
        data_filled$day[i] <- "weekend"
      }
      else{
        data_filled$day[i] <- "weekday"
      }
    }
    data_filled$day <- factor(data_filled$day, levels = c("weekday", "weekend"))
    unique(data_filled[data_filled$day == "weekend", ]$date)

    ##  [1] "2012-10-06" "2012-10-07" "2012-10-13" "2012-10-14" "2012-10-20"
    ##  [6] "2012-10-21" "2012-10-27" "2012-10-28" "2012-11-03" "2012-11-04"
    ## [11] "2012-11-10" "2012-11-11" "2012-11-17" "2012-11-18" "2012-11-24"
    ## [16] "2012-11-25"

    library(lattice)
    aggregated <- aggregate(steps ~ interval + day, data_filled, mean)
    xyplot(steps ~ interval | factor(day), data = aggregated, aspect = 1/2, type = "l")

<img src="proj1_files/figure-markdown_strict/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />
