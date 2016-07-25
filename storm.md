# Analysis of U.S. Severe Weather Events to Understand the Most Impactful on Human and Economic Health
wdsteck  
`r format(Sys.Date(), "%B %d, %Y")`  


```r
require(knitr)
```

```
## Loading required package: knitr
```

```r
require(markdown)
```

```
## Loading required package: markdown
```

```r
require(lubridate)
```

```
## Loading required package: lubridate
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following object is masked from 'package:base':
## 
##     date
```

```r
knitr::opts_chunk$set(echo = TRUE)
```

## Synopsis

Storms and other severe weather events can cause both public health
and economic impacts for communities and municipalities impacted
by those events. Many severe events can result in fatalities,
injuries, and property damage, and preventing such outcomes to the
extent possible is a key concern. Understanding the impacts and
then educating the public on those impacts
and methods of remediating the impact is the goal.

This analysis shows the event type reported in the data with the
greatest impact on population health is, by far, the Tornado causing more
than twice the number of fatalities over the next closest event type.

It also shows the event type reported in the data wih the greatest
economic impact is the Flood causing about twice the cost impact over the
second event type.

## Data Processing

This project involves exploring the U.S. National Oceanic and
Atmospheric Administration's (NOAA) storm database. This database
tracks characteristics of major storms and weather events in the
United States, including when and where they occur, as well as
estimates of any fatalities, injuries, and property damage.

The data comes in the form of a comma-separated-value file compressed
via the bzip2 algorithm to reduce its size. The data file was
downloaded from this location:

* [Storm Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2).

Documentation of the data can be found at this location:

* [National Weather Service Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)
* [National Climatic Data Center Storm Events FAQ](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2FNCDC%20Storm%20Events-FAQ%20Page.pdf)

### Load Data for Processing

The data is first downloaded and brought into R.


```r
dataFileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
dataFile <- "repdata-data-StormData.csv.bz2"

if (!file.exists(dataFile)) {
        print("Downloading Data Zip File")
        if (download.file(dataFileURL, dataFile)) {
                stop(paste("Could not download data file <",
                           dataFileURL, "> to zip file <",
                           dataFile, ">", sep = ""))
        }
        print("Data Files Successfully downloaded.")
}

df <- read.csv(dataFile, stringsAsFactors = FALSE)
```

### Eliminate Unneeded Variables from Data Structure
Once the data has been loaded, the following is a list of variables
that will be used in the analysis:

* BGN_DATE - Date Event Started
* EVTYPE - Event Type
* FATALITIES - Numerical Count of Event Fatalities
* PROPDMG - Numerical Estimate of Property Damage Costs
* PROPDMGEXP - Exponent of 10 to multiply against PROPDMG to get total cost
* CROPDMG - Numerical Estimate of Crop Damage Costs
* CROPDMGEXP - Exponent of 10 to multiply against CROPDMG to get total cost

To reduce memory usage and speed processing, remove unneeded variables and
ensure data types are correct for the variables:


```r
df <- df[, c("BGN_DATE", "EVTYPE", "FATALITIES", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")]

df$BGN_DATE <- as.Date(df$BGN_DATE, "%m/%d/%Y %H:%M:%S")
df$YEAR <- year(df$BGN_DATE)
df$EVTYPE <- as.factor(df$EVTYPE)
```

The final data conversion that needs to occur is to consolidate costs.
First, convert the exponent
variables to numbers and then create a total costs column and remove the
costs and exponent columns. This TOTALCOST variable is the property damage
cost plus the crop damage cost in billions of dollars.


```r
cols <- c("PROPDMGEXP", "CROPDMGEXP")
df[,cols] <- as.numeric(apply(df[,cols], 2,
                  function (x) {
                          x[x %in% c("", "-", "?", "+")] <- 0
                          x[x %in% c("b", "B")] <- 9
                          x[x %in% c("h", "H")] <- 2
                          x[x %in% c("k", "K")] <- 3
                          x[x %in% c("m", "M")] <- 6
                          x
                  } ) )
df$TOTALCOST <- with(df, (PROPDMG*10^PROPDMGEXP + CROPDMG*10^CROPDMGEXP)/10^9)
df <- df[, c("YEAR", "EVTYPE", "FATALITIES", "TOTALCOST")]
```

## Results

We have 2 questions to answer with the meteorological data. The first is regarding
population health and the second around cost.

There are many ways to answer these questions, I chose to count the number of impacts
(fatalities or cost)
of each event and find the events with the largest impact.

### Impact of Meteorological Events on Population Health

To find the events causing the greatest impact on
population health (fatalities), must first
add up the number of fatalities by event in the data and sort from largest to
smallest. Display the 10 events with the most fatalities.


```r
# calculate the number of fatalities by Type and Year
fatalSumByEvtype <- aggregate(FATALITIES ~ EVTYPE, sum, data = df)

# extract the events with the largest number of fatalities.
top10 <- head(fatalSumByEvtype[order(-fatalSumByEvtype$FATALITIES),], n=10)

kable(top10, format="markdown", row.names = FALSE)
```



|EVTYPE         | FATALITIES|
|:--------------|----------:|
|TORNADO        |       5633|
|EXCESSIVE HEAT |       1903|
|FLASH FLOOD    |        978|
|HEAT           |        937|
|LIGHTNING      |        816|
|TSTM WIND      |        504|
|FLOOD          |        470|
|RIP CURRENT    |        368|
|HIGH WIND      |        248|
|AVALANCHE      |        224|

The event type reporting
the most fatalities in the data provided is the TORNADO event.

To see when these events occurred, plot them against time.


```r
fatalSumByEvtypeYear <- aggregate(FATALITIES ~ EVTYPE + YEAR, sum, data = df)

plot(x = range(fatalSumByEvtypeYear$YEAR), y = range(fatalSumByEvtypeYear$FATALITIES),
     type = "n",
     xlab = "YEAR",
     ylab = "Number of Fatalities"
     )

colors <- rainbow(10)

for (i in 1:10) { 
  lines(fatalSumByEvtypeYear$YEAR[fatalSumByEvtypeYear$EVTYPE == top10$EVTYPE[i]],
        fatalSumByEvtypeYear$FATALITIES[fatalSumByEvtypeYear$EVTYPE == top10$EVTYPE[i]],
         type="b", lwd=1.5,
         col=colors[i], pch=18) 
} 

# add a title and subtitle 
title("Fatalities per Year\nfor Top 10 Meteorological Events")

# add a legend 
legend("topleft", legend = top10$EVTYPE, cex=0.8, col=colors,
  	pch=18, title="Events")
```

![](storm_files/figure-html/Fatalities Plot-1.png)<!-- -->

### Impact of Meteorological Events on Economic Health

To find the events causing the greatest impact on
economic health (cost), must first
add up the total cost by event in the data and sort from largest to
smallest. Display the 10 events causing the most economic damage.


```r
# calculate the costs by Type and Year
costSumByEvtype <- aggregate(TOTALCOST ~ EVTYPE, sum, data = df)

# extract the events with the largest costs
top10 <- head(costSumByEvtype[order(-costSumByEvtype$TOTALCOST),], n=10)

kable(top10, format="markdown", row.names = FALSE)
```



|EVTYPE            |  TOTALCOST|
|:-----------------|----------:|
|FLOOD             | 150.319678|
|HURRICANE/TYPHOON |  71.913713|
|TORNADO           |  57.362334|
|STORM SURGE       |  43.323541|
|HAIL              |  18.761222|
|FLASH FLOOD       |  18.243991|
|DROUGHT           |  15.018672|
|HURRICANE         |  14.610229|
|RIVER FLOOD       |  10.148404|
|ICE STORM         |   8.967041|

The event type reporting
the most economic impact in the data provided is the FLOOD event.

To see when these events occurred, plot them against time.


```r
costSumByEvtypeYear <- aggregate(TOTALCOST ~ EVTYPE + YEAR, sum, data = df)

plot(x = range(costSumByEvtypeYear$YEAR), y = range(costSumByEvtypeYear$TOTALCOST),
     type = "n",
     xlab = "YEAR",
     ylab = "Cost (in Billions of Dollars)"
     )

colors <- rainbow(10)

for (i in 1:10) { 
  lines(costSumByEvtypeYear$YEAR[costSumByEvtypeYear$EVTYPE == top10$EVTYPE[i]],
        (costSumByEvtypeYear$TOTALCOST[costSumByEvtypeYear$EVTYPE == top10$EVTYPE[i]]),
         type="b", lwd=1.5,
         col=colors[i], pch=18) 
} 

# add a title and subtitle 
title("Economic Loss per Year\nfor Top 10 Meteorological Events")

# add a legend 
legend("topleft", legend = top10$EVTYPE, cex=0.8, col=colors,
  	pch=18, title="Events")
```

![](storm_files/figure-html/Cost Plot-1.png)<!-- -->
