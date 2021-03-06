# Reproducible Research - Peer Assessment 1

```{r, echo=FALSE, results='hide', warning=FALSE, message=FALSE}
library(ggplot2)
library(scales)
library(Hmisc)
```

## Loading and preprocessing the data
##### 1. Load the data (i.e. read.csv())
```{r, results='markup', warning=TRUE, message=TRUE}
if(!file.exists('activity.csv')){
    unzip('activity.zip')
}
Data_activity <- read.csv('activity.csv')
```
##### 2. Process/transform the data (if necessary) into a format suitable for your analysis

## What is mean total number of steps taken per day?
```{r}
steps_By_Day <- tapply(Data_activity$steps, Data_activity$date, sum, na.rm=TRUE)
```

##### 1. Make a histogram of the total number of steps taken each day
```{r}
qplot(steps_By_Day, xlab='Total steps per day', ylab='Frequency Using Binwidth 2000', binwidth=2000)
```

##### 2. Calculate and report the mean and median total number of steps taken per day
```{r}
Mean_steps_By_Day <- mean(steps_By_Day)
Median_steps_By_Day <- median(steps_By_Day)
```
* Mean =  `r Mean_steps_By_Day`
* Median =  `r Median_steps_By_Day`

-----

## What is the average daily activity pattern?
```{r}
average_Steps_Per_Time_Block <- aggregate(x=list(mean_Steps=Data_activity$steps), by=list(interval=Data_activity$interval), FUN=mean, na.rm=TRUE)
```

##### 1. Make a time series plot
```{r}
ggplot(data=average_Steps_Per_Time_Block, aes(x=interval, y=mean_Steps)) +
    geom_line() +
    xlab("5-minute interval") +
    ylab("Average number of steps taken") 
```

##### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r}
Max_Steps <- which.max(average_Steps_Per_Time_Block$mean_Steps)
time_Max_Steps <-  gsub("([0-9]{1,2})([0-9]{2})", "\\1:\\2", average_Steps_Per_Time_Block[Max_Steps,'interval'])
```

* Max Steps at = `r time_Max_Steps`

----

## Imputing missing values
##### 1. Calculate and report the total number of missing values in the dataset 
```{r}
num_of_Missing_Values <- length(which(is.na(Data_activity$steps)))
```

* Number of missing values =  `r num_of_Missing_Values`

##### 2. Devise a strategy for filling in all of the missing values in the dataset.
##### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

##### Impute missing value with mean values from steps data by using function impute.

```{r}
Imputed_Data_activity <- Data_activity
Imputed_Data_activity$steps <- impute(Data_activity$steps, fun=mean)
```



##### 4. Make a histogram of the total number of steps taken each day 
```{r}
Imputed_steps_By_Day <- tapply(Imputed_Data_activity$steps, Imputed_Data_activity$date, sum)
qplot(Imputed_steps_By_Day, xlab='Total steps per day (Imputed)', ylab='Frequency Using Binwidth 2000', binwidth=2000)
```

##### Calculate and report the mean and median total number of steps taken per day. 
```{r}
Mean_Imputed_steps_By_Day <- mean(Imputed_steps_By_Day)
Median_Imputed_steps_By_Day <- median(Imputed_steps_By_Day)
```
* Mean (Imputed) = `r Mean_Imputed_steps_By_Day`
* Median (Imputed) =  `r Median_Imputed_steps_By_Day`



* Mean and median values are higher after imputing missing data. The reason is that in the original data, there are some days with `steps` values `NA` for any `interval`. The total number of steps taken in such days are set to 0s by  default. However, after replacing missing `steps` values with the mean `steps` of associated `interval` value, these 0 values are removed from the histogram of total number of steps taken each day.





----

## Are there differences in activity patterns between weekdays and weekends?
##### 1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```{r}
Imputed_Data_activity$dateType <-  ifelse(as.POSIXlt(Imputed_Data_activity$date)$wday %in% c(0,6), 'weekend', 'weekday')
```

##### 2. Make a panel plot containing a time series plot

```{r}
Averaged_Imputed_Data_activity <- aggregate(steps ~ interval + dateType, data=Imputed_Data_activity, mean)
ggplot(Averaged_Imputed_Data_activity, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("Average number of steps")
```
