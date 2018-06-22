
# Analysis

Observations:
1. As the coordinates approach to the equator, or the closer the city is to the equator, the higher the temperature. 
2. According to this random data, majority of these citie's Latitudes seem to be greater than 0 which applies they are from the northern part of the hemisphere, however there is no relationship between their latitudes and huminity levels.
3. Similarly, there is no relationship between latitudes and cloudiness percentage for these random cities
4. Lastly, the further away from the equator, the higher the wind speed.However, is it a very weak correlation.


```python
# Dependencies
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import requests as req
import numpy as np
import time
from citipy import citipy
from weatherfig import api_key
from datetime import datetime
```


```python
# Generating random latitudes and longitudes for 1200 cities
lat = np.random.uniform(low=-90, high=90, size=1200)
lng = np.random.uniform(low=-180, high=180, size=1200)

# Creating coordinate pairs
coordinates = []
for x in range(0,len(lat)):
    coordinates.append((lat[x], lng[x]))
# print (coordinates)
```


```python
# Finding the cities nearest to the coordinates
cities = []
for coordinate_pair in coordinates:
    lat, lon = coordinate_pair
    cities.append(citipy.nearest_city(lat, lon))

# Creating DataFrame
cities_df = pd.DataFrame(cities)
cities_df["City Name"] = ""
cities_df["Country Code"] = ""

for index, row in cities_df.iterrows():
    row["City Name"] = cities_df.iloc[index,0].city_name
    row["Country Code"] = cities_df.iloc[index,0].country_code

# Dropping duplicate cities
cities_df.drop_duplicates(['City Name', 'Country Code'], inplace=True)
cities_df.reset_index(inplace=True)

# Deleting unnecessary columns
del cities_df[0]
del cities_df['index']

cities_df.head()
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
      <th>City Name</th>
      <th>Country Code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>neiafu</td>
      <td>to</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ushuaia</td>
      <td>ar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>port blair</td>
      <td>in</td>
    </tr>
    <tr>
      <th>3</th>
      <td>puerto ayora</td>
      <td>ec</td>
    </tr>
    <tr>
      <th>4</th>
      <td>mys shmidta</td>
      <td>ru</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Adding columns for values
cities_df['Latitude'] = ""
cities_df['Longitude'] = ""
cities_df['Temperature (F)'] = ""
cities_df['Humidity (%)'] = ""
cities_df['Cloudiness (%)'] = ""
cities_df['Wind Speed (mph)'] = ""

cities_df.head()
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
      <th>City Name</th>
      <th>Country Code</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Temperature (F)</th>
      <th>Humidity (%)</th>
      <th>Cloudiness (%)</th>
      <th>Wind Speed (mph)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>neiafu</td>
      <td>to</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>port blair</td>
      <td>in</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>puerto ayora</td>
      <td>ec</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>mys shmidta</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
# Accessing data
print("Beginning Data Retrieval")
print("---------------------------------")

# Limiting pull requests
start_time = time.time()

for index, row in cities_df.iterrows():
    # Building target url
    url = "http://api.openweathermap.org/data/2.5/weather?q=%s,%s&units=imperial&appid=%s" % (row['City Name'], 
                                                                                              row['Country Code'], 
                                                                                              api_key)
    print(url)
    # Printing to ensure loop is correct
    print("Now retrieving City #" + str(index) + ": " + row['City Name'] + ", " + row['Country Code'])
    print(url)
    
    # Running request
    weather_data = req.get(url).json()
    
    try:
        # Appending latitude and longitude to correct location
        row['Latitude'] = weather_data['coord']['lat']
        row['Longitude'] = weather_data['coord']['lon']
    
        # Appending temperature to correct location
        row['Temperature (F)'] = weather_data['main']['temp']
    
        # Appending humidity to correct location
        row['Humidity (%)'] = weather_data['main']['humidity']
    
        # Appending cloudiness to correct location
        row['Cloudiness (%)'] = weather_data['clouds']['all']
    
        # Appending wind speed to correct location
        row['Wind Speed (mph)'] = weather_data['wind']['speed']
    except:
        print("Error with city data. Skipping")
        continue
        
    # Pausing to limit pull requests
    if (index + 1) % 60 == 0:
        run_time = time.time() - start_time
        time.sleep(60 - run_time)
        start_time = time.time()
    
print("---------------------------------")
print("Data Retrieval Complete")
print("---------------------------------")

# Changing strings to floats
columns = ['Latitude', 'Temperature (F)', 'Humidity (%)', 'Cloudiness (%)', 'Wind Speed (mph)']
for column in columns:
    cities_df[column] = pd.to_numeric(cities_df[column], errors='coerce')
    
# Dropping NaN values
cities_df.dropna(inplace=True)

cities_df.head()
```

    Beginning Data Retrieval
    ---------------------------------
    http://api.openweathermap.org/data/2.5/weather?q=neiafu,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #0: neiafu, to
    http://api.openweathermap.org/data/2.5/weather?q=neiafu,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #1: ushuaia, ar
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port blair,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #2: port blair, in
    http://api.openweathermap.org/data/2.5/weather?q=port blair,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #3: puerto ayora, ec
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kununurra,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #5: kununurra, au
    http://api.openweathermap.org/data/2.5/weather?q=kununurra,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #6: saskylakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #7: vaini, to
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=porto santo,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #8: porto santo, pt
    http://api.openweathermap.org/data/2.5/weather?q=porto santo,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port keats,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #10: port keats, au
    http://api.openweathermap.org/data/2.5/weather?q=port keats,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port alfred,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #12: port alfred, za
    http://api.openweathermap.org/data/2.5/weather?q=port alfred,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint george,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #13: saint george, bm
    http://api.openweathermap.org/data/2.5/weather?q=saint george,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rocha,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #14: rocha, uy
    http://api.openweathermap.org/data/2.5/weather?q=rocha,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #15: rikitea, pf
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #17: saint-philippe, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=east london,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #18: east london, za
    http://api.openweathermap.org/data/2.5/weather?q=east london,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=huaicheng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #20: huaicheng, cn
    http://api.openweathermap.org/data/2.5/weather?q=huaicheng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=palana,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #22: palana, ru
    http://api.openweathermap.org/data/2.5/weather?q=palana,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #23: hermanus, za
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hambantota,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #24: hambantota, lk
    http://api.openweathermap.org/data/2.5/weather?q=hambantota,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #25: albany, au
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #26: hilo, us
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #27: faanui, pf
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #28: kahului, us
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=garowe,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #30: garowe, so
    http://api.openweathermap.org/data/2.5/weather?q=garowe,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #31: upernavik, gl
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #32: butaritari, ki
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #33: yellowknife, ca
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=monte patria,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #34: monte patria, cl
    http://api.openweathermap.org/data/2.5/weather?q=monte patria,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #35: pevek, ru
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oranjemund,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #36: oranjemund, na
    http://api.openweathermap.org/data/2.5/weather?q=oranjemund,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #37: tasiilaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=naze,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #39: naze, jp
    http://api.openweathermap.org/data/2.5/weather?q=naze,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ancud,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #40: ancud, cl
    http://api.openweathermap.org/data/2.5/weather?q=ancud,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=manoel urbano,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #41: manoel urbano, br
    http://api.openweathermap.org/data/2.5/weather?q=manoel urbano,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #42: punta arenas, cl
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #43: chokurdakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #45: busselton, au
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port lincoln,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #46: port lincoln, au
    http://api.openweathermap.org/data/2.5/weather?q=port lincoln,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=beidao,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #47: beidao, cn
    http://api.openweathermap.org/data/2.5/weather?q=beidao,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #48: lompoc, us
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=karatau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #49: karatau, kz
    http://api.openweathermap.org/data/2.5/weather?q=karatau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #50: port elizabeth, za
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cape town,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #51: cape town, za
    http://api.openweathermap.org/data/2.5/weather?q=cape town,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #53: leningradskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #54: viedma, ar
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kavieng,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #55: kavieng, pg
    http://api.openweathermap.org/data/2.5/weather?q=kavieng,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dillon,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #56: dillon, us
    http://api.openweathermap.org/data/2.5/weather?q=dillon,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #57: bluff, nz
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #58: nome, us
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aginskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #59: aginskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=aginskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tilichiki,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #60: tilichiki, ru
    http://api.openweathermap.org/data/2.5/weather?q=tilichiki,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alice springs,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #61: alice springs, au
    http://api.openweathermap.org/data/2.5/weather?q=alice springs,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ruwi,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #62: ruwi, om
    http://api.openweathermap.org/data/2.5/weather?q=ruwi,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #63: ilulissat, gl
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #64: wau, pg
    http://api.openweathermap.org/data/2.5/weather?q=wau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ust-kuyga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #65: ust-kuyga, ru
    http://api.openweathermap.org/data/2.5/weather?q=ust-kuyga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ulaangom,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #66: ulaangom, mn
    http://api.openweathermap.org/data/2.5/weather?q=ulaangom,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=caxito,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #67: caxito, ao
    http://api.openweathermap.org/data/2.5/weather?q=caxito,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kidal,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #68: kidal, ml
    http://api.openweathermap.org/data/2.5/weather?q=kidal,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #69: hithadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #71: arraial do cabo, br
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=stratford-upon-avon,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #72: stratford-upon-avon, gb
    http://api.openweathermap.org/data/2.5/weather?q=stratford-upon-avon,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #73: bredasdorp, za
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #74: carnarvon, au
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #75: barrow, us
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=norman wells,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #76: norman wells, ca
    http://api.openweathermap.org/data/2.5/weather?q=norman wells,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=panorama,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #77: panorama, br
    http://api.openweathermap.org/data/2.5/weather?q=panorama,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #78: jamestown, sh
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wamba,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #79: wamba, cd
    http://api.openweathermap.org/data/2.5/weather?q=wamba,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #80: touros, br
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #81: georgetown, sh
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=roma,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #82: roma, au
    http://api.openweathermap.org/data/2.5/weather?q=roma,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=acari,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #83: acari, pe
    http://api.openweathermap.org/data/2.5/weather?q=acari,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kawalu,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #84: kawalu, id
    http://api.openweathermap.org/data/2.5/weather?q=kawalu,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #85: torbay, ca
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #86: dingle, ie
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=verkhnyaya inta,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #87: verkhnyaya inta, ru
    http://api.openweathermap.org/data/2.5/weather?q=verkhnyaya inta,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=baghdad,iq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #89: baghdad, iq
    http://api.openweathermap.org/data/2.5/weather?q=baghdad,iq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #90: constitucion, mx
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kita,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #91: kita, ml
    http://api.openweathermap.org/data/2.5/weather?q=kita,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #93: new norfolk, au
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ellensburg,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #94: ellensburg, us
    http://api.openweathermap.org/data/2.5/weather?q=ellensburg,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #95: bethel, us
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #96: pangnirtung, ca
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #97: sitka, us
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #98: tiksi, ru
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=guaira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #100: guaira, br
    http://api.openweathermap.org/data/2.5/weather?q=guaira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #101: kapaa, us
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #102: mar del plata, ar
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #103: nanortalik, gl
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vila velha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #104: vila velha, br
    http://api.openweathermap.org/data/2.5/weather?q=vila velha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=abatskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #105: abatskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=abatskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san patricio,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #106: san patricio, mx
    http://api.openweathermap.org/data/2.5/weather?q=san patricio,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #107: yulara, au
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #108: chuy, uy
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kozhva,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #109: kozhva, ru
    http://api.openweathermap.org/data/2.5/weather?q=kozhva,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alyangula,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #110: alyangula, au
    http://api.openweathermap.org/data/2.5/weather?q=alyangula,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kosh-agach,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #111: kosh-agach, ru
    http://api.openweathermap.org/data/2.5/weather?q=kosh-agach,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #112: mahebourg, mu
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #113: lebu, cl
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=matara,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #114: matara, lk
    http://api.openweathermap.org/data/2.5/weather?q=matara,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=filingue,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #115: filingue, ne
    http://api.openweathermap.org/data/2.5/weather?q=filingue,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chimbote,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #117: chimbote, pe
    http://api.openweathermap.org/data/2.5/weather?q=chimbote,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #118: dikson, ru
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #119: katsuura, jp
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sayat,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #120: sayat, tm
    http://api.openweathermap.org/data/2.5/weather?q=sayat,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san roque,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #121: san roque, ph
    http://api.openweathermap.org/data/2.5/weather?q=san roque,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=luderitz,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #122: luderitz, na
    http://api.openweathermap.org/data/2.5/weather?q=luderitz,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #123: atuona, pf
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=emerald,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #124: emerald, au
    http://api.openweathermap.org/data/2.5/weather?q=emerald,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=abalak,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #126: abalak, ne
    http://api.openweathermap.org/data/2.5/weather?q=abalak,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #127: ostrovnoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mecca,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #128: mecca, sa
    http://api.openweathermap.org/data/2.5/weather?q=mecca,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #129: thompson, ca
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #130: alofi, nu
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fredericksburg,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #131: fredericksburg, us
    http://api.openweathermap.org/data/2.5/weather?q=fredericksburg,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vostok,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #132: vostok, ru
    http://api.openweathermap.org/data/2.5/weather?q=vostok,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nizhniye sergi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #133: nizhniye sergi, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhniye sergi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=roald,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #134: roald, no
    http://api.openweathermap.org/data/2.5/weather?q=roald,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=farap,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #135: farap, tm
    http://api.openweathermap.org/data/2.5/weather?q=farap,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=clyde river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #136: clyde river, ca
    http://api.openweathermap.org/data/2.5/weather?q=clyde river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dafeng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #137: dafeng, cn
    http://api.openweathermap.org/data/2.5/weather?q=dafeng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=polunochnoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #138: polunochnoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=polunochnoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kiama,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #139: kiama, au
    http://api.openweathermap.org/data/2.5/weather?q=kiama,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #140: talnakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #141: beringovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rio gallegos,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #142: rio gallegos, ar
    http://api.openweathermap.org/data/2.5/weather?q=rio gallegos,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=semikarakorsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #143: semikarakorsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=semikarakorsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mangrol,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #144: mangrol, in
    http://api.openweathermap.org/data/2.5/weather?q=mangrol,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=luanda,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #146: luanda, ao
    http://api.openweathermap.org/data/2.5/weather?q=luanda,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mount gambier,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #147: mount gambier, au
    http://api.openweathermap.org/data/2.5/weather?q=mount gambier,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #148: kruisfontein, za
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=trinidad,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #149: trinidad, co
    http://api.openweathermap.org/data/2.5/weather?q=trinidad,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #150: castro, cl
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #152: nikolskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oceanside,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #153: oceanside, us
    http://api.openweathermap.org/data/2.5/weather?q=oceanside,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sainte-marie,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #154: sainte-marie, re
    http://api.openweathermap.org/data/2.5/weather?q=sainte-marie,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carballo,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #155: carballo, es
    http://api.openweathermap.org/data/2.5/weather?q=carballo,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao jose da coroa grande,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #156: sao jose da coroa grande, br
    http://api.openweathermap.org/data/2.5/weather?q=sao jose da coroa grande,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=egvekinot,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #157: egvekinot, ru
    http://api.openweathermap.org/data/2.5/weather?q=egvekinot,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #160: severo-kurilsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hofn,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #162: hofn, is
    http://api.openweathermap.org/data/2.5/weather?q=hofn,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kenora,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #163: kenora, ca
    http://api.openweathermap.org/data/2.5/weather?q=kenora,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=komsomolskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #164: komsomolskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=komsomolskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #165: longyearbyen, sj
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mattru,sl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #166: mattru, sl
    http://api.openweathermap.org/data/2.5/weather?q=mattru,sl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=blyznyuky,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #167: blyznyuky, ua
    http://api.openweathermap.org/data/2.5/weather?q=blyznyuky,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=raudeberg,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #168: raudeberg, no
    http://api.openweathermap.org/data/2.5/weather?q=raudeberg,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thinadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #169: thinadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=thinadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #170: avarua, ck
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=benghazi,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #171: benghazi, ly
    http://api.openweathermap.org/data/2.5/weather?q=benghazi,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tawau,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #172: tawau, my
    http://api.openweathermap.org/data/2.5/weather?q=tawau,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bosaso,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #173: bosaso, so
    http://api.openweathermap.org/data/2.5/weather?q=bosaso,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lechinkay,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #174: lechinkay, ru
    http://api.openweathermap.org/data/2.5/weather?q=lechinkay,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=prainha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #175: prainha, br
    http://api.openweathermap.org/data/2.5/weather?q=prainha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mogzon,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #176: mogzon, ru
    http://api.openweathermap.org/data/2.5/weather?q=mogzon,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #179: husavik, is
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zhanaozen,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #180: zhanaozen, kz
    http://api.openweathermap.org/data/2.5/weather?q=zhanaozen,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #181: geraldton, au
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=male,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #182: male, mv
    http://api.openweathermap.org/data/2.5/weather?q=male,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #183: sao filipe, cv
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #184: ribeira grande, pt
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=novikovo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #185: novikovo, ru
    http://api.openweathermap.org/data/2.5/weather?q=novikovo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #186: fortuna, us
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #189: khatanga, ru
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zeya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #190: zeya, ru
    http://api.openweathermap.org/data/2.5/weather?q=zeya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #192: kaitangata, nz
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=adelaide,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #193: adelaide, au
    http://api.openweathermap.org/data/2.5/weather?q=adelaide,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=haines junction,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #194: haines junction, ca
    http://api.openweathermap.org/data/2.5/weather?q=haines junction,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shingu,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #195: shingu, jp
    http://api.openweathermap.org/data/2.5/weather?q=shingu,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=batticaloa,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #196: batticaloa, lk
    http://api.openweathermap.org/data/2.5/weather?q=batticaloa,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=poyarkovo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #197: poyarkovo, ru
    http://api.openweathermap.org/data/2.5/weather?q=poyarkovo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=samarai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #198: samarai, pg
    http://api.openweathermap.org/data/2.5/weather?q=samarai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fort nelson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #199: fort nelson, ca
    http://api.openweathermap.org/data/2.5/weather?q=fort nelson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bandarban,bd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #200: bandarban, bd
    http://api.openweathermap.org/data/2.5/weather?q=bandarban,bd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=moerai,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #201: moerai, pf
    http://api.openweathermap.org/data/2.5/weather?q=moerai,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #202: cherskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ningxiang,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #203: ningxiang, cn
    http://api.openweathermap.org/data/2.5/weather?q=ningxiang,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #204: kodiak, us
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ramos arizpe,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #206: ramos arizpe, mx
    http://api.openweathermap.org/data/2.5/weather?q=ramos arizpe,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #207: victoria, sc
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dunedin,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #208: dunedin, nz
    http://api.openweathermap.org/data/2.5/weather?q=dunedin,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zheleznodorozhnyy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #209: zheleznodorozhnyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=zheleznodorozhnyy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=general teran,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #210: general teran, mx
    http://api.openweathermap.org/data/2.5/weather?q=general teran,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=burns lake,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #211: burns lake, ca
    http://api.openweathermap.org/data/2.5/weather?q=burns lake,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=voru,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #212: voru, ee
    http://api.openweathermap.org/data/2.5/weather?q=voru,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=le vauclin,mq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #213: le vauclin, mq
    http://api.openweathermap.org/data/2.5/weather?q=le vauclin,mq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=loandjili,cg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #214: loandjili, cg
    http://api.openweathermap.org/data/2.5/weather?q=loandjili,cg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sibu,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #215: sibu, my
    http://api.openweathermap.org/data/2.5/weather?q=sibu,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #216: qaanaaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tuatapere,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #217: tuatapere, nz
    http://api.openweathermap.org/data/2.5/weather?q=tuatapere,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wagar,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #218: wagar, sd
    http://api.openweathermap.org/data/2.5/weather?q=wagar,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tutoia,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #220: tutoia, br
    http://api.openweathermap.org/data/2.5/weather?q=tutoia,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chumikan,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #221: chumikan, ru
    http://api.openweathermap.org/data/2.5/weather?q=chumikan,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pirna,de&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #223: pirna, de
    http://api.openweathermap.org/data/2.5/weather?q=pirna,de&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #224: vao, nc
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #225: esperance, au
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #226: hobart, au
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #227: adrar, dz
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #229: hasaki, jp
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=xiaoweizhai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #230: xiaoweizhai, cn
    http://api.openweathermap.org/data/2.5/weather?q=xiaoweizhai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nosy varika,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #231: nosy varika, mg
    http://api.openweathermap.org/data/2.5/weather?q=nosy varika,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=olavarria,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #233: olavarria, ar
    http://api.openweathermap.org/data/2.5/weather?q=olavarria,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ahuimanu,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #234: ahuimanu, us
    http://api.openweathermap.org/data/2.5/weather?q=ahuimanu,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=miraflores,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #235: miraflores, co
    http://api.openweathermap.org/data/2.5/weather?q=miraflores,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=virginia beach,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #236: virginia beach, us
    http://api.openweathermap.org/data/2.5/weather?q=virginia beach,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=poronaysk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #237: poronaysk, ru
    http://api.openweathermap.org/data/2.5/weather?q=poronaysk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tshela,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #238: tshela, cd
    http://api.openweathermap.org/data/2.5/weather?q=tshela,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cruzeiro do sul,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #239: cruzeiro do sul, br
    http://api.openweathermap.org/data/2.5/weather?q=cruzeiro do sul,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-paul,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #240: saint-paul, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-paul,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=puri,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #241: puri, in
    http://api.openweathermap.org/data/2.5/weather?q=puri,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=turukhansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #242: turukhansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=turukhansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=muscat,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #243: muscat, om
    http://api.openweathermap.org/data/2.5/weather?q=muscat,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=salalah,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #245: salalah, om
    http://api.openweathermap.org/data/2.5/weather?q=salalah,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kandrian,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #246: kandrian, pg
    http://api.openweathermap.org/data/2.5/weather?q=kandrian,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #247: carnarvon, za
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pont-rouge,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #248: pont-rouge, ca
    http://api.openweathermap.org/data/2.5/weather?q=pont-rouge,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=caravelas,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #250: caravelas, br
    http://api.openweathermap.org/data/2.5/weather?q=caravelas,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mount pleasant,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #251: mount pleasant, us
    http://api.openweathermap.org/data/2.5/weather?q=mount pleasant,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=marawi,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #253: marawi, sd
    http://api.openweathermap.org/data/2.5/weather?q=marawi,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=padang,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #254: padang, id
    http://api.openweathermap.org/data/2.5/weather?q=padang,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #255: cabo san lucas, mx
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zaraza,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #256: zaraza, ve
    http://api.openweathermap.org/data/2.5/weather?q=zaraza,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san carlos de bariloche,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #258: san carlos de bariloche, ar
    http://api.openweathermap.org/data/2.5/weather?q=san carlos de bariloche,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san matias,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #259: san matias, bo
    http://api.openweathermap.org/data/2.5/weather?q=san matias,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=christchurch,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #261: christchurch, nz
    http://api.openweathermap.org/data/2.5/weather?q=christchurch,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #262: ponta do sol, cv
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=igboho,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #263: igboho, ng
    http://api.openweathermap.org/data/2.5/weather?q=igboho,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=inuvik,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #264: inuvik, ca
    http://api.openweathermap.org/data/2.5/weather?q=inuvik,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dawei,mm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #266: dawei, mm
    http://api.openweathermap.org/data/2.5/weather?q=dawei,mm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #267: guerrero negro, mx
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=balotra,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #268: balotra, in
    http://api.openweathermap.org/data/2.5/weather?q=balotra,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tomigusuku,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #270: tomigusuku, jp
    http://api.openweathermap.org/data/2.5/weather?q=tomigusuku,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=qaqortoq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #271: qaqortoq, gl
    http://api.openweathermap.org/data/2.5/weather?q=qaqortoq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bonavista,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #272: bonavista, ca
    http://api.openweathermap.org/data/2.5/weather?q=bonavista,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=los llanos de aridane,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #273: los llanos de aridane, es
    http://api.openweathermap.org/data/2.5/weather?q=los llanos de aridane,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-joseph,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #274: saint-joseph, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-joseph,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=iralaya,hn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #275: iralaya, hn
    http://api.openweathermap.org/data/2.5/weather?q=iralaya,hn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ibra,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #276: ibra, om
    http://api.openweathermap.org/data/2.5/weather?q=ibra,om&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gigmoto,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #278: gigmoto, ph
    http://api.openweathermap.org/data/2.5/weather?q=gigmoto,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=itarema,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #280: itarema, br
    http://api.openweathermap.org/data/2.5/weather?q=itarema,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tabasalu,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #281: tabasalu, ee
    http://api.openweathermap.org/data/2.5/weather?q=tabasalu,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=anadyr,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #282: anadyr, ru
    http://api.openweathermap.org/data/2.5/weather?q=anadyr,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #283: vila franca do campo, pt
    http://api.openweathermap.org/data/2.5/weather?q=vila franca do campo,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ingham,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #284: ingham, au
    http://api.openweathermap.org/data/2.5/weather?q=ingham,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san rafael,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #285: san rafael, ar
    http://api.openweathermap.org/data/2.5/weather?q=san rafael,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hay river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #286: hay river, ca
    http://api.openweathermap.org/data/2.5/weather?q=hay river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ixtapa,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #287: ixtapa, mx
    http://api.openweathermap.org/data/2.5/weather?q=ixtapa,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=antigonish,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #289: antigonish, ca
    http://api.openweathermap.org/data/2.5/weather?q=antigonish,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=altamira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #290: altamira, br
    http://api.openweathermap.org/data/2.5/weather?q=altamira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-augustin,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #291: saint-augustin, ca
    http://api.openweathermap.org/data/2.5/weather?q=saint-augustin,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #292: san cristobal, ec
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mogadishu,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #293: mogadishu, so
    http://api.openweathermap.org/data/2.5/weather?q=mogadishu,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tupik,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #295: tupik, ru
    http://api.openweathermap.org/data/2.5/weather?q=tupik,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pierre,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #296: pierre, us
    http://api.openweathermap.org/data/2.5/weather?q=pierre,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kirakira,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #297: kirakira, sb
    http://api.openweathermap.org/data/2.5/weather?q=kirakira,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lahad datu,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #298: lahad datu, my
    http://api.openweathermap.org/data/2.5/weather?q=lahad datu,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hobyo,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #299: hobyo, so
    http://api.openweathermap.org/data/2.5/weather?q=hobyo,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alice town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #301: alice town, bs
    http://api.openweathermap.org/data/2.5/weather?q=alice town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ostersund,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #302: ostersund, se
    http://api.openweathermap.org/data/2.5/weather?q=ostersund,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=eureka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #303: eureka, us
    http://api.openweathermap.org/data/2.5/weather?q=eureka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shakawe,bw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #304: shakawe, bw
    http://api.openweathermap.org/data/2.5/weather?q=shakawe,bw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #305: namibe, ao
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #306: vestmannaeyjar, is
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rawson,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #307: rawson, ar
    http://api.openweathermap.org/data/2.5/weather?q=rawson,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=umm kaddadah,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #309: umm kaddadah, sd
    http://api.openweathermap.org/data/2.5/weather?q=umm kaddadah,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=omaruru,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #310: omaruru, na
    http://api.openweathermap.org/data/2.5/weather?q=omaruru,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #311: bambous virieux, mu
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zhigansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #312: zhigansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=zhigansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san francisco,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #313: san francisco, ar
    http://api.openweathermap.org/data/2.5/weather?q=san francisco,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #315: sao joao da barra, br
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #316: klaksvik, fo
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=toktogul,kg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #317: toktogul, kg
    http://api.openweathermap.org/data/2.5/weather?q=toktogul,kg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mutoko,zw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #318: mutoko, zw
    http://api.openweathermap.org/data/2.5/weather?q=mutoko,zw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=la carolina,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #320: la carolina, es
    http://api.openweathermap.org/data/2.5/weather?q=la carolina,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wuda,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #321: wuda, cn
    http://api.openweathermap.org/data/2.5/weather?q=wuda,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=katangli,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #322: katangli, ru
    http://api.openweathermap.org/data/2.5/weather?q=katangli,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #323: cidreira, br
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wolfville,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #324: wolfville, ca
    http://api.openweathermap.org/data/2.5/weather?q=wolfville,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kloulklubed,pw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #325: kloulklubed, pw
    http://api.openweathermap.org/data/2.5/weather?q=kloulklubed,pw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zhytomyr,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #326: zhytomyr, ua
    http://api.openweathermap.org/data/2.5/weather?q=zhytomyr,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=porto walter,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #327: porto walter, br
    http://api.openweathermap.org/data/2.5/weather?q=porto walter,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wewak,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #328: wewak, pg
    http://api.openweathermap.org/data/2.5/weather?q=wewak,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #329: cockburn town, bs
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mabaruma,gy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #330: mabaruma, gy
    http://api.openweathermap.org/data/2.5/weather?q=mabaruma,gy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aguimes,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #331: aguimes, es
    http://api.openweathermap.org/data/2.5/weather?q=aguimes,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #332: ponta do sol, pt
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aksu,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #333: aksu, kz
    http://api.openweathermap.org/data/2.5/weather?q=aksu,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=luba,gq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #334: luba, gq
    http://api.openweathermap.org/data/2.5/weather?q=luba,gq&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cayenne,gf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #335: cayenne, gf
    http://api.openweathermap.org/data/2.5/weather?q=cayenne,gf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=naliya,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #336: naliya, in
    http://api.openweathermap.org/data/2.5/weather?q=naliya,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=horsham,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #337: horsham, au
    http://api.openweathermap.org/data/2.5/weather?q=horsham,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=palmer,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #338: palmer, us
    http://api.openweathermap.org/data/2.5/weather?q=palmer,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=letka,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #339: letka, ru
    http://api.openweathermap.org/data/2.5/weather?q=letka,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shakiso,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #340: shakiso, et
    http://api.openweathermap.org/data/2.5/weather?q=shakiso,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=taksimo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #341: taksimo, ru
    http://api.openweathermap.org/data/2.5/weather?q=taksimo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=noumea,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #342: noumea, nc
    http://api.openweathermap.org/data/2.5/weather?q=noumea,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #343: bandarbeyla, so
    http://api.openweathermap.org/data/2.5/weather?q=bandarbeyla,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lavrentiya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #344: lavrentiya, ru
    http://api.openweathermap.org/data/2.5/weather?q=lavrentiya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=naryan-mar,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #345: naryan-mar, ru
    http://api.openweathermap.org/data/2.5/weather?q=naryan-mar,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=senneterre,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #346: senneterre, ca
    http://api.openweathermap.org/data/2.5/weather?q=senneterre,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=olga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #347: olga, ru
    http://api.openweathermap.org/data/2.5/weather?q=olga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=druzhba,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #348: druzhba, ua
    http://api.openweathermap.org/data/2.5/weather?q=druzhba,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=veraval,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #349: veraval, in
    http://api.openweathermap.org/data/2.5/weather?q=veraval,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gananoque,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #350: gananoque, ca
    http://api.openweathermap.org/data/2.5/weather?q=gananoque,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kamenka,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #351: kamenka, ru
    http://api.openweathermap.org/data/2.5/weather?q=kamenka,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=asyut,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #352: asyut, eg
    http://api.openweathermap.org/data/2.5/weather?q=asyut,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=berdigestyakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #355: berdigestyakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=berdigestyakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #356: namatanai, pg
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jesup,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #358: jesup, us
    http://api.openweathermap.org/data/2.5/weather?q=jesup,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hamilton,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #359: hamilton, bm
    http://api.openweathermap.org/data/2.5/weather?q=hamilton,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=college,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #361: college, us
    http://api.openweathermap.org/data/2.5/weather?q=college,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #362: isangel, vu
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #363: flinders, au
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hami,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #364: hami, cn
    http://api.openweathermap.org/data/2.5/weather?q=hami,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=morehead,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #366: morehead, pg
    http://api.openweathermap.org/data/2.5/weather?q=morehead,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=agadez,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #367: agadez, ne
    http://api.openweathermap.org/data/2.5/weather?q=agadez,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kalemie,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #368: kalemie, cd
    http://api.openweathermap.org/data/2.5/weather?q=kalemie,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nalvo,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #369: nalvo, ph
    http://api.openweathermap.org/data/2.5/weather?q=nalvo,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jardim,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #370: jardim, br
    http://api.openweathermap.org/data/2.5/weather?q=jardim,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #371: nemuro, jp
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kamskiye polyany,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #372: kamskiye polyany, ru
    http://api.openweathermap.org/data/2.5/weather?q=kamskiye polyany,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=matay,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #373: matay, eg
    http://api.openweathermap.org/data/2.5/weather?q=matay,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=scutelnici,ro&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #374: scutelnici, ro
    http://api.openweathermap.org/data/2.5/weather?q=scutelnici,ro&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=great falls,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #375: great falls, us
    http://api.openweathermap.org/data/2.5/weather?q=great falls,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tarko-sale,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #376: tarko-sale, ru
    http://api.openweathermap.org/data/2.5/weather?q=tarko-sale,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=voskresenskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #377: voskresenskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=voskresenskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=berlevag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #378: berlevag, no
    http://api.openweathermap.org/data/2.5/weather?q=berlevag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=byron bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #379: byron bay, au
    http://api.openweathermap.org/data/2.5/weather?q=byron bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bilma,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #380: bilma, ne
    http://api.openweathermap.org/data/2.5/weather?q=bilma,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=artigas,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #381: artigas, uy
    http://api.openweathermap.org/data/2.5/weather?q=artigas,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #382: tuktoyaktuk, ca
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=flin flon,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #383: flin flon, ca
    http://api.openweathermap.org/data/2.5/weather?q=flin flon,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ducheng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #384: ducheng, cn
    http://api.openweathermap.org/data/2.5/weather?q=ducheng,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pisco,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #385: pisco, pe
    http://api.openweathermap.org/data/2.5/weather?q=pisco,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shush,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #386: shush, ir
    http://api.openweathermap.org/data/2.5/weather?q=shush,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chenghai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #387: chenghai, cn
    http://api.openweathermap.org/data/2.5/weather?q=chenghai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pacific grove,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #388: pacific grove, us
    http://api.openweathermap.org/data/2.5/weather?q=pacific grove,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shimoda,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #390: shimoda, jp
    http://api.openweathermap.org/data/2.5/weather?q=shimoda,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lighthouse point,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #391: lighthouse point, us
    http://api.openweathermap.org/data/2.5/weather?q=lighthouse point,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rorvik,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #392: rorvik, no
    http://api.openweathermap.org/data/2.5/weather?q=rorvik,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #393: iqaluit, ca
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mumford,gh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #394: mumford, gh
    http://api.openweathermap.org/data/2.5/weather?q=mumford,gh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=soe,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #395: soe, id
    http://api.openweathermap.org/data/2.5/weather?q=soe,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kieta,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #396: kieta, pg
    http://api.openweathermap.org/data/2.5/weather?q=kieta,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bachatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #397: bachatskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=bachatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kampot,kh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #398: kampot, kh
    http://api.openweathermap.org/data/2.5/weather?q=kampot,kh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #399: coquimbo, cl
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #400: sola, vu
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=asosa,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #401: asosa, et
    http://api.openweathermap.org/data/2.5/weather?q=asosa,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=henties bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #402: henties bay, na
    http://api.openweathermap.org/data/2.5/weather?q=henties bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hervey bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #403: hervey bay, au
    http://api.openweathermap.org/data/2.5/weather?q=hervey bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nantong,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #404: nantong, cn
    http://api.openweathermap.org/data/2.5/weather?q=nantong,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tabou,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #405: tabou, ci
    http://api.openweathermap.org/data/2.5/weather?q=tabou,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=abha,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #406: abha, sa
    http://api.openweathermap.org/data/2.5/weather?q=abha,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #407: provideniya, ru
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=laguna de perlas,ni&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #408: laguna de perlas, ni
    http://api.openweathermap.org/data/2.5/weather?q=laguna de perlas,ni&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carthage,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #409: carthage, us
    http://api.openweathermap.org/data/2.5/weather?q=carthage,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ballina,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #410: ballina, au
    http://api.openweathermap.org/data/2.5/weather?q=ballina,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oxbow,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #411: oxbow, ca
    http://api.openweathermap.org/data/2.5/weather?q=oxbow,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=corning,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #412: corning, us
    http://api.openweathermap.org/data/2.5/weather?q=corning,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bollnas,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #413: bollnas, se
    http://api.openweathermap.org/data/2.5/weather?q=bollnas,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=along,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #415: along, in
    http://api.openweathermap.org/data/2.5/weather?q=along,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zaozhuang,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #416: zaozhuang, cn
    http://api.openweathermap.org/data/2.5/weather?q=zaozhuang,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thurso,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #417: thurso, gb
    http://api.openweathermap.org/data/2.5/weather?q=thurso,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jasper,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #418: jasper, ca
    http://api.openweathermap.org/data/2.5/weather?q=jasper,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=andijon,uz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #419: andijon, uz
    http://api.openweathermap.org/data/2.5/weather?q=andijon,uz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tual,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #420: tual, id
    http://api.openweathermap.org/data/2.5/weather?q=tual,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=manacapuru,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #421: manacapuru, br
    http://api.openweathermap.org/data/2.5/weather?q=manacapuru,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #422: bathsheba, bb
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=karasuyama,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #423: karasuyama, jp
    http://api.openweathermap.org/data/2.5/weather?q=karasuyama,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gilgit,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #424: gilgit, pk
    http://api.openweathermap.org/data/2.5/weather?q=gilgit,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arcata,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #425: arcata, us
    http://api.openweathermap.org/data/2.5/weather?q=arcata,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sobolevo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #426: sobolevo, ru
    http://api.openweathermap.org/data/2.5/weather?q=sobolevo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=poya,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #427: poya, nc
    http://api.openweathermap.org/data/2.5/weather?q=poya,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #428: honiara, sb
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #429: constitucion, cl
    http://api.openweathermap.org/data/2.5/weather?q=constitucion,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #430: saldanha, za
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=xining,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #431: xining, cn
    http://api.openweathermap.org/data/2.5/weather?q=xining,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oume,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #432: oume, ci
    http://api.openweathermap.org/data/2.5/weather?q=oume,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=annonay,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #433: annonay, fr
    http://api.openweathermap.org/data/2.5/weather?q=annonay,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=warmbad,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #434: warmbad, na
    http://api.openweathermap.org/data/2.5/weather?q=warmbad,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kuty,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #437: kuty, ua
    http://api.openweathermap.org/data/2.5/weather?q=kuty,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sinnai,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #438: sinnai, it
    http://api.openweathermap.org/data/2.5/weather?q=sinnai,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=doha,qa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #442: doha, qa
    http://api.openweathermap.org/data/2.5/weather?q=doha,qa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=farafangana,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #443: farafangana, mg
    http://api.openweathermap.org/data/2.5/weather?q=farafangana,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lagunas,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #444: lagunas, pe
    http://api.openweathermap.org/data/2.5/weather?q=lagunas,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oistins,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #446: oistins, bb
    http://api.openweathermap.org/data/2.5/weather?q=oistins,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mao,td&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #447: mao, td
    http://api.openweathermap.org/data/2.5/weather?q=mao,td&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hihya,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #448: hihya, eg
    http://api.openweathermap.org/data/2.5/weather?q=hihya,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aripuana,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #449: aripuana, br
    http://api.openweathermap.org/data/2.5/weather?q=aripuana,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dong hoi,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #450: dong hoi, vn
    http://api.openweathermap.org/data/2.5/weather?q=dong hoi,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aklavik,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #452: aklavik, ca
    http://api.openweathermap.org/data/2.5/weather?q=aklavik,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=portland,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #453: portland, au
    http://api.openweathermap.org/data/2.5/weather?q=portland,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=paita,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #454: paita, pe
    http://api.openweathermap.org/data/2.5/weather?q=paita,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=araxa,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #455: araxa, br
    http://api.openweathermap.org/data/2.5/weather?q=araxa,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=roswell,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #456: roswell, us
    http://api.openweathermap.org/data/2.5/weather?q=roswell,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nouakchott,mr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #457: nouakchott, mr
    http://api.openweathermap.org/data/2.5/weather?q=nouakchott,mr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jacareacanga,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #458: jacareacanga, br
    http://api.openweathermap.org/data/2.5/weather?q=jacareacanga,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #459: sorland, no
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gondanglegi,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #460: gondanglegi, id
    http://api.openweathermap.org/data/2.5/weather?q=gondanglegi,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rathenow,de&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #461: rathenow, de
    http://api.openweathermap.org/data/2.5/weather?q=rathenow,de&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tessalit,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #462: tessalit, ml
    http://api.openweathermap.org/data/2.5/weather?q=tessalit,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #463: ambilobe, mg
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=toucheng,tw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #464: toucheng, tw
    http://api.openweathermap.org/data/2.5/weather?q=toucheng,tw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=beckley,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #465: beckley, us
    http://api.openweathermap.org/data/2.5/weather?q=beckley,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zhuhai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #466: zhuhai, cn
    http://api.openweathermap.org/data/2.5/weather?q=zhuhai,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=new iberia,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #467: new iberia, us
    http://api.openweathermap.org/data/2.5/weather?q=new iberia,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sidi ali,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #469: sidi ali, dz
    http://api.openweathermap.org/data/2.5/weather?q=sidi ali,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lengshuitan,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #471: lengshuitan, cn
    http://api.openweathermap.org/data/2.5/weather?q=lengshuitan,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nizhniy tsasuchey,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #472: nizhniy tsasuchey, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhniy tsasuchey,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=paamiut,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #474: paamiut, gl
    http://api.openweathermap.org/data/2.5/weather?q=paamiut,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nueva concepcion,gt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #475: nueva concepcion, gt
    http://api.openweathermap.org/data/2.5/weather?q=nueva concepcion,gt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shitanjing,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #476: shitanjing, cn
    http://api.openweathermap.org/data/2.5/weather?q=shitanjing,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cravo norte,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #477: cravo norte, co
    http://api.openweathermap.org/data/2.5/weather?q=cravo norte,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bonthe,sl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #478: bonthe, sl
    http://api.openweathermap.org/data/2.5/weather?q=bonthe,sl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gunnedah,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #479: gunnedah, au
    http://api.openweathermap.org/data/2.5/weather?q=gunnedah,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aranos,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #480: aranos, na
    http://api.openweathermap.org/data/2.5/weather?q=aranos,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovsk-kamchatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #481: petropavlovsk-kamchatskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovsk-kamchatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=senanga,zm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #482: senanga, zm
    http://api.openweathermap.org/data/2.5/weather?q=senanga,zm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jega,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #484: jega, ng
    http://api.openweathermap.org/data/2.5/weather?q=jega,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cotonou,bj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #485: cotonou, bj
    http://api.openweathermap.org/data/2.5/weather?q=cotonou,bj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tecoanapa,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #486: tecoanapa, mx
    http://api.openweathermap.org/data/2.5/weather?q=tecoanapa,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #487: souillac, mu
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=podgornoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #490: podgornoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=podgornoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ponta delgada,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #491: ponta delgada, pt
    http://api.openweathermap.org/data/2.5/weather?q=ponta delgada,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kortessem,be&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #493: kortessem, be
    http://api.openweathermap.org/data/2.5/weather?q=kortessem,be&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=belmonte,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #495: belmonte, br
    http://api.openweathermap.org/data/2.5/weather?q=belmonte,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lorengau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #498: lorengau, pg
    http://api.openweathermap.org/data/2.5/weather?q=lorengau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=road town,vg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #499: road town, vg
    http://api.openweathermap.org/data/2.5/weather?q=road town,vg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=trofors,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #500: trofors, no
    http://api.openweathermap.org/data/2.5/weather?q=trofors,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=touba,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #501: touba, ci
    http://api.openweathermap.org/data/2.5/weather?q=touba,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=moyo,ug&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #502: moyo, ug
    http://api.openweathermap.org/data/2.5/weather?q=moyo,ug&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=leh,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #503: leh, in
    http://api.openweathermap.org/data/2.5/weather?q=leh,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #504: mayo, ca
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=riberalta,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #505: riberalta, bo
    http://api.openweathermap.org/data/2.5/weather?q=riberalta,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=honningsvag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #506: honningsvag, no
    http://api.openweathermap.org/data/2.5/weather?q=honningsvag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=riverton,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #508: riverton, nz
    http://api.openweathermap.org/data/2.5/weather?q=riverton,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dalbandin,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #509: dalbandin, pk
    http://api.openweathermap.org/data/2.5/weather?q=dalbandin,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-louis,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #510: saint-louis, sn
    http://api.openweathermap.org/data/2.5/weather?q=saint-louis,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hanstholm,dk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #511: hanstholm, dk
    http://api.openweathermap.org/data/2.5/weather?q=hanstholm,dk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lagoa,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #512: lagoa, pt
    http://api.openweathermap.org/data/2.5/weather?q=lagoa,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ola,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #513: ola, ru
    http://api.openweathermap.org/data/2.5/weather?q=ola,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=te anau,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #514: te anau, nz
    http://api.openweathermap.org/data/2.5/weather?q=te anau,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=riyadh,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #515: riyadh, sa
    http://api.openweathermap.org/data/2.5/weather?q=riyadh,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=monroe,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #516: monroe, us
    http://api.openweathermap.org/data/2.5/weather?q=monroe,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    ---------------------------------
    Data Retrieval Complete
    ---------------------------------
    




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
      <th>City Name</th>
      <th>Country Code</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Temperature (F)</th>
      <th>Humidity (%)</th>
      <th>Cloudiness (%)</th>
      <th>Wind Speed (mph)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>neiafu</td>
      <td>to</td>
      <td>-18.65</td>
      <td>-173.98</td>
      <td>82.40</td>
      <td>83.0</td>
      <td>40.0</td>
      <td>10.29</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ushuaia</td>
      <td>ar</td>
      <td>-54.81</td>
      <td>-68.31</td>
      <td>30.20</td>
      <td>86.0</td>
      <td>75.0</td>
      <td>24.16</td>
    </tr>
    <tr>
      <th>2</th>
      <td>port blair</td>
      <td>in</td>
      <td>11.67</td>
      <td>92.75</td>
      <td>82.08</td>
      <td>100.0</td>
      <td>56.0</td>
      <td>16.15</td>
    </tr>
    <tr>
      <th>3</th>
      <td>puerto ayora</td>
      <td>ec</td>
      <td>-0.74</td>
      <td>-90.35</td>
      <td>71.37</td>
      <td>100.0</td>
      <td>20.0</td>
      <td>11.45</td>
    </tr>
    <tr>
      <th>5</th>
      <td>kununurra</td>
      <td>au</td>
      <td>-15.77</td>
      <td>128.74</td>
      <td>80.60</td>
      <td>12.0</td>
      <td>8.0</td>
      <td>13.87</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Exporting DataFrame
cities_df.to_csv("Output/weather_data.csv")
```

# Latitude vs. Temperature Plot 


```python
plt.scatter(cities_df["Latitude"], 
            cities_df["Temperature (F)"], c=cities_df["Temperature (F)"],
            edgecolor="black", linewidths=1, marker="o", 
            cmap='plasma', alpha=0.8, label="City")

# Incorporate the other graph properties
plt.style.use('seaborn')
plt.title(f"City Latitude vs. Max Temperature {datetime.now().strftime('%m/%d/%Y')}")
plt.ylabel("Max Temperature (F)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim([-80, 100])
plt.ylim([-60, 125])

# Save the figure
plt.savefig("Weather_Analysis/Latitude_Temperature.png")

# Show plot
plt.show()
```


![png](output_9_0.png)


# Lattitude vs Humidity 


```python
plt.scatter(cities_df["Latitude"], 
            cities_df["Humidity (%)"], c=cities_df["Humidity (%)"],
            edgecolor="black", linewidths=1, marker="o", 
            cmap='plasma', alpha=0.8, label="City")

# Incorporate the other graph properties
plt.style.use('seaborn')
plt.title(f"City Latitude vs. Humidity {datetime.now().strftime('%m/%d/%Y')}")
plt.ylabel("Humidity (%)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim([-80, 100])
plt.ylim([-60, 125])

# Save the figure
plt.savefig("Weather_Analysis/Latitude_Humidity.png")

# Show plot
plt.show()
```


![png](output_11_0.png)


# City Latitude vs Cloudiness (%)


```python
plt.scatter(cities_df["Latitude"], 
            cities_df["Cloudiness (%)"], c=cities_df["Cloudiness (%)"],
            edgecolor="black", linewidths=1, marker="o", 
            cmap='plasma', alpha=0.8, label="City")

# Incorporate the other graph properties
plt.style.use('seaborn')
plt.title(f"City Latitude vs. Cloudiness (%) {datetime.now().strftime('%m/%d/%Y')}")
plt.ylabel("Cloudiness (%)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim([-80, 100])
plt.ylim([-60, 125])

# Save the figure
plt.savefig("Weather_Analysis/Latitude_Cloudiness.png")

# Show plot
plt.show()
```


![png](output_13_0.png)


# Wind Speed (mph) vs. Latitude



```python
#Build a scatter plot for Latitude vs. Wind Speed (mph)

plt.scatter(cities_df["Latitude"], 
            cities_df["Wind Speed (mph)"], c=cities_df["Wind Speed (mph)"],
            edgecolor="black", linewidths=1, marker="o", 
            cmap='plasma', alpha=0.8, label="City")

# Incorporate the other graph properties
plt.style.use('seaborn')
plt.title(f"City Latitude vs. Wind Speed (mph) {datetime.now().strftime('%m/%d/%Y')}")
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid(True)
plt.xlim([-80, 100])
plt.ylim([-10, 50])

# Save the figure
plt.savefig("Weather_Analysis/Latitude_WindSpeed.png")

# Show plot
plt.show()
```


![png](output_15_0.png)

