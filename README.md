
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
      <td>rikitea</td>
      <td>pf</td>
    </tr>
    <tr>
      <th>1</th>
      <td>srednekolymsk</td>
      <td>ru</td>
    </tr>
    <tr>
      <th>2</th>
      <td>carnarvon</td>
      <td>au</td>
    </tr>
    <tr>
      <th>3</th>
      <td>louisbourg</td>
      <td>ca</td>
    </tr>
    <tr>
      <th>4</th>
      <td>yulara</td>
      <td>au</td>
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
      <td>rikitea</td>
      <td>pf</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>srednekolymsk</td>
      <td>ru</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>carnarvon</td>
      <td>au</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>louisbourg</td>
      <td>ca</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>yulara</td>
      <td>au</td>
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
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #0: rikitea, pf
    http://api.openweathermap.org/data/2.5/weather?q=rikitea,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=srednekolymsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #1: srednekolymsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=srednekolymsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #2: carnarvon, au
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=louisbourg,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #3: louisbourg, ca
    http://api.openweathermap.org/data/2.5/weather?q=louisbourg,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #4: yulara, au
    http://api.openweathermap.org/data/2.5/weather?q=yulara,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #5: belushya guba, ru
    http://api.openweathermap.org/data/2.5/weather?q=belushya guba,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=noumea,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #6: noumea, nc
    http://api.openweathermap.org/data/2.5/weather?q=noumea,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mittagong,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #7: mittagong, au
    http://api.openweathermap.org/data/2.5/weather?q=mittagong,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #8: busselton, au
    http://api.openweathermap.org/data/2.5/weather?q=busselton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lorengau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #9: lorengau, pg
    http://api.openweathermap.org/data/2.5/weather?q=lorengau,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=petrozavodsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #10: petrozavodsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=petrozavodsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #11: avarua, ck
    http://api.openweathermap.org/data/2.5/weather?q=avarua,ck&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=one hundred mile house,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #12: one hundred mile house, ca
    http://api.openweathermap.org/data/2.5/weather?q=one hundred mile house,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #13: albany, au
    http://api.openweathermap.org/data/2.5/weather?q=albany,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #14: barentsburg, sj
    http://api.openweathermap.org/data/2.5/weather?q=barentsburg,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #15: kapaa, us
    http://api.openweathermap.org/data/2.5/weather?q=kapaa,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #16: nikolskoye, ru
    http://api.openweathermap.org/data/2.5/weather?q=nikolskoye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lubao,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #17: lubao, cd
    http://api.openweathermap.org/data/2.5/weather?q=lubao,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=keuruu,fi&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #18: keuruu, fi
    http://api.openweathermap.org/data/2.5/weather?q=keuruu,fi&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aguilas,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #19: aguilas, es
    http://api.openweathermap.org/data/2.5/weather?q=aguilas,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #20: vaini, to
    http://api.openweathermap.org/data/2.5/weather?q=vaini,to&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #21: hermanus, za
    http://api.openweathermap.org/data/2.5/weather?q=hermanus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #22: bluff, nz
    http://api.openweathermap.org/data/2.5/weather?q=bluff,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #23: ushuaia, ar
    http://api.openweathermap.org/data/2.5/weather?q=ushuaia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #24: georgetown, sh
    http://api.openweathermap.org/data/2.5/weather?q=georgetown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mataura,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #25: mataura, pf
    http://api.openweathermap.org/data/2.5/weather?q=mataura,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=san patricio,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #26: san patricio, mx
    http://api.openweathermap.org/data/2.5/weather?q=san patricio,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nioki,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #27: nioki, cd
    http://api.openweathermap.org/data/2.5/weather?q=nioki,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #28: punta arenas, cl
    http://api.openweathermap.org/data/2.5/weather?q=punta arenas,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=marcona,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #29: marcona, pe
    http://api.openweathermap.org/data/2.5/weather?q=marcona,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=fez,ma&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #30: fez, ma
    http://api.openweathermap.org/data/2.5/weather?q=fez,ma&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #31: coquimbo, cl
    http://api.openweathermap.org/data/2.5/weather?q=coquimbo,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #32: geraldton, au
    http://api.openweathermap.org/data/2.5/weather?q=geraldton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=avera,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #33: avera, pf
    http://api.openweathermap.org/data/2.5/weather?q=avera,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #34: puerto ayora, ec
    http://api.openweathermap.org/data/2.5/weather?q=puerto ayora,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=muisne,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #35: muisne, ec
    http://api.openweathermap.org/data/2.5/weather?q=muisne,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #36: new norfolk, au
    http://api.openweathermap.org/data/2.5/weather?q=new norfolk,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #37: bredasdorp, za
    http://api.openweathermap.org/data/2.5/weather?q=bredasdorp,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #38: hithadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=hithadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sinor,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #39: sinor, in
    http://api.openweathermap.org/data/2.5/weather?q=sinor,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=grindavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #40: grindavik, is
    http://api.openweathermap.org/data/2.5/weather?q=grindavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #41: atuona, pf
    http://api.openweathermap.org/data/2.5/weather?q=atuona,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #42: katsuura, jp
    http://api.openweathermap.org/data/2.5/weather?q=katsuura,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=southbridge,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #43: southbridge, nz
    http://api.openweathermap.org/data/2.5/weather?q=southbridge,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=east london,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #44: east london, za
    http://api.openweathermap.org/data/2.5/weather?q=east london,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #45: isangel, vu
    http://api.openweathermap.org/data/2.5/weather?q=isangel,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #46: illoqqortoormiut, gl
    http://api.openweathermap.org/data/2.5/weather?q=illoqqortoormiut,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #47: talnakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=talnakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #48: chuy, uy
    http://api.openweathermap.org/data/2.5/weather?q=chuy,uy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #49: cidreira, br
    http://api.openweathermap.org/data/2.5/weather?q=cidreira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pishin,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #50: pishin, pk
    http://api.openweathermap.org/data/2.5/weather?q=pishin,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #51: ribeira grande, pt
    http://api.openweathermap.org/data/2.5/weather?q=ribeira grande,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ruse,bg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #52: ruse, bg
    http://api.openweathermap.org/data/2.5/weather?q=ruse,bg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dubbo,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #53: dubbo, au
    http://api.openweathermap.org/data/2.5/weather?q=dubbo,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port hardy,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #54: port hardy, ca
    http://api.openweathermap.org/data/2.5/weather?q=port hardy,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=padang,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #55: padang, id
    http://api.openweathermap.org/data/2.5/weather?q=padang,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port alfred,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #56: port alfred, za
    http://api.openweathermap.org/data/2.5/weather?q=port alfred,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jalu,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #57: jalu, ly
    http://api.openweathermap.org/data/2.5/weather?q=jalu,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ravar,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #58: ravar, ir
    http://api.openweathermap.org/data/2.5/weather?q=ravar,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bondo,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #59: bondo, cd
    http://api.openweathermap.org/data/2.5/weather?q=bondo,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #60: hobart, au
    http://api.openweathermap.org/data/2.5/weather?q=hobart,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gat,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #61: gat, ly
    http://api.openweathermap.org/data/2.5/weather?q=gat,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #62: dingle, ie
    http://api.openweathermap.org/data/2.5/weather?q=dingle,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tsihombe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #63: tsihombe, mg
    http://api.openweathermap.org/data/2.5/weather?q=tsihombe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #64: yellowknife, ca
    http://api.openweathermap.org/data/2.5/weather?q=yellowknife,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #65: jamestown, sh
    http://api.openweathermap.org/data/2.5/weather?q=jamestown,sh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=stornoway,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #66: stornoway, gb
    http://api.openweathermap.org/data/2.5/weather?q=stornoway,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=marsa matruh,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #67: marsa matruh, eg
    http://api.openweathermap.org/data/2.5/weather?q=marsa matruh,eg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=acarau,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #68: acarau, br
    http://api.openweathermap.org/data/2.5/weather?q=acarau,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=olinda,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #69: olinda, br
    http://api.openweathermap.org/data/2.5/weather?q=olinda,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #70: ponta do sol, cv
    http://api.openweathermap.org/data/2.5/weather?q=ponta do sol,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #71: nemuro, jp
    http://api.openweathermap.org/data/2.5/weather?q=nemuro,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=morshyn,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #72: morshyn, ua
    http://api.openweathermap.org/data/2.5/weather?q=morshyn,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #73: dikson, ru
    http://api.openweathermap.org/data/2.5/weather?q=dikson,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arani,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #74: arani, bo
    http://api.openweathermap.org/data/2.5/weather?q=arani,bo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #75: mar del plata, ar
    http://api.openweathermap.org/data/2.5/weather?q=mar del plata,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #76: nome, us
    http://api.openweathermap.org/data/2.5/weather?q=nome,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #77: bathsheba, bb
    http://api.openweathermap.org/data/2.5/weather?q=bathsheba,bb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mount gambier,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #78: mount gambier, au
    http://api.openweathermap.org/data/2.5/weather?q=mount gambier,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #79: sentyabrskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=sentyabrskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #80: cockburn town, bs
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,bs&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cape town,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #81: cape town, za
    http://api.openweathermap.org/data/2.5/weather?q=cape town,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #82: bengkulu, id
    http://api.openweathermap.org/data/2.5/weather?q=bengkulu,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #83: namatanai, pg
    http://api.openweathermap.org/data/2.5/weather?q=namatanai,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=airai,pw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #84: airai, pw
    http://api.openweathermap.org/data/2.5/weather?q=airai,pw&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #85: nizhneyansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=nizhneyansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=aflu,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #86: aflu, dz
    http://api.openweathermap.org/data/2.5/weather?q=aflu,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=jablah,sy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #87: jablah, sy
    http://api.openweathermap.org/data/2.5/weather?q=jablah,sy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=the valley,ai&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #88: the valley, ai
    http://api.openweathermap.org/data/2.5/weather?q=the valley,ai&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #89: mahebourg, mu
    http://api.openweathermap.org/data/2.5/weather?q=mahebourg,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kostroma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #90: kostroma, ru
    http://api.openweathermap.org/data/2.5/weather?q=kostroma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pacific grove,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #91: pacific grove, us
    http://api.openweathermap.org/data/2.5/weather?q=pacific grove,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tagusao,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #92: tagusao, ph
    http://api.openweathermap.org/data/2.5/weather?q=tagusao,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kasongo-lunda,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #93: kasongo-lunda, cd
    http://api.openweathermap.org/data/2.5/weather?q=kasongo-lunda,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arys,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #94: arys, kz
    http://api.openweathermap.org/data/2.5/weather?q=arys,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=jaleswar,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #95: jaleswar, in
    http://api.openweathermap.org/data/2.5/weather?q=jaleswar,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-georges,gf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #96: saint-georges, gf
    http://api.openweathermap.org/data/2.5/weather?q=saint-georges,gf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=broken hill,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #97: broken hill, au
    http://api.openweathermap.org/data/2.5/weather?q=broken hill,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #98: severo-kurilsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=severo-kurilsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #99: kodiak, us
    http://api.openweathermap.org/data/2.5/weather?q=kodiak,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=prince rupert,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #100: prince rupert, ca
    http://api.openweathermap.org/data/2.5/weather?q=prince rupert,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=guatire,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #101: guatire, ve
    http://api.openweathermap.org/data/2.5/weather?q=guatire,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=muyezerskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #102: muyezerskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=muyezerskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=utica,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #103: utica, us
    http://api.openweathermap.org/data/2.5/weather?q=utica,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=teguldet,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #104: teguldet, ru
    http://api.openweathermap.org/data/2.5/weather?q=teguldet,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hambantota,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #105: hambantota, lk
    http://api.openweathermap.org/data/2.5/weather?q=hambantota,lk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hanmer springs,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #106: hanmer springs, nz
    http://api.openweathermap.org/data/2.5/weather?q=hanmer springs,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #107: leningradskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=leningradskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #108: tuktoyaktuk, ca
    http://api.openweathermap.org/data/2.5/weather?q=tuktoyaktuk,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=markapur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #109: markapur, in
    http://api.openweathermap.org/data/2.5/weather?q=markapur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=opuwo,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #110: opuwo, na
    http://api.openweathermap.org/data/2.5/weather?q=opuwo,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lahaina,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #111: lahaina, us
    http://api.openweathermap.org/data/2.5/weather?q=lahaina,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #112: pevek, ru
    http://api.openweathermap.org/data/2.5/weather?q=pevek,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=malatya,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #113: malatya, tr
    http://api.openweathermap.org/data/2.5/weather?q=malatya,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=olga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #114: olga, ru
    http://api.openweathermap.org/data/2.5/weather?q=olga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=narsaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #115: narsaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=narsaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #116: vao, nc
    http://api.openweathermap.org/data/2.5/weather?q=vao,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=raudeberg,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #117: raudeberg, no
    http://api.openweathermap.org/data/2.5/weather?q=raudeberg,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #118: tasiilaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=tasiilaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=norman wells,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #119: norman wells, ca
    http://api.openweathermap.org/data/2.5/weather?q=norman wells,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=orissaare,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #120: orissaare, ee
    http://api.openweathermap.org/data/2.5/weather?q=orissaare,ee&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lac-megantic,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #121: lac-megantic, ca
    http://api.openweathermap.org/data/2.5/weather?q=lac-megantic,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=adzope,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #122: adzope, ci
    http://api.openweathermap.org/data/2.5/weather?q=adzope,ci&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #123: lebu, cl
    http://api.openweathermap.org/data/2.5/weather?q=lebu,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #124: barrow, us
    http://api.openweathermap.org/data/2.5/weather?q=barrow,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #125: kaitangata, nz
    http://api.openweathermap.org/data/2.5/weather?q=kaitangata,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=waipawa,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #126: waipawa, nz
    http://api.openweathermap.org/data/2.5/weather?q=waipawa,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #127: taolanaro, mg
    http://api.openweathermap.org/data/2.5/weather?q=taolanaro,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=ust-kulom,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #128: ust-kulom, ru
    http://api.openweathermap.org/data/2.5/weather?q=ust-kulom,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=beloha,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #129: beloha, mg
    http://api.openweathermap.org/data/2.5/weather?q=beloha,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=garmsar,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #130: garmsar, ir
    http://api.openweathermap.org/data/2.5/weather?q=garmsar,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=comodoro rivadavia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #131: comodoro rivadavia, ar
    http://api.openweathermap.org/data/2.5/weather?q=comodoro rivadavia,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ulaanbaatar,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #132: ulaanbaatar, mn
    http://api.openweathermap.org/data/2.5/weather?q=ulaanbaatar,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #133: honiara, sb
    http://api.openweathermap.org/data/2.5/weather?q=honiara,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=taguatinga,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #134: taguatinga, br
    http://api.openweathermap.org/data/2.5/weather?q=taguatinga,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cabra,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #135: cabra, ph
    http://api.openweathermap.org/data/2.5/weather?q=cabra,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #136: nanortalik, gl
    http://api.openweathermap.org/data/2.5/weather?q=nanortalik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #137: arraial do cabo, br
    http://api.openweathermap.org/data/2.5/weather?q=arraial do cabo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chagoda,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #138: chagoda, ru
    http://api.openweathermap.org/data/2.5/weather?q=chagoda,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=banyuwangi,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #139: banyuwangi, id
    http://api.openweathermap.org/data/2.5/weather?q=banyuwangi,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tuatapere,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #140: tuatapere, nz
    http://api.openweathermap.org/data/2.5/weather?q=tuatapere,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tiznit,ma&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #141: tiznit, ma
    http://api.openweathermap.org/data/2.5/weather?q=tiznit,ma&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=luanda,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #142: luanda, ao
    http://api.openweathermap.org/data/2.5/weather?q=luanda,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=charters towers,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #143: charters towers, au
    http://api.openweathermap.org/data/2.5/weather?q=charters towers,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wuwei,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #144: wuwei, cn
    http://api.openweathermap.org/data/2.5/weather?q=wuwei,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=berlevag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #145: berlevag, no
    http://api.openweathermap.org/data/2.5/weather?q=berlevag,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dasoguz,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #146: dasoguz, tm
    http://api.openweathermap.org/data/2.5/weather?q=dasoguz,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #147: qaanaaq, gl
    http://api.openweathermap.org/data/2.5/weather?q=qaanaaq,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mbour,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #148: mbour, sn
    http://api.openweathermap.org/data/2.5/weather?q=mbour,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nabire,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #149: nabire, id
    http://api.openweathermap.org/data/2.5/weather?q=nabire,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=walvis bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #150: walvis bay, na
    http://api.openweathermap.org/data/2.5/weather?q=walvis bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=college,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #151: college, us
    http://api.openweathermap.org/data/2.5/weather?q=college,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #152: saint-philippe, re
    http://api.openweathermap.org/data/2.5/weather?q=saint-philippe,re&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kamaishi,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #153: kamaishi, jp
    http://api.openweathermap.org/data/2.5/weather?q=kamaishi,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=slave lake,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #154: slave lake, ca
    http://api.openweathermap.org/data/2.5/weather?q=slave lake,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alghero,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #155: alghero, it
    http://api.openweathermap.org/data/2.5/weather?q=alghero,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #156: hilo, us
    http://api.openweathermap.org/data/2.5/weather?q=hilo,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kaeo,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #157: kaeo, nz
    http://api.openweathermap.org/data/2.5/weather?q=kaeo,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vanavara,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #158: vanavara, ru
    http://api.openweathermap.org/data/2.5/weather?q=vanavara,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=talara,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #159: talara, pe
    http://api.openweathermap.org/data/2.5/weather?q=talara,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #160: beringovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=beringovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yanan,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #161: yanan, cn
    http://api.openweathermap.org/data/2.5/weather?q=yanan,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=gornopravdinsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #162: gornopravdinsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=gornopravdinsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=brandon,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #163: brandon, us
    http://api.openweathermap.org/data/2.5/weather?q=brandon,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=victor harbor,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #164: victor harbor, au
    http://api.openweathermap.org/data/2.5/weather?q=victor harbor,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kharp,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #165: kharp, ru
    http://api.openweathermap.org/data/2.5/weather?q=kharp,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=koratla,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #166: koratla, in
    http://api.openweathermap.org/data/2.5/weather?q=koratla,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=uthal,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #167: uthal, pk
    http://api.openweathermap.org/data/2.5/weather?q=uthal,pk&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sunrise manor,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #168: sunrise manor, us
    http://api.openweathermap.org/data/2.5/weather?q=sunrise manor,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #169: butaritari, ki
    http://api.openweathermap.org/data/2.5/weather?q=butaritari,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ust-maya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #170: ust-maya, ru
    http://api.openweathermap.org/data/2.5/weather?q=ust-maya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #171: esperance, au
    http://api.openweathermap.org/data/2.5/weather?q=esperance,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ergani,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #172: ergani, tr
    http://api.openweathermap.org/data/2.5/weather?q=ergani,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=redencao,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #173: redencao, br
    http://api.openweathermap.org/data/2.5/weather?q=redencao,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #174: cherskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=cherskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=atlantis,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #175: atlantis, za
    http://api.openweathermap.org/data/2.5/weather?q=atlantis,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=morehead,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #176: morehead, pg
    http://api.openweathermap.org/data/2.5/weather?q=morehead,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ewa beach,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #177: ewa beach, us
    http://api.openweathermap.org/data/2.5/weather?q=ewa beach,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint george,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #178: saint george, bm
    http://api.openweathermap.org/data/2.5/weather?q=saint george,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kontagora,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #179: kontagora, ng
    http://api.openweathermap.org/data/2.5/weather?q=kontagora,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #180: klaksvik, fo
    http://api.openweathermap.org/data/2.5/weather?q=klaksvik,fo&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ayagoz,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #181: ayagoz, kz
    http://api.openweathermap.org/data/2.5/weather?q=ayagoz,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #182: saskylakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=saskylakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ustye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #183: ustye, ru
    http://api.openweathermap.org/data/2.5/weather?q=ustye,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=djougou,bj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #184: djougou, bj
    http://api.openweathermap.org/data/2.5/weather?q=djougou,bj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #185: kruisfontein, za
    http://api.openweathermap.org/data/2.5/weather?q=kruisfontein,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovsk-kamchatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #186: petropavlovsk-kamchatskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=petropavlovsk-kamchatskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bantry,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #187: bantry, ie
    http://api.openweathermap.org/data/2.5/weather?q=bantry,ie&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=boo,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #188: boo, se
    http://api.openweathermap.org/data/2.5/weather?q=boo,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tumannyy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #189: tumannyy, ru
    http://api.openweathermap.org/data/2.5/weather?q=tumannyy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #190: bambous virieux, mu
    http://api.openweathermap.org/data/2.5/weather?q=bambous virieux,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kenai,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #191: kenai, us
    http://api.openweathermap.org/data/2.5/weather?q=kenai,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aracaju,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #192: aracaju, br
    http://api.openweathermap.org/data/2.5/weather?q=aracaju,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=celestun,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #193: celestun, mx
    http://api.openweathermap.org/data/2.5/weather?q=celestun,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=samusu,ws&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #194: samusu, ws
    http://api.openweathermap.org/data/2.5/weather?q=samusu,ws&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=zeya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #195: zeya, ru
    http://api.openweathermap.org/data/2.5/weather?q=zeya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vyazma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #196: vyazma, ru
    http://api.openweathermap.org/data/2.5/weather?q=vyazma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dwarka,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #197: dwarka, in
    http://api.openweathermap.org/data/2.5/weather?q=dwarka,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dauphin,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #198: dauphin, ca
    http://api.openweathermap.org/data/2.5/weather?q=dauphin,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=olafsvik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #199: olafsvik, is
    http://api.openweathermap.org/data/2.5/weather?q=olafsvik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=kisangani,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #200: kisangani, cd
    http://api.openweathermap.org/data/2.5/weather?q=kisangani,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=clyde river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #201: clyde river, ca
    http://api.openweathermap.org/data/2.5/weather?q=clyde river,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chagda,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #202: chagda, ru
    http://api.openweathermap.org/data/2.5/weather?q=chagda,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #203: ambilobe, mg
    http://api.openweathermap.org/data/2.5/weather?q=ambilobe,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #204: namibe, ao
    http://api.openweathermap.org/data/2.5/weather?q=namibe,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=formoso do araguaia,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #205: formoso do araguaia, br
    http://api.openweathermap.org/data/2.5/weather?q=formoso do araguaia,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=amderma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #206: amderma, ru
    http://api.openweathermap.org/data/2.5/weather?q=amderma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #207: cabo san lucas, mx
    http://api.openweathermap.org/data/2.5/weather?q=cabo san lucas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bezhetsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #208: bezhetsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=bezhetsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tucuman,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #209: tucuman, ar
    http://api.openweathermap.org/data/2.5/weather?q=tucuman,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=poya,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #210: poya, nc
    http://api.openweathermap.org/data/2.5/weather?q=poya,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=grand river south east,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #211: grand river south east, mu
    http://api.openweathermap.org/data/2.5/weather?q=grand river south east,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=teya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #212: teya, ru
    http://api.openweathermap.org/data/2.5/weather?q=teya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yerbogachen,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #213: yerbogachen, ru
    http://api.openweathermap.org/data/2.5/weather?q=yerbogachen,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #214: faanui, pf
    http://api.openweathermap.org/data/2.5/weather?q=faanui,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hakkari,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #215: hakkari, tr
    http://api.openweathermap.org/data/2.5/weather?q=hakkari,tr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=oga,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #216: oga, jp
    http://api.openweathermap.org/data/2.5/weather?q=oga,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=asau,tv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #217: asau, tv
    http://api.openweathermap.org/data/2.5/weather?q=asau,tv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=richard toll,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #218: richard toll, sn
    http://api.openweathermap.org/data/2.5/weather?q=richard toll,sn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=manta,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #219: manta, ec
    http://api.openweathermap.org/data/2.5/weather?q=manta,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #220: viedma, ar
    http://api.openweathermap.org/data/2.5/weather?q=viedma,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #221: tiksi, ru
    http://api.openweathermap.org/data/2.5/weather?q=tiksi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ereymentau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #222: ereymentau, kz
    http://api.openweathermap.org/data/2.5/weather?q=ereymentau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sakakah,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #223: sakakah, sa
    http://api.openweathermap.org/data/2.5/weather?q=sakakah,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,tc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #224: cockburn town, tc
    http://api.openweathermap.org/data/2.5/weather?q=cockburn town,tc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #225: castro, cl
    http://api.openweathermap.org/data/2.5/weather?q=castro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chicama,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #226: chicama, pe
    http://api.openweathermap.org/data/2.5/weather?q=chicama,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=outjo,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #227: outjo, na
    http://api.openweathermap.org/data/2.5/weather?q=outjo,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=andevoranto,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #228: andevoranto, mg
    http://api.openweathermap.org/data/2.5/weather?q=andevoranto,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=socorro,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #229: socorro, us
    http://api.openweathermap.org/data/2.5/weather?q=socorro,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kirakira,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #230: kirakira, sb
    http://api.openweathermap.org/data/2.5/weather?q=kirakira,sb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=columbus,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #231: columbus, us
    http://api.openweathermap.org/data/2.5/weather?q=columbus,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=camacha,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #232: camacha, pt
    http://api.openweathermap.org/data/2.5/weather?q=camacha,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carinhanha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #233: carinhanha, br
    http://api.openweathermap.org/data/2.5/weather?q=carinhanha,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #234: ilulissat, gl
    http://api.openweathermap.org/data/2.5/weather?q=ilulissat,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #235: thompson, ca
    http://api.openweathermap.org/data/2.5/weather?q=thompson,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bridlington,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #236: bridlington, gb
    http://api.openweathermap.org/data/2.5/weather?q=bridlington,gb&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mundybash,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #237: mundybash, ru
    http://api.openweathermap.org/data/2.5/weather?q=mundybash,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=anadyr,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #238: anadyr, ru
    http://api.openweathermap.org/data/2.5/weather?q=anadyr,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=teixeira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #239: teixeira, br
    http://api.openweathermap.org/data/2.5/weather?q=teixeira,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #240: torbay, ca
    http://api.openweathermap.org/data/2.5/weather?q=torbay,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vardo,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #241: vardo, no
    http://api.openweathermap.org/data/2.5/weather?q=vardo,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=meulaboh,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #242: meulaboh, id
    http://api.openweathermap.org/data/2.5/weather?q=meulaboh,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=toliary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #243: toliary, mg
    http://api.openweathermap.org/data/2.5/weather?q=toliary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=mae ramat,th&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #244: mae ramat, th
    http://api.openweathermap.org/data/2.5/weather?q=mae ramat,th&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao lourenco do sul,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #245: sao lourenco do sul, br
    http://api.openweathermap.org/data/2.5/weather?q=sao lourenco do sul,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san buenaventura,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #246: san buenaventura, mx
    http://api.openweathermap.org/data/2.5/weather?q=san buenaventura,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mingshui,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #247: mingshui, cn
    http://api.openweathermap.org/data/2.5/weather?q=mingshui,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=muros,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #248: muros, es
    http://api.openweathermap.org/data/2.5/weather?q=muros,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=voh,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #249: voh, nc
    http://api.openweathermap.org/data/2.5/weather?q=voh,nc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #250: adrar, dz
    http://api.openweathermap.org/data/2.5/weather?q=adrar,dz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #251: victoria, sc
    http://api.openweathermap.org/data/2.5/weather?q=victoria,sc&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lodeynoye pole,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #252: lodeynoye pole, ru
    http://api.openweathermap.org/data/2.5/weather?q=lodeynoye pole,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ulladulla,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #253: ulladulla, au
    http://api.openweathermap.org/data/2.5/weather?q=ulladulla,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sirur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #254: sirur, in
    http://api.openweathermap.org/data/2.5/weather?q=sirur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=te anau,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #255: te anau, nz
    http://api.openweathermap.org/data/2.5/weather?q=te anau,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=abadan,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #256: abadan, ir
    http://api.openweathermap.org/data/2.5/weather?q=abadan,ir&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mayumba,ga&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #257: mayumba, ga
    http://api.openweathermap.org/data/2.5/weather?q=mayumba,ga&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aitape,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #258: aitape, pg
    http://api.openweathermap.org/data/2.5/weather?q=aitape,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bonavista,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #259: bonavista, ca
    http://api.openweathermap.org/data/2.5/weather?q=bonavista,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=porbandar,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #260: porbandar, in
    http://api.openweathermap.org/data/2.5/weather?q=porbandar,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=umuarama,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #261: umuarama, br
    http://api.openweathermap.org/data/2.5/weather?q=umuarama,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=diego de almagro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #262: diego de almagro, cl
    http://api.openweathermap.org/data/2.5/weather?q=diego de almagro,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=holme,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #263: holme, no
    http://api.openweathermap.org/data/2.5/weather?q=holme,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=najran,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #264: najran, sa
    http://api.openweathermap.org/data/2.5/weather?q=najran,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dzhusaly,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #265: dzhusaly, kz
    http://api.openweathermap.org/data/2.5/weather?q=dzhusaly,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #266: port elizabeth, za
    http://api.openweathermap.org/data/2.5/weather?q=port elizabeth,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alice springs,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #267: alice springs, au
    http://api.openweathermap.org/data/2.5/weather?q=alice springs,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=basco,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #268: basco, ph
    http://api.openweathermap.org/data/2.5/weather?q=basco,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=attawapiskat,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #269: attawapiskat, ca
    http://api.openweathermap.org/data/2.5/weather?q=attawapiskat,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=balimo,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #270: balimo, pg
    http://api.openweathermap.org/data/2.5/weather?q=balimo,pg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=portmore,jm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #271: portmore, jm
    http://api.openweathermap.org/data/2.5/weather?q=portmore,jm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #272: iqaluit, ca
    http://api.openweathermap.org/data/2.5/weather?q=iqaluit,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=road town,vg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #273: road town, vg
    http://api.openweathermap.org/data/2.5/weather?q=road town,vg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #274: sitka, us
    http://api.openweathermap.org/data/2.5/weather?q=sitka,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tarquinia,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #275: tarquinia, it
    http://api.openweathermap.org/data/2.5/weather?q=tarquinia,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #276: khatanga, ru
    http://api.openweathermap.org/data/2.5/weather?q=khatanga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kavaratti,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #277: kavaratti, in
    http://api.openweathermap.org/data/2.5/weather?q=kavaratti,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=victoria,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #278: victoria, us
    http://api.openweathermap.org/data/2.5/weather?q=victoria,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port blair,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #279: port blair, in
    http://api.openweathermap.org/data/2.5/weather?q=port blair,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kamenskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #280: kamenskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=kamenskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=andenes,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #281: andenes, no
    http://api.openweathermap.org/data/2.5/weather?q=andenes,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pochutla,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #282: pochutla, mx
    http://api.openweathermap.org/data/2.5/weather?q=pochutla,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=miranda de ebro,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #283: miranda de ebro, es
    http://api.openweathermap.org/data/2.5/weather?q=miranda de ebro,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bousso,td&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #284: bousso, td
    http://api.openweathermap.org/data/2.5/weather?q=bousso,td&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=khani,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #285: khani, ru
    http://api.openweathermap.org/data/2.5/weather?q=khani,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #286: san cristobal, ec
    http://api.openweathermap.org/data/2.5/weather?q=san cristobal,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #287: husavik, is
    http://api.openweathermap.org/data/2.5/weather?q=husavik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #288: sorland, no
    http://api.openweathermap.org/data/2.5/weather?q=sorland,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tucurui,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #289: tucurui, br
    http://api.openweathermap.org/data/2.5/weather?q=tucurui,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rokytne,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #290: rokytne, ua
    http://api.openweathermap.org/data/2.5/weather?q=rokytne,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lagoa,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #291: lagoa, pt
    http://api.openweathermap.org/data/2.5/weather?q=lagoa,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #292: alofi, nu
    http://api.openweathermap.org/data/2.5/weather?q=alofi,nu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mende,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #293: mende, fr
    http://api.openweathermap.org/data/2.5/weather?q=mende,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fukue,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #294: fukue, jp
    http://api.openweathermap.org/data/2.5/weather?q=fukue,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #295: kahului, us
    http://api.openweathermap.org/data/2.5/weather?q=kahului,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kosh-agach,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #296: kosh-agach, ru
    http://api.openweathermap.org/data/2.5/weather?q=kosh-agach,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #297: touros, br
    http://api.openweathermap.org/data/2.5/weather?q=touros,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chore,py&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #298: chore, py
    http://api.openweathermap.org/data/2.5/weather?q=chore,py&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=singleton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #299: singleton, au
    http://api.openweathermap.org/data/2.5/weather?q=singleton,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=weiser,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #300: weiser, us
    http://api.openweathermap.org/data/2.5/weather?q=weiser,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #301: hasaki, jp
    http://api.openweathermap.org/data/2.5/weather?q=hasaki,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=falun,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #302: falun, se
    http://api.openweathermap.org/data/2.5/weather?q=falun,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #303: upernavik, gl
    http://api.openweathermap.org/data/2.5/weather?q=upernavik,gl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=santa elena,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #304: santa elena, ec
    http://api.openweathermap.org/data/2.5/weather?q=santa elena,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ritchie,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #305: ritchie, za
    http://api.openweathermap.org/data/2.5/weather?q=ritchie,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dalvik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #306: dalvik, is
    http://api.openweathermap.org/data/2.5/weather?q=dalvik,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=arbuzynka,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #307: arbuzynka, ua
    http://api.openweathermap.org/data/2.5/weather?q=arbuzynka,ua&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #308: pangnirtung, ca
    http://api.openweathermap.org/data/2.5/weather?q=pangnirtung,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pamyat parizhskoy kommuny,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #309: pamyat parizhskoy kommuny, ru
    http://api.openweathermap.org/data/2.5/weather?q=pamyat parizhskoy kommuny,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=praia da vitoria,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #310: praia da vitoria, pt
    http://api.openweathermap.org/data/2.5/weather?q=praia da vitoria,pt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #311: guerrero negro, mx
    http://api.openweathermap.org/data/2.5/weather?q=guerrero negro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=batemans bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #312: batemans bay, au
    http://api.openweathermap.org/data/2.5/weather?q=batemans bay,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #313: souillac, mu
    http://api.openweathermap.org/data/2.5/weather?q=souillac,mu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #314: sola, vu
    http://api.openweathermap.org/data/2.5/weather?q=sola,vu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ligayan,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #315: ligayan, ph
    http://api.openweathermap.org/data/2.5/weather?q=ligayan,ph&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rawannawi,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #316: rawannawi, ki
    http://api.openweathermap.org/data/2.5/weather?q=rawannawi,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=tarko-sale,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #317: tarko-sale, ru
    http://api.openweathermap.org/data/2.5/weather?q=tarko-sale,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kokino,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #318: kokino, ru
    http://api.openweathermap.org/data/2.5/weather?q=kokino,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=amiens,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #319: amiens, fr
    http://api.openweathermap.org/data/2.5/weather?q=amiens,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nanakuli,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #320: nanakuli, us
    http://api.openweathermap.org/data/2.5/weather?q=nanakuli,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nowy dwor gdanski,pl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #321: nowy dwor gdanski, pl
    http://api.openweathermap.org/data/2.5/weather?q=nowy dwor gdanski,pl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=anloga,gh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #322: anloga, gh
    http://api.openweathermap.org/data/2.5/weather?q=anloga,gh&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lanja,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #323: lanja, in
    http://api.openweathermap.org/data/2.5/weather?q=lanja,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #324: chokurdakh, ru
    http://api.openweathermap.org/data/2.5/weather?q=chokurdakh,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cururupu,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #325: cururupu, br
    http://api.openweathermap.org/data/2.5/weather?q=cururupu,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vaitupu,wf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #326: vaitupu, wf
    http://api.openweathermap.org/data/2.5/weather?q=vaitupu,wf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #327: ostrovnoy, ru
    http://api.openweathermap.org/data/2.5/weather?q=ostrovnoy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=atambua,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #328: atambua, id
    http://api.openweathermap.org/data/2.5/weather?q=atambua,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=umm lajj,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #329: umm lajj, sa
    http://api.openweathermap.org/data/2.5/weather?q=umm lajj,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #330: longyearbyen, sj
    http://api.openweathermap.org/data/2.5/weather?q=longyearbyen,sj&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #331: lompoc, us
    http://api.openweathermap.org/data/2.5/weather?q=lompoc,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=aktau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #332: aktau, kz
    http://api.openweathermap.org/data/2.5/weather?q=aktau,kz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thanh hoa,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #333: thanh hoa, vn
    http://api.openweathermap.org/data/2.5/weather?q=thanh hoa,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=henties bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #334: henties bay, na
    http://api.openweathermap.org/data/2.5/weather?q=henties bay,na&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=peleduy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #335: peleduy, ru
    http://api.openweathermap.org/data/2.5/weather?q=peleduy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=seoul,kr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #336: seoul, kr
    http://api.openweathermap.org/data/2.5/weather?q=seoul,kr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=damara,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #337: damara, cf
    http://api.openweathermap.org/data/2.5/weather?q=damara,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ajdabiya,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #338: ajdabiya, ly
    http://api.openweathermap.org/data/2.5/weather?q=ajdabiya,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port-gentil,ga&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #339: port-gentil, ga
    http://api.openweathermap.org/data/2.5/weather?q=port-gentil,ga&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mount isa,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #340: mount isa, au
    http://api.openweathermap.org/data/2.5/weather?q=mount isa,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bac lieu,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #341: bac lieu, vn
    http://api.openweathermap.org/data/2.5/weather?q=bac lieu,vn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=rodrigues alves,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #342: rodrigues alves, br
    http://api.openweathermap.org/data/2.5/weather?q=rodrigues alves,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=portland,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #343: portland, au
    http://api.openweathermap.org/data/2.5/weather?q=portland,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=amboasary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #344: amboasary, mg
    http://api.openweathermap.org/data/2.5/weather?q=amboasary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=westport,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #345: westport, nz
    http://api.openweathermap.org/data/2.5/weather?q=westport,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rio grande,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #346: rio grande, br
    http://api.openweathermap.org/data/2.5/weather?q=rio grande,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pizarro,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #347: pizarro, co
    http://api.openweathermap.org/data/2.5/weather?q=pizarro,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=loreto,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #348: loreto, mx
    http://api.openweathermap.org/data/2.5/weather?q=loreto,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nisia floresta,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #349: nisia floresta, br
    http://api.openweathermap.org/data/2.5/weather?q=nisia floresta,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tucupita,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #350: tucupita, ve
    http://api.openweathermap.org/data/2.5/weather?q=tucupita,ve&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #351: vestmannaeyjar, is
    http://api.openweathermap.org/data/2.5/weather?q=vestmannaeyjar,is&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #352: flinders, au
    http://api.openweathermap.org/data/2.5/weather?q=flinders,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pelham,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #353: pelham, us
    http://api.openweathermap.org/data/2.5/weather?q=pelham,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=vedea,ro&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #354: vedea, ro
    http://api.openweathermap.org/data/2.5/weather?q=vedea,ro&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=junqueiro,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #355: junqueiro, br
    http://api.openweathermap.org/data/2.5/weather?q=junqueiro,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #356: sao joao da barra, br
    http://api.openweathermap.org/data/2.5/weather?q=sao joao da barra,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=grimshaw,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #357: grimshaw, ca
    http://api.openweathermap.org/data/2.5/weather?q=grimshaw,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #358: mys shmidta, ru
    http://api.openweathermap.org/data/2.5/weather?q=mys shmidta,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=tateyama,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #359: tateyama, jp
    http://api.openweathermap.org/data/2.5/weather?q=tateyama,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tabiauea,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #360: tabiauea, ki
    http://api.openweathermap.org/data/2.5/weather?q=tabiauea,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=ariquemes,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #361: ariquemes, br
    http://api.openweathermap.org/data/2.5/weather?q=ariquemes,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=marzuq,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #362: marzuq, ly
    http://api.openweathermap.org/data/2.5/weather?q=marzuq,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=tabuk,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #363: tabuk, sa
    http://api.openweathermap.org/data/2.5/weather?q=tabuk,sa&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lazaro cardenas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #364: lazaro cardenas, mx
    http://api.openweathermap.org/data/2.5/weather?q=lazaro cardenas,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lanzhou,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #365: lanzhou, cn
    http://api.openweathermap.org/data/2.5/weather?q=lanzhou,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kamien pomorski,pl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #366: kamien pomorski, pl
    http://api.openweathermap.org/data/2.5/weather?q=kamien pomorski,pl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=rezzato,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #367: rezzato, it
    http://api.openweathermap.org/data/2.5/weather?q=rezzato,it&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mantsala,fi&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #368: mantsala, fi
    http://api.openweathermap.org/data/2.5/weather?q=mantsala,fi&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=antofagasta,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #369: antofagasta, cl
    http://api.openweathermap.org/data/2.5/weather?q=antofagasta,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ciras,af&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #370: ciras, af
    http://api.openweathermap.org/data/2.5/weather?q=ciras,af&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=carauari,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #371: carauari, br
    http://api.openweathermap.org/data/2.5/weather?q=carauari,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=linxia,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #372: linxia, cn
    http://api.openweathermap.org/data/2.5/weather?q=linxia,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kuching,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #373: kuching, my
    http://api.openweathermap.org/data/2.5/weather?q=kuching,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=port hope,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #374: port hope, ca
    http://api.openweathermap.org/data/2.5/weather?q=port hope,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bahia honda,cu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #375: bahia honda, cu
    http://api.openweathermap.org/data/2.5/weather?q=bahia honda,cu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=blois,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #376: blois, fr
    http://api.openweathermap.org/data/2.5/weather?q=blois,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=potgietersrus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #377: potgietersrus, za
    http://api.openweathermap.org/data/2.5/weather?q=potgietersrus,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=fairbanks,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #378: fairbanks, us
    http://api.openweathermap.org/data/2.5/weather?q=fairbanks,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #379: saldanha, za
    http://api.openweathermap.org/data/2.5/weather?q=saldanha,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wanning,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #380: wanning, cn
    http://api.openweathermap.org/data/2.5/weather?q=wanning,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bartica,gy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #381: bartica, gy
    http://api.openweathermap.org/data/2.5/weather?q=bartica,gy&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=atar,mr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #382: atar, mr
    http://api.openweathermap.org/data/2.5/weather?q=atar,mr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mehamn,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #383: mehamn, no
    http://api.openweathermap.org/data/2.5/weather?q=mehamn,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=eyl,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #384: eyl, so
    http://api.openweathermap.org/data/2.5/weather?q=eyl,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=antalaha,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #385: antalaha, mg
    http://api.openweathermap.org/data/2.5/weather?q=antalaha,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=gander,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #386: gander, ca
    http://api.openweathermap.org/data/2.5/weather?q=gander,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=key west,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #387: key west, us
    http://api.openweathermap.org/data/2.5/weather?q=key west,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kaikalur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #388: kaikalur, in
    http://api.openweathermap.org/data/2.5/weather?q=kaikalur,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=baykit,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #389: baykit, ru
    http://api.openweathermap.org/data/2.5/weather?q=baykit,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=thinadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #390: thinadhoo, mv
    http://api.openweathermap.org/data/2.5/weather?q=thinadhoo,mv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yeppoon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #391: yeppoon, au
    http://api.openweathermap.org/data/2.5/weather?q=yeppoon,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=buldana,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #392: buldana, in
    http://api.openweathermap.org/data/2.5/weather?q=buldana,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=batagay,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #393: batagay, ru
    http://api.openweathermap.org/data/2.5/weather?q=batagay,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mungaa,tz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #394: mungaa, tz
    http://api.openweathermap.org/data/2.5/weather?q=mungaa,tz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=semporna,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #395: semporna, my
    http://api.openweathermap.org/data/2.5/weather?q=semporna,my&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ventspils,lv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #396: ventspils, lv
    http://api.openweathermap.org/data/2.5/weather?q=ventspils,lv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=garowe,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #397: garowe, so
    http://api.openweathermap.org/data/2.5/weather?q=garowe,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ekhabi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #398: ekhabi, ru
    http://api.openweathermap.org/data/2.5/weather?q=ekhabi,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=adra,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #399: adra, es
    http://api.openweathermap.org/data/2.5/weather?q=adra,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kushima,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #400: kushima, jp
    http://api.openweathermap.org/data/2.5/weather?q=kushima,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=turukhansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #401: turukhansk, ru
    http://api.openweathermap.org/data/2.5/weather?q=turukhansk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=okhotsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #402: okhotsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=okhotsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sobolevo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #403: sobolevo, ru
    http://api.openweathermap.org/data/2.5/weather?q=sobolevo,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=banda aceh,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #404: banda aceh, id
    http://api.openweathermap.org/data/2.5/weather?q=banda aceh,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hede,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #405: hede, cn
    http://api.openweathermap.org/data/2.5/weather?q=hede,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fare,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #406: fare, pf
    http://api.openweathermap.org/data/2.5/weather?q=fare,pf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=qinhuangdao,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #407: qinhuangdao, cn
    http://api.openweathermap.org/data/2.5/weather?q=qinhuangdao,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=skelleftea,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #408: skelleftea, se
    http://api.openweathermap.org/data/2.5/weather?q=skelleftea,se&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=porto walter,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #409: porto walter, br
    http://api.openweathermap.org/data/2.5/weather?q=porto walter,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mutsamudu,km&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #410: mutsamudu, km
    http://api.openweathermap.org/data/2.5/weather?q=mutsamudu,km&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=coihaique,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #411: coihaique, cl
    http://api.openweathermap.org/data/2.5/weather?q=coihaique,cl&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yarmouth,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #412: yarmouth, ca
    http://api.openweathermap.org/data/2.5/weather?q=yarmouth,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ijaki,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #413: ijaki, ki
    http://api.openweathermap.org/data/2.5/weather?q=ijaki,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=canberra,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #414: canberra, au
    http://api.openweathermap.org/data/2.5/weather?q=canberra,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=wanaka,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #415: wanaka, nz
    http://api.openweathermap.org/data/2.5/weather?q=wanaka,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=pokhara,np&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #416: pokhara, np
    http://api.openweathermap.org/data/2.5/weather?q=pokhara,np&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dingzhou,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #417: dingzhou, cn
    http://api.openweathermap.org/data/2.5/weather?q=dingzhou,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san andres,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #418: san andres, co
    http://api.openweathermap.org/data/2.5/weather?q=san andres,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=amazar,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #419: amazar, ru
    http://api.openweathermap.org/data/2.5/weather?q=amazar,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mango,tg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #420: mango, tg
    http://api.openweathermap.org/data/2.5/weather?q=mango,tg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=halalo,wf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #421: halalo, wf
    http://api.openweathermap.org/data/2.5/weather?q=halalo,wf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=mandvi,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #422: mandvi, in
    http://api.openweathermap.org/data/2.5/weather?q=mandvi,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=yakeshi,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #423: yakeshi, cn
    http://api.openweathermap.org/data/2.5/weather?q=yakeshi,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #424: sao filipe, cv
    http://api.openweathermap.org/data/2.5/weather?q=sao filipe,cv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=soyo,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #425: soyo, ao
    http://api.openweathermap.org/data/2.5/weather?q=soyo,ao&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=iskateley,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #426: iskateley, ru
    http://api.openweathermap.org/data/2.5/weather?q=iskateley,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=karamay,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #427: karamay, cn
    http://api.openweathermap.org/data/2.5/weather?q=karamay,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=bara,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #428: bara, sd
    http://api.openweathermap.org/data/2.5/weather?q=bara,sd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=gardan diwal,af&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #429: gardan diwal, af
    http://api.openweathermap.org/data/2.5/weather?q=gardan diwal,af&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=murgab,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #430: murgab, tm
    http://api.openweathermap.org/data/2.5/weather?q=murgab,tm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=guisa,cu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #431: guisa, cu
    http://api.openweathermap.org/data/2.5/weather?q=guisa,cu&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #432: bethel, us
    http://api.openweathermap.org/data/2.5/weather?q=bethel,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mahadday weyne,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #433: mahadday weyne, so
    http://api.openweathermap.org/data/2.5/weather?q=mahadday weyne,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=eseka,cm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #434: eseka, cm
    http://api.openweathermap.org/data/2.5/weather?q=eseka,cm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saleaula,ws&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #435: saleaula, ws
    http://api.openweathermap.org/data/2.5/weather?q=saleaula,ws&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=haines junction,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #436: haines junction, ca
    http://api.openweathermap.org/data/2.5/weather?q=haines junction,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ambon,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #437: ambon, id
    http://api.openweathermap.org/data/2.5/weather?q=ambon,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sidi bu zayd,tn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #438: sidi bu zayd, tn
    http://api.openweathermap.org/data/2.5/weather?q=sidi bu zayd,tn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=todos santos,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #439: todos santos, mx
    http://api.openweathermap.org/data/2.5/weather?q=todos santos,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lunenburg,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #440: lunenburg, ca
    http://api.openweathermap.org/data/2.5/weather?q=lunenburg,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bemidji,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #441: bemidji, us
    http://api.openweathermap.org/data/2.5/weather?q=bemidji,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=erdenet,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #442: erdenet, mn
    http://api.openweathermap.org/data/2.5/weather?q=erdenet,mn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ploemeur,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #443: ploemeur, fr
    http://api.openweathermap.org/data/2.5/weather?q=ploemeur,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tual,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #444: tual, id
    http://api.openweathermap.org/data/2.5/weather?q=tual,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=temaraia,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #445: temaraia, ki
    http://api.openweathermap.org/data/2.5/weather?q=temaraia,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=kaduy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #446: kaduy, ru
    http://api.openweathermap.org/data/2.5/weather?q=kaduy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sistranda,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #447: sistranda, no
    http://api.openweathermap.org/data/2.5/weather?q=sistranda,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=zarubino,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #448: zarubino, ru
    http://api.openweathermap.org/data/2.5/weather?q=zarubino,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lima,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #449: lima, pe
    http://api.openweathermap.org/data/2.5/weather?q=lima,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=san jose,gt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #450: san jose, gt
    http://api.openweathermap.org/data/2.5/weather?q=san jose,gt&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=santa rosalia,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #451: santa rosalia, mx
    http://api.openweathermap.org/data/2.5/weather?q=santa rosalia,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lasa,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #452: lasa, cn
    http://api.openweathermap.org/data/2.5/weather?q=lasa,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=dunedin,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #453: dunedin, nz
    http://api.openweathermap.org/data/2.5/weather?q=dunedin,nz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #454: provideniya, ru
    http://api.openweathermap.org/data/2.5/weather?q=provideniya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=imeni zhelyabova,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #455: imeni zhelyabova, ru
    http://api.openweathermap.org/data/2.5/weather?q=imeni zhelyabova,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hamilton,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #456: hamilton, bm
    http://api.openweathermap.org/data/2.5/weather?q=hamilton,bm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=saint anthony,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #457: saint anthony, ca
    http://api.openweathermap.org/data/2.5/weather?q=saint anthony,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=utiroa,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #458: utiroa, ki
    http://api.openweathermap.org/data/2.5/weather?q=utiroa,ki&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=armacao dos buzios,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #459: armacao dos buzios, br
    http://api.openweathermap.org/data/2.5/weather?q=armacao dos buzios,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=abashiri,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #460: abashiri, jp
    http://api.openweathermap.org/data/2.5/weather?q=abashiri,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mandera,ke&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #461: mandera, ke
    http://api.openweathermap.org/data/2.5/weather?q=mandera,ke&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=sharan,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #462: sharan, ru
    http://api.openweathermap.org/data/2.5/weather?q=sharan,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=inhambane,mz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #463: inhambane, mz
    http://api.openweathermap.org/data/2.5/weather?q=inhambane,mz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tukrah,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #464: tukrah, ly
    http://api.openweathermap.org/data/2.5/weather?q=tukrah,ly&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #465: carnarvon, za
    http://api.openweathermap.org/data/2.5/weather?q=carnarvon,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ternate,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #466: ternate, id
    http://api.openweathermap.org/data/2.5/weather?q=ternate,id&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=katherine,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #467: katherine, au
    http://api.openweathermap.org/data/2.5/weather?q=katherine,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lolua,tv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #468: lolua, tv
    http://api.openweathermap.org/data/2.5/weather?q=lolua,tv&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=papasquiaro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #469: papasquiaro, mx
    http://api.openweathermap.org/data/2.5/weather?q=papasquiaro,mx&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=sheboygan,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #470: sheboygan, us
    http://api.openweathermap.org/data/2.5/weather?q=sheboygan,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=ouadda,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #471: ouadda, cf
    http://api.openweathermap.org/data/2.5/weather?q=ouadda,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=anshun,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #472: anshun, cn
    http://api.openweathermap.org/data/2.5/weather?q=anshun,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=naze,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #473: naze, jp
    http://api.openweathermap.org/data/2.5/weather?q=naze,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=abalak,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #474: abalak, ne
    http://api.openweathermap.org/data/2.5/weather?q=abalak,ne&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=odweyne,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #475: odweyne, so
    http://api.openweathermap.org/data/2.5/weather?q=odweyne,so&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=codrington,ag&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #476: codrington, ag
    http://api.openweathermap.org/data/2.5/weather?q=codrington,ag&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=mapiripan,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #477: mapiripan, co
    http://api.openweathermap.org/data/2.5/weather?q=mapiripan,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=barra do garcas,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #478: barra do garcas, br
    http://api.openweathermap.org/data/2.5/weather?q=barra do garcas,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=kabo,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #479: kabo, cf
    http://api.openweathermap.org/data/2.5/weather?q=kabo,cf&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tazovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #480: tazovskiy, ru
    http://api.openweathermap.org/data/2.5/weather?q=tazovskiy,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=grand forks,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #481: grand forks, ca
    http://api.openweathermap.org/data/2.5/weather?q=grand forks,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=angoche,mz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #482: angoche, mz
    http://api.openweathermap.org/data/2.5/weather?q=angoche,mz&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=carbondale,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #483: carbondale, us
    http://api.openweathermap.org/data/2.5/weather?q=carbondale,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nara,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #484: nara, ml
    http://api.openweathermap.org/data/2.5/weather?q=nara,ml&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=catamarca,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #485: catamarca, ar
    http://api.openweathermap.org/data/2.5/weather?q=catamarca,ar&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=daura,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #486: daura, ng
    http://api.openweathermap.org/data/2.5/weather?q=daura,ng&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=krasnoselkup,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #487: krasnoselkup, ru
    http://api.openweathermap.org/data/2.5/weather?q=krasnoselkup,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=ballina,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #488: ballina, au
    http://api.openweathermap.org/data/2.5/weather?q=ballina,au&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lalibela,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #489: lalibela, et
    http://api.openweathermap.org/data/2.5/weather?q=lalibela,et&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=conakry,gn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #490: conakry, gn
    http://api.openweathermap.org/data/2.5/weather?q=conakry,gn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=labutta,mm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #491: labutta, mm
    http://api.openweathermap.org/data/2.5/weather?q=labutta,mm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=pak phanang,th&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #492: pak phanang, th
    http://api.openweathermap.org/data/2.5/weather?q=pak phanang,th&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=hammerfest,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #493: hammerfest, no
    http://api.openweathermap.org/data/2.5/weather?q=hammerfest,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=houma,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #494: houma, us
    http://api.openweathermap.org/data/2.5/weather?q=houma,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lamu,ke&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #495: lamu, ke
    http://api.openweathermap.org/data/2.5/weather?q=lamu,ke&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=dzhebariki-khaya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #496: dzhebariki-khaya, ru
    http://api.openweathermap.org/data/2.5/weather?q=dzhebariki-khaya,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=tashtyp,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #497: tashtyp, ru
    http://api.openweathermap.org/data/2.5/weather?q=tashtyp,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=novaya igirma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #498: novaya igirma, ru
    http://api.openweathermap.org/data/2.5/weather?q=novaya igirma,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bose,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #499: bose, cn
    http://api.openweathermap.org/data/2.5/weather?q=bose,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Error with city data. Skipping
    http://api.openweathermap.org/data/2.5/weather?q=oksfjord,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #500: oksfjord, no
    http://api.openweathermap.org/data/2.5/weather?q=oksfjord,no&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=humberto de campos,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #501: humberto de campos, br
    http://api.openweathermap.org/data/2.5/weather?q=humberto de campos,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=salinas,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #502: salinas, ec
    http://api.openweathermap.org/data/2.5/weather?q=salinas,ec&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #503: fortuna, us
    http://api.openweathermap.org/data/2.5/weather?q=fortuna,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=chengde,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #504: chengde, cn
    http://api.openweathermap.org/data/2.5/weather?q=chengde,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=juneau,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #505: juneau, us
    http://api.openweathermap.org/data/2.5/weather?q=juneau,us&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=toyooka,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #506: toyooka, jp
    http://api.openweathermap.org/data/2.5/weather?q=toyooka,jp&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=lukulu,zm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #507: lukulu, zm
    http://api.openweathermap.org/data/2.5/weather?q=lukulu,zm&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=huancavelica,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #508: huancavelica, pe
    http://api.openweathermap.org/data/2.5/weather?q=huancavelica,pe&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=millau,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #509: millau, fr
    http://api.openweathermap.org/data/2.5/weather?q=millau,fr&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=bijie,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #510: bijie, cn
    http://api.openweathermap.org/data/2.5/weather?q=bijie,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mwene-ditu,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #511: mwene-ditu, cd
    http://api.openweathermap.org/data/2.5/weather?q=mwene-ditu,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=usogorsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #512: usogorsk, ru
    http://api.openweathermap.org/data/2.5/weather?q=usogorsk,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mananjary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #513: mananjary, mg
    http://api.openweathermap.org/data/2.5/weather?q=mananjary,mg&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=richards bay,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #514: richards bay, za
    http://api.openweathermap.org/data/2.5/weather?q=richards bay,za&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=binga,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #515: binga, cd
    http://api.openweathermap.org/data/2.5/weather?q=binga,cd&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=khandyga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #516: khandyga, ru
    http://api.openweathermap.org/data/2.5/weather?q=khandyga,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=shenjiamen,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #517: shenjiamen, cn
    http://api.openweathermap.org/data/2.5/weather?q=shenjiamen,cn&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=valparai,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #518: valparai, in
    http://api.openweathermap.org/data/2.5/weather?q=valparai,in&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=okha,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #519: okha, ru
    http://api.openweathermap.org/data/2.5/weather?q=okha,ru&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=cabedelo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #520: cabedelo, br
    http://api.openweathermap.org/data/2.5/weather?q=cabedelo,br&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=nigran,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #521: nigran, es
    http://api.openweathermap.org/data/2.5/weather?q=nigran,es&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=la primavera,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #522: la primavera, co
    http://api.openweathermap.org/data/2.5/weather?q=la primavera,co&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
    Now retrieving City #523: mayo, ca
    http://api.openweathermap.org/data/2.5/weather?q=mayo,ca&units=imperial&appid=997ac1bb517def3f586c903aa0200103
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
      <td>rikitea</td>
      <td>pf</td>
      <td>-23.12</td>
      <td>-134.97</td>
      <td>70.41</td>
      <td>100.0</td>
      <td>92.0</td>
      <td>10.04</td>
    </tr>
    <tr>
      <th>1</th>
      <td>srednekolymsk</td>
      <td>ru</td>
      <td>67.46</td>
      <td>153.71</td>
      <td>76.62</td>
      <td>32.0</td>
      <td>12.0</td>
      <td>3.11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>carnarvon</td>
      <td>au</td>
      <td>-24.87</td>
      <td>113.63</td>
      <td>63.66</td>
      <td>90.0</td>
      <td>0.0</td>
      <td>15.79</td>
    </tr>
    <tr>
      <th>4</th>
      <td>yulara</td>
      <td>au</td>
      <td>-25.24</td>
      <td>130.99</td>
      <td>59.00</td>
      <td>41.0</td>
      <td>40.0</td>
      <td>9.17</td>
    </tr>
    <tr>
      <th>6</th>
      <td>noumea</td>
      <td>nc</td>
      <td>-22.28</td>
      <td>166.46</td>
      <td>73.40</td>
      <td>53.0</td>
      <td>75.0</td>
      <td>8.05</td>
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

