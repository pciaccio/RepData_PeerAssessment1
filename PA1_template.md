This assignment uses data from a personal monitoring. The dataset comes
from activity.csv in the following zip file
(<https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip>)

The variables included in the dataset are:

-   Steps: Number of steps taken in a 5-minute interval
-   Date: The date on which the measurement was takenin YYYY-MM-DD
    format
-   Interval: Identifierfor the 5-minute interval in which measurement
    was taken

Loading and preprocessing the data
----------------------------------

Loading the data from the website listed above.

    temp<-tempfile()
    download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
    data<-read.csv(unz(temp,"activity.csv"))
    unlink(temp)

Processing the data so that the Date variable is of class "Date".

    data$date<-as.Date(as.character(data$date))

What is the mean total of steps taken per day?
----------------------------------------------

The number of steps needs to be aggregated by day to come up with the
total number of steps per day. This data was then used to make a
histogram of the total number of steps per day.

    total<-aggregate(data$steps,by=list(data$date),FUN=sum,na.rm=TRUE)
    colnames(total)<-c("Date","Steps")
    hist(total$Steps,main="Total Steps Taken Each Day",xlab="Steps Per Day",col="red")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)<!-- -->

The mean and the median for steps taken per day is then calculated.

The mean number of steps taken in a day is:

    mean(total$Steps,na.rm=TRUE)

    ## [1] 9354.23

The median number of steps taken in a day is:

    median(total$Steps,na.rm=TRUE)

    ## [1] 10395

What is the average daily activity pattern?
-------------------------------------------

To examine the average daily activity pattern the steps needs to be
aggregated by interval and then plotted.

    inter<-aggregate(data$steps,by=list(data$interval),FUN=mean,na.rm=TRUE)
    colnames(inter)<-c("Interval","Steps")
    plot(inter$Interval,inter$Steps,type="l",ylab="Average Number of Steps", xlab="5-Minute Intervals",main="Average Daily Activity Pattern")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-6-1.png)<!-- -->

The 5-minute interval with the maximum number of steps is calculated as:

    max<-inter$Interval[which.max(inter$Steps)]

The 5-minute interval, on average across all the days, that contains the
maximum number of steps is the interval from 835 to 840.

Input Missing Values
--------------------

This step calculates the number of missing values for each variable.

    s<-sum(is.na(data$steps))
    d<-sum(is.na(data$date))
    i<-sum(is.na(data$interval))

-   The total number of missing steps is 2304.
-   The total number of missing dates is 0.
-   The total number of missing intervals is 0.

The only variable with missing data is steps.

Since there were days with all of the intervals missing, I decided to
use the mean steps of the 5-minute intervals to fill in the missing
number of steps.

    data_1<-subset(data,is.na(data$steps))
    data_2<-subset(data,!is.na(data$steps))
    data_1$steps<-inter$Steps[match(data_1$interval,inter$Interval)]
    data_3<-rbind(data_1,data_2)

The new number of steps per day is then aggregated and used to make a
historgram of the updated data.

    total_1<-aggregate(data_3$steps,by=list(data_3$date),FUN=sum,na.rm=TRUE)
    colnames(total_1)<-c("Date","Steps")
    hist(total_1$Steps,main="Total Steps Taken Each Day",xlab="Steps Per Day",col="blue")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-10-1.png)<!-- -->

The mean of the new dataset is:

    mean(total_1$Steps)

    ## [1] 10766.19

The median of the new dataset is:

    median(total_1$Steps)

    ## [1] 10766.19

The mean and median both changed with the replacement of the NA values
with the average number of steps taken in the interval.

The mean has increased by 1411.959171.

The median has increased by 371.1886792.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

I used the completed data from the previous step for this analysis. A
variable was added to the dataset, to indicate whether the date
represented a day from the weekend or weekday.

    Day<-weekdays(data_3$date,abbreviate=TRUE)
    data_3<-cbind(data_3,Day)
    w_end<-subset(data_3,data_3$Day=="Sat"|data_3$Day=="Sun")
    w_day<-subset(data_3,data_3$Day=="Mon"|data_3$Day=="Tue"|data_3$Day=="Wed"|data_3$Day=="Thu"|data_3$Day=="Fri")
    w_end$Day<-"Weekend"
    w_day$Day<-"Weekday"
    data_4<-rbind(w_day,w_end)

To examine the activity patterns of the weekend and weekdays a lattice
plot was used. Therefore, the Lattice and Dataset libraries hadto be
called. The plot was then created from the data.

    library(datasets)
    library(lattice)
    xyplot(steps~interval|Day,data=data_4,type="l",layout=c(1,2))

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)<!-- -->

Based on the plots shown above, it would appear that there is a
difference between the activity patterns of weekdays and weekends. One
difference is that periods of high activity start at an earlier interval
on the Weekdays than on Weekends. It also appears that the activity
level on Weekdays is generally higher than the Weekends.
