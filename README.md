
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
- **Created cron jobs** to collect Citi Bike station statuses for all ~858 stations, every 3 minutes, for ~2 months.
    - Total rows in final table: 5,800,274
    - "Why stop after 2 months," you ask? Because my server ran out of space while I was on vacation. Here's what that looks like: 

![My server crashed July 14](https://blog.alhan.co/storage/images/posts/2/web-server-crashed_2_1568434613_sm.jpg)
- **Created a mini-ETL process** to transform data into the final output used below. 
    - Along the way, there were many errors, some of which I will resolve here.

### Table of Contents
- [Packages](#Packages)
- [Extracting](#Extracting)
    - [Stations-Raw](#Stations-Raw)
    - [Stations-Flat](#Stations-Flat)
    - [Geocoding](#Geocoding)
    - [Weather](#Weather)
    - [Cron Jobs](#Cron-Jobs)
- [Transforming](#Transforming)
    - [Availability by Station](#Availability-by-Station)
- [Exploratory Data Analysis](#Exploratory-Data-Analysis)
    

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
pd.set_option('display.max_columns', None)
```

### Extracting
I started by find an interesting data source. In this case, I found the [Citi Bike Station Feed](https://feeds.citibikenyc.com/stations/stations.json) via the [NYC Open Data site](https://opendata.cityofnewyork.us/).

The feed shows the latest statuses of ~858 Citi Bike stations. Below is a list of values per station and sample data for each. Any keys left blank are often blank in the data source as well, which I'll address in later steps. 

| key | sample value |
|------------:|:---------|
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

#### Stations-Raw

To have a back-up in case any of the subsequent steps went awry, I wanted to store the source data in the simplest way possible: a table `stations_raw` that stored the following: 

|column_name|data_type|sample value|
|-----------|-----------|----------|
|id         |int4|31419|
|status     |json|{"executionTime": "2019-06-22 01:53:41 PM", "s...|

Once the table created, I needed a way to collect data. A quick solution -- for me -- was to create [a Laravel application](https://github.com/alhankeser/citibike-tracker/)  that [makes it easy create console commands](https://laravel.com/docs/5.8/artisan#writing-commands). In combination with [Laravel Forge](https://forge.laravel.com), it's easy to set up a cron job that triggers [the necessary command](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L49) at set intervals.

Once the commands created, I set up a [cron job](#Cron-Jobs) that ran once every 3 minutes. This resulted in the collection of 41,325 rows.

#### Stations-Flat
As part of the same command that creates the [stations_raw](#Stations-Raw) table, I [flattened out the JSON](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L80) and created a table with a single row per 3-minute interval, per station. We'll call this table `stations_flat` (probably could have used a better naming convention throughout this project). 

Here is the structure of `stations_flat` and some sample data:

|column_name|data_type|sample value|description|
|-----------|---------|------------|-----------|
|id         |int4     |10511778    |row id|
|station_id |int4     |72          |unique id for each station|
|available_bikes|int4 |4           |number of available bikes at the station|
|available_docks|int4 |49          |number of available docks (places to park a bike) at the station|
|station_status|text  |In Service  |whether the station is in or out of service|
|last_communication_time|timestamp|2019-05-15 01:14:15|the last time the station sent back data|

After just over 2 months of this, I ended up with **34,301,048 rows** in this table. Luckily, I took some steps to make the volume of data more manageable when analyzing outside of a high CPU/RAM environment. 

#### Stations-Static
As the name suggests, `stations_static` contains information about each station that doesn't change minute-to-minute. Since there was a likelihood that stations be added, removed, renamed, I inserted or updated on duplicate each time `stations_flat` was updated. 

|column_name|data_type|sample value|description|
|-----------|---------|------------|-----------|
|id| int4|3119|unique `station_id` found throughout db|
|name| text|Vernon Blvd & 50 Ave||
|latitude |float8|40.74232744||
|longitude |float8|-73.95411749||
|status_key| int4|1||
|postal_code| text|NULL||
|st_address_1| text|Vernon Blvd & 50 Ave||
|st_address_2| text|NULL||
|total_docks| int4|45||
|status| text|In Service||
|altitude| text|NULL||
|location| text|NULL||
|land_mark| text|NULL||
|city| text|NULL||
|is_test_station| int4|0||

#### Geocoding
As can be seen from the `stations_static` table above, many of the location-related values are null. This was the case for all stations. I wanted to be able to group stations by neighborhood and zip. Also, I wanted to use zip to associate weather data to each station, without having to make separate requests for each station (to stay within the free tier of the [Dark Sky Weather API](https://darksky.net/dev/docs)). 

To geocode from lat/long for each station into human-readeable location info, I used the [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/intro). [See the command I used to create the below table](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L92)

|column_name|data_type|sample value|description|
|-----------|---------|------------|-----------|
|id|int4|1||
|station_id|int4|3119|unique station id|
|zip|text|11101|zip code of station|
|hood_1|text|LIC|neighborhood or the closest thing provided by Google|
|hood_2|text|Hunters Point|another level of neighborhood|
|borough|text|Queens|one of 5 NYC boroughs or New Jersey|

#### Weather
Grouping stations by zip, I then called the [Dark Sky Weather API](https://darksky.net/dev/docs) once every hour to build the `weather` table. Reducing the scope to zip made it possible to stay within the free plan limits of Dark Sky. 

|column_name|data_type|sample value|description|
|-----------|---------|------------|-----------|
|id|int4|1||
|time_interval|timestamptz|2019-05-02 01:00:00-04|the 15-minute interval of time to associate the weather data to|
|summary|text|Foggy|a categorical label for weather conditions|
|precip_intensity|float8|0|percent percipitation intensity|
|temperature|float8|61.45|temperature in Fahrenheit|
|apparent_temperature|float8|61.89|"feels-like" temperature|
|dew_point|float8|60.82||
|humidity|float8|0.98|percent humidity|
|wind_speed|float8|3.11|speed in MPH|
|wind_gust|float8|5.38|gusts in MPH|
|cloud_cover|float8|1|percent cloud cover|
|uv_index|float8|0||
|visibility|float8|3.18||
|ozone|float8|316.23||
|status|text|observed|one of two values ("predicted"/"observed") depending on if the weather values are from the past or the future|

#### Cron Jobs
I'm not going to spend a lot of time on discussing cron jobs, but here are the patterns I was using to run everything. There is probably a more optimal approach that I am not aware of. 

|Cron       |Command|
|-----------|-------|
|\*/3 \* \* \* \* | get:docks && update:availability 0 && update:weather|
|0 \*/2 \* \* \*  | get:weather 0|

View the code behind each command:
- `get:docks` [view](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L49)
- `update:availability 0` [view](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L179)
- `update:weather` [view](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L359)
- `get:weather 0` [view](https://github.com/alhankeser/citibike-tracker/blob/d61f82adde88c90430205785297abf9f3de07c4d/app/Console/Kernel.php#L276)

### Transforming
Once I had the 1 raw table and 4 tables above updating as expected, I created a single, flat table that had the final data I intended to use for analysis. I kept most columns except some of the minutiae of the `weather` table. This table was updated regularly such that I could run analyses on an on-going basis and avoid having to run a massive, single query after all data had been collected.   

#### Availability by Station
Below is the flat table `availability` that combined the above tables, purpose-built for analysis. Note that available_bikes is the minimum number of bikes available during any of the 3-minute intervals during which samples were collected over the course of each 15-minute interval. 

|column_name|data_type|
|-----------|---------|
|time_interval|timestamptz|
|station_id|int4|
|station_name|text|
|station_status|text|
|latitude|float8|
|longitude|float8|
|zip|text|
|borough|text|
|hood|text|
|available_bikes|int4|
|available_docks|int4|
|weather_summary|text|
|precip_intensity|float8|
|temperature|float8|
|humidity|float8|
|wind_speed|float8|
|wind_gust|float8|
|cloud_cover|float8|
|weather_status|text|
|created_at|timestamptz|
|updated_at|timestamptz|

### Exploratory Data Analysis
On to the fun part! Now that I've got all of my station-by-station availability by 15-minute interval, it's time to explore. 

#### Reducing Complexity
First things first, I wanted to reduce the number of stations I was analyzing. The `availability` table resulted in nearly 6 million rows after 2 months, so I decided to export a subset of "interesting" stations to begin analyzing. Below is the query I used to find the interesting stations, based on whether there is a high variability in number of bikes, that the bikes regularly get refilled, and that the station has a decent number of bikes. I also limited the number of stations per neighborhood to 1.  
https://gist.github.com/alhankeser/9fbaf67a8ce052de72f22ab1630cd91c

```sql
with variability as (
    select
		borough,
		hood,
		station_name,
		station_id,
		max(available_bikes) as max_bikes,
		sum(case when available_bikes = 0 then 1 else 0 end) as times_no_bikes,
		sum(case when available_docks = 0 then 1 else 0 end) as times_replenished
	from 
		availability
	where
		station_status = 'In Service'
	group by
		station_id, station_name, hood, borough
),
percentiles as ( 
	select
		*,
        ntile(100) over (order by max_bikes asc) max_bikes_percentile,
		ntile(100) over (order by times_no_bikes asc) no_bikes_percentile,
	 	ntile(100) over (order by times_replenished asc) times_replenished_percentile
	from
		variability
	order by times_no_bikes
),
ranks as (
	select
		*,
		(max_bikes_percentile + no_bikes_percentile + times_replenished_percentile) as score,
		rank() over (partition by hood order by (max_bikes_percentile + no_bikes_percentile + times_replenished_percentile) desc) as rank
	from 
		percentiles
	where
		max_bikes_percentile > 40 
		and no_bikes_percentile > 50
		and times_replenished_percentile > 50
),
ranked_by_hood as (
	select
		*
	from 
		ranks
	where
		rank = 1 
	order by
		score desc
)
select
	a.*
from
	availability as a
join 
	ranked_by_hood as rbh 
	on a.station_id = rbh.station_id;
```

The query above reduced the nearly 6 million rows down to 186,000. The csv export used for the analysis below can be [found here](https://github.com/alhankeser/citibike-analysis/blob/master/input/availability_interesting_original.csv). 

#### Load Data


```python
date_cols = ['time_interval', 'updated_at', 'created_at']
df = pd.read_csv('../input/availability_interesting_original.csv', parse_dates=date_cols)
```


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 186030 entries, 0 to 186029
    Data columns (total 21 columns):
    station_id          186030 non-null int64
    station_name        186030 non-null object
    station_status      186030 non-null object
    latitude            186030 non-null float64
    longitude           186030 non-null float64
    zip                 186030 non-null int64
    borough             186030 non-null object
    hood                186030 non-null object
    available_bikes     186030 non-null int64
    available_docks     186030 non-null int64
    time_interval       186030 non-null datetime64[ns]
    created_at          186030 non-null datetime64[ns]
    weather_summary     88053 non-null object
    precip_intensity    88053 non-null float64
    temperature         88053 non-null float64
    humidity            88053 non-null float64
    wind_speed          88053 non-null float64
    wind_gust           88053 non-null float64
    cloud_cover         88053 non-null float64
    weather_status      88053 non-null object
    updated_at          186030 non-null datetime64[ns]
    dtypes: datetime64[ns](3), float64(8), int64(4), object(6)
    memory usage: 29.8+ MB



```python
print(df.head(3)) #printing to improve how this looks in the README.md markdown file
```

       station_id station_name station_status   latitude  longitude   zip  \
    0        3195      Sip Ave     In Service  40.730897 -74.063913  7306   
    1        3195      Sip Ave     In Service  40.730897 -74.063913  7306   
    2        3195      Sip Ave     In Service  40.730897 -74.063913  7306   
    
          borough            hood  available_bikes  available_docks  \
    0  New Jersey  Journal Square                1               33   
    1  New Jersey  Journal Square                0               34   
    2  New Jersey  Journal Square                0               34   
    
            time_interval          created_at weather_summary  precip_intensity  \
    0 2019-05-13 02:45:00 2019-05-13 02:45:04        Overcast               0.0   
    1 2019-05-13 02:30:00 2019-05-13 02:30:04        Overcast               0.0   
    2 2019-05-13 02:15:00 2019-05-13 02:15:05        Overcast               0.0   
    
       temperature  humidity  wind_speed  wind_gust  cloud_cover weather_status  \
    0        44.86      0.91        6.85       9.65          1.0      predicted   
    1        44.86      0.91        6.85       9.65          1.0      predicted   
    2        44.86      0.91        6.85       9.65          1.0      predicted   
    
               updated_at  
    0 2019-05-13 02:45:04  
    1 2019-05-13 02:45:04  
    2 2019-05-13 02:45:04  


#### Data Quality Issues
Without much digging, it's easy to spot some data quality/consistency issues: 
1. `weather_status` should be 'observed' for all locations rather than 'predicted' since the dates are in the past. 
1. `zip` is being converted to an integer and thus dropping the 0, which is may or may not be an issue. If wish to solve problem #1 then this chould be an issue. 

Auto-Generate README.md:


```python
!jupyter nbconvert --output-dir='..' --to markdown analysis.ipynb --output README.md
```

    [NbConvertApp] Converting notebook analysis.ipynb to markdown
    [NbConvertApp] Writing 17126 bytes to ../README.md

