I will first setup the environment and load some of the packages that will be used.

The good think in using R Markdown is creating code in chunks for clarity and ease of debugging. \[Table 2\] ("<https://www.bls.gov/news.release/empsit.t02.htm>") of the Economic News Release contains general labor force data including employment/unemployment rates.

``` r
#This code will extract the data.
EconNews <- as.data.frame(readHTMLTable(getURL("https://www.bls.gov/news.release/empsit.t02.htm"),
               stringsAsFactors=FALSE, trim = TRUE))

#see what the first few lines of the data look like. We also need to assign appropriate columns and rows later.
head(EconNews[,c(1,5:10)])
```

    ##                      cps_empsit_a02.V1 cps_empsit_a02.V5 cps_empsit_a02.V6
    ## 1                                WHITE              <NA>              <NA>
    ## 2 Civilian noninstitutional population           198,380           198,685
    ## 3                 Civilian labor force           124,736           124,925
    ## 4                   Participation rate              62.9              62.9
    ## 5                             Employed           119,269           120,142
    ## 6          Employment-population ratio              60.1              60.5
    ##   cps_empsit_a02.V7 cps_empsit_a02.V8 cps_empsit_a02.V9 cps_empsit_a02.V10
    ## 1              <NA>              <NA>              <NA>               <NA>
    ## 2           198,775           198,872           198,974            199,082
    ## 3           124,481           124,890           124,968            125,037
    ## 4              62.6              62.8              62.8               62.8
    ## 5           119,896           120,091           120,262            120,209
    ## 6              60.3              60.4              60.4               60.4

``` r
#"," needs to be removed so we can coveniently convert from one form to another form of numbers/factors etc.

EconNews[] <- lapply(EconNews, gsub, pattern=',', replacement='')


#Now I will just make columns for the table and assign those to columns 2-10 (the first column is variable names)


colName <- c("Aug2016", "Jul2017", "Aug2017", "Aug2016a",
              "Apr2017a", "May2017a", "Jun2017a", "Jul2017a", "Aug2017a")

names(EconNews)[2:10] <- colName
```

I will first setup the environment and loaded some of the packages that will be used. The good think in using R Markdown is creating code in chunks for clarity and ease of debugging. Take a look at [Table 2](https://www.bls.gov/news.release/empsit.t02.htm) & [Table 3](https://www.bls.gov/news.release/empsit.t03.htm) of the *Economic News Release* data that I used for this section.

``` r
EconNewsLat <- as.data.frame(readHTMLTable(getURL("https://www.bls.gov/news.release/empsit.t03.htm"),
               stringsAsFactors=FALSE, trim = TRUE))

EconNewsLat[] <- lapply(EconNewsLat, gsub, pattern=',', replacement='')

names(EconNewsLat)[2:10] <- colName

#look at first few lines
head(EconNewsLat[,c(1,5:10)])
```

    ##                      cps_empsit_a03.V1 Aug2016a Apr2017a May2017a Jun2017a
    ## 1         HISPANIC OR LATINO ETHNICITY     <NA>     <NA>     <NA>     <NA>
    ## 2 Civilian noninstitutional population    40825    41162    41241    41323
    ## 3                 Civilian labor force    26988    27241    27239    27290
    ## 4                   Participation rate     66.1     66.2     66.0     66.0
    ## 5                             Employed    25460    25832    25833    25974
    ## 6          Employment-population ratio     62.4     62.8     62.6     62.9
    ##   Jul2017a Aug2017a
    ## 1     <NA>     <NA>
    ## 2    41404    41492
    ## 3    27487    27322
    ## 4     66.4     65.8
    ## 5    26078    25914
    ## 6     63.0     62.5

Names of the first rows for "EconNews" & "EcoNewsLat" are different, so we can't do row bind when column names are different.
=============================================================================================================================

Remove different names of first column or rename them with the same column name. I will remove them.
====================================================================================================

``` r
colnames(EconNewsLat)[1] <- ""
colnames(EconNews)[1] <- ""
```

row\_bind the two tables, using the dplyr package and call the giant table "EconNewsAll"

``` r
EconNewsAll <- dplyr::bind_rows(EconNews,EconNewsLat)

#see few first lines
head(EconNewsAll[,c(1,5:10)])
```

    ##                                        Aug2016a Apr2017a May2017a Jun2017a
    ## 1                                WHITE     <NA>     <NA>     <NA>     <NA>
    ## 2 Civilian noninstitutional population   198380   198685   198775   198872
    ## 3                 Civilian labor force   124736   124925   124481   124890
    ## 4                   Participation rate     62.9     62.9     62.6     62.8
    ## 5                             Employed   119269   120142   119896   120091
    ## 6          Employment-population ratio     60.1     60.5     60.3     60.4
    ##   Jul2017a Aug2017a
    ## 1     <NA>     <NA>
    ## 2   198974   199082
    ## 3   124968   125037
    ## 4     62.8     62.8
    ## 5   120262   120209
    ## 6     60.4     60.4

Now I can subset data by race, gender and age. I also need to rename rows.

``` r
#grab sub-table for all White group as "W1data"
W1data <- EconNewsAll[2:9,2:10] 

#Rename rows
rownames(W1data)<- c("Civil_noninst_pop","civil_labor_force", "Particip_rate", "Employed", 
                      "Emp_pop_ratio", "Unemployed","Unemp_rate", "Not_in_labor_force")

#transpose table for analysis & plotting

W1data <- as.data.frame(t(W1data))
```

Add indicator variables for "Race", "Quarters" or "time", "Class" Quarters (Qtrs) is months data was released, Classes indicate age & gender status in case we want to look at plots by age, gender and race...

``` r
#add indicator variables for "Race", "Quarters" or "time", "Class"
#Quarters is months data was released, Classes indicate age & gender status
#in case we want to look at plots by age, gender and race...

#again with the dplyr package
W1data <- W1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(1,9))),
           Class = as.factor(c(rep(1,9))))

#convert variables to numeric
W1data[,1:8] <- lapply(W1data[,1:8], function(x) as.numeric(as.character(x)))
```

All other data manipulation activities below are copies of the above for specific race/ethnic/gender categories, so I will not elaborate further. Only things I need to change are data frame names, appropriate rows, & classes. You can jump to the code for the plots if you like.

``` r
#grab sub-table for All Blacks
B1data <- EconNewsAll[33:40,2:10] 

rownames(B1data)<- c("Civil_noninst_pop","civil_labor_force", "Particip_rate", "Employed", 
                      "Emp_pop_ratio", "Unemployed","Unemp_rate", "Not_in_labor_force")

B1data <- as.data.frame(t(B1data))

B1data <- B1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(2,9))),
           Class = as.factor(c(rep(5,9))))

B1data[,1:8] <- lapply(B1data[,1:8], function(x) as.numeric(as.character(x)))
```

``` r
#grab sub-table for All Asians (Note: Asians were not categorized by gender and age)
A1data <- EconNewsAll[64:71,2:10]

rownames(A1data)<- c("Civil_noninst_pop","civil_labor_force","Particip_rate",
                      "Employed","Emp_pop_ratio","Unemployed","Unemp_rate","Not_in_labor_force")

A1data <- as.data.frame(t(A1data))

A1data <- A1data %>% 
    mutate(Qtrs = as.numeric(c(seq(1:9))),
           Race = as.factor(c(rep(3,9))),
           Class = as.factor(c(rep(9,9))))

A1data[,1:8] <- lapply(A1data[,1:8], function(x) as.numeric(as.character(x)))
```

``` r
#grab sub-table for All Latino/Hispanic
L1data <- EconNewsAll[73:80,2:10]

rownames(L1data)<- c("Civil_noninst_pop","civil_labor_force","Particip_rate",
                      "Employed","Emp_pop_ratio","Unemployed","Unemp_rate","Not_in_labor_force")

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

    ##   Civil_noninst_pop civil_labor_force Particip_rate Employed Emp_pop_ratio
    ## 1            198380            124998          63.0   119477          60.2
    ## 2            198974            126046          63.3   121029          60.8
    ## 3            199082            125280          62.9   120365          60.5
    ## 4            198380            124736          62.9   119269          60.1
    ## 5            198685            124925          62.9   120142          60.5
    ## 6            198775            124481          62.6   119896          60.3
    ##   Unemployed Unemp_rate Not_in_labor_force Qtrs Race Class
    ## 1       5521        4.4              73382    1    1     1
    ## 2       5017        4.0              72928    2    1     1
    ## 3       4915        3.9              73802    3    1     1
    ## 4       5466        4.4              73644    4    1     1
    ## 5       4783        3.8              73760    5    1     1
    ## 6       4585        3.7              74294    6    1     1

Time to plot our data! You can now start guessing what unemployment trends will look like for Americans of different race/ethnic/gender identities.

Seasonally adjusted unemployment rates by Race/Ethnicity

``` r
#Plot for Race/ethnic groups using ggplot

All_plot <- ggplot(data=WBAL1data[c(4:9,13:18,22:27,31:36),], #restrict to these rows.
                  aes(x=Qtrs, y=Unemp_rate, shape=Race, color=Race, group=Race )) + 
  geom_point() + 
  geom_line() + 
  scale_x_discrete(limits =c(4:9),
                   labels=c("Aug 2016", "Apr 2017", "May 2017", "Jun 2017", "Jul 2017", "Aug 2017"))+ 
  scale_color_discrete(breaks=c(1:4), labels=c("All White", "Black", "Asian", "Latino/Hispanic")) +
  scale_shape_discrete(breaks=c(1:4), labels=c("All White", "Black", "Asian", "Latino/Hispanic")) +
  theme(axis.line = element_line(color="blue", linetype = "solid"), plot.title = element_text(hjust = .5)) +  
  ggtitle("Adjusted Unemployment Rates by Race\n (Aug 2016 - Aug 2017)") +
  theme(legend.title = element_blank(), legend.text = element_text(size = 12), 
        legend.background = element_rect(fill="NA", size=.5, linetype = "solid"))
All_plot
```

![](LaborForceData_I_files/figure-markdown_github/All_plot-1.png)

The above plot was for 6 time points within a year, now I will look at 10 years' monthly data. A Time Series Plot of Unemployment Rates by Race/ethnicity (2007 - 2017)

data are available in excel for various groups in bls website [here](https://www.bls.gov/webapps/legacy/cpsatab2.htm) I selected all & saved each excel file in my local drive. Then import into R.

``` r
WhiteAll_unemp <- as.data.frame(read_excel("C:/Users/Aman/Desktop/Scrapping/CPSwhiteAll.xlsx", skip=12)) #'skip' first 12 rows are notes...

#Let's assign original data frame to an new data frame.
WhiteAll_unempl <- WhiteAll_unemp[,-1]

#& made the firt column to be row names like so:
rownames(WhiteAll_unempl) <- WhiteAll_unemp[,1]

# create & convert to time series. 
#You will appreciate this step if you look at the data before and after the following code is executed. 
WhiteAll_unemp2 <- ts(as.vector(t(as.matrix(WhiteAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

The above four lines of code are applied to the tables for Black, Hispanic & Asian groups below.

Black group

``` r
BlackAll_unemp <- as.data.frame(read_excel("C:/Users/Aman/Desktop/Scrapping/CPSblackAll.xlsx", skip=12))
BlackAll_unempl <- BlackAll_unemp[,-1]
rownames(BlackAll_unempl) <- BlackAll_unemp[,1]
BlackAll_unemp2 <- ts(as.vector(t(as.matrix(BlackAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

Hispanic group

``` r
HispanicAll_unemp <- as.data.frame(read_excel("C:/Users/Aman/Desktop/Scrapping/CPShispanicAll.xlsx", skip=12))
HispanicAll_unempl <- HispanicAll_unemp[,-1]
rownames(HispanicAll_unempl) <- HispanicAll_unemp[,1]
HispanicAll_unemp2 <- ts(as.vector(t(as.matrix(HispanicAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

Asian group

``` r
AsianAll_unemp <- as.data.frame(read_excel("C:/Users/Aman/Desktop/Scrapping/CPSasianAll.xlsx", skip=12))
AsianAll_unempl <- AsianAll_unemp[,-1]
rownames(AsianAll_unempl) <- AsianAll_unemp[,1]
AsianAll_unemp2 <- ts(as.vector(t(as.matrix(AsianAll_unempl))), 
                      start = c(2007, 1), end=c(2017, 12), frequency=12)
```

``` r
#Use the zoo library for simple quick plot.
plot.zoo(cbind(WhiteAll_unemp2, BlackAll_unemp2, HispanicAll_unemp2, AsianAll_unemp2), #data are in long form, so column bind the 4 data frames.
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

![](LaborForceData_I_files/figure-markdown_github/unnamed-chunk-14-1.png)

Alternatively, you can use ggplot, but first we need few more steps to prepare the data.

``` r
#prepare data in a format ggplot2 likes
ts_All_race <- (cbind(WhiteAll_unemp2, BlackAll_unemp2, HispanicAll_unemp2, AsianAll_unemp2))

df_all_race <- fortify.zoo(ts_All_race, melt=TRUE, yearmon_trans(format="%b%Y", n=12))
df_all_race <- rename(df_all_race, Year=Index, unemployment_rate=Value, Race=Series)

#Plot the time series (can reduce some of the specifications below)
ggplot(data=df_all_race, aes(x=Year, y=unemployment_rate, group=Race, shape=Race, color=Race)) + 
  geom_line() + geom_point() +
  scale_x_discrete(limit=c(2007:2017)) +
  scale_y_discrete(limits=c(2:17)) +
  scale_color_discrete(breaks=c("WhiteAll_unemp2", "BlackAll_unemp2", "HispanicAll_unemp2", "AsianAll_unemp2"), 
                       labels=c("White", "Black", "Latino", "Asian")) +
  scale_shape_discrete(breaks=c("WhiteAll_unemp2", "BlackAll_unemp2",  "HispanicAll_unemp2", "AsianAll_unemp2"), 
                       labels=c("White", "Black", "Latino", "Asian")) +
  ggtitle("Adjusted Seasonal Unemployment Rates by Race/Ethnicity (for 16 & older)\n(2007 - 2017)") + 
  xlab("Year \n 
Date source: https://www.bls.gov/webapps/legacy/cpsatab2.htm 
                      https://www.bls.gov/webapps/legacy/cpsatab3.htm") +
  theme(axis.line = element_line(color="blue", linetype = "solid"), 
        plot.title = element_text(hjust = .5), legend.title = element_blank())
```

![](LaborForceData_I_files/figure-markdown_github/unnamed-chunk-15-1.png)
