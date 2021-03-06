# Analysis of Severe Weather Events and its Impact on Population Health and the Economy

## **SYNOPSIS**
1. This report analyzes the the impact of severe weather events across United States on population health and the economy. 

2. The information provided in this report is aimed at supporting the government in prioritizing resources for the different type of events across the United States. 

3. The data from this report are retrieved from the U.S. National Oceanic and Atmospheric Administration's storm database which also contains information such as the location and beginning and ending times of events. 

4. However, given the focus on population health and economic consequences, and the inconsistencies that exists in the earlier data, the report will primarily analyze the events resulting in injuries, fatalities, damage to properties and damage to crops from the year 2001 to 2011. 

5. The top fatalities and injuries are both caused by: **tornados**

6. **Tornados are most harmful to population health in terms of combined fatalities and injuries**

7. The top property damages are caused by: **floods**

8. The top crop damages are caused by: **droughts**

9. **Floods causes the most economic damage in terms of combined property and crop damages.**

## **DATA PROCESSING**
This section describes how the data were loaded into R and processed for analysis.

#### **Reading the Data**
Read the dataset into the variable df. 

```r
mypath <- "repdata_data_StormData.csv.bz2"
df <- read.csv(mypath)
```

The column names of the dataset are as such.

```r
colnames(df)
```

```
##  [1] "STATE__"    "BGN_DATE"   "BGN_TIME"   "TIME_ZONE"  "COUNTY"    
##  [6] "COUNTYNAME" "STATE"      "EVTYPE"     "BGN_RANGE"  "BGN_AZI"   
## [11] "BGN_LOCATI" "END_DATE"   "END_TIME"   "COUNTY_END" "COUNTYENDN"
## [16] "END_RANGE"  "END_AZI"    "END_LOCATI" "LENGTH"     "WIDTH"     
## [21] "F"          "MAG"        "FATALITIES" "INJURIES"   "PROPDMG"   
## [26] "PROPDMGEXP" "CROPDMG"    "CROPDMGEXP" "WFO"        "STATEOFFIC"
## [31] "ZONENAMES"  "LATITUDE"   "LONGITUDE"  "LATITUDE_E" "LONGITUDE_"
## [36] "REMARKS"    "REFNUM"
```

#### **Subsetting the Data**
Subset the data to contain columns relevant to the type of events, population health impact and economic consequences.  

The variable *REFNUM* is included to provide a reference back to the original dataset if the need arises.  
The variable *BGN_DATE* is included to indicate when did the event happen. 

```r
processingData <- df[c(37,2,8,23:28)]
colnames(processingData) <- tolower(colnames(processingData))
```

The data is further subsetted to contain only events where there are population health impacts and economic consequences. 

```r
processingData <- processingData[processingData$fatalities!=0 | 
                             processingData$injuries!=0 |
                             processingData$propdmg!=0 |
                             processingData$cropdmg!=0,]
```

As indicated by the documentation of the Storms Events Database, the drop-down selector was only added in the year 2000 to standardize the Event Type values. Prior to the year 2000, there were significant number of variations of event types. Hence, to minimize the impact of the inconsistencies in the analysis, only data from the year 2001 to year 2011 was used. 


```r
processingData$bgn_date <- as.character(processingData$bgn_date)
processingData$bgn_date <- as.Date(processingData$bgn_date,"%m/%d/%Y %H:%M:%S")
processingData <- processingData[processingData$bgn_date >= as.Date("2001-01-01"),]
```

Refer to Storms Events Database - [Collection Sources](https://www.ncdc.noaa.gov/stormevents/details.jsp?type=collection) for details. 

#### **Cleaning the Data**
Convert all event type, and damages to lower case. 

```r
processingData$evtype <- tolower(processingData$evtype)
processingData$propdmgexp <- tolower(processingData$propdmgexp)
processingData$cropdmgexp<- tolower(processingData$cropdmgexp)
```

The list of available events in provided by NOAA is listed below.

A to H                  | I to W
----------------------- | -----------------
Astronomical Low Tide   | Ice Storm
Avalanche               | Lake Effect Snow
Blizzard                | Lakeshore Flood
Coastal Flood           | Lightning
Cold or Wind Chill      | Marine Hail
Debris Flow             | Marine High Wind
Dense Fog               | Marine Strong Wind
Dense Smoke             | Marine Thunderstorm Wind
Drought                 | Rip Current
Dust Devil              | Seiche
Dust Storm              | Sleet
Excessive Heat          | Storm Surge or Tide
Extreme Cold or Wind Chill | Strong Wind
Flash Flood             | Thunderstorm Wind
Flood                   | Tornado
Frost or Freeze         | Tropical Depression
Funnel Cloud            | Tropical Storm
Freezing Fog            | Tsunami
Hail                    | Volcanic Ash
Heat                    | Waterspout
Heavy Rain              | Wildfire
Heavy Snow              | Winter Storm
High Surf               | Winter Weather
High Wind               | -
Hurricane (Typhoon)     | -

The following creates a function to process the event type using regular expressions. The objective is to classify event types that may have significant impact with regard to population health and the economy. 

The regular expressions should be refined if unclassified weather events are listed in the top 10 weather events for impact to population health and economic damages. 

```r
getCleanEvent <- function(category){
    cleanStr <- tolower(c("Astronomical Low Tide","Avalanche",
                  "Blizzard","Coastal Flood",
                  "Cold or Wind Chill","Debris Flow",
                  "Dense Fog","Dense Smoke",
                  "Drought","Dust Devil",
                  "Dust Storm","Excessive Heat",
                  "Extreme Cold or Wind Chill","Flash Flood",
                  "Flood","Frost or Freeze",
                  "Funnel Cloud","Freezing Fog",
                  "Hail","Heat",
                  "Heavy Rain","Heavy Snow",
                  "High Surf","High Wind",
                  "Hurricane (Typhoon)","Ice Storm",
                  "Lake Effect Snow","Lakeshore Flood",
                  "Lightning","Marine Hail",
                  "Marine High Wind","Marine Strong Wind",
                  "Marine Thunderstorm Wind","Rip Current",
                  "Seiche","Sleet",
                  "Storm Surge or Tide","Strong Wind",
                  "Thunderstorm Wind","Tornado",
                  "Tropical Depression","Tropical Storm",
                  "Tsunami","Volcanic Ash",
                  "Waterspout","Wildfire",
                  "Winter Storm","Winter Weather"))
    
    regexStr <- c("(astro).*(low).*(tide)","(avalanc)h?e",
                  "(blizzard)","(coast).*((flood)|(storm)|(surge))",
                  "(^(cold)|((wind).*(chill)))","(debris).*(flow).*",
                  "(dense).*(fog).*","(dense).*(flow).*",
                  "(drought).*","(dust).*(devil).*",
                  "(dust).*(storm)","(excessive).*(heat).*",
                  "(extreme).*(cold).*","(flash).*(flood).*",
                  "(flood).*","((frost)|((freez)(e|i))).*",
                  "(funnel).*(cloud).*","(freezing).*(fog).*",
                  "(hail).*","(heat).*",
                  "(heavy).*(rain).*","(heavy).*(snow).*",
                  "((high)|(heavy)).*((surf)|(wave)|(tide)).*","(high).*(wind).*",
                  "((hurricane)|(typhoon))","(ic)(e|y)+.*(storm)?.*",
                  "(lake).*(snow).*","(lake).*(flood).*",
                  "(lightning).*","(marine).*(hail).*",
                  "(marine).*(high).*(wind).*","(marine).*(strong).*(wind).*",
                  "(marine).*(thunderstorm).*","(rip).*(current).*",
                  "(seiche).*","(sleet).*",
                  "(storm).*(surge).*","(strong).*(wind).*",
                  "((th?un?)+d?(e+r)?s?(to?r?m)|(tstm)|(thunderstrom)).*(wind)?",
                  "(torn((ad)|(da))o).*",
                  "(tropical).*(depression).*","(tropical).*(storm).*",
                  "(tsunami).*","(volcanic).*(ash).*",
                  "(water).*(spout).*","(wild).*(fire).*",
                  "(winter).*(storm).*","(winter).*(weather).*")
    
    mapping <- cbind(cleanStr,regexStr)
    
    for(x in 1:nrow(mapping)){
            match <- "unclassified"
            if(grepl(mapping[x,2],category, ignore.case=T)){
                    match <- mapping[x,1]
                    return(match)
                    break
            }        
    }
    match
}
```

The following creates a function to derive the economic damage in dollars. The following approach was taken.

* cropdmg and propdmg values are assumed to be in thousands unless otherwise stated by cropdmgexp and propdmgexp respectively. 
* Numeric values (with the exception of 0) in cropdmgexp and propdmgexp are assumed to be powers of data. 
* Letter values in cropdmgexp and propdmgexp are treated as described in the [Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)


```r
getCleanExp<- function (x){
        result = 0
        switch (x,
        "-"={result <- 1000},
        "?"={result <- 1000},
        "+"={result <- 1000},
        "0"={result <- 1000},
        "1"={result <- 10^1},
        "2"={result <- 10^2},
        "3"={result <- 10^3},
        "4"={result <- 10^4},
        "5"={result <- 10^5},
        "6"={result <- 10^6},
        "7"={result <- 10^7},
        "8"={result <- 10^8},
        "h"={result <- 10^2},
        "k"={result <- 10^3},
        "m"={result <- 10^6},
        "b"={result <- 10^9}
    )
    result
}
```

Create the columns cleanevtype, cleanpropdmg, cleancropdmg to store the values cleaned by the above 2 functions. 


```r
# Creates the event type mapping table based on the regex 
evtypeMapping <- as.data.frame(sort(unique(processingData["evtype"][,1])))
colnames(evtypeMapping) <- "evtype"
evtypeMapping$cleanevtype <- apply(evtypeMapping,1,getCleanEvent)

# Merge the cleaned event type to the analysis dataset
processingData <- merge(processingData, evtypeMapping, by="evtype")

# Clean the propdmgexp column
processingData$propmultiplier <- apply(as.data.frame(processingData$propdmgexp),1,getCleanExp)
processingData$cropmultiplier <- apply(as.data.frame(processingData$cropdmgexp),1,getCleanExp)

# Create the cleanpropdmg and cleancropdmg columns to store the actual damages
processingData$propdmgactual <- processingData$propdmg*processingData$propmultiplier
processingData$cropdmgactual <- processingData$cropdmg*processingData$cropmultiplier
```

Subset processingData to the columns needed for the analysis and remove all intermediate columns used to process the data. Compute the total fatalities and injuries as well as the total economic damages. 


```r
analysisData <- processingData[,c(2,3,4,5,10,13,14)]
analysisData$cleanevtype <- as.factor(analysisData$cleanevtype)
analysisData$totalhealthimpact <- analysisData$fatalities + analysisData$injuries
analysisData$totaleconomicdmg <- analysisData$propdmgactual + analysisData$cropdmgactual
```

#### **Aggregating the Population Health Impact Data for Analysis**
The following aggregates the fatalities data and derive the top 10 event types that causes fatalities.

```r
fatalitiesData <- as.data.frame(tapply(analysisData$fatalities, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(fatalitiesData) <- "fatalities"
fatalitiesData$evtype <- rownames(fatalitiesData)
rownames(fatalitiesData) <- NULL
top10fatalities <- head(fatalitiesData[order(-fatalitiesData$fatalities),],10)
```

The following aggregates the injuries data and derive the top 10 event types that causes injuries.

```r
injuriesData <- as.data.frame(tapply(analysisData$injuries, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(injuriesData) <- "injuries"
injuriesData$evtype <- rownames(injuriesData)
rownames(injuriesData) <- NULL
top10injuries <- head(injuriesData[order(-injuriesData$injuries),],10)
```

The following aggregates both injuries and fatalities data and derive the top 10 event types that has an impact to population health. 

```r
library(tidyr)
```

```
## Warning: package 'tidyr' was built under R version 3.1.3
```

```r
healthImpactData <- as.data.frame(tapply(analysisData$totalhealthimpact, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(healthImpactData) <- "totalhealthimpact"
healthImpactData$evtype <- rownames(healthImpactData)
rownames(healthImpactData) <- NULL
healthImpactData <- merge(healthImpactData,fatalitiesData,by="evtype")
healthImpactData <- merge(healthImpactData,injuriesData,by="evtype")

top10healthimpact <- head(healthImpactData[order(-healthImpactData$totalhealthimpact),],10)

#Remove the dimensions so that gather can be used.
dim(top10healthimpact$fatalities) <-NULL
dim(top10healthimpact$injuries) <-NULL
dim(top10healthimpact$totalhealthimpact) <-NULL

top10healthimpactGroup <- gather(top10healthimpact[,c(1,3,4)], healthimpacttype, healthimpact, 2:3)
```

#### **Aggregating the Economic Consequences Data for Analysis**
The following aggregates the property damage data and derive the top 10 event types that causes property damages.

```r
propdmgData <- as.data.frame(tapply(analysisData$propdmgactual, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(propdmgData) <- "propdmg"
propdmgData$evtype <- rownames(propdmgData)
rownames(propdmgData) <- NULL
top10propdmg <- head(propdmgData[order(-propdmgData$propdmg),],10)

#Represent the damage in millions
top10propdmg$propdmg <- top10propdmg$propdmg / (10^6)
```

The following aggregates the crop damage data and derive the top 10 event types that causes crop damages.

```r
cropdmgData <- as.data.frame(tapply(analysisData$cropdmgactual, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(cropdmgData) <- "cropdmg"
cropdmgData$evtype <- rownames(cropdmgData)
rownames(cropdmgData) <- NULL
top10cropdmg <- head(cropdmgData[order(-cropdmgData$cropdmg),],10)

#Represent the damage in millions
top10cropdmg$cropdmg <- top10cropdmg$cropdmg / (10^6)
```

The following aggregates both property and crop damage data and derive the top 10 event types that resulted in economic damages.

```r
library(tidyr)

economicdmgData <- as.data.frame(tapply(analysisData$totaleconomicdmg, 
                                       INDEX=analysisData$cleanevtype, FUN=sum))
colnames(economicdmgData) <- "totaleconomicdmg"
economicdmgData$evtype <- rownames(economicdmgData)
rownames(economicdmgData) <- NULL
economicdmgData <- merge(economicdmgData,propdmgData,by="evtype")
economicdmgData <- merge(economicdmgData,cropdmgData,by="evtype")

#Derive the top 10 events with population health impact.
top10economicdmg <- head(economicdmgData[order(-economicdmgData$totaleconomicdmg),],10)

#Remove the dimensions so that gather can be used.
dim(top10economicdmg$propdmg) <-NULL
dim(top10economicdmg$cropdmg) <-NULL
dim(top10economicdmg$totaleconomicdmg) <-NULL

top10economicdmgGroup <- gather(top10economicdmg[,c(1,3,4)], economicdmgtype, economicdmg, 2:3)

#Represent the damage in millions
top10economicdmgGroup$economicdmg <- top10economicdmgGroup$economicdmg / (10^9)
```

## **RESULTS**
This section presents the results of the analysis performed on Severe Weather Events and its impact on population health and the economy.

#### **Population Health Impact Analysis Results**

The following plots the population health impact of the different event types broken down by injuries and fatalities.


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.1.3
```

```r
library(grid)
library(gridExtra)
```

```
## Warning: package 'gridExtra' was built under R version 3.1.3
```

```r
fatalitiesPlot <- 
        ggplot(data=top10fatalities, aes(x=evtype, y=fatalities)) +
        geom_bar(stat="identity",fill="#CC6666") +
        coord_flip() +
        xlab("Event Type") +
        ylab("Total Fatalities") +
        theme(legend.position = "none", 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10fatalities$evtype)) +
        ylim(0,15000) + 
        ggtitle("Total Fatalities Caused by\nDifferent Weather Events\nfrom 2001 to 2011\n")

injuriesPlot <- 
        ggplot(data=top10injuries, aes(x=evtype, y=injuries)) +
        geom_bar(stat="identity",fill="#56B4E9") +
        coord_flip() +
        xlab("Event Type") +
        ylab("Total Injuries") +
        theme(legend.position = "none", 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10injuries$evtype)) +
        ylim(0,15000) + 
        ggtitle("Total Injuries Caused by\nDifferent Weather Events\nfrom 2001 to 2011\n")

overallhealthPlot <- 
        ggplot(data=top10healthimpactGroup, 
               aes(x=evtype, y=healthimpact, group=healthimpacttype, fill=healthimpacttype)) + 
        geom_bar(stat="identity") +
        coord_flip() + 
        xlab("Event Type") +
        ylab("Population Health Impact") +
        theme(legend.position = c(.7,.1), 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10healthimpact$evtype)) +
        scale_fill_manual("Pop. Health Impact Type",
                          values=c("#CC6666","#56B4E9"),
                          labels=c("Fatalities","Injuries")) +
        guides(colour=guide_legend(override.aes=list(size=4))) +
        ggtitle("Overall Population Health Impact\nCaused by Different Weather Events\nfrom 2001 to 2011\nBroken Down by Health Impact Type")

populationHealthPlot <- grid.arrange(overallhealthPlot, arrangeGrob(fatalitiesPlot,injuriesPlot, ncol=1), ncol=2)
```

![](SWEAnalysis_files/figure-html/unnamed-chunk-17-1.png) 

**Figure 1. Top 10 Population Health Impact of Severe Weather Events Broken Down by Number of Fatalities and Injuries from 2001 - 2011.**

#### **Economic Consequences Analysis Results**
The following plots the economic damages resulting from the different event types broken down by injuries and fatalities.


```r
library(ggplot2)
library(grid)
library(gridExtra)

propdmgPlot <- 
        ggplot(data=top10propdmg, aes(x=evtype, y=propdmg, fill="#56B4E9")) +
        geom_bar(stat="identity",fill="#CC6666") +
        coord_flip() +
        xlab("Event Type") +
        ylab("Total Property Damage ($ Millions)") +
        theme(legend.position = "none", 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10propdmg$evtype)) +
        ylim(0,150000) + 
        ggtitle("Total Property Damage Caused by\nDifferent Weather Events\nfrom 2001 to 2011\n")

cropdmgPlot <- 
        ggplot(data=top10cropdmg, aes(x=evtype, y=cropdmg, fill="#CC6666")) +
        geom_bar(stat="identity",fill="#56B4E9") +
        coord_flip() +
        xlab("Event Type") +
        ylab("Total Crop Damage ($ Millions)") +
        theme(legend.position = "none", 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10cropdmg$evtype)) +
        ylim(0,150000) + 
        ggtitle("Total Crop Damage Caused by\nDifferent Weather Events\nfrom 2001 to 2011\n")

overalldmgPlot <- 
        ggplot(data=top10economicdmgGroup, 
               aes(x=evtype, y=economicdmg, group=economicdmgtype, fill=economicdmgtype)) + 
        geom_bar(stat="identity") +
        coord_flip() + 
        xlab("Event Type") +
        ylab("Economic Damage ($ Billions)") +
        theme(legend.position = c(.7,.1), 
              plot.title=element_text(face="bold",size=11),
              axis.title=element_text(face="bold",size=11)) +
        scale_x_discrete(limits=rev(top10economicdmg$evtype)) +
        scale_fill_manual("Economic Damage Type",
                          values=c("#CC6666","#56B4E9"),
                          labels=c("Property Damage","Crop Damage")) +
        guides(colour=guide_legend(override.aes=list(size=10))) +
        ggtitle("Overall Economic Consequences\nCaused by Different Weather Events\nfrom 2001 to 2011\nBroken Down by Damage Type")

economicdmgPlot <- grid.arrange(overalldmgPlot, 
                                     arrangeGrob(propdmgPlot,cropdmgPlot, ncol=1), ncol=2)
```

![](SWEAnalysis_files/figure-html/unnamed-chunk-18-1.png) 

**Figure 2. Top 10 Economic Damage of Severe Weather Events Broken Down by Property Damages and Crop Damages from 2001 - 2011. **


