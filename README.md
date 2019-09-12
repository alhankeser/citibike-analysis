
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


```python
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
```

### Extracting

#### Stations


```python
stations_raw = pd.read_csv("../input/stations_raw.csv") # 16.49 GB output from base SQL table
```


```python
stations_raw.shape
```




    (41325, 3)




```python
stations_raw.iloc[1000:1005,:]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>data</th>
      <th>created_at</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1000</th>
      <td>1001</td>
      <td>{"executionTime": "2019-05-02 03:07:52 PM", "s...</td>
      <td>2019-05-02 15:08:01</td>
    </tr>
    <tr>
      <th>1001</th>
      <td>1002</td>
      <td>{"executionTime": "2019-05-02 03:08:57 PM", "s...</td>
      <td>2019-05-02 15:09:02</td>
    </tr>
    <tr>
      <th>1002</th>
      <td>1003</td>
      <td>{"executionTime": "2019-05-02 03:09:51 PM", "s...</td>
      <td>2019-05-02 15:10:01</td>
    </tr>
    <tr>
      <th>1003</th>
      <td>1004</td>
      <td>{"executionTime": "2019-05-02 03:10:55 PM", "s...</td>
      <td>2019-05-02 15:11:01</td>
    </tr>
    <tr>
      <th>1004</th>
      <td>1005</td>
      <td>{"executionTime": "2019-05-02 03:12:00 PM", "s...</td>
      <td>2019-05-02 15:12:02</td>
    </tr>
  </tbody>
</table>
</div>




```python
print(stations_raw.iloc[1000]['data'][:1000] + '...')
```

    {"executionTime": "2019-05-02 03:07:52 PM", "stationBeanList": [{"id": 168, "city": "", "altitude": "", "landMark": "", "latitude": 40.73971301, "location": "", "longitude": -73.99456405, "statusKey": 1, "postalCode": "", "stAddress1": "W 18 St & 6 Ave", "stAddress2": "", "totalDocks": 47, "stationName": "W 18 St & 6 Ave", "statusValue": "In Service", "testStation": false, "availableBikes": 14, "availableDocks": 31, "lastCommunicationTime": "2019-05-02 03:06:26 PM"}, {"id": 281, "city": "", "altitude": "", "landMark": "", "latitude": 40.7643971, "location": "", "longitude": -73.97371465, "statusKey": 1, "postalCode": "", "stAddress1": "Grand Army Plaza & Central Park S", "stAddress2": "", "totalDocks": 66, "stationName": "Grand Army Plaza & Central Park S", "statusValue": "In Service", "testStation": false, "availableBikes": 5, "availableDocks": 58, "lastCommunicationTime": "2019-05-02 03:05:15 PM"}, {"id": 285, "city": "", "altitude": "", "landMark": "", "latitude": 40.73454567, "loca...



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


```python
"""Creates README.md file for a better GitHub reading experience. 
"""
!jupyter nbconvert --output-dir='..' --to markdown analysis.ipynb --output README.md
```

    [NbConvertApp] Converting notebook analysis.ipynb to markdown
    [NbConvertApp] Writing 5518 bytes to ../README.md

