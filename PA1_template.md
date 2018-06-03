The data
--------

    activity <- read.csv("activity.csv", header = TRUE)
    activity$date <- as.Date(activity$date)

What is mean total number of steps taken per day?
-------------------------------------------------

### 1. Calculate the total number of steps taken per day

    TotalSteps <- aggregate(x = list(Steps = activity$steps), by = list(Date = activity$date), FUN = sum, na.rm = TRUE)

### 2. Make a histogram of the total number of steps taken each day

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.3.3

    h <- ggplot(TotalSteps, aes(Steps))
    h + geom_histogram(bins = 30)

![](PA1_template_files/figure-markdown_strict/h-1.png)

### 3. Calculate and report the mean and median of the total number of steps taken per day

    mean(TotalSteps$Steps)

    ## [1] 9354.23

    median(TotalSteps$Steps)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

    DailySteps <- aggregate(x = list(Steps = activity$steps), by = list(Interval = activity$interval), FUN = mean, na.rm = TRUE)
    ts <- ggplot(DailySteps, aes(Interval, Steps))
    ts + geom_line()

![](PA1_template_files/figure-markdown_strict/tsplot-1.png)

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

    DailySteps[DailySteps$Steps == max(DailySteps$Steps), ]

    ##     Interval    Steps
    ## 104      835 206.1698

Imputing missing values
-----------------------

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

    i <- 0; c <- 0
    for(i in 1:nrow(activity)){
      if (is.na(activity[i, ]$steps) == TRUE) c <- c + 1
    }; c

    ## [1] 2304

### 2. Devise a strategy for filling in all of the missing values in the dataset.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

Strategy: to fill in the missing values with the mean values for the
5-min interval.

    activity.NAfilled <- activity
    for(i in 1:nrow(activity.NAfilled)){
      if (is.na(activity.NAfilled[i, ]$steps) == TRUE) activity.NAfilled[i, ]$steps <- DailySteps$Steps[activity.NAfilled[i, ]$interval == DailySteps$Interval]
    }

### 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

    TotalSteps.NAfilled <- aggregate(x = list(Steps = activity.NAfilled$steps), by = list(Date = activity.NAfilled$date), FUN = sum, na.rm = TRUE)
    ts.NAfilled <- ggplot(TotalSteps.NAfilled, aes(Steps))
    ts.NAfilled + geom_histogram(bins = 30)

![](PA1_template_files/figure-markdown_strict/impute-1.png)

    mean(TotalSteps.NAfilled$Steps)

    ## [1] 10766.19

    median(TotalSteps.NAfilled$Steps)

    ## [1] 10766.19

Impact of imputing: Data exhibits less skewed, more of a bell-shaped
distribution.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

    day <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")
    end <- c("Saturday", "Sunday")
    for(i in 1:nrow(activity.NAfilled)){
      activity.NAfilled$week[i] <- ifelse(weekdays(activity.NAfilled$date[i]) %in% day, "weekday", "weekend")
    }
    activity.NAfilled$week <- as.factor(activity.NAfilled$week)

### 2.Make a panel plot containing a time series plot

    library(gtable); library(grid)

    ## Warning: package 'gtable' was built under R version 3.3.3

    DailySteps.NAfilled <- aggregate(x = list(Steps = activity.NAfilled$steps), by = list(Interval = activity.NAfilled$interval, Week = activity.NAfilled$week), FUN = mean)
    ts1 <- qplot(x = Interval, y = Steps, data = DailySteps.NAfilled[DailySteps.NAfilled$Week == "weekday", ], geom = "line") + ggtitle("Weekday")
    ts2 <- qplot(x = Interval, y = Steps, data = DailySteps.NAfilled[DailySteps.NAfilled$Week == "weekend", ], geom = "line") + ggtitle("Weekend")
    g1 <- ggplotGrob(ts1); g2 <- ggplotGrob(ts2)
    g <- rbind(g1, g2, size = "first")
    g$widths <- unit.pmax(g1$widths, g2$widths)
    grid.newpage(); grid.draw(g)

![](PA1_template_files/figure-markdown_strict/panel-1.png)
