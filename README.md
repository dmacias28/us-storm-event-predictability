## ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Capstone

<h1 align="center">U.S. Storm Event Predictability</h1>

## Purpose

The purpose of this project will be to determine the predictability of recent storm events in the U.S. through historical data.

---

## Data

### Original Data

The original data was obtained through the [National Climatic Data Center](https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/) website. Their U.S. storm event data goes back to 1950 and, conveniently, there was a CSV file available for each year. However, given that there were 72 years worth of data, and that the later files, in particular, were quite large, they were webscraped and compiled into one single DataFrame.

That DataFrame contained 51 features and 1,740,597 observations, which spanned 70 different storm event types.

### Cleaned Data

The DataFrame required quite a bit of cleaning as there were a number of columns that either did not provide value to the project, such as IDs, codes and coordinates, or contained repeated information. There was also a region column added that allowed for each observation to be categorized under its appropriate U.S. region using the state column.

Additionally, there were columns whose value required substantial clean up, such as:
- *begin_date_time & end_date_time:*
    - These columns were object data types and needed to be converted to datetime data types. With a time series model in mind, the begin_date_time column would eventually become the index, and would need to be sorted from the earliest date to the latest date. However, only the last 2 numbers of the year were listed for each event, making it difficult for the datetime conversion function to distinguish between years that belonged to the 1900s and the 2000s. As such, the first 2 numbers of the year needed to be added manually for proper century assignment.
- *damage_crops & damage_property:*
    - These 2 columns were object data types but needed to be converted to integer data types. Their values were entered in shorthand and therefore, a mix of integer and letter characters, and they needed to be the full dollar amount values in damages. For example, 5K needed to be converted to 5,000 and 12.95M to 12,950,000.

In the end, the resulting DataFrame was reduced to containing 17 features. No observations were removed due to the vast amount of event types. It was noted throughout the cleaning process that certain columns only pertained to certain event types, and it was too early to drop observations based on missing values. That would come later in the EDA process when specific events would be looked into.

### Data Dictionary

|Feature|Type|Description|
|---|---|---|
|**state**|*object*|The state name where the event occurred.|
|**region**|*integer*|The region name where the event occurred.|
|**year**|*integer*|The four digit year for the event.|
|**event_type**|*object*|Ex: Hail, Thunderstorm Wind, Tornado, etc.|
|**begin_date_time**|*datetime*|Begin date and time for the event. Format: MM/DD/YYYY hh:mm:ss (24 hour time usually in LST).|
|**end_date_time**|*datetime*|End date and time for the event. Format: MM/DD/YYYY hh:mm:ss (24 hour time usually in LST).|
|**injuries_direct**|*integer*|The number of injuries directly caused by the weather event.|
|**injuries_indirect**|*integer*|The number of injuries indirectly caused by the weather event.|
|**deaths_direct**|*integer*|The number of deaths directly caused by the weather event.|
|**deaths_indirect**|*integer*|The number of deaths indirectly caused by the weather event.| 
|**damage_property**|*integer*|The estimated amount of damage to property incurred by the weather event.|
|**damage_crops**|*integer*|The estimated amount of damage to crops incurred by the weather event.|
|**magnitude**|*float*|The measured extent of the magnitude type ~ only used for wind speeds (in knots) and hail size (in inches to the hundredth).|
|**magnitude_type**|*object*|EG = Wind Estimated Gust; ES = Estimated Sustained Wind; MS = Measured Sustained Wind; MG = Measured Wind Gust (no magnitude is included for instances of hail).|
|**tor_f_scale**|*object*|EF0 – Light Damage (40 – 72 mph); EF1 – Moderate Damage (73 – 112 mph); EF2 – Significant damage (113 – 157 mph); EF3 – Severe Damage (158 – 206 mph); EF4 – Devastating Damage (207 – 260 mph); EF5 – Incredible Damage (261 – 318 mph)|
|**tor_length**|*float*|Length of the tornado or tornado segment while on the ground (in miles to the tenth).|
|**tor_width**|*float*|Width of the tornado or tornado segment while on the ground (in feet).|

---

## Exploratory Data Analysis

Having worked with the features enough to get a sense of which would be best to predict, I decided on magnitude, so the focus of the EDA became to identify storm events that hold the most promise for a reliable forecast.

### High-Level Exploration

#### Count of reported events per year (1950-2022)

Of the 1,740,596 events, the vast majority occur after 1995. Between 1950 and 1995, there was a steady increase up toward 20,000 events, but in 1996, there was a sudden spike with over 50,000 events recorded. From there, events increased toward 60,000, though 2008 and 2011 surpassed that and reached 70,000 and 80,000, respectively.

According to the National Climatic Data Center, the storm events database was moved online in 1996. They've also stated that the number of events dramatically increased due to advances in technology and population density in previously sparse areas, so there's some insight into the sudden spike.

#### Count of reported events per event type (1950-2022)

There are 70 event types recorded in the dataset, but it's the thunderstorm wind and hail events that account for half of the data. They make up 28% and 22% of the data, respectively.

This makes sense as thunderstorm wind and hail events were among the first to be recorded by the Storm Prediction Center in 1955. They're also more common than many of the other events. Most of the other events are more severe and require special conditions in order to occur, so aside from the fact their recordings didn't begin until 1993 under the National Climatic Data Center, it makes sense that they have far less occurrences.

#### Count of reported events by state (1950-2022)

When counting events by state, it was discovered that the state column had more than just states recorded. U.S. territories and bodies of water are also included. Among the states, Texas has the highest event count with nearly 140,000 events recorded and Kansas has the second-highest with about 80,000. Most states are in the 20,000-60,000 range and those below 20,000 are either smaller states or have a sparse population density. As for the territories and bodies of water, none of them exceed 20,000 events.

#### Count of reported events by region (1950-2022)

Given that the count of storm events vary greatly between the states, I moved to the regional level to see what things looked like there. The South has the highest event count with nearly 700,000 events recorded. In second is the Midwest with close to 600,000, followed by the West with about 200,000 and lastly the Northeast with just under 200,000.

### Event types for which magnitude is tracked

According to the original data dictionary provided by the National Climatic Data Center, magnitude is only used for wind speeds (knots) and hail size (inches to the hundredth). Given that not all event types track magnitude, I needed to see what events I could work with.

There were 39 events that track, but tornado, hail and thunderstorm wind events were the standouts. Because they've all been tracked since the 1950s, I'll have a better chance at long-term trends here than with any other events. Hail and thunderstorm wind also have the highest event counts, so it was reassuring that there would be a good amount of data to work with for those events.

### Event types by total event counts and casualties (injuries, deaths and damage)

The purpose of this was to get a closer look at the sum of the direct and indirect injuries, direct and indirect deaths, and damage to property and crops by event type. While the intention wasn't to consider these as potential additional variables, looking into this data helped further narrow down the events by seeing which were top threats in terms of casualties.

For a more complete look at the casualty sums, I combined the injury columns, the death columns and the damage columns to. The observations were as follows:

- *All Events:* As seen in the 'Count of reported events per event type' plot, thunderstorm wind and hail events have the highest counts with 484K and 382K events recorded, respectively. The event to follow is flash flood with 94K.

- *All Injuries:* Tornados have the highest count of injuries, both direct and indirect, with 97K. The event to follow is thunderstorm wind with 11K.

- *All Deaths:* Tornados again have the highest count of deaths, both direct and indirect, with 12K. The event to follow is heat with 5K.

- *All Damage:* The events that seem to cause the most damage are water events such as floods, hurricanes, and tides. Among the mix are also tornados.

Thunderstorm wind and tornado events are the most recurring in each of the sortings. This confirmed that these are 2 events worth continuing to explore.

### Exploration by Event Type

By focusing on 1.) events for which magnitude is tracked, 2.) events with the highest event counts, and 3.) events with the highest casualties, I narrowed it down to 4 events that seemed worth continuing to explore. They were thunderstorm winds, tornados, hail and high winds. The focus here became to uncover magnitude trends and seeing how qualitative the data for each of the events actually was.

#### Thunderstorm Wind

There were 484,908 thunderstorm wind observations to start, but after removing the observations that had missing magnitude values, the total dropped to 453,604.

##### Count of reported thunderstorm wind events per year (1955-2022)

Thunderstorm wind events are seen to have steadily increased over time. In the early years of the data, there were less than 1000 events per year, and in recent years, there have consistently been 15,000+ events per year. There are spikes observed in some years (1995, 2008, 2011, 2019 and 2020), but there's been a consistent upward trend for the most part.

##### Count of reported thunderstorm wind events by state (1955-2022)

The count of thunderstorm wind events vary greatly by state. Texas has the highest count with nearly 30,000, followed by Kansas with a little more than 20,000. About half of the rest of the states fall in the 10,000-20,000 range and the other half fall mostly under the 5,000 range.

##### Count of reported thunderstorm wind events by region (1955-2022)

At the regional level, the South has the highest event count with over 200,000 events recorded and accounts for nearly half of all thunderstorm wind events. The Midwest comes in second with over 150,000, followed by the Northeast with around 50,000 and lastly the West with about 25,000.

##### Distribution of thunderstorm wind event magnitudes (1955-2022)

The magnitude for thunderstorm wind is measured in knots. The distribution of magnitudes for this event ranges between 0 and 175 knots and is extremely right-skewed.

Most events fall in the 35-50 range with over 200,000 events, followed by the 50-70 range with about 130,000, then, the 0-15 range with around 80,000, and then, lastly the 70-85 range with about 20,000. Anything beyond an 85 can likely be considered an outlier.

##### Thunderstorm wind magnitude averages (1955-2022)

The national level of magnitude averages over time could not have been expected. The data is very chaotic with the averages fluctuating wildly between 0 and 30 knots from 1955 to the mid-90s. Then, there's a dramatic spike from one year to the next where the average surpass 50. From there, the averages become steadier but seem to have a slight downward trend.

The spike in the mid-90s aligns with the previous knowledge that the database moved online in 1996 and that the number of events dramatically increased due to advances in technology and population density in previously sparse areas.

Also, with the knowledge that conditions vary greatly in different parts of the country, the averages were also looked at on the regional level, which helped, but the same level of chaos remained prior to the spike in the mid-90s. Because that data would be harmful to the model, I decided to focus on the data after 1995.

At the national level for data after 1995, the chaos was eliminated but the downward trend initially previously detected became more evident. Breaking it down by region, the South was identified to be the likely cause of the downward trend at the national level given that it accounts for nearly half of all thunderstorm wind events and its own shape is similar to that of the national level shape.

##### Distribution of thunderstorm wind magnitude by region (1996-2022)

The distribution of thunderstorm wind magnitudes by region showed that there are a great number of outliers for each of the regions. The interquartile range (25th-75th percentile) for each of the regions is right above 50 knots. And while all regions have outliers, the Midwest and the South have outliers reaching as high as 175 knots.

##### Thunderstorm Wind Summary

The data for this event started off quite chaotic when analyzing at the national level and prior to 1996. Because that data is not representative of the current data, in order to move forward with this event, I'd need to focus on the data from 1996 and on.

The national trends also seem to differ quite a bit from the regional trends. Given that the South accounts for nearly half of all observations, it's the region that is driving the national trend.

Regardless, this data seems promising to work with and is a contender for moving forward.

#### Tornado

There were 73,987 tornado observations to start, but after removing the observations that had missing magnitude values, the total dropped to 37,635.

##### Count of reported tornado events per year (1950-2006)

In general, tornado events have increased over time, but it hasn't been steady. There's been a lot of fluctuation. However, what's interesting is that the number of events drops suddenly from 1200 events in 1995 to hardly an at all in 1996. The numbers don't pick back up at any point after that and the events actually stop altogether in 2006.

##### Tornado Summary

The fact that the data for this event stopped in 2006 was enough to disqualify it from moving forward. In order to forecast into the future, I need recent data, which this event does not have.

#### Hail

There were 382,234 hail observations to start, but after removing the observations that had missing magnitude values, the total dropped to 382,234.

##### Count of reported hail events per year (1955-2022)

Hail events have steadily increased over time until the 1990s. After that, they continue increasing until 2006 but at a more dramatic rate. From then on, there's more of a downward trend, though, there are some large spikes in 2008 and 2011 making them the years with the highest event counts within the entire hail dataset.

##### Count of reported hail events by state (1955-2022)

The count of hail events vary greatly by state, even more so than in any of the previous events analyzed so far. Though, this makes sense as hail is heavily dependent on freezing weather and not all states are subject to those conditions as often as others. What's interesting though, is that Texas still has the highest event count.

##### Count of reported hail events by region (1955-2022)

At the regional level, the Midwest, for the first time, suprasses the South's count of events, though the South is not far behind. The Midwest totals to a little under 170,000 events and the South is right around 160,000. Next is the West with around 40,000 events and the Northeast lags further behind with around 20,000 events.

##### Distribution of hail event magnitudes (1955-2022)

The magnitude for hail is measured in inches to the hundreth. The distribution of magnitudes for this event ranges between 0 and 10 inches to the hundreth and is extremely right-skewed.

Most events fall in the 1-2 range with over 200,000 events, followed by the 0-1 range with over 150,000, and then, the 2-3 range with nearly 25,000. Anything beyond a 3 is likely considered an outlier.

##### Hail magnitude averages (1955-2022)

The hail event magnitude averages start off high and fluctuate between 1.4 and 1.6 between 1955 and 1980. There's a downward trend toward 1.0 for the next 28 years until 2008 when the averages begin to rise until they hit 1.27 in 2021. In 2022, there's a steep drop back to 1.1.

The downward trend between the 1980s and early 2000s make sense, given the steady increase in hail events during that time. As does the upward trend after 2008 when the number of events begin decreasing.

Looking at the averages at a regional level didn't do much to further explain the ups and downs over time. All regions showed the same trend, though at different severities, with the Midwest and South experiencing some of the most severes hail events.

##### Distribution of hail magnitude by region (1996-2022)

The distribution of hail magnitudes by region showed that the interquartile range (25th-75th percentile) for each region is within the 0-2 range. The South interestingly had the largest IQR range, followed by the Midwest and the West, with comparable IQRs and the Northeast with the smallest IQR.

All regions have numerous outliers that extend beyond the maximum, but only the Northeast has outliers below the minimum. We can also see that the Midwest and the South are the regions to have experienced the most severe hail events.

##### Hail Summary

The data for this event seems promising. It's not as chaotic as the thunderstorm wind event data, but I'd have to stick to data after the year 2000 since the trends before then are not at all representative of the more recent trends. This would give me even less data to work with in comparison to the thunderstorm wind event.

#### High Wind

There were 79,894 high wind observations to start, but after removing the observations that had missing magnitude values, the total dropped to 74,834.

##### Count of reported high wind events per year (1996-2022)

The earliest records of high wind events are in 1996. There's heavy fluctuation and there isn't as much of a clear trend as there has been for the previous events, but there has been a general increase in events when comparing the number of events in recent years to those in the early years of this event.

##### Count of reported high wind events by state (1996-2022)

Surprisingly, for the first time, Texas is not the state with the most event counts. It's Wyoming with over 7,000 events. It's followed by Montana with about 6,000, then, California with about 5,000, and then, comes Texas with around 4,000. The rest of the states vary wildly between 0 and 4,000.

##### Count of reported high wind events by region (1996-2022)

And another first on the regional level, as well. It's not the South with the highest event count, but the West with over 30,000 events. It's followed by the Midwest with just under 200,000, then the South with around 10,000, and then the Northeast with around 7,500.

##### Distribution of high wind event magnitudes (1996-2022)

The magnitude for high wind is measured in knots. The distribution of magnitudes for this event ranges between 0 and 175 knots and is right-skewed.

Most events fall in the 35-50 range with nearly 40,000 events, followed by the 50-70 range with just over 30,000, and then the 70-85 range with just under 5,000. Anything beyond an 85 can likely be considered an outlier.

This distribution is actually very similar to the thunderstorm wind distribution, except there are hardly any events under the 25 range, whereas the thunderstorm wind distribution had a decent amount.

##### High wind magnitude averages (1996-2022)

Between 1995 and 2015, the high wind magnitudes show a downward trend while fluctuating between 51 and 55.5 knots. In 2016, there are beginnings of a steep increase up until 2021 when it surpasses an unprecedented 56 knots. In 2022, however, the averages begin to decline back toward 55 knots.

The fluctuations are interesting seeing as there are no major fluctuations in yearly event counts as there have with previous events. Any steep downward or upward trends were explained by large increases or decreases in event counts then.

Looking at the averages at a regional level helps explain the sudden national increase in high wind magnitude averages. While there was a slight increase between 2015 and 2022 for the Midwest, South and Northeast region, it's the West region that's driving the steep increase. This makes sense seeing as the West accounts for more than 1/3 of the events and it's the region to experience the highest magnitudes.

##### Distribution of high wind magnitude by region (1996-2022)

The distribution of high wind magnitudes by region shows that the interquartile range (25th-75th percentile) for each region is right around 50 knots. The West has the largest IQR, and the rest of the regions share smaller, more comparable IQRs. All regions have numerous outliers that extend beyond both the minimums and the maximums, but the West has the largest outliers reaching as many as 175 knots.

##### High Wind Summary

The data for this event does not seem as promising as the data for the thunderstorm wind and hail events. Though it was apparent from the start that it had substantially less observations than the thunderstorm wind and hail events, but it seemed worth looking into considering the data would be averaged. However, the data did not show any long-term consistent trends, which would not make it ideal data to forecast on.

### EDA Summary

To summarize, I explored 4 storm events: thunderstorm wind, tornado, hail and high wind. I narrowed it down to these events by focusing on the following:

- Events for which magnitude is tracked
- Event with the highest event counts
- Events with the highest casualties

As I explored these 4 events, I found that the tornado and high wind events would not make for good data to run a time series model on. The tornado event did not have any data beyond 2006 and the high wind event had far too few observations that did not allow for any decent trends. That narrowed it down to thunderstorm wind and hail, which I've both decided to move forward with.

I'd also like to note that although outliers are usually areas for concern and should be dealt with, I will not be removing them. They do not seem to be due to error and removing them would just suppress the naturally high variability of the data. It would reduce my error metrics and make my predictions look more accurate, but that doesn't necessarily mean they'll be more accurate when applied to the real world.

---
## Time Series Modeling

I'll be working with the event data on the quarterly level instead of the annual level, for the purpose of increasing the amount of observations and for more accurate forecasts, since for thunderstorm wind, I'm working with 26 years of data, and for hail, I'm working with 22 years of data. 

Since I'm only working with one variable, the ARIMA model will be used to determine the predictability of these events. Each event will consist of 5 models, one for each of the regions, and then, one on the national level.

For each model, the data was made stationary, that is without trends or seasonality, as is required by the ARIMA model. The optimal p, d, and q parameters were determined for each, and the models were trained on 90% of the data.

### Thunderstorm Wind

Below are the findings for the each of the 5 thunderstorm wind models:

- National
    - The ARIMA (2,1,4) model explained -2% of the variability
    - Did not produce a reliable forecast, according to the RMSE (1.80) and standard deviation (0.98) comparison
- Midwest
    - The ARIMA (2,0,2) model explained -4% of the variability
    - Did not produce a reliable forecast, according to the RMSE (1.49) and standard deviation (1.32) comparison
- South
    - The ARIMA (4,2,4) model explained 10% of the variability
    - Did not produce a reliable forecast, according to the RMSE (2.37) and standard deviation (2.12) comparison
- Northeast
    - The ARIMA (0,1,3) model explained 4% of the variability
    - Produced a reliable forecast, according to the RMSE (1.28) and standard deviation (1.52) comparison
- West
    - The ARIMA (2,0,2) model explained -30% of the variability
    - Produced a reliable forecast, according to the RMSE (2.16) and standard deviation (2.66) comparison

The national model indicated that the levels of variability are too high to explain, and was unable to produce a reliable forecast. The regions individually did not fare much better, as none of the models explained more than 10%. The Northeast and West regions were the only models to produce a reliable forecast according to the evaluation metrics, but they were not enough to influence the national model, as the Midwest and the South regions account for more than 80% of the observations.

---

### Hail

Below are the findings for the each of the 5 hail models:

- National
    - The ARIMA (4,2,4) model was able to explain 12% of the variability
    - Produced a reliable forecast, according to the RMSE (0.17) and standard deviation (0.12) comparison
- Midwest
    - The ARIMA (2,1,4) model was able to explain 55% of the variability
    - Produced a reliable forecast, according to the RMSE (0.09) and standard deviation (0.12) comparison
- South
    - The ARIMA (3,1,4) model was able to explain 31% of the variability
    - Did not produce a reliable forecast, according to the RMSE (0.118) and standard deviation (0.114) comparison
- Northeast
    - The ARIMA (0,1,2) model was able to explain 21% of the variability
    - Did not produce a reliable forecast, according to the RMSE (0.16) and standard deviation (0.13) comparison
- West
    - The ARIMA (3,1,1) model was able to explain 58% of the variability
    - Did not produce a reliable forecast, according to the RMSE (0.33) and standard deviation (0.30) comparison

Though the hail event fared better than the thunderstorm wind event overall, the levels of variability remain too high for the models to explain. The national model was able to produce a reliable forecast, but was only able to explain 12% of the variability. Of the regions, the Midwest region was the only one to produce a reliable forecast according to the evaluation metrics. Given that it accounts for more than 40% of the data, it was likely enough to influence the national model into producing a reliable forecast.

---

## Conclusion

By modeling on the 2 most promising storm events, as determined by the exploratory data analysis, we can see that storm events are not very predictable on the regional level, and much less at the national level. While removing outliers would've allowed the models to explain the variability better, and therefore, produce more "reliable" forecasts, I'd be suppressing the naturally high variability of the data, and would be creating biased models that wouldn't necessarily translate into the real world.

### Next Steps

As I continue to explore this data, I'll consider changing the timeframe of the averages from quarterly to monthly, introducing exogenous variables, and exploring other models such as SARIMA, since seasonality trends will likely be present with monthly data. And considering that I didn't end up using any of the data prior to 1996, this opens up the door to exploring additional events that were disqualified simply because they lacked historical data.