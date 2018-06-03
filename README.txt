Observations.
1. Latitude has a significant affect on the temperature though this clearly is not the only factor. Trends show that cities with the highest max temperature are more likely to be closer to or on the equator. 
2. The Latitude v Humidity table implies no direct correlation between temperatures and proximity to the equator however in more coastal areas, water vapour in the atmosphere increases with temperature and in desert regions, water vapour is not affected with temperature rises!
3. The Latitude v Wind Speed graph shows that most of the cities sampled are located in areas with wind speeds below 20mph. The randomly selected sample contains just 668 cities. It appears that wind speed does not affect temperatures though we would need to look at a larger sample for a more concrete observation.

#Dependencies
import seaborn as sns
iris = sns.load_dataset('iris')
from citipy import citipy
from config import api_key
import random
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import requests
import json

#Random generate latitude and longitude numbers then lookup nearest city using citipy

# Empty series to hold names of cities and countries
randCity = []
randCountry = []
loopcnt = 0

#Loop through untill 500 unique city names are found
#increase to 750. len is 438 using 500!
while len(randCity) < 750 :
    # Loop counter just to track number of loops made
    loopcnt +=1
    
    # Random generate latitude
    lat_pos = random.randint(0,2)
    base_lat = random.randint(0,91)
    dec_lat = random.random()/100
    lat = base_lat + dec_lat
    if lat_pos == 1:
        lat = lat * -1
    #print(f"The lattitude is: {lat}") 
    
    # Random generate longitude
    lng_pos = random.randint(0,2)
    base_lng = random.randint(0,181)
    dec_lng = random.random()/100
    lng = base_lng + dec_lng
    if lng_pos == 1:
        lng = lng * -1
    #print(f"The longitude is: {lng}") 
    
    # Call citipy to get nearest city based on random lat and lng
    city = citipy.nearest_city(lat, lng)
    
    # Test to see if the random city is already in the list, if not, add to the city and country lists
    foundCity = False
    for j in range(len(randCity)):
        if city.city_name == randCity[j]:
            foundCity = True
    if foundCity==False  :   
        randCity.append(city.city_name)
        randCountry.append(city.country_code)
        
        # Build query Url
base_url = "http://api.openweathermap.org/data/2.5/weather?"
api_key = api_key
units = "imperial"

query_url = base_url + "appid=" + api_key + "&units=" + units + "&q="

# Loop through the list of cities and perform a request for data on each
search_data = []
cityList = []
countryList = []

searchCnt = 0
print("Start API Call to OpenWeather Database ")
print("---------------------------------------")
for city in randCity:
    searchCnt += 1
    print(f"Processing record {searchCnt} for the city: {city}")   
    response = requests.get(query_url + city).json()
    searchCityID = response.get("id")
    if response.get("id"):
        print(f"     Record found for city: {city}    city id: {searchCityID}")
        search_data.append(response)
        cityList.append(city)
        countryList.append(randCountry[searchCnt-1])
    else:
        print(f"     No weather record found for city: {city}")
          
print("---------------------------------------")
print("End")

#print(json.dumps(search_data[2], indent=4))
len(search_data)

# Use list comprehension to gather necessary data series

lat_data = [data.get("coord").get("lat") for data in search_data]
lng_data = [data.get("coord").get("lon") for data in search_data]
temp_data = [data.get("main").get("temp") for data in search_data]
humid_data = [data.get("main").get("humidity") for data in search_data]
cloud_data =[data.get("clouds").get("all") for data in search_data]
wind_data = [data.get("wind").get("speed") for data in search_data]

# Create data frame with aggregated data
weather_data = pd.DataFrame({"cityName":cityList,
                            "country":countryList,
                            "lat":lat_data,
                            "lng":lng_data,
                            "temp":temp_data,
                            "humidity":humid_data,
                            "cloudiness":cloud_data,
                            "winds":wind_data})
                            
                            # change column order
weather_data = weather_data[["cityName","country","lat","lng","temp","humidity","cloudiness","winds"]]
weather_data.head()

# Save weather data to csv file
weather_data.to_csv("weather_data.csv", encoding="utf-8", index=False)

# Latitude vs Max Temp
sns.regplot(x='lat',y='temp', data=weather_data, scatter = True, fit_reg=False)

# Add chart labels
plt.title("City Latitude vs Max Temperature")
plt.ylabel("Temperature (F) ")
plt.xlabel("Latitude")
plt.grid(True)
#plt.xlim(-90,90)

# Save the figure
plt.savefig("Latitude_vs_MaxTemp.png")

# Latitude vs Humidity
sns.regplot(x='lat',y='humidity', data=weather_data, scatter = True, fit_reg=False)

# Add chart labels
plt.title("City Latitude vs Humidity")
plt.ylabel("Humidity")
plt.xlabel("Latitude")
plt.grid(True)
#plt.xlim(-90,90)

# Save the figure
plt.savefig("Latitude_vs_Humidity.png")

# Latitude vs Cloudiness
sns.regplot(x='lat',y='cloudiness', data=weather_data, scatter = True, fit_reg=False)

plt.title("City Latitude vs Cloudiness")
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid(True)

plt.savefig("Latitude_vs_Cloudiness.png")

# Latitude vs Wind Speed
sns.regplot(x='lat',y='winds', data=weather_data, scatter = True, fit_reg=False)

# Add chart labels
plt.title("City Latitude vs Wind Speed")
plt.ylabel("Wind Speed (mph)")
plt.xlabel("Latitude")
plt.grid(True)
#plt.xlim(-90,90)

# Save the figure
plt.savefig("Latitude_vs_WindSpeed.png")
