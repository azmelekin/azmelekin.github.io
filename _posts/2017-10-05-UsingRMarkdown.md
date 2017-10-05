---
layout: post
title:  "Using R Markdown & knitr"
date:   2017-10-05
excerpt_separator: <!--more-->
categories: Unemployment
---

I will first setup the environment and load some of the packages that will be used.
The good thing in using R Markdown is creating code in chunks for clarity and ease of debugging. 
[Table 2](https://www.bls.gov/news.release/empsit.t02.htm) & [Table 3](https://www.bls.gov/news.release/empsit.t03.htm) of the _Economic News Release_ contain
the relevant general labor force data including employment/unemployment rates for various groups.
I will extract and plot unemployment rate data by race/ethnicity and by gender. 

I will first setup the environment and load some of the packages that will be used. 
Remember to begin and end the code below with  '```'  when using R Markdown.

{% highlight ruby %}

{r setup, include=FALSE}
knitr::opts_chunk$set(echo=TRUE, invisible=TRUE, warning = FALSE)

#load libraries
pkg <- c("XML", "RCurl", "rlist", "curl", "stringr", "gapminder", "ggplot2", "xml2", "dplyr", "zoo", "readxl")
lapply(pkg, require, character.only = TRUE)

{% endhighlight %}

<!--more-->

Obtain (scrape) the relevant tables:
``` r
#This code will extract the data.
EconNews <- as.data.frame(readHTMLTable(getURL("https://www.bls.gov/news.release/empsit.t02.htm"),
               stringsAsFactors=FALSE, trim = TRUE))

#see what the first few lines of the data look like. We also need to assign appropriate columns and rows later.
head(EconNews[,c(1,5:10)])
```
Some more cleaning & adjustment:
"," needs to be removed so we can coveniently convert from one to another form of number types.

``` r
EconNews[] <- lapply(EconNews, gsub, pattern=',', replacement='')

#Now I will just make columns for the table and assign those to columns 2-10 (the first column is variable names)

colName <- c("Aug2016", "Jul2017", "Aug2017", "Aug2016a", "Apr2017a", "May2017a", "Jun2017a", "Jul2017a", "Aug2017a")

names(EconNews)[2:10] <- colName
```

Scrapping the table for Latino/Hispanic group.
Essentially, the code is the same as the above but need to point to the appropriate url.

``` r
EconNewsLat <- as.data.frame(readHTMLTable(getURL("https://www.bls.gov/news.release/empsit.t03.htm"), stringsAsFactors=FALSE, trim = TRUE))

EconNewsLat[] <- lapply(EconNewsLat, gsub, pattern=',', replacement='')

names(EconNewsLat)[2:10] <- colName

#look at first few lines
head(EconNewsLat[,c(1,5:10)])
```

Names of the first rows for "EconNews" & "EcoNewsLat" are different, so we can't do row bind when column names are different.


Remove different names of first column or rename them with the same column name. I will remove them.

``` r
colnames(EconNewsLat)[1] <- ""
colnames(EconNews)[1] <- ""
```

Row_bind (append) the two tables, using the dplyr package and call the giant table "EconNewsAll"
and can have a look at the first few lines of the relevant columns.

``` r
EconNewsAll <- dplyr::bind_rows(EconNews,EconNewsLat)

#see few first lines
head(EconNewsAll[,c(1,5:)])
```

Now I can subset data by race, gender and age. I also need to rename rows.

``` r
#grab sub-table for all White group as "W1data"
W1data <- EconNewsAll[2:9,2:10] 

#Rename rows
rownames(W1data)<- c("Civil_noninst_pop","civil_labor_force", "Particip_rate", "Employed", "Emp_pop_ratio", "Unemployed","Unemp_rate", "Not_in_labor_force")

#transpose table for analysis & plotting

W1data <- as.data.frame(t(W1data))
```

Add indicator variables for 'Race', 'Quarters' or 'time', 'Class'. Quarters (Qtrs) is months data were released; 
Classes indicate age & gender status in case we want to look at plots by age, gender and race...

``` r
#again with the dplyr package
W1data <- W1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(1,9))),
           Class = as.factor(c(rep(1,9))))

#convert variables to numeric
W1data[,1:8] <- lapply(W1data[,1:8], function(x) as.numeric(as.character(x)))
```

All other data manipulation activities below are copies of the above for specific race/ethnic/gender categories, 
so I will not elaborate further. I only needed to change data frame names, appropriate rows, & classes. 
You can jump to the code for the plots if you like.

For the Black/African American group
``` r
#grab sub-table for All Blacks
B1data <- EconNewsAll[33:40,2:10] 

rownames(B1data)<- c("Civil_noninst_pop","civil_labor_force", "Particip_rate", "Employed", "Emp_pop_ratio", "Unemployed","Unemp_rate", "Not_in_labor_force")

B1data <- as.data.frame(t(B1data))

B1data <- B1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(2,9))),
           Class = as.factor(c(rep(5,9))))

B1data[,1:8] <- lapply(B1data[,1:8], function(x) as.numeric(as.character(x)))
```

For the Asian group
``` r
#grab sub-table for All Asians (Note: Asians were not categorized by gender and age)
A1data <- EconNewsAll[64:71,2:10]

rownames(A1data)<- c("Civil_noninst_pop","civil_labor_force","Particip_rate", "Employed","Emp_pop_ratio","Unemployed","Unemp_rate","Not_in_labor_force")

A1data <- as.data.frame(t(A1data))

A1data <- A1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(3,9))),
           Class = as.factor(c(rep(9,9))))

A1data[,1:8] <- lapply(A1data[,1:8], function(x) as.numeric(as.character(x)))
```

For the Latino/Hispanic group
``` r
#grab sub-table for All Latino/Hispanic
L1data <- EconNewsAll[73:80,2:10]

rownames(L1data)<- c("Civil_noninst_pop","civil_labor_force","Particip_rate", "Employed","Emp_pop_ratio","Unemployed","Unemp_rate","Not_in_labor_force")

L1data <- as.data.frame(t(L1data))

L1data <- L1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(4,9))),
           Class = as.factor(c(rep(10,9))))

L1data[,1:8] <- lapply(L1data[,1:8], function(x) as.numeric(as.character(x)))
```

``` r
#Combine (append) Asian, Hispanic, Black, White tables to create one table for all.
WBAL1data <- dplyr:: bind_rows(W1data, B1data, A1data, L1data)

WBAL1data[,10:11] <- lapply(WBAL1data[,10:11], function(x) as.factor(as.character(x)))

head(WBAL1data)
```

Time to plot the data!
``` r
#Plot for Race/ethnic groups using ggplot
#Note I am plotting subset of data as indicated by the selected rows bewlo
All_plot <- ggplot(data=WBAL1data[c(4:9,13:18,22:27,31:36),], aes(x=Qtrs, y=Unemp_rate, shape=Race, color=Race, group=Race )) + 
  geom_point() + 
  geom_line() + 
  scale_x_discrete(limits =c(4:9), labels=c("Aug 2016", "Apr 2017", "May 2017", "Jun 2017", "Jul 2017", "Aug 2017"))+ 
  scale_color_discrete(breaks=c(1:4), labels=c("All White", "Black", "Asian", "Latino/Hispanic")) +
  scale_shape_discrete(breaks=c(1:4), labels=c("All White", "Black", "Asian", "Latino/Hispanic")) +
  theme(axis.line = element_line(color="blue", linetype = "solid"), plot.title = element_text(hjust = .5)) +  
  ggtitle("Adjusted Unemployment Rates by Race\n (Aug 2016 - Aug 2017)") +
  theme(legend.title = element_blank(), legend.text = element_text(size = 12), legend.background = element_rect(fill="NA", size=.5, linetype = "solid"))
All_plot
```
<img src="/images/All_plot-1.png"/>


The above plot was for 6 time points within a year, now I will look at 10 years' monthly data. 

**Time Series Data**

A Time Series Plot of Unemployment Rates by Race/ethnicity (2007 - 2017)
data are available in excel for various groups in bls website [here](https://www.bls.gov/webapps/legacy/cpsatab2.htm) 
I selected all & saved each excel file in my local drive. Then import into R.

- Time series data for the White group

``` r
WhiteAll_unemp <- as.data.frame(read_excel("yourPath/CPSwhiteAll.xlsx", skip=12)) #'skip' first 12 rows are notes...

I will assign original data frame to an new data frame.
WhiteAll_unempl <- WhiteAll_unemp[,-1]

& made the first column to be row names like:
rownames(WhiteAll_unempl) <- WhiteAll_unemp[,1]

create & convert to time series. 
You will appreciate this step if you look at the data before and after the following code is executed. 
WhiteAll_unemp2 <- ts(as.vector(t(as.matrix(WhiteAll_unempl))), start = c(2007, 1), end=c(2017, 12), frequency=12)
```

The above four lines of code are applied to the tables for Black, Hispanic & Asian groups below.

- Time series data for the Black/African American group

``` r
BlackAll_unemp <- as.data.frame(read_excel("yourPath/CPSblackAll.xlsx", skip=12))
BlackAll_unempl <- BlackAll_unemp[,-1]
rownames(BlackAll_unempl) <- BlackAll_unemp[,1]
BlackAll_unemp2 <- ts(as.vector(t(as.matrix(BlackAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

- Time series data for the Hispanic/Latino group

``` r
HispanicAll_unemp <- as.data.frame(read_excel("yourPath/CPShispanicAll.xlsx", skip=12))
HispanicAll_unempl <- HispanicAll_unemp[,-1]
rownames(HispanicAll_unempl) <- HispanicAll_unemp[,1]
HispanicAll_unemp2 <- ts(as.vector(t(as.matrix(HispanicAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

- Time series for the Asian group

``` r
AsianAll_unemp <- as.data.frame(read_excel("yourPath/CPSasianAll.xlsx", skip=12))
AsianAll_unempl <- AsianAll_unemp[,-1]
rownames(AsianAll_unempl) <- AsianAll_unemp[,1]
AsianAll_unemp2 <- ts(as.vector(t(as.matrix(AsianAll_unempl))), start = c(2007, 1), end=c(2017, 12), frequency=12)
```

Plotting using the zoo library...
``` r
#Use the zoo library for simple quick plot.
#data are in long form, so column bind the 4 data frames.

plot.zoo(cbind(WhiteAll_unemp2, BlackAll_unemp2, HispanicAll_unemp2, AsianAll_unemp2), 
	plot.type = "single", xy.labels = TRUE,
    xlab="Year \n Data source: https://www.bls.gov/cps", ylab = "Unemployment_rate",
    col = c("red", "blue", "black", "green"), main = "Adjusted Seasonal Unemployment Rates by Race/Ethnicity\n (2007 - 2017)")
legend("topright", inset = c(0,0), y.intersp = 1, legend = c("White", "Black", "Hispanic/Latino", "Asian"),
       lty=1, bty = "n", col = c("red", "blue", "black", "green"), cex = 1)
axis(1, 2007:2017)
#axis(2, 3:16)
grid (NULL,NULL, lty = 6, col = "cornsilk3")
abline(h=3:17, v=2007:2017, col="cornsilk2", lty=3)
```

<img src="/images/TimeSereisZoo.png"/>

Alternatively, you can use ggplot, but first we need few more steps to prepare the data.
``` r
#prepare data in a format ggplot2 likes
ts_All_race <- (cbind(WhiteAll_unemp2, BlackAll_unemp2, HispanicAll_unemp2, AsianAll_unemp2))

df_all_race <- fortify.zoo(ts_All_race, melt=TRUE, yearmon_trans(format="%b%Y", n=12))
df_all_race <- rename(df_all_race, Year=Index, unemployment_rate=Value, Race=Series)
```

Plot the time series (can reduce some of the specifications below)

``` r
ggplot(data=df_all_race, aes(x=Year, y=unemployment_rate, group=Race, shape=Race, color=Race)) + 
geom_line() + geom_point() +
scale_x_discrete(limit=c(2007:2017)) +
scale_y_discrete(limits=c(2:17)) +
  
scale_color_discrete(breaks=c("WhiteAll_unemp2", "BlackAll_unemp2", "HispanicAll_unemp2", "AsianAll_unemp2"), labels=c("White", "Black", "Latino", "Asian")) +
  
scale_shape_discrete(breaks=c("WhiteAll_unemp2", "BlackAll_unemp2", "HispanicAll_unemp2", "AsianAll_unemp2"), labels=c("White", "Black", "Latino", "Asian")) +
  
ggtitle("Adjusted Seasonal Unemployment Rates by Race/Ethnicity (for 16 & older)\n(2007 - 2017)") + 
  xlab("Year \n 
Date source: https://www.bls.gov/webapps/legacy/cpsatab2.htm 
                      https://www.bls.gov/webapps/legacy/cpsatab3.htm") +

theme(axis.line = element_line(color="blue", linetype = "solid"), plot.title = element_text(hjust = .5), legend.title = element_blank())
```

<img src="/images/TimeSeriesByRace.png"/>

End.

