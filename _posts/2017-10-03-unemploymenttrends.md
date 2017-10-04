---
layout: post
title:  "Unemployment Rates Over Time"
date:   2017-10-03
excerpt_separator: <!--more-->
categories: Unemployment
---

I was curious to see seasonal unemployment rates by race/ethnicity and gender over time.  
Another reason for the next few posts is to show how to scrape html data from the web (specifically Bureaue of Labor Statistics (BLS) website), prepare and plot the data by important grouping variables. The time series plot data was obtained from [here](https://www.bls.gov/webapps/legacy/cpsatab2.htm) -- I selected the seasonally adjusted and saved in my local drive. All data wrangling and plotting was done in R.

In the second post, I will look at unemployment rates by race/ethnicity and gender over six time points (Aug 2016 - Aug 2017) as provided in the BLS website [here](https://www.bls.gov/news.release/empsit.t02.htm) and [here](https://www.bls.gov/news.release/empsit.t03.htm) 
for Hispanic/Latino. In the third post, I will provide a quick look at R codes that created all these plots.

### Seasonal unemployment rates by Race/Ethnicity (Aug 2016 - Aug 2017)

<img src="/images/new%20plot-1.png"/>

##### What would 10 year monthly unemployment rates look like? ... see plot at the end of the post!
<!--more-->

The plot above shows seasonal unemployment rates by race/ethnicity at six time points during the last year. Note that the 'All White' 
category includes non-Caucasian Whites that maybe of Hispanic/Latino origin. It is not straight forward to remove the Hispanic/Latino group from the 'All White' category and for demonstration purposes I left the groups as the are. 

Unequal distribution of wealth, burden of diseases and mortality has been observed for generations. 
There are several sources for these differences. Difference in occupational status is one of the sources.

I first became interested in disparities research after taking a quantitative health research class where I 
read about differences in how people from different race/ethnic backgrounds interacted with the healthcare system.

In my dissertation, I looked at the direct and indirect relationships of socioeconomic status (SES) factors to 
health status and healthcare services utilization. Until I looked at the literature and reports, I did not realize 
the great wealth disparities by race/ethnicity. 

So, I was curious about employment/unemployment rates over time given official governemnt data. Obviously, employment, 
education, income, wealth and health are related but this relationship maybe different for different groups. 

From the plot above it would seem that Asians have lower unemployment rates than Caucasian Whites but once the Latino/Hispanic group is removed from the "All White" group, unemployment rates among Caucasian Whites is likely to be lower than observed in the plot. What is interesting in this plot is that unemployment rates among Hispanic/Latino groups is much lower than unemployment rates among Black/African Americans. 

## A Time Series Plot of seasonal unemployment rates by Race/Ethnicity (2007 - 2017) 
#### What would a plot of unemployment rates for 10 years look like for White, Black and Asian groups?

Unemployment rates for Black/African American groups is almost always two times higher than that of Asians and Whites. 
You can also observe that month to month changes in unemployment are higher in the Asian and Black groups. Why?

<img src="/images/unnamed-chunk-7-1.png"/>

Despite the wide perception (& actual gains in many aspects of life for minorities) about improved experience regarding 
better employment, wealth accumulation and health outcomes, published indicators show consistent 
lag for minorities in general but for Blacks/African Americans in particular. Monthly unemployment rate trends over the last 10 years (as provided in the plot above) show similar rize and decline for each race/ethnic group. What will the plot for the next 10 years look like?
