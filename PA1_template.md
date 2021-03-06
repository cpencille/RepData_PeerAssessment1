---
title: "Reproducible Research Assignment Week 2"
author: "Chris Pencille"
date: "July 24, 2016"
output: html_document
keep_md: true
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

#Loading and preprocessing the data
Loading libraries needed for whole assignment, loading data, and reordering columns for personal preference of order

```{r echo=TRUE}
library(plyr)
library(dplyr)
library(lattice)
data1<-read.csv("activity.csv")
data1<-data1[,c(2,3,1)]
```

#What is the mean total number of steps taken per day?
1. Look at the data to see what we will need to do some calculations
2. Aggregate the data and then output the data to see what the average steps by day is
3. Rename column names for usage in histogram and output
4. Create Histogram

```{r echo=TRUE}
str(data1)
datasum<-aggregate(data1$steps,by = list(Category = data1$date),FUN = sum, na.rm = TRUE)
names(datasum)[names(datasum)=="Category"]<-"Day"
names(datasum)[names(datasum)=="x"]<-"Steps"
with(data=datasum,hist(Steps))
data1median<-median(data1$steps,na.rm = TRUE)
data1average<-mean(data1$steps,na.rm = TRUE)
data1average
data1median
```

#What is the average daily pattern activity?
1. Create datatable of average steps and rename columns
2. Create time series chart
3. Figure out which time interval is the max average
```{r echo=TRUE}
data1averages<-aggregate(data1$steps,by = list(Category = data1$interval),FUN = mean, na.rm = TRUE)
names(data1averages)[names(data1averages)=="Category"]<-"Interval"
names(data1averages)[names(data1averages)=="x"]<-"Steps"
with(data = data1averages, plot(Interval,Steps,type = "l"))
maxsteps<-max(data1averages$Steps,na.rm=TRUE)
data1averages[data1averages$Steps==maxsteps,]
```
Time interval 835 contains the max average step count


#Imputting Missing Values
1. Create a data table of all the NA rows
2. Join that to add the average of steps by interval and removing the NA steps column
3. Create a data table without the NA rows and join that to the imputted table
4. Make a histogram of total steps after creating an aggregated table
5. The values don't differ at all
```{r echo=TRUE}
narows<-data1[is.na(data1$steps),]
str(narows)
names(narows)[names(narows)=="interval"]<-"Interval"
data1nona<-data1[!is.na(data1$steps),]
mergeddata<-join(narows,data1averages,by = "Interval",type = "left")
mergeddata<-mergeddata[,c(1,2,4)]
names(mergeddata)[names(mergeddata)=="Interval"]<-"interval"
names(mergeddata)[names(mergeddata)=="Steps"]<-"steps"
data1all<-union(data1nona,mergeddata)

sumnona<-aggregate(data1all$steps,by = list(Category = data1all$date),FUN = sum)
hist(sumnona$x)
nonamean<-mean(data1all$steps)
nonamedian<-median(data1all$steps)
nonamean
nonamedian
```
#Are there differences in activity patterns between weekdays and weekends?
1. Reassign the date column to a date format
2. Create a column of the weekday from the date
3. Create a column of whether the weekday is a Weekday or Weekend day
4. Make the panel plot

```{r echo=TRUE}
data1$date<-as.Date(data1$date)
data1$weekday<-weekdays(data1$date)
for (i in 1:length(data1$date)){
if (data1$weekday[i]=="Monday"){
    data1$wdwe[i]<-"Weekday"
} else if (data1$weekday[i]=="Tuesday"){
    data1$wdwe[i]<-"Weekday"
} else if (data1$weekday[i]=="Wednesday"){
    data1$wdwe[i]<-"Weekday"
} else if (data1$weekday[i]=="Thursday"){
    data1$wdwe[i]<-"Weekday"
} else if (data1$weekday[i]=="Friday"){
    data1$wdwe[i]<-"Weekday"
} else if (data1$weekday[i]=="Saturday"){
    data1$wdwe[i]<-"Weekend"
} else {data1$wdwe[i]<-"Weekend"}
}
data1agg<-aggregate(data1$steps,by = list(data1$wdwe,data1$interval),FUN = mean,na.rm=TRUE)
names(data1agg)[names(data1agg)=="x"]<-"Steps"
names(data1agg)[names(data1agg)=="Group.2"]<-"Interval"
names(data1agg)[names(data1agg)=="Group.1"]<-"WeekdayWeekend"
xyplot(Steps ~ Interval | WeekdayWeekend, data = data1agg,type="l",layout = c(2,1))
```