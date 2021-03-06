# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
# Set working directory to the folder which contains activity.zip Check if
# file 'activity.csv' is present in working directory. If it isn't unzip
# 'activity.zip'. This will create the file 'activity.csv'in the working
# directory.
if (!file.exists("activity.csv")) {
    unzip("activity.zip")
}

# Read the csv file into a data frame called activity
activity <- read.csv("activity.csv")

# Display the first few entries in activity data frame
head(activity)
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

```r

# Convert the interval column into a factor, since it is parsed as an
# integer.  This preprocessing step helps with some of the later questions
activity$interval <- as.factor(activity$interval)
```



## What is mean total number of steps taken per day?

```r
# Split steps column in the activity data frame by the date column.
steps_by_day <- split(activity$steps, activity$date)

# Apply the sum function to the list steps_by_day to count the number of
# steps walked by the person in a given day.
total_steps_by_day <- sapply(steps_by_day, sum, na.rm = T)

# Plot a histogram of total number of steps taken per day
hist(total_steps_by_day, breaks = 20, xlab = "Total steps taken per day", ylab = "Frequency (number of days)", 
    main = "Histogram of total steps taken per day")
```

![plot of chunk histogram_missing](figure/histogram_missing.png) 

```r

# Find the mean steps per day
mean_steps_by_day <- mean(total_steps_by_day)
print(mean_steps_by_day)
```

```
## [1] 9354
```

```r

# Find the median steps per day
median_steps_by_day <- median(total_steps_by_day)
print(median_steps_by_day)
```

```
## [1] 10395
```



## What is the average daily activity pattern?


```r
# Split steps column in the activity data frame by the interval column.
steps_by_interval <- split(activity$steps, activity$interval)

# Apply the mean function to the list steps_by_interval
mean_steps_by_interval <- sapply(steps_by_interval, mean, na.rm = T)

# Make a time series plot of the 5-minute interval (x-axis) and the average
# number of steps taken, averaged across all days (y-axis)
plot(names(mean_steps_by_interval), mean_steps_by_interval, type = "l", xlab = "Time interval", 
    ylab = "Mean number of steps taken", main = "Mean number of steps taken per 5 minute interval")
```

![plot of chunk time_series](figure/time_series.png) 

```r

# Find the five minute interval that contains the maximum number of steps on
# average across all the days in the dataset
names(which.max(mean_steps_by_interval))
```

```
## [1] "835"
```


## Imputing missing values

```r
# Find the total number of rows that have NAs Note complete.cases applied to
# activity returns a logical vector with False whenever a row has NA in any
# column and True otherwise.
sum(!complete.cases(activity))
```

```
## [1] 2304
```

```r

# Replace missing values (i.e. NAs) with the mean value corresponding to the
# interval First create a copy of the activity data frame
activity_no_missing <- activity

activity_no_missing$steps[is.na(activity_no_missing$steps)] <- mean_steps_by_interval[activity$interval[is.na(activity$steps)]]

# Make sure there are no NAs in the new data frame
sum(!complete.cases(activity_no_missing))
```

```
## [1] 0
```

```r

# Display the first few rows of the new data frame. Note that the NAs have
# been replaced.
head(activity_no_missing)
```

```
##     steps       date interval
## 1 1.71698 2012-10-01        0
## 2 0.33962 2012-10-01        5
## 3 0.13208 2012-10-01       10
## 4 0.15094 2012-10-01       15
## 5 0.07547 2012-10-01       20
## 6 2.09434 2012-10-01       25
```

```r

# Split steps column in the activity_no_missing data frame by the date
# column.
steps_by_day_no_missing <- split(activity_no_missing$steps, activity_no_missing$date)

# Apply the sum function to the list steps_by_day_no_missing to count the
# number of steps walked by the person in a given day.
total_steps_by_day_no_missing <- sapply(steps_by_day_no_missing, sum)

# Plot a histogram of total number of steps taken per day with the missing
# data filled in
hist(total_steps_by_day_no_missing, breaks = 20, xlab = "Total steps taken per day", 
    ylab = "Frequency (number of days)", main = "Histogram of total steps taken per day")
```

![plot of chunk histogram_no_missing](figure/histogram_no_missing.png) 

```r

# Find the mean steps per day with the missing data filled in
mean_steps_by_day_no_missing <- mean(total_steps_by_day_no_missing)
print(mean_steps_by_day_no_missing)
```

```
## [1] 10766
```

```r

# Find the median steps per day with the missing data filled in
median_steps_by_day_no_missing <- median(total_steps_by_day_no_missing)
print(median_steps_by_day_no_missing)
```

```
## [1] 10766
```


The mean and median are larger than the ones calculated from the data with missing values.


## Are there differences in activity patterns between weekdays and weekends?


```r
# Add another column to the data set to indicate whether the given day is a
# weekday or weekend.  We will be using the filled-in data frame for this
# part.

# First convert the date column to POSIXlt format to use with the weekdays
# function.
activity_no_missing$date <- as.POSIXlt(activity_no_missing$date)

# Append a new column 'Day' to the data frame to indicate whether it is
# weekend or weekday.  We accomplish this using the ifelse function. This
# function uses the weekdays function to find the day of the week for the
# entries in activity_no_missing$date. If the day is Saturday or Sunday, it
# adds an entry 'Weekend' in the 'Day' column, else adds an entry 'Weekday'.
activity_no_missing$Day <- factor(ifelse(weekdays(activity_no_missing$date) == 
    "Sunday" | weekdays(activity_no_missing$date) == "Saturday", "Weekend", 
    "Weekday"))

# Inspect the first few rows of the new data frame.
head(activity_no_missing)
```

```
##     steps       date interval     Day
## 1 1.71698 2012-10-01        0 Weekday
## 2 0.33962 2012-10-01        5 Weekday
## 3 0.13208 2012-10-01       10 Weekday
## 4 0.15094 2012-10-01       15 Weekday
## 5 0.07547 2012-10-01       20 Weekday
## 6 2.09434 2012-10-01       25 Weekday
```

```r

# Split the data frame according to weekday and weekend.
activity_no_missing_weekday <- activity_no_missing[activity_no_missing$Day == 
    "Weekday", ]
activity_no_missing_weekend <- activity_no_missing[activity_no_missing$Day == 
    "Weekend", ]

# Split steps column in the activity_no_missing_weekday data frame by the
# interval column.
steps_by_interval_weekday <- split(activity_no_missing_weekday$steps, activity_no_missing_weekday$interval)

# Apply the mean function to the list steps_by_interval_weekday
mean_steps_by_interval_weekday <- sapply(steps_by_interval_weekday, mean, na.rm = T)

# Split steps column in the activity_no_missing_weekend data frame by the
# interval column.
steps_by_interval_weekend <- split(activity_no_missing_weekend$steps, activity_no_missing_weekend$interval)

# Apply the mean function to the list steps_by_interval_weekday
mean_steps_by_interval_weekend <- sapply(steps_by_interval_weekend, mean, na.rm = T)

# Convert mean_steps_by_interval_weekday vector to a data frame for the time
# series plot.
mean_steps_by_interval_weekday_df <- data.frame(interval = as.numeric(names(mean_steps_by_interval)), 
    steps = mean_steps_by_interval_weekday, Day = "Weekday")

# Convert mean_steps_by_interval_weekday vector to a data frame for the time
# series plot.
mean_steps_by_interval_weekend_df <- data.frame(interval = as.numeric(names(mean_steps_by_interval)), 
    steps = mean_steps_by_interval_weekend, Day = "Weekend")

# Combine mean_steps_by_interval_weekday_df and
# mean_steps_by_interval_weekend_df into a single data frame to create the
# panel time series plot.
mean_steps_by_interval_df <- rbind(mean_steps_by_interval_weekday_df, mean_steps_by_interval_weekend_df)

# Load the lattice library
library(lattice)

# Create a panel plot containing a time series plot of th 5-minute interval
# (x-axis) and the average number of steps taken, averaged across all
# weekday days or weekend days (y-axis).
xyplot(mean_steps_by_interval_df$steps ~ mean_steps_by_interval_df$interval | 
    mean_steps_by_interval_df$Day, type = "l", xlab = "Time interval", ylab = "Mean number of steps taken", 
    layout = c(1, 2))
```

![plot of chunk panel](figure/panel.png) 

