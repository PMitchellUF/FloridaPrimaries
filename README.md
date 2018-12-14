---
title: "Florida Extreme?"
date: "`r Sys.Date()`"
output:
  rmdformats::material:
    highlight: kate
    code_folding: hide
    self_contained: no
---


```{r knitr_init, echo=FALSE, cache=FALSE}
library(knitr)
library(rmdformats)

## Global options
options(max.print="100")
opts_chunk$set(echo=TRUE,
	             prompt=FALSE,
               tidy=TRUE,
               comment=NA,
               message=FALSE,
               warning=FALSE)
opts_knit$set(width=100)
```

# Florida's Extreme Electoral Choices

A great debate exists in the study of American politics regarding partisanship, the ideological extremity of voters, and how these things effect political attitudes and behaviors (i.e. vote choice & turnout). Using Florida as an example, exploring how (and where) certain candidates win nomination to the General Election is important to many apects of politics, including campaigns.

## Florida

In Florida, where almost one-third of voters are registered with ["No Party Affiliation"](https://dos.myflorida.com/elections/data-statistics/voter-registration-statistics/voter-registration-monthly-reports/voter-registration-by-party-affiliation/) (NPA), and the Primary Election system is closed (meaning only voters registered with the two major parties, **Republican Party of Florida** & **Florida Democratic Party**, are entitled to vote for the candidates running for the parties' respective nominations), the choices made by Primary Election voters often become the *only* options for *all* of the state's General Election voters.  

Additionally, with a population that more closely reflects the Nation as a whole than any other State, responsible for casting the third-most electoral votes in Presidential elections, and widely-regarded as the largest swing-state, Florida's elections impact (and draw the interest of) the entire Country.  

Ultimately, Florida's elections are important, and it behoves us to understand them!  

#### 2010, 2016, 2018

2010, 2016 and 2018 provide some of the most recent statewide, high profile Primary Elections in the State of Florida, with 2010 & 2018 including open (non-incumbant) Gubernatorial Primary elections in August, and 2016 an open Presidential Preference Primary in March.  

I make no pretense of suggesting that these races are perfectly similar, or that a Presidential Preference Primary ("PPP") is equatable to a Midterm Gubanatorial Primary (as they fall in Florida).  Obviously, there are several components of these election that are different: from the candidates themselves, to perhaps even more influential, environmental factors (which could mobilize or suppress turnout). 

However, few could deny that the turnout observed in 2018's midterm election cycle far exceeded its typical showing, more closely mimicing the 2016 PPP, making the comparasion at least worthy of visualization. 

#### 67 Counties

Florida has 67 individual Counties, with *North Florida* home to a more rural/blue-collar & Republican population, *South Florida* containing a large urban/metropolitan & Democratic demographic, and *Central Florida*'s "I-4 Corridor," a political hot-bed of urban, rural & suburban households closely divided in party identicfication. 

This project will break down Florida's most recent non-incumbant and statewide elections by County, to observe any developing trends in voter choice.

## Why I Care

For my purpose, as a student of political campaigns and voting behavior, regular analyses such as this educate campaigns, including candidate messaging, voter targeting, and GOTV efforts. 

Understanding trends, and making conjectures as to whom and when to target (including geographic & demographic blocks) is an incredibly important strategy that can make a difference, whether it be tens-of-thousands or [just a few thousand votes](https://floridaelectionwatch.gov/StateOffices/Governor) (the margin of victory in Florida's last few statewide elections). 

# R Packages & Data

This section will include information on the RStudio packages that are used in this analysis, as well as the methods used to clean the data for Mapping.

## Packages for Mapping

This project will rely heavily on viewing geographic trends, and as such on geospatial data. In this effort, I have employed the [**'tmap'**](https://github.com/mtennekes/tmap) suite of "Thematic Maps"," in which spatial data distributions are visualized. 

To get these ready for use in R Studio, install the **'tmap'** packages and load the libraries into the working session: 
```{r eval=-(1:2)}
#install.packages("tmap")
#install.packages("tmaptools")

library("tmap")
library("tmaptools")
```
Additionally, I will use specific RStudio packages to format the data in a way that makes it compatible with **'tmap'**, including **'sf'**, **'magick'** (for animations), **'leaflet'** (for interactive maps), and **'viridis'**, which is an alternative color palette including options not included in base R. 
```{r eval=-(1:4), message = FALSE}
#install.packages("sf")
#install.packages("leaflet")
#install.packages("magick")
#install.packages("viridis")

library("sf")
library("leaflet")
library("magick")
library("viridis")
```
## Datasets

First, I will map the party registration density, per County. This information can be obtained from the Florida Division of Elections [Bookclosing Reports](https://dos.myflorida.com/elections/data-statistics/voter-registration-statistics/bookclosing/).

For this project, I used the bookclosing reports for the month prior to each year's respective primary election (July of 2010 and 2018, and February of 2016).

Next, to evaluate the candidate performance in each of these counties, I used the data provided by the Florida Division of Elections [Results Archive](https://dos.myflorida.com/elections/data-statistics/elections-data/election-results-archive/). This site allows a user to obtain a breakdown of results by a number of factors, including county, party, race, etc. Simply chose the election cycle.  

For each type of data, I simply downloaded the results to a file on my hardrive, and established that location as the R working directory. 

#### Cleaning & Tidying Data

As with most secondary-datasets retrieved online, the files will reqire some cleaning and tidying, for effective translation into R. In this venture, I emplo the **'tidyr'** **'deplyr'** and **'reshape2'** packages throughout. 
```{r eval=-(1:3), message = FALSE}
#install.packages("dplyr")
#install.packages("tidyr")
#install.packages("reshape2")

library("dplyr")
library("tidyr")
library("reshape2")
```
#### Importing Data

Finally, in order to import the Florida Division of Elections data files into the current R session, I use the [**'rio'**](https://github.com/leeper/rio) package, which I find is easiest for data from multiple files/sources, as it can import and list data files by their respective file extensions. 
```{r eval=-1, message = FALSE}
#install.packages("rio")

library("rio")
```
Once the Working Directory is set to the path which includes the saved data files, they are ready to be uploaded.

As every dataset is different (unfortunately), cleaning the data requires one to `view()` and reformat/drop dat to conform to the easiest format for using the R visualization tools. This process begins in the code below. 
```{r include = FALSE}
setwd("C:/Users/rumpl/Desktop/Electoral Data Science/Final Project")
```
```{r}
##Upload the Data Sets 

fl2010primaryregdatafile <- 
  "2010VoterRegistrationSnapshots/pri2010_countyparty.xls"
fl2016primaryregdatafile <- 
  "2016VoterRegistrationSnapshots/ppp2016_countyparty.xlsx"
fl2018primaryregdatafile <- 
  "2018VoterRegistrationSnapshots/Primary/2018pri_party.xlsx"

##For all three election cycles, the State files have mutiple 
  ##rows that do not contain data, so we will skip the first 
  ##7 rows in generating our data frame:

flprimary_partyreg_2010 <- 
  rio::import(fl2010primaryregdatafile, skip=7)
flprimary_partyreg_2016 <- 
  rio::import(fl2016primaryregdatafile, skip=7)
flprimary_partyreg_2018 <- 
  rio::import(fl2018primaryregdatafile, skip=8)

##Drop the rows and columns that we do not need: 

flprimary_partyreg_2010 <- 
  flprimary_partyreg_2010 [c(1:28, 30:63, 65:69),c(1,4,7)]
flprimary_partyreg_2016 <- 
  flprimary_partyreg_2016 [,c(4:6)]
flprimary_partyreg_2018 <- 
  flprimary_partyreg_2018 [-68,c(1:3)]

##Rename the columns so we can easily and uniformly reference each:

colnames(flprimary_partyreg_2010) <- 
  c("CountyName", "Republican", "Democratic")
colnames(flprimary_partyreg_2016) <- 
  c("CountyName", "Republican", "Democratic")
colnames(flprimary_partyreg_2018) <- 
  c("CountyName", "Republican", "Democratic")

```
The data cleaning complete, check to see that all three data.frames frames now contain 3 columns (CountyName = character, Republican & Democrat = numeric) and 67 rows of data, correctly labeled:
```{r, results = "hold"}
str(flprimary_partyreg_2010)
str(flprimary_partyreg_2016)
str(flprimary_partyreg_2018)
```
*note that the __Rupublican__ & __Democratic__ columns of the 2010 data.frame are reporting as character strings (instead of numeric, those from 2016 & 2018). The `as.numeric` conversion will fixes this.*
```{r}
flprimary_partyreg_2010$Republican <- 
  as.numeric(flprimary_partyreg_2010$Republican)
flprimary_partyreg_2010$Democratic <- 
  as.numeric(flprimary_partyreg_2010$Democratic)

##Advise: check the 2010 data.frame again, to see that 
  ##it matches the others
```
### Make New Variables to Map

In order to make the choropleth of Florida Counties' *primary eligible* party registration, calculate the percentage of those registered **'Republican'**. 

(As there are only 2 variables (parties), 50% is the natural midpoint of registration density, with those counties below a 50% majority registered **Republican** representing a majority of registered **Democratic** voters. 
```{r}

flprimary_partyreg_2010$RepPct <- 
  (flprimary_partyreg_2010$Republican / 
     (flprimary_partyreg_2010$Republican + flprimary_partyreg_2010$Democratic) *100)

flprimary_partyreg_2016$RepPct <- 
  (flprimary_partyreg_2016$Republican / 
     (flprimary_partyreg_2016$Republican + flprimary_partyreg_2016$Democratic) *100)

flprimary_partyreg_2018$RepPct <- 
  (flprimary_partyreg_2018$Republican / 
     (flprimary_partyreg_2018$Republican + flprimary_partyreg_2018$Democratic) *100)
```
Finally, at this stage (and so we can use the **'tmap'** `facet()` and `animate()` functions later), create an additional data.frame that can be transformed from wide to long data. 
```{r, results = "hide"}
## first add the "year" identifer to the 'RepPct' variable 
  ##(voters registered Republican), to better identify the 
  ##correct year later:

    ##2010
flprimary_partyreg_2010$RepPct2010 <- 
  (flprimary_partyreg_2010$Republican / 
     (flprimary_partyreg_2010$Republican + flprimary_partyreg_2010$Democratic) *100)
    ##2016
flprimary_partyreg_2016$RepPct2016 <- 
  (flprimary_partyreg_2016$Republican / 
     (flprimary_partyreg_2016$Republican + flprimary_partyreg_2016$Democratic) *100)
    ##2018
flprimary_partyreg_2018$RepPct2018 <- 
  (flprimary_partyreg_2018$Republican / 
     (flprimary_partyreg_2018$Republican + flprimary_partyreg_2018$Democratic) *100)

  ##sort

flprimary_partyreg_2010 <- 
  flprimary_partyreg_2010[order(flprimary_partyreg_2010$CountyName),]
flprimary_partyreg_2016 <- 
  flprimary_partyreg_2016[order(flprimary_partyreg_2010$CountyName),]
flprimary_partyreg_2018 <- 
  flprimary_partyreg_2018[order(flprimary_partyreg_2018$CountyName),]

  ##compare: the `identical()` logical argument should return `[1] True` 
    ##if the data match. 

identical(flprimary_partyreg_2010$CountyName,
          flprimary_partyreg_2016$CountyName )
identical(flprimary_partyreg_2010$CountyName,
          flprimary_partyreg_2018$CountyName )

## use the dplyr cbind() function to bind the variables 
  ##for each year from 2010-2018 into a new data.frame

FLCountyReg_Percents <- 
  cbind(flprimary_partyreg_2010, flprimary_partyreg_2016$RepPct2016, 
        flprimary_partyreg_2018$RepPct2018)

## tidy this new data.frame:  drop the columns we do not need, 
  ##and rename the columns to reflect their corresponding value

FLCountyReg_Percents <- FLCountyReg_Percents [,c(1, 5:7)] 
colnames(FLCountyReg_Percents) <- 
  c("CountyName", "2010PctRep", "2016PctRep", "2018PctRep")
```
## Election Results Data

Turning to the Candidate Election Results, and similar to before, the 2010, 2016 & 2018 Election Results files must be loaded from the working directory into respective data.frame of the R session, including performing any necessary cleaning and tidying. 
```{r}
## import
fl2010primaryresultsdatafile <- 
  "2010PrimaryResults/08242010Election.txt"
  flprimaryResults2010 <- 
    rio::import(fl2010primaryresultsdatafile)

fl2016primaryresultsdatafile <- 
  "2016PrimaryResults/2016PrimaryResults.txt"
  flprimaryResults2016 <- 
    rio::import(fl2016primaryresultsdatafile)

fl2018primaryresultsdatafile <- 
  "2018PrimaryResults/Results/08282018Election.txt"
  flprimaryResults2018 <- 
    rio::import(fl2018primaryresultsdatafile)
  
## reduce the size of the dataframe's by dropping 
  ##the columns not needed: 

flprimaryResults2010 <- flprimaryResults2010 [,c(2,4,7,12,15)]
flprimaryResults2016 <- flprimaryResults2016[,c(2,4,7,12,15)]
flprimaryResults2018 <- flprimaryResults2018 [,c(2,4,7,12,15)]

## drop rows that include races other than those at Subject

flprimaryResults2010 <- 
  filter(flprimaryResults2010, RaceCode == 'GOV')
flprimaryResults2018 <- 
  filter(flprimaryResults2018, RaceCode == 'GOV')

## create dataframe's individually, for the 'Republican' 
  ##and 'Democratic' candidates (respective to year)

flprimaryRepub2010 <- 
  filter(flprimaryResults2010, PartyCode == 'REP')
flprimaryDem2010 <- 
  filter(flprimaryResults2010, PartyCode == 'DEM')

flprimaryRepub2016 <- 
  filter(flprimaryResults2016, PartyCode == 'REP')
flprimaryDem2016 <- 
  filter(flprimaryResults2016, PartyCode == 'DEM')

flprimaryRepub2018 <- 
  filter(flprimaryResults2018, PartyCode == 'REP')
flprimaryDem2018 <- 
  filter(flprimaryResults2018, PartyCode == 'DEM')

## finally, reduce the size of the data.frames by 
  ##dropping irrelevant variables 

flprimaryRepub2010 <- flprimaryRepub2010 [,c(3:5)]
flprimaryDem2010 <- flprimaryDem2010 [,c(3:5)]

flprimaryRepub2016 <- flprimaryRepub2016 [,c(3:5)]
flprimaryDem2016 <- flprimaryDem2016 [,c(3:5)]

flprimaryRepub2018 <- flprimaryRepub2018 [,c(3:5)]
flprimaryDem2018 <- flprimaryDem2018 [,c(3:5)]
```
Ensure structure of the new data.frame, to see that they are numeric (for reshaping, applying math functions, and utilizing **'tmap'** tools):
```{r, results = "hold"}
  ##2010
str(flprimaryRepub2010)
str(flprimaryDem2010)
  ##2016
str(flprimaryRepub2016)
str(flprimaryDem2016)
  ##2018
str(flprimaryRepub2018)
str(flprimaryDem2018)
```
*note that the the "CanVotes"" (vote total) variables of all datasets are reporting as integers, instead of `as.numeric`: they will need to be converted.*  
```{r}
flprimaryRepub2010$CanVotes <- 
  as.numeric(flprimaryRepub2010$CanVotes)
flprimaryDem2010$CanVotes <- 
  as.numeric(flprimaryDem2010$CanVotes)

flprimaryRepub2016$CanVotes <- 
  as.numeric(flprimaryRepub2016$CanVotes)
flprimaryDem2016$CanVotes <- 
  as.numeric(flprimaryDem2016$CanVotes)

flprimaryRepub2018$CanVotes <- 
  as.numeric(flprimaryRepub2018$CanVotes)
flprimaryDem2018$CanVotes <- 
  as.numeric(flprimaryDem2018$CanVotes)

## confirm structure is correct after change
```
#### Transform Data for **'tmap'**

In order to compute new variables, as well as execute some **'tmap'** functions, long data (*such as was provided by the Florida Division of Elections*) will need to be transformed into wide data, for which the **'reshape2'** package's `dcast()` makes quick work of the task: 
```{r, results = "hold", message = TRUE}
  ##2010
flprimaryRepub2010_Cand <- 
  dcast(flprimaryRepub2010, CountyName ~ 
          CanNameLast, value.var="CanVotes")
flprimaryDem2010_Cand <- 
  dcast(flprimaryDem2010, CountyName ~ 
          CanNameLast, value.var="CanVotes")
  ##2016
flprimaryRepub2016_Cand <- 
  dcast(flprimaryRepub2016, CountyName ~ 
          CanNameLast, value.var="CanVotes")
flprimaryDem2016_Cand <- 
  dcast(flprimaryDem2016, CountyName ~ 
          CanNameLast, value.var="CanVotes")
  ##2018
flprimaryRepub2018_Cand <- 
  dcast(flprimaryRepub2018, CountyName ~ 
          CanNameLast, value.var="CanVotes")
flprimaryDem2018_Cand <- 
  dcast(flprimaryDem2018, CountyName ~ 
          CanNameLast, value.var="CanVotes")
```
*note: R is retruning a warning re the 2016 data.frame - "Aggregation function missing: defaulting to length."* 

Upon further inspection, it appears the State's results file has multiple rows for each county/candidate (by "District Court" to be exact), and therefore it provided **counts**, instead of **vote totals**. To fix & standardize this `group` and `summarise` the rows for each county/candidate, and again transform the data:
```{r}

flprimaryRepub2016 <- flprimaryRepub2016 %>% 
  group_by(CountyName, CanNameLast) %>% 
  summarise(CandVotes=sum(CanVotes))

flprimaryDem2016 <- flprimaryDem2016 %>% 
  group_by(CountyName, CanNameLast) %>% 
  summarise(CandVotes=sum(CanVotes))

    ###Re-run the transform functions for 2016:
flprimaryRepub2016_Cand <- 
  dcast(flprimaryRepub2016, CountyName ~ 
          CanNameLast, value.var="CandVotes")
flprimaryDem2016_Cand <- 
  dcast(flprimaryDem2016, CountyName ~ 
          CanNameLast, value.var="CandVotes")
```
For the last bit of tidying, drop candidates (*variables/columns*) that are not of interest to this assessment, namely those who achieved less than 2% of the statewide vote.
```{r}

flprimaryRepub2016_Cand <- 
  flprimaryRepub2016_Cand[,c(1,5,10,12,14)]
flprimaryDem2016_Cand <- flprimaryDem2016_Cand[,c(1,2,4)]

flprimaryRepub2018_Cand <- 
  flprimaryRepub2018_Cand [,c(1,3,8)]
flprimaryDem2018_Cand <- flprimaryDem2018_Cand [,c(1:4,6)]
```
#### Make New Variables 

As this analysis looks at whether the electorate is trending toward more "Extreme" candidates, and away from "Establishment" candidates, categorizing each candidate in such *Groups* per their respective election cycles is necessary. From there, combine the *Group* percent of the total vote and MOV (*margin of victory*) so that the vote shares for each candidate type can be plotted. 
```{r}
## Total Republican Votes Cast
flprimaryRepub2010_Cand$TotalRVotesCast <- 
  (flprimaryRepub2010_Cand$Scott + 
     flprimaryRepub2010_Cand$McCalister + 
     flprimaryRepub2010_Cand$McCollum)

flprimaryRepub2010_Cand$EstCandPercent2010 <- 
  ((flprimaryRepub2010_Cand$McCalister + 
      flprimaryRepub2010_Cand$McCollum) / 
     (flprimaryRepub2010_Cand$TotalRVotesCast) *100)
flprimaryRepub2010_Cand$ExtCandPercent2010 <- 
  ((flprimaryRepub2010_Cand$Scott) / 
     (flprimaryRepub2010_Cand$TotalRVotesCast) *100)

flprimaryRepub2010_Cand$EstCandPercent2010_MOV <- 
  ((flprimaryRepub2010_Cand$McCollum + 
      flprimaryRepub2010_Cand$McCalister - 
      flprimaryRepub2010_Cand$Scott) / 
     (flprimaryRepub2010_Cand$TotalRVotesCast) *100)

flprimaryRepub2010_Cand$ExtCandPercent2010_MOV <- 
  ((flprimaryRepub2010_Cand$Scott - 
      flprimaryRepub2010_Cand$McCollum - 
      flprimaryRepub2010_Cand$McCalister) / 
     (flprimaryRepub2010_Cand$TotalRVotesCast) *100)

##2010 
    ###Democrat
flprimaryDem2010_Cand$TotalDVotesCast <- 
  (flprimaryDem2010_Cand$Moore + flprimaryDem2010_Cand$Sink)

flprimaryDem2010_Cand$EstCandPercent2010 <- 
  ((flprimaryDem2010_Cand$Sink) / 
     (flprimaryDem2010_Cand$TotalDVotesCast) *100)
flprimaryDem2010_Cand$ExtCandPercent2010 <- 
  ((flprimaryDem2010_Cand$Moore) / 
     (flprimaryDem2010_Cand$TotalDVotesCast) *100)

flprimaryDem2010_Cand$EstCandPercent2010_MOV <- 
  ((flprimaryDem2010_Cand$Sink - flprimaryDem2010_Cand$Moore) / 
     (flprimaryDem2010_Cand$TotalDVotesCast) *100)
flprimaryDem2010_Cand$ExtCandPercent2010_MOV <- 
  ((flprimaryDem2010_Cand$Moore - flprimaryDem2010_Cand$Sink) / 
     (flprimaryDem2010_Cand$TotalDVotesCast) *100)
  
  ##2016  
    ###Republican
flprimaryRepub2016_Cand$TotalRVotesCast <- 
  (flprimaryRepub2016_Cand$Cruz + flprimaryRepub2016_Cand$Kasich + 
     flprimaryRepub2016_Cand$Rubio + flprimaryRepub2016_Cand$Trump)

flprimaryRepub2016_Cand$EstCandPercent2016 <- 
  ((flprimaryRepub2016_Cand$Rubio + flprimaryRepub2016_Cand$Kasich) / 
     (flprimaryRepub2016_Cand$TotalRVotesCast) *100)
flprimaryRepub2016_Cand$ExtCandPercent2016 <- 
  ((flprimaryRepub2016_Cand$Trump + flprimaryRepub2016_Cand$Cruz) / 
     (flprimaryRepub2016_Cand$TotalRVotesCast) *100)

flprimaryRepub2016_Cand$EstCandPercent2016_MOV <- 
  ((flprimaryRepub2016_Cand$Rubio + flprimaryRepub2016_Cand$Kasich - 
      flprimaryRepub2016_Cand$Trump + flprimaryRepub2016_Cand$Cruz) / 
     (flprimaryRepub2016_Cand$TotalRVotesCast) *100) 
flprimaryRepub2016_Cand$ExtCandPercent2016_MOV <- 
  ((flprimaryRepub2016_Cand$Trump + flprimaryRepub2016_Cand$Cruz - 
      flprimaryRepub2016_Cand$Rubio + flprimaryRepub2016_Cand$Kasich) / 
     (flprimaryRepub2016_Cand$TotalRVotesCast) *100) 

    ###Democrat
flprimaryDem2016_Cand$TotalDVotesCast <- 
  (flprimaryDem2016_Cand$Clinton + flprimaryDem2016_Cand$Sanders)

flprimaryDem2016_Cand$EstCandPercent2016 <- 
  ((flprimaryDem2016_Cand$Clinton) / 
     (flprimaryDem2016_Cand$TotalDVotesCast) *100)
flprimaryDem2016_Cand$ExtCandPercent2016 <- 
  ((flprimaryDem2016_Cand$Sanders) /  
     (flprimaryDem2016_Cand$TotalDVotesCast) *100)

flprimaryDem2016_Cand$EstCandPercent2016_MOV <- 
  ((flprimaryDem2016_Cand$Clinton - flprimaryDem2016_Cand$Sanders) / 
     (flprimaryDem2016_Cand$TotalDVotesCast) *100)
flprimaryDem2016_Cand$ExtCandPercent2016_MOV <- 
  ((flprimaryDem2016_Cand$Sanders - flprimaryDem2016_Cand$Clinton) / 
     (flprimaryDem2016_Cand$TotalDVotesCast) *100)

  ##2018
    ###Republican
flprimaryRepub2018_Cand$TotalRVotesCast <- 
  (flprimaryRepub2018_Cand$DeSantis + 
     flprimaryRepub2018_Cand$Putnam )

flprimaryRepub2018_Cand$EstCandPercent2018 <- 
  ((flprimaryRepub2018_Cand$Putnam) / 
     (flprimaryRepub2018_Cand$TotalRVotesCast) *100)
flprimaryRepub2018_Cand$ExtCandPercent2018 <- 
  ((flprimaryRepub2018_Cand$DeSantis) / (
    flprimaryRepub2018_Cand$TotalRVotesCast) *100)

flprimaryRepub2018_Cand$EstCandPercent2018_MOV <- 
  ((flprimaryRepub2018_Cand$Putnam - flprimaryRepub2018_Cand$DeSantis) / 
     (flprimaryRepub2018_Cand$TotalRVotesCast) *100)
flprimaryRepub2018_Cand$ExtCandPercent2018_MOV <- 
  ((flprimaryRepub2018_Cand$DeSantis - flprimaryRepub2018_Cand$Putnam) / 
     (flprimaryRepub2018_Cand$TotalRVotesCast) *100)

    ###Democrat
flprimaryDem2018_Cand$TotalDVotesCast <- 
  (flprimaryDem2018_Cand$Gillum + flprimaryDem2018_Cand$Graham + 
     flprimaryDem2018_Cand$Greene + flprimaryDem2018_Cand$Levine)

flprimaryDem2018_Cand$EstCandPercent2018 <- 
  ((flprimaryDem2018_Cand$Graham + flprimaryDem2018_Cand$Levine) / 
     (flprimaryDem2018_Cand$TotalDVotesCast) *100)
flprimaryDem2018_Cand$ExtCandPercent2018 <- 
  ((flprimaryDem2018_Cand$Gillum + flprimaryDem2018_Cand$Green) /  
     (flprimaryDem2018_Cand$TotalDVotesCast) *100)

flprimaryDem2018_Cand$EstCandPercent2018_MOV <- 
  ((flprimaryDem2018_Cand$Graham + flprimaryDem2018_Cand$Levine - 
      flprimaryDem2018_Cand$Gillum - flprimaryDem2018_Cand$Green) / 
     (flprimaryDem2018_Cand$TotalDVotesCast) *100)
flprimaryDem2018_Cand$ExtCandPercent2018_MOV <- 
  ((flprimaryDem2018_Cand$Gillum + flprimaryDem2018_Cand$Green - 
      flprimaryDem2018_Cand$Graham - flprimaryDem2018_Cand$Levine) / 
     (flprimaryDem2018_Cand$TotalDVotesCast) *100)
```
Finally, as we did with the Registration Data, make a data.frame which includes our percentage variables from all 3 elections,  for **Republican** and **Democratic** candidates 

*for purposes of this analysis, and since these are also binary variables, I will bind just the Extreme Candidate Percentages, as those Counties in which <50% of the vote goes to the Extreme Candidate are Establisment Candidate victories.*
```{r, results="hide"}
##sort
flprimaryRepub2010_Cand <- 
  flprimaryRepub2010_Cand[order(flprimaryRepub2010_Cand$CountyName),]
flprimaryRepub2016_Cand <- 
  flprimaryRepub2016_Cand[order(flprimaryRepub2016_Cand$CountyName),]
flprimaryRepub2018_Cand <- 
  flprimaryRepub2018_Cand[order(flprimaryRepub2018_Cand$CountyName),]
flprimaryDem2010_Cand <- 
  flprimaryDem2010_Cand[order(flprimaryDem2010_Cand$CountyName),]
flprimaryDem2016_Cand <- 
  flprimaryDem2016_Cand[order(flprimaryDem2016_Cand$CountyName),]
flprimaryDem2018_Cand <- 
  flprimaryDem2018_Cand[order(flprimaryDem2018_Cand$CountyName),]

##compare: the `identical()` logical argument should return back 
  ##`[1] True` if the data match. 
identical(flprimaryRepub2010_Cand$CountyName,flprimaryRepub2016_Cand$CountyName )
identical(flprimaryRepub2010_Cand$CountyName,flprimaryRepub2018_Cand$CountyName )
identical(flprimaryDem2010_Cand$CountyName,flprimaryDem2016_Cand$CountyName )
identical(flprimaryDem2010_Cand$CountyName,flprimaryDem2018_Cand$CountyName )

## note the 2016 & 2018 data.frames are returning a “FALSE” in the identical() test. 
  ##Upon inspection of the County name variables tested here, it appears that 
  ##“DeSoto” County was entered as “Desoto” in the 2016 & 2018 Florida election 
  ##results, and needs to be changed to match the 2010 file. 

flprimaryRepub2016_Cand[13, 1] = "DeSoto"
flprimaryDem2016_Cand[13, 1] = "DeSoto"
flprimaryRepub2018_Cand[13, 1] = "DeSoto"
flprimaryDem2018_Cand[13, 1] = "DeSoto"

## bind
FLRepubCountyElection_Results <- 
  cbind(flprimaryRepub2010_Cand,flprimaryRepub2016_Cand$ExtCandPercent2016, 
        flprimaryRepub2016_Cand$ExtCandPercent2016_MOV,
        flprimaryRepub2018_Cand$ExtCandPercent2018, 
        flprimaryRepub2018_Cand$ExtCandPercent2018_MOV)

FLDemCountyElection_Results <- 
  cbind(flprimaryDem2010_Cand, flprimaryDem2016_Cand$ExtCandPercent2016, 
        flprimaryDem2016_Cand$ExtCandPercent2016_MOV, 
        flprimaryDem2018_Cand$ExtCandPercent2018, 
        flprimaryDem2018_Cand$ExtCandPercent2018_MOV)

## tidy

FLRepubCountyElection_Results <- 
  FLRepubCountyElection_Results [,-c({2:6}, 8)] 

FLDemCountyElection_Results <- 
  FLDemCountyElection_Results [,-c({2:5}, 7)] 
```
# Mapping 

In order to create the outline of Florida and its 67 Counties, geospatial vector data is necessary to make a shapefile.  

I downloaded the geospacial data from [United States Census Bureau's Cartographic Boundary Shapefiles](https://www.census.gov/geo/maps-data/data/cbf/cbf_counties.html), and have saved in my working directory. 
```{r warning = FALSE}

usshapefile <- "CountyShapeFiles/cb_2014_us_county_5m.shp"

usgeo <- read_shape(file=usshapefile, as.sf = TRUE)
```
To extract just Florida Counties from this U.S. shapefile, use the [U.S. Census Bureau FIPS Code for the State of Florida](https://www.census.gov/geo/reference/ansi_statetables.html) (*FL == 12*) to `filter` data that relates to Florida.
```{r warning = FALSE}
flgeo <- filter(usgeo, STATEFP=="12")
```
### Quick Thematic Map 

Use tmap's `qtm()` (*quick thematic map*) function to ensure that the shapefile for Florida _'flgeo'_ imported and appears correctly:
```{r fig.width=10, fig.height=5, fig.cap = "Shapefile of Florida's 67 Counties"}
qtm(flgeo)
```
### Choropleth Map: Party Registration

To visualize the trends in party registration in Florida Counties, make a choropleth of voters registered with the two *primary eligible* parties:  **Republican** Party of Florida & Florida **Democratic** Party. 

This will aide in visualizing which counties lean heavily **Republican** or **Democratic**, and those which are more evenly distributed.  

Begin the choropleth by combining the Primary Registration data.frame with the the Florida shapefile. 

The first step is to ensure that the shapefile's vectors containing County Names `flgeo$NAME` is the same data type as those from the Primary Registration data.frames `df$CountyName`.
```{r, results = "hide"}

##confirm data type 

str(flgeo$NAME)
str(flprimary_partyreg_2010$CountyName)
str(flprimary_partyreg_2016$CountyName)
str(flprimary_partyreg_2018$CountyName)
str(FLCountyReg_Percents$CountyName)

## note: the shapefile lists Counties as factors, while they’re 
  ##character text in the party registration data.frames. As before, 
  ##we can easily convert the data type, from factor level values to character:

flgeo$NAME <- as.character(flgeo$NAME)
```
Next, in order to assign the geospatial vector data to its corresponding rows in the data.frame, employ the `identical()` logical to ensure uniformity.
```{r, results = "hide"}
##sort
flgeo <- flgeo[order(flgeo$NAME),]
flprimary_partyreg_2010 <- 
  flprimary_partyreg_2010[order(flprimary_partyreg_2010$CountyName),]
flprimary_partyreg_2016 <- 
  flprimary_partyreg_2016[order(flprimary_partyreg_2010$CountyName),]
flprimary_partyreg_2018 <- 
  flprimary_partyreg_2018[order(flprimary_partyreg_2018$CountyName),]
FLCountyReg_Percents <- 
  FLCountyReg_Percents[order(FLCountyReg_Percents$CountyName),]

##compare: the `identical()` logical argument should return 
  ##`[1] True` if the data match. 
identical(flgeo$NAME,flprimary_partyreg_2010$CountyName )
identical(flgeo$NAME,flprimary_partyreg_2016$CountyName )
identical(flgeo$NAME,flprimary_partyreg_2018$CountyName )
identical(flgeo$NAME,FLCountyReg_Percents$CountyName )
```
Finally, join the Florida shapefile to the data.frame for each primary election using the `append_data()` tool, with County names as the key.
```{r, results= "hide"}
flPri_reg2010map <- 
  append_data(flgeo, flprimary_partyreg_2010,
              key.shp = "NAME", key.data="CountyName")
flPri_reg2016map <- 
  append_data(flgeo, flprimary_partyreg_2016, 
              key.shp = "NAME", key.data="CountyName")
flPri_reg2018map <- 
  append_data(flgeo, flprimary_partyreg_2018, 
              key.shp = "NAME", key.data="CountyName")
FLPri_Reg3ElectionMap <- 
  append_data(flgeo, FLCountyReg_Percents,
              key.shp = "NAME", key.data="CountyName")
```
#### Simple Static Maps

A `qtm()` will show us if the data and shapefile have merged, and we are on track to start customizing: 
```{r}

qtm(flPri_reg2010map, "RepPct")

```
#### The **'tmap'** Aesthetic Features

Improve the look of maps by changing the aesthetics features, including: 
  + color {using a typical *Democrat = Blue*, *Republican = Red* palettes}; 
  + add titles; 
  + standardize the legend breaks; & 
  + add lables, etc.
```{r fig.width = 13, fig.height = 7, fig.cap = "2010 2-Party Registration Density Per Florida County"}

tm_shape(flPri_reg2010map) +
  tm_fill("RepPct", title="% Two Party Registration Per County", 
          palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
    labels = c("80%-90% Dem.", "70%-80% Dem.","60%-70% Dem.", "50%-60% Dem.", 
                     "50%-60% Rep.", "60%-70% Rep.", "70%-80% Rep."), 
    midpoint = 50) + tm_borders(alpha=.5) + tm_text("NAME", size=0.3) +
  tm_layout(main.title = "2010 Primary Election Party Registration", 
            legend.height = 6,
            legend.title.size = 1, main.title.position = "center")
```
#### The **'tmap'** Facet Features

Using **'tmap'**s `facet()` feature, a better comparasion of changes in a Counties' 2-party Registration over the course of the 8-years is possible. 

This allows maps to be plotted side-by-side. To do this, first convert*wide* data into *long* data. 
```{r}
  ##convert the key to factor level data:
FLPri_Reg3ElectionMap$NAME <- as.factor(FLPri_Reg3ElectionMap$NAME)

  ##use tidyr's gather() function to transform the data:
FLRegistrationPercentages <- gather(FLPri_Reg3ElectionMap, 
  Year, RepublicanPercent, '2010PctRep':'2018PctRep', factor_key=FALSE)
```
Now, the maps of the three election years can be plotted in the same frame, each `facet` its own, and the aesthetics applied, as such:
```{r fig.width = 13, fig.height = 7}
tm_shape(FLRegistrationPercentages) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("RepublicanPercent", title="Registered Voter Majority %", 
          palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_borders(alpha=.5) + tm_text("NAME", size=0.2) +
  tm_layout(main.title = "2-Party Registration Density per County",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.20,.5), 
            legend.width = 2, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "blue",
            main.title.position = "center")
```
Or, finally, we can animate these facets, to play one-at-a-time or in the same loop:
```{r fig.width = 13, fig.height = 10, fig.cap = "Changes in 2-Party Registration Density Per Florida County"}

Reg_Anim <- tm_shape(FLRegistrationPercentages) + 
  tm_facets(along = "Year", ncol = 1, nrow = 1) + 
  tm_fill("RepublicanPercent", title="Registered Voter Majority %", 
          palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_borders(alpha=.5) + tm_text("NAME", size=0.2) +
  tm_layout(main.title = "2-Party Registration Density per County",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.05,.5), 
            legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "blue",
            main.title.position = "center")

tmap_animation(Reg_Anim, filename = "reg_anim.gif", 
               delay = 140, width =6, height = 8, loop = TRUE, 
               restart.delay = 140)

knitr::include_graphics('reg_anim.gif', dpi = NA)

```
## Election Results Mapping

Patty, here, explain the tmpap functions (bubbles, layers, etc.), as well as the categorization of candidates into different groups. 

To visualize the trends in party registration in Florida Counties, we can make a choropleth of voters registered with the *primary eligible* parties, the **Republican** Party of Florida & Florida **Democratic** Party. This will allow us to see which counties lean heavily **Republican** or **Democratic**, and those which are more even. 

A complete breakdown of the candidates, and their categorization in these models is as follows: 

**Year**| **Extreme**   | **Establishment**
--------|---------------|---------------------
2010 (R)| Scott         | McCollum/McCalister
2010 (D)| Moore         | Sink
        |               |
2016 (R)| Trump/Cruz    | Rubio/Kasich
2016 (D)| Sanders       | Clinton
        |               |
2018 (R)| DeSantis      | Putnam
2018 (D)| Gillum/Greene | Graham/Levine

#### Append Results & Florida Shapefile

As before, the first step to mapping the election reults data is the data.frames with the results to the shapefile data, which begins by confirming the County names vectors are `identical`, and if all is well, join the Florida shapefile with the Elections data.frames, again with the `append_data()` tool.
```{r, results= "hide"}

  ##Confirm 'identical'
identical(flgeo$NAME,FLDemCountyElection_Results$CountyName )
identical(flgeo$NAME,FLRepubCountyElection_Results$CountyName )

  ##Append
FlRepubCountyElectionResultsMap <- 
  append_data(flgeo, FLRepubCountyElection_Results, key.shp = 
                "NAME", key.data="CountyName")
FlDemCountyElectionResultsMap <- 
  append_data(flgeo, FLDemCountyElection_Results, key.shp = 
                "NAME", key.data="CountyName")
```
#### Simple Static Maps
 
The state and county borders will be created by the first map layer, `tm_polygon`; 
  + elections results can be mapped by county with the second layer `tm_shape`; 
  + the percentage of the vote received by each candidate group, *Extreme* & *Establishment*, as the third layer, `tm_fill`; 
  + and finally the aesthetic features detailed by the fourth layer.  
```{r fig.width = 13, fig.height = 7}
tm_shape(FlRepubCountyElectionResultsMap) + 
  tm_fill(c("ExtCandPercent2010"), palette = "-YlOrRd", 
          breaks = c(20, 30, 40, 50, 60, 70, 80),
          labels = c("  70%-80% Est  ", "  60%-70% Est  ",
                     "  50%-60% Est  ", "  50%-60% Ext  ", 
                     "  60%-70% Ext  ", "  70%-80% Ext  "), 
          title = "Candidate % of Vote Achieved", legend.show = TRUE, 
          legend.is.portrait = FALSE) + 
tm_polygons(flgeo) + tm_text("NAME", size=0.2) + 
  tm_layout(main.title = "2010 Republican Primary", 
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = 
              c(.05,.6), legend.width = 2, legend.frame = FALSE, 
            main.title.size = 2, main.title.position = "center")
```
#### Facets
The `tmap_arrange` function allows individual maps to be pictured in the same frame (even dictating in what order and layout format). 

To do this, you would just create and name a map for each party's results, per election cycle. 

However, for ease with the data (as we now have 7 working data.frames) I will combine all **Republicant**, **Democratic**, and **Registration** data into one long data frame each, to use all **'tmap'** functions with one call: 
```{r}
## Rename the colums so they are more uniform, and will order by year.

colnames(FlRepubCountyElectionResultsMap
         )[colnames(FlRepubCountyElectionResultsMap) == 
             'flprimaryRepub2016_Cand$ExtCandPercent2016_MOV'] <- 
  'ExtCandPercent2016_MOV'
colnames(FlRepubCountyElectionResultsMap
         )[colnames(FlRepubCountyElectionResultsMap) == 
             'flprimaryRepub2018_Cand$ExtCandPercent2018_MOV'] <- 
  'ExtCandPercent2018_MOV'
colnames(FlRepubCountyElectionResultsMap
         )[colnames(FlRepubCountyElectionResultsMap) == 
             'flprimaryRepub2016_Cand$ExtCandPercent2016'] <- 
  'ExtCandPercent2016'
colnames(FlRepubCountyElectionResultsMap
         )[colnames(FlRepubCountyElectionResultsMap) == 
             'flprimaryRepub2018_Cand$ExtCandPercent2018'] <- 
  'ExtCandPercent2018'

FlRepubCountyElectionResultsMap1 <- 
  melt(FlRepubCountyElectionResultsMap, id.var = 
         c("STATEFP", "COUNTYFP", "COUNTYNS", "AFFGEOID", "GEOID", 
           "NAME", "LSAD", "ALAND", "AWATER", "ExtCandPercent2010_MOV", 
           "ExtCandPercent2016_MOV", "ExtCandPercent2018_MOV", "geometry"), 
       variable.name = "Year", value.name = "ExtremeCandPercent" )

FlRepubCountyElectionResultsMap2 <- 
  melt(FlRepubCountyElectionResultsMap, id.var = 
         c("STATEFP", "COUNTYFP", "COUNTYNS", "AFFGEOID", "GEOID", 
           "NAME", "LSAD", "ALAND", "AWATER", "ExtCandPercent2010", 
           "ExtCandPercent2016", "ExtCandPercent2018", "geometry"), 
       variable.name = "YearMOV", value.name = "ExtremeCandPercent_MOV" )

## Reorder columns
FlRepubCountyElectionResultsMap1 <- 
  FlRepubCountyElectionResultsMap1[c(1:9, 14:15, 13)]
FlRepubCountyElectionResultsMap2 <- 
  FlRepubCountyElectionResultsMap2[c(1:9, 14:15, 13)]

## Return to shape.file
FlRepubCountyElectionResultsMap1 <- 
  st_as_sf(FlRepubCountyElectionResultsMap1)
FlRepubCountyElectionResultsMap2 <- 
  st_as_sf(FlRepubCountyElectionResultsMap2)
 
 ##Prepare to cbind the registration percents in by sorting both
 
FLRegistrationPercentages <- 
  FLRegistrationPercentages[with (
    FLRegistrationPercentages, order
    (FLRegistrationPercentages$Year, FLRegistrationPercentages$NAME)),]
FlRepubCountyElectionResultsMap1 <- 
  FlRepubCountyElectionResultsMap1[with (
    FlRepubCountyElectionResultsMap1, order
    (FlRepubCountyElectionResultsMap1$Year, 
      FlRepubCountyElectionResultsMap1$NAME)),]
FlRepubCountyElectionResultsMap2 <- 
  FlRepubCountyElectionResultsMap2[with (
    FlRepubCountyElectionResultsMap2, order
    (FlRepubCountyElectionResultsMap2$YearMOV, 
      FlRepubCountyElectionResultsMap2$NAME)),]

## Bind the MOV and Registration Data to the shapefile map
 
FlRepubCountyElectionResultsMap <- 
  cbind(FlRepubCountyElectionResultsMap1,
        FlRepubCountyElectionResultsMap2$ExtremeCandPercent_MOV, 
        FLRegistrationPercentages$RepublicanPercent)

## Again, rename the columns so thhey are more easily referenceable:

colnames(FlRepubCountyElectionResultsMap)[colnames(
  FlRepubCountyElectionResultsMap) == 
    'FlRepubCountyElectionResultsMap2.ExtremeCandPercent_MOV'] <- 
  'ExtCandPercent_MOV'
colnames(FlRepubCountyElectionResultsMap)[colnames(
  FlRepubCountyElectionResultsMap) == 
    'FLRegistrationPercentages.RepublicanPercent'] <- 
  'RepublicanPercent'

## Repeat for Democratic Data: 
colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'flprimaryDem2016_Cand$ExtCandPercent2016_MOV'] <- 
  'ExtCandPercent2016_MOV'
colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'flprimaryDem2018_Cand$ExtCandPercent2018_MOV'] <- 
  'ExtCandPercent2018_MOV'
colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'flprimaryDem2016_Cand$ExtCandPercent2016'] <- 
  'ExtCandPercent2016'
colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'flprimaryDem2018_Cand$ExtCandPercent2018'] <- 
  'ExtCandPercent2018'

FlDemCountyElectionResultsMap1 <- 
  melt(FlDemCountyElectionResultsMap, id.var = c(
    "STATEFP", "COUNTYFP", "COUNTYNS", "AFFGEOID", "GEOID", "NAME", 
    "LSAD", "ALAND", "AWATER", "ExtCandPercent2010_MOV", "ExtCandPercent2016_MOV", 
    "ExtCandPercent2018_MOV", "geometry"), variable.name = "Year", 
    value.name = "ExtremeCandPercent" )

FlDemCountyElectionResultsMap2 <- 
  melt(FlDemCountyElectionResultsMap, id.var = c(
    "STATEFP", "COUNTYFP", "COUNTYNS", "AFFGEOID", "GEOID", "NAME", 
    "LSAD", "ALAND", "AWATER", "ExtCandPercent2010", "ExtCandPercent2016", 
    "ExtCandPercent2018", "geometry"), variable.name = "YearMOV", 
    value.name = "ExtremeCandPercent_MOV" )

## Reorder columns
FlDemCountyElectionResultsMap1 <- 
  FlDemCountyElectionResultsMap1[c(1:9, 14:15, 13)]
FlDemCountyElectionResultsMap2 <- 
  FlDemCountyElectionResultsMap2[c(1:9, 14:15, 13)]

## Return to shape.file
FlDemCountyElectionResultsMap1 <- 
  st_as_sf(FlDemCountyElectionResultsMap1)
FlDemCountyElectionResultsMap2 <- 
  st_as_sf(FlDemCountyElectionResultsMap2)
 
 ##Prepare to cbind the registration percents in by sorting both
 
FLRegistrationPercentages <- 
  FLRegistrationPercentages[with (FLRegistrationPercentages, 
                                  order(FLRegistrationPercentages$Year, 
                                        FLRegistrationPercentages$NAME)),]
FlDemCountyElectionResultsMap1 <- 
  FlDemCountyElectionResultsMap1[with (FlDemCountyElectionResultsMap1, 
                                       order(FlDemCountyElectionResultsMap1$Year, 
                                             FlDemCountyElectionResultsMap1$NAME)),]
FlDemCountyElectionResultsMap2 <- 
  FlDemCountyElectionResultsMap2[with (FlDemCountyElectionResultsMap2, 
                                       order(FlDemCountyElectionResultsMap2$YearMOV, 
                                             FlDemCountyElectionResultsMap2$NAME)),]

## Bind the MOV and Registration Data to our Map
 
FlDemCountyElectionResultsMap <- cbind(
  FlDemCountyElectionResultsMap1,FlDemCountyElectionResultsMap2$
    ExtremeCandPercent_MOV, FLRegistrationPercentages$RepublicanPercent)

## Again, rename the columns so thhey are more easily referenced:

colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'FlDemCountyElectionResultsMap2.ExtremeCandPercent_MOV'] <- 
  'ExtCandPercent_MOV'
colnames(FlDemCountyElectionResultsMap)[colnames(
  FlDemCountyElectionResultsMap) == 'FLRegistrationPercentages.RepublicanPercent'] <- 
  'RepublicanPercent'
```
Now, any combination of these maps can be plotted within the same frame, which allows us to make comparasions, and see trends in the data:
```{r fig.width = 11, fig.height = 10, message = FALSE}

  ##2010-2018 Democratic Groups Percent per County:
tm_shape(FlDemCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("ExtremeCandPercent", title="Candidate Group % of Vote",
          palette = "YlGnBu", n = 7,
         breaks = c(20, 30, 40, 50, 60, 70, 80),
         labels = c("  70% - 80% Est  ", "  60% - 70% Est  ",
                    "  50% - 60% Est  ", "  50% - 60% Ext  ", "  60% - 70% Ext  ",
                    "  70% - 80% Ext  "),
         midpoint = 50, legend.is.portrait = FALSE) + 
  tm_borders(alpha=.5) + tm_text("NAME", size=0.3) +
  tm_layout(main.title = "Democratic Primary 2010-2018: Establishment v. Extreme",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.30,.7), 
            legend.width = 4, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "blue", main.title.position = "center")
```
```{r fig.width = 11, fig.height = 10, message = FALSE}
  
 ##Facet 2010-2018 Republican Groups Percent per County:
tm_shape(FlRepubCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("ExtremeCandPercent", title="Candidate Group % of Vote", 
          palette = "YlOrRd", n = 7,
         breaks = c(20, 30, 40, 50, 60, 70, 80),
         labels = c("  70% - 80% Est  ", "  60% - 70% Est  ",
                    "  50% - 60% Est  ", "  50% - 60% Ext  ", 
                    "  60% - 70% Ext  ", "  70% - 80% Ext  "),
         midpoint = 50, legend.is.portrait = FALSE) + 
  tm_borders(alpha=.5) + tm_text("NAME", size=0.3) +
  tm_layout(main.title = "Republican Primary 2010-2018: Establishment v. Extreme",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = 
              c(.30,.7), legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "red", main.title.position = "center")
```
#### Combine Layers 
The `shapes()` function of **'tmap'** allows layering of different variables over chloropleths, like our Party Registration maps. 

Here, I will display Elections Results (depicting candidate group margins of victory) atop the County 2-Party Registration choropleths (showing registration density).
```{r fig.width = 13, fig.height = 10, message = FALSE, fig.cap = "2010 - 2018 Republican Primary: Extreme v. Establishment Candidate"}

tm_shape(FlRepubCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent", 
              title="Registered Voter Majority %", 
              palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  "
                     ,"  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  "
                     ,"  70% - 80% Rep.  "), midpoint = 50, 
          legend.is.portrait = FALSE) +
  tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",
             shape = 21, alpha = 1, 
             scale = 2, sizes.legend = c(10, 20, 30, 40, 50, 60),
             border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%",
                        "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-viridis") + 
  tm_layout(main.title = "Republican Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",
            legend.outside = TRUE, 
            legend.outside.position = "bottom",
            legend.position = c(.30,.05), legend.width = 4,
            legend.height = 3, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white",
            panel.label.bg.color = "red",
            main.title.position = "center")
```
```{r fig.width = 13, fig.height = 10, message = FALSE, fig.cap = "2010 - 2018 Democratic Primary: Extreme v. Establishment Candidate"}

tm_shape(FlDemCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent",
              title="Registered Voter Majority %",
              palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ",
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50,
          legend.is.portrait = FALSE) +
  tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",
             shape = 21, alpha = 1, 
             scale = 2, sizes.legend = c(10, 20, 30, 40, 50, 60),
             border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%",
                        "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-plasma") + 
  tm_layout(main.title = "Democratic Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.30,.05),
            legend.width = 4, legend.height = 3, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white",
            panel.label.bg.color = "blue",
            main.title.position = "center")
```
#### Interactive Maps

Any map can become interactive, simply by switching the **'tmap'** mode to `tmap_mode("view')`:
```{r}
tmap_mode("view")
```
#### Extreme Republican Candidates: 

Try:  *scrolling in and out;* 
      *changing layers*
```{r}

tm_shape(FlRepubCountyElectionResultsMap) +
  tm_facets(by = "Year", as.layers = TRUE, ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent", title="Registered Voter Majority %",
              palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50,
          legend.is.portrait = FALSE) + tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",
             shape = 21, alpha = 1, 
             scale = 2, sizes.legend = c(10, 20, 30, 40, 50, 60),
             border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%",
                        "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-viridis", id = "NAME", popup.vars=c(
               "RepublicanPercent","ExtremeCandPercent",
               "ExtCandPercent_MOV")) + 
  tm_layout(main.title = "Republican Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",
            legend.show=FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white",
            panel.label.bg.color = "red",
            main.title.position = "center") 
```
#### Extreme Democratic Candidates: 

Try:  *scrolling in and out;* 
      *changing layers*
```{r}

tm_shape(FlDemCountyElectionResultsMap) +
  tm_facets(by = "Year", as.layers = TRUE, ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent", title="Registered Voter Majority %",
              palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",
             shape = 21, alpha = 1, 
             scale = 2, sizes.legend = c(10, 20, 30, 40, 50, 60),
             border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%",
                        "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-plasma", id = "NAME", popup.vars=c(
               "RepublicanPercent","ExtremeCandPercent", "ExtCandPercent_MOV")) + 
  tm_layout(main.title = "Democratic Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",
            legend.show=FALSE, panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white",
            panel.label.bg.color = "red",
            main.title.position = "center") 
```
```{r include = FALSE}
tmap_mode("plot")
```
#### Limit Results to See Trends

Finally, to see if any trends are occuring within counties based upon their party composition, plot the *% Vote Total* and *Margin of Victory* for only those Counties in which a majority of the two-party registration corresponds to the respective candidate group. (Available in the next section).

# Analysis

## Registration Trends

Our first map reviews the changes in two-party registration density from 2010 to 2018: 
```{r fig.width = 13, fig.height = 10, fig.cap = "2010, 2016 & 2018 Two-Party Registration Density Per Florida County"}
tm_shape(FLRegistrationPercentages) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("RepublicanPercent", title="Registered Voter Majority %", 
          palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ",
                     "  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ",
                     "  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_borders(alpha=.5) + tm_text("NAME", size=0.2) +
  tm_layout(main.title = "2-Party Registration Density per County",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.20,.5), 
            legend.width = 2, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "blue",
            main.title.position = "center")
```
Here, we can see that: 

* As we can see, Florida's two-party registration appears to have switched from favoring the **Democratic** party in 2010 to favoring the **Republican** party in 2010, in terms of *count*.
  + Also evident is that *South Florida*'s two most populous counties are also now more densly populated with **Democratic** registrants. 

This, most likely, makes up for the switch from light blue to light red counties in terms of registrants, in our choropleth. 

## Extreme Candidate Trends

Next, we will look at the "Extreme Candidate" percentage of the vote for each party, and see if any patterns emerge in the data. 

#### Republican
```{r fig.width = 11, fig.height = 10, message = FALSE}
tm_shape(FlRepubCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("ExtremeCandPercent", title="Candidate Group % of Vote", 
          palette = "YlOrRd", n = 7,
         breaks = c(20, 30, 40, 50, 60, 70, 80),
         labels = c("  70% - 80% Est  ", "  60% - 70% Est  ",
                    "  50% - 60% Est  ", "  50% - 60% Ext  ", 
                    "  60% - 70% Ext  ", "  70% - 80% Ext  "),
         midpoint = 50, legend.is.portrait = FALSE) + 
  tm_borders(alpha=.5) + tm_text("NAME", size=0.3) +
  tm_layout(main.title = "Republican Primary 2010-2018: Establishment v. Extreme",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = 
              c(.30,.7), legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "red", main.title.position = "center")
```
Here, we can see that:

* The more extreme candidate was overwhelmingly favored by voters of the Republican party in 2018, over 2010. 
  + Specifically, the locus of this support has centered around the *South* and *Costal* Florida, with the exception of the *Northern Gulf Coast*.

#### Democratic
```{r fig.width = 11, fig.height = 10, message = FALSE}

  ##2010-2018 Democratic Groups Percent per County:
tm_shape(FlDemCountyElectionResultsMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_fill("ExtremeCandPercent", title="Candidate Group % of Vote",
          palette = "YlGnBu", n = 7,
         breaks = c(20, 30, 40, 50, 60, 70, 80),
         labels = c("  70% - 80% Est  ", "  60% - 70% Est  ",
                    "  50% - 60% Est  ", "  50% - 60% Ext  ", "  60% - 70% Ext  ",
                    "  70% - 80% Ext  "),
         midpoint = 50, legend.is.portrait = FALSE) + 
  tm_borders(alpha=.5) + tm_text("NAME", size=0.3) +
  tm_layout(main.title = "Democratic Primary 2010-2018: Establishment v. Extreme",  
            legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.30,.7), 
            legend.frame = FALSE,             panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", 
            panel.label.bg.color = "blue", main.title.position = "center")
```
Here, we can see that:

* The more extreme candidate was ALSO overwhelmingly favored by voters of the Democratic party in 2018, over 2010. 
  + However, the Democratic voters supporting the Extreme Candidate seem to be more evenly dispersed around the State, including *North*, *Central* and  *South* Florida.
  
### Limit Results to Highlight Trends

Combine the Extreme Candidate's Percent Vote share with a layer of those counties which share a majoritarian registration with candidates from that party.

##### Republican
```{r, fig.width = 13, fig.height = 10, message = FALSE, fig.cap = "Republican Majority Counties"}

  ##Drop Counties where the 2-party registration is majority Democratic
RepubCountiesDataMap <- FlRepubCountyElectionResultsMap[!(FlRepubCountyElectionResultsMap$RepublicanPercent <= 50),]

  ## Map the  remaining data

qtm(flgeo) + tm_shape(RepubCountiesDataMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent", title="Registered Voter Majority %", palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ","  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ","  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",  shape = 21, alpha = 1, 
             scale = 1.5, sizes.legend = c(10, 20, 30, 40, 50, 60), border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%", "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-viridis") + 
  tm_layout(main.title = "Republican Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",  legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.30,.05), legend.width = 4, legend.height = 3, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", panel.label.bg.color = "red",
            main.title.position = "center")
```
Here, we see the following:

* Republican voters in the *I-4 Corridor*, and *Southwest Florida* and in the *Panhandle* are in fact trending more "extreme" since 2010.
  + The Counties that are newer to the two-party Republican majority are more likely to favor the Establishment Candidates. 
  
##### Democratic
```{r, fig.width = 13, fig.height = 10, message = FALSE, fig.cap = "Democratic Majority Counties"}

  ##Drop Counties where the 2-party registration is majority Republican
DemCountiesDataMap <- FlDemCountyElectionResultsMap[!(FlDemCountyElectionResultsMap$RepublicanPercent >= 50),]

  ## Map the  remaining data

qtm(flgeo) + tm_shape(DemCountiesDataMap) +
  tm_facets(by = "Year", ncol = 3, nrow = 1) + 
  tm_polygons("RepublicanPercent", title="Registered Voter Majority %", palette = "-RdBu", n = 7,
          breaks = c(10, 20, 30, 40, 50, 60, 70, 80), 
          labels = c("  80% - 90% Dem.  ", "  70% - 80% Dem.  ","  60% - 70% Dem.  ", "  50% - 60% Dem.  ", 
                     "  50% - 60% Rep.  ", "  60% - 70% Rep.  ","  70% - 80% Rep.  "), midpoint = 50, legend.is.portrait = FALSE) +
  tm_text("NAME", size=0.2) +
  tm_bubbles(size = "ExtremeCandPercent", col = "ExtCandPercent_MOV",  shape = 21, alpha = 1, 
             scale = 1.5, sizes.legend = c(10, 20, 30, 40, 50, 60), border.alpha = .5, 
             breaks = c(-50, -25, 0, 10, 20, 30, 40, 50), midpoint = NA,
             labels = c("-50% to -25%", "-25% to 0%", "0% to 10%", "10%-20%", "20%-30%","30%-40%", "40%-50%"), 
             palette = "-plasma") + 
  tm_layout(main.title = "Democratic Primary 2010-2018: County Reg. & Extreme v. Establishment Candidate",  legend.outside = TRUE, 
            legend.outside.position = "bottom", legend.position = c(.30,.05), legend.width = 4, legend.height = 3, legend.frame = FALSE, 
            panel.labels = c('2010', '2016', '2018'),
            panel.label.size = 2, panel.label.color = "white", panel.label.bg.color = "red",
            main.title.position = "center")
```
Here, we see the following:

* Democratic majorities have condensed in *Northcentral Florida*, and *Southeast Florida* since 2010. 
  + As a counties share of Democratic voters increases, it becomes more likey to support a non-Establisment or Extreme Candidate.
