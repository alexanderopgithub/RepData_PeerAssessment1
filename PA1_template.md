Question 1: Loading and preprocessing the data
----------------------------------------------

We load the activity data from csv into dataframe activity0. The
original data contains 17568 observations which is partially missing in
variable steps.

    activity0 <- read.csv("activity.csv")
    str(activity0) # 17568 observations

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

We remove the rows with missing data and obtain data frame activity.

    wd  <- complete.cases(activity0)
    activity = activity0[wd,] # 15264 left
    str(activity)

    ## 'data.frame':    15264 obs. of  3 variables:
    ##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

Question 2: What is mean total number of steps taken per day?
-------------------------------------------------------------

We use library dplyr to calculate the total number of steps per day.

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    resQ2 <- activity %>%
      group_by(date) %>%
      summarize(totalsteps = sum(steps,na.rm=TRUE))

The mean and median are

    mean(resQ2$totalsteps)

    ## [1] 10766.19

    median(resQ2$totalsteps)

    ## [1] 10765

Use library ggplot2 to create a histogram

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.4.4

    qplot(totalsteps,data = resQ2,bins = 10,xlab = "Total number of steps",main = "Total number of steps per day [remove rows missing values]")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Question 3: What is the average daily activity pattern?
-------------------------------------------------------

Calculate the average number of steps taken averaged across all days in
the dataset.

    resQ3 <- activity %>%
      group_by(interval) %>%
      summarize(avsteps = mean(steps,na.rm=TRUE))

We find the maximum number of steps is on average made on interval 8:35

    maxsteps <- which.max(resQ3$avsteps)
    resQ3$interval[maxsteps]

    ## [1] 835

This brings the following lineplot

    qplot(interval,avsteps, data = resQ3, ylab= "Average number of steps", main = "Average number of steps per 5 minute interval") + geom_line()

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)

Question 4: Imputing missing values
-----------------------------------

There were missing data in the original dataset.

    wd  <- complete.cases(activity0)
    sum(!wd) # 2304 rows with NA

    ## [1] 2304

Above we took out the rows with missing values. Here we impute values.
We recognize that we cannot add mean/median from that specific day as
there are days which are completely NA. Thus we replace missing values
with the mean for the 5 minute interval. Create dataframe activity\_ext
which picks the original number of steps when it is available, and
otherwise the mean value.

    activity_ext <- merge(activity0,resQ3,by.x="interval",by.y="interval")
    activity_ext$steps2 = ifelse (is.na(activity_ext$steps), activity_ext$avsteps,activity_ext$steps)

And derive the total number of steps per day

    resQ4 <- activity_ext %>%
      group_by(date) %>%
      summarize(totalsteps = sum(steps2,na.rm=TRUE))

The mean and median have hardly changed from the above situation where
we removed the rows

    mean(resQ4$totalsteps)

    ## [1] 10766.19

    median(resQ4$totalsteps)

    ## [1] 10766.19

The histogram becomes

    qplot(totalsteps,data = resQ4,bins = 10,xlab = "Total number of steps",main = "Total number of steps per day [imputed values]")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-13-1.png)

Thus we conclude the impact of imputing missing data versus deleting the
rows is limited.

Q5 Are there differences in activity patterns between weekdays and weekends?
----------------------------------------------------------------------------

Add a new factor variable to our dataframe indicating whether it was
weekday or weekend

    library(lubridate)

    ## 
    ## Attaching package: 'lubridate'

    ## The following object is masked from 'package:base':
    ## 
    ##     date

    activity_ext$date = ymd(activity_ext$date) # make a date object

    activity_ext$weekday = wday(activity_ext$date)
    indWeekend = (activity_ext$weekday== 7 |  activity_ext$weekday== 1)
    activity_ext$week_weekend[indWeekend] = "weekend"
    activity_ext$week_weekend[!indWeekend] = "week"
    activity_ext$week_weekend =as.factor(activity_ext$week_weekend)

Create panel plot with the average number of steps

    resQ5 <- activity_ext %>%
      group_by(interval,week_weekend) %>%
      summarize(avsteps = mean(steps,na.rm=TRUE))

    qplot(interval,avsteps, data = resQ5,facets = .~week_weekend, ylab= "Average number of steps") + geom_line()

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-15-1.png)
