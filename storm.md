# Analysis of U.S. Severe Weather Events to Understand the Most Impactful on Human and Economic Health
wdsteck  
`r format(Sys.Date(), "%B %d, %Y")`  



## Synopsis

Storms and other severe weather events can cause both public health
and economic impacts for communities and municipalities impacted
by those events. Many severe events can result in fatalities,
injuries, and property damage, and preventing such outcomes to the
extent possible is a key concern. Understanding the impacts and
then educating the public on those impacts
and methods of remediating the impact is the goal.

### Questions to be answered

* Across the United States, which types of events are most harmful with respect to population health?
* Across the United States, which types of events have the greatest economic consequences?

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
rm(list=ls())
setwd("/Users/bill/Desktop/class/reproducable research/proj2")
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

stormData <- read.csv(dataFile, stringsAsFactors = FALSE)
```

### Eliminate Unneeded Variables from Data Structure
Once the data has been loaded, the following is a list of variables
that will be used in the analysis:

* BGN_DATE - Date Event Started
* STATE - 2 Letter Abbreviation of State Event Occurred In
* EVTYPE - Event Type
* FATALITIES - Numerical Count of Event Fatalities
* INJURIES - Numerical Count of Event Injuries
* PROPDMG - Numerical Estimate of Property Damage Costs
* PROPDMGEXP - Exponent of 10 to multiply against PROPDMG to get total cost
* CROPDMG - Numerical Estimate of Crop Damage Costs
* CROPDMGEXP - Exponent of 10 to multiply against CROPDMG to get total cost

To reduce memory usage and speed processing, remove unneeded variables and
ensure data types are correct for the variables:


```r
df <- stormData[, c("BGN_DATE", "STATE", "EVTYPE", "FATALITIES", "INJURIES", "PROPDMG", "PROPDMGEXP", "CROPDMG", "CROPDMGEXP")]

df$BGN_DATE <- as.Date(df$BGN_DATE, "%m/%d/%Y %H:%M:%S")
df$STATE <- as.factor(df$STATE)
df$EVTYPE <- as.factor(df$EVTYPE)
```

The final data conversion that needs to occur is to perpare the exponent
variables as numbers. First convert the symbols in the data to the exponent
value they represent (i.e. 'H' refers to Hundreds and converts to 2).


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
```

The data is now properly formatted for the analysis.

Synopsis: Immediately after the title, there should be a synopsis
which describes and summarizes your analysis in at most 10 complete
sentences.
There should be a section titled Data Processing which describes (in words and code) how the data were loaded into R and processed for analysis. In particular, your analysis must start from the raw CSV file containing the data. You cannot do any preprocessing outside the document. If preprocessing is time-consuming you may consider using the 𝚌𝚊𝚌𝚑𝚎 = 𝚃𝚁𝚄𝙴 option for certain code chunks.
There should be a section titled Results in which your results are presented.
You may have other sections in your analysis, but Data Processing and Results are required.
The analysis document must have at least one figure containing a plot.
Your analysis must have no more than three figures. Figures may have multiple plots in them (i.e. panel plots), but there cannot be more than three figures total.
You must show all your code for the work in your analysis document. This may make the document a bit verbose, but that is okay. In general, you should ensure that 𝚎𝚌𝚑𝚘 = 𝚃𝚁𝚄𝙴 for every code chunk (this is the default setting in knitr).
Publishing Your Analysisless 
For this assignment you will need to publish your analysis on RPubs.com. If you do not already have an account, then you will have to create a new account. After you have completed writing your analysis in RStudio, you can publish it to RPubs by doing the following:

In RStudio, make sure your R Markdown document (.𝚁𝚖𝚍) document is loaded in the editor
Click the 𝙺𝚗𝚒𝚝 𝙷𝚃𝙼𝙻 button in the doc toolbar to preview your document.
In the preview window, click the 𝙿𝚞𝚋𝚕𝚒𝚜𝚑 button.
Once your document is published to RPubs, you should get a unique URL to that document. Make a note of this URL as you will need it to submit your assignment.

NOTE: If you are having trouble connecting with RPubs due to proxy-related or other issues, you can upload your final analysis document file as a PDF to Coursera instead.
Submitting Your Assignmentless 
In order to submit this assignment, you must copy the RPubs URL for your completed data analysis document in to the peer assessment question.

If you choose to submit as a PDF, please insert an obvious placeholder URL (e.g. https://google.com) in order to allow submission.
