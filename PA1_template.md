# Reproducible Research: Course Project 1

```{r setup, include=TRUE}
knitr::opts_chunk$set(echo = TRUE)
```

## Analysis on Original Dataset

Load the activity monitoring dataset into the R environment and check the first few entries using head().

```{r activity}
activity_data <- read.csv("activity.csv", sep=",",header=TRUE)
head(activity_data)
```

Plot the total number of steps for each day.

```{r stepsperday, echo = TRUE}
spd <- with(activity_data, tapply(steps, date, sum, na.rm=TRUE))
with(
  activity_data,
  plot(
    1:length(spd),
    spd,
    xlab = "Day",
    ylab = "Number of Steps",
    main = "Total Number of Steps per Day"
  )
)
```

The following plot shows the histogram of number of steps taken per day. Note that the bin-width is set to '10' to give us a better visualization of the data.

```{r plot_hist, echo=TRUE}
hist(
  spd,
  main = "Histogram of Steps per Day",
  xlab = "Steps per Day",
  breaks = 10
)
```

Now take a look into the data's mean and median for each day.

```{r mean_median, echo = TRUE}
steps_mean <- with(activity_data, tapply(steps, date, mean, na.rm=TRUE))
steps_median <- with(activity_data, tapply(steps, date, median, na.rm=TRUE))
par(mfcol = c(2,1))
plot(
  1:length(steps_mean),
  steps_mean,
  xlab = "Day",
  ylab = "",
  main = "Mean"
)
plot(
  1:length(steps_median),
  steps_median,
  xlab = "Day",
  ylab = "",
  main = "Median"
)
```

Notice that steps_median plot either shows 0s or NAs. Looking at its summary, we can verify that the median data contains a vector of 0s and NAs.

```{r median_verify, echo = TRUE}
summary(steps_median)
```

Plot time-series data of average number of steps per interval across all days. 

```{r time-series, echo=TRUE}
ave_daily <- with(activity_data, tapply(steps, interval, mean, na.rm = TRUE))
plot(
  ave_daily,
  xlab = "Interval Index",
  ylab = "Average Number of Steps",
  main = "Average Number of Steps per Interval",
  type = "l"
)
```

The maximum average value of number of steps is also seen below along with its corresponding interval. The maximum average is 206.1698 steps during the 835th interval for the day.

```{r max, echo=TRUE}
ave_daily[which.max(as.numeric(ave_daily))]
```

##Analysis of Imputed Dataset

Calculate the number of NAs present in the dataset.

```{r na_impute, echo = TRUE}
nas <- which(is.na(activity_data))
length(nas)
```

###Technique for Imputing Data

Impute missing NA values using the daily interval averages computed from the previous section. Now check the first few entries of the new dataset by using head().

```{r impute_check, echo = TRUE}
new_data <- activity_data
new_data$steps[nas] <- ave_daily
head(new_data)
```

Looking at the graph below, the graph differs significantly when it comes to the frequency of the distribution of number of steps. Since the imputed data came from the interval averages, the distribution values are more frequent at the center of the graph compared to the previous histogram. As with the previous histogram, bin-width is set to '10'.

The new mean and median is also reported below using a histogram as was previously done in the previous section. Due to imputed data, the median plot shows us 2 value groups as compared to the original median vector which only held 0s and NAs.

```{r impute_analysis, echo = TRUE}

new_spd <- with(new_data, tapply(steps, date, sum, na.rm=TRUE))
hist(
  new_spd,
  main = "Histogram of Steps per Day",
  xlab = "Steps per Day",
  breaks = 10
)

new_steps_mean <- with(new_data, tapply(steps, date, mean, na.rm=TRUE))
new_steps_median <- with(new_data, tapply(steps, date, median, na.rm=TRUE))

par(mfcol = c(2,1))
plot(
  1:length(new_steps_mean),
  new_steps_mean,
  xlab = "Day",
  ylab = "",
  main = "Mean"
)
plot(
  1:length(new_steps_median),
  new_steps_median,
  xlab = "Day",
  ylab = "",
  main = "Median"
)

```

A new column, day_type, is added to the new dataset for this section. The variable day_type can take only 1 of two values: "weekday" and "weekend". The 1st few entries are shown using head(). The graph below shows the differences between weekday and weekend daily averages. From the graphs, weekend daily averages are greater during the middle and latter parts of the day which indicates greater activity during these times.

```{r weekdays, echo=TRUE, message=FALSE, warning=FALSE}
library(lubridate)
library(dplyr)

wday_activity <- mutate(new_data, day_type = factor(wday(as.Date(date)) %in% 2:6, levels = c(TRUE, FALSE), labels = c("weekday","weekend")))
head(wday_activity)

wday_ave <- with(wday_activity, tapply(steps, list(interval,day_type), mean, na.rm = TRUE))
par(mfrow = c(2,1), mar = c(2,2,1.5,1.5))
plot(
  wday_ave[,1],
  main = "Weekday",
  xlab = "",
  xaxt = 'n',
  ylab = "",
  type = "l"
)

plot(
  wday_ave[,2],
  xlab = "Interval Index",
  ylab = "",
  main = "Weekend",
  type = "l"
)
```


