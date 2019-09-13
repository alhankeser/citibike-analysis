
# Extracting and Transforming Citi Bike Data for Analysis 
## How is Citi Bike availability affected by various factors like time of day and weather?

### Who:
I am [Alhan Keser](https://blog.alhan.co/), a [10+ year specialist in Web Experimentation](https://www.linkedin.com/in/alhankeser/) (aka A/B Testing, Conversion Optimization), on my way to a Master's in Data Science.

### What:
This is an original analysis of Citi Bike station data from May-June 2019 to find out what affect the day of week, time of day, and weather (temperature, precipitation, etc...) have on the availability of bikes at station-,  neighborhood-, and borough-levels. 

### Why:
- I wanted to push myself to extract and transform my own data. Skipping the entire ETL process and going straight into analysis is a luxury: it does not reflect reality. 
- Doing a time-series analysis is something that I wanted practice with. 
- I commute by bike every day (despite weather) so I have first-hand evidence that Citi Bike riders tend to shy away from biking in inclement weather. It will be interesting to visualize the differences here.   

### How:
- **Combined original data sources:**
    - [Citi Bike Live Station Status](https://feeds.citibikenyc.com/stations/stations.json)
    - [Dark Sky Weather API](https://darksky.net/dev/docs)
    - [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/intro)
- **Created cron jobs** to collect Citi Bike station statuses for all ~858 station, every 3 minutes, for ~2 months.
    - Total rows in final table: 5,800,274
    - "Why stop after 2 months," you ask? Because my server ran out of space while I was on vacation. Oops! 
- **Created a mini-ETL process** to transform data into the final output used below. 
    - Along the way, there were many errors, some of which I will resolve here.

### Table of Contents
- [Packages](#Packages)
- [Extracting](#Extracting)
    - [Stations](#Stations)
    - [Geocoding](#Geocoding)
    - [Weather](#Weather)
    - [Cron Jobs](#Cron-Jobs)
- [Transforming](#Transforming)
    - [Availability by Station](#Availability-by-Station)
    - [Predicted vs Observed Weather](#Predicted-vs-Observed-Weather)

### Packages
Importing a few packages that will help with describing, cleaning and visualizing things. 


```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import random
import warnings
warnings.filterwarnings('ignore')
```

### Extracting
I started by find an interesting data source. In this case, I found the [Citi Bike Station Feed](https://feeds.citibikenyc.com/stations/stations.json) via the [NYC Open Data site](https://opendata.cityofnewyork.us/).

The feed shows the latest statuses of ~858 Citi Bike stations. Below is a list of values per station and sample data for each. Any keys left blank are often blank in the data source as well, which I'll address in later steps. 

| key | sample value |
|:------------|:---------:|
| `id`        | 285|
| `stationName` |"Broadway & E 14St"|
| `availableDocks` |20|
| `totalDocks` |53|
| `latitude`|40.73454567|
| `longitude`   |-73.99074142|
| `statusValue` |"In Service"|
| `statusKey`   |1|
| `availableBikes` |31|
| `stAddress1`  |"Broadway & E 14 St"|
| `stAddress2`  |""|
| `city`        |""|
| `postalCode`  |""|
| `location`    |""|
| `altitude`    |""|
| `testStation` |false|
| `lastCommunicationTime` |"2019-09-12 08:38:21 PM"|
| `landMark`    |""|

#### Stations

To capture the raw station data without any transformations, I created a simple table that stored the following: 

|column_name|data_type|
|-----------|---------|
|created_at|timestamp|
|data|json|

Once the table created, I needed a way to collect data. A quick solution -- for me -- was to create a [Laravel](https://laravel.com/) application that [makes it easy to run console commands](https://laravel.com/docs/5.8/artisan#writing-commands). In combination with [Laravel Forge](https://forge.laravel.com), it's easy to set up cron jobs that trigger the necessary commands at set intervals.

**Resources**:
- [See the Laravel application I created to query station data](https://github.com/alhankeser/citibike-tracker/). 
- [See the request that extracts data for the base table](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L49)

Once the commands created, I set up a cron job that ran once every 3 minutes. This resulted in the collection of 41,325 rows. Below I've done a quick analysis of the base table:


```python
stations_raw = pd.read_csv("../input/stations_raw.csv") # 16.49 GB output from base SQL table
```


```python
print('---Sample Row---')
print(stations_raw.iloc[random.randint(0,len(stations_raw))])
print('\n---Info---')
print(stations_raw.info())
```

    ---Sample Row---
    id                                                        31419
    data          {"executionTime": "2019-06-22 01:53:41 PM", "s...
    created_at                                  2019-06-22 13:54:01
    Name: 31418, dtype: object
    
    ---Info---
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 41325 entries, 0 to 41324
    Data columns (total 3 columns):
    id            41325 non-null int64
    data          41325 non-null object
    created_at    41325 non-null object
    dtypes: int64(1), object(2)
    memory usage: 968.6+ KB
    None



```python
"""Create pandas DataFrame with flat "availability" table. 
"""
df = pd.read_csv("../input/availability_sep2_2019.csv")
```


```python
df.shape
```




    (5800274, 21)



#### Geocoding

#### Weather

#### Cron Jobs

### Transforming

#### Availability by Station

#### Predicted vs Observed Weather

#### Auto-Generate README.md


```python
!jupyter nbconvert --output-dir='..' --to markdown analysis.ipynb --output README.md
```

    [NbConvertApp] Converting notebook analysis.ipynb to markdown
    [NbConvertApp] Writing 5901 bytes to ../README.md

