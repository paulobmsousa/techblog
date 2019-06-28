---
layout: post
title:  "Data Science Capstone"
date:   2019-06-27 09:38:58 -0400
categories: personal
permalink: "/datascience_capstone/"
---

![DevCorner][devblog2]

# Introduction

![Jekyll Logo][logo]

In this second post, I decided to reproduce the final project developed to the  **IBM Data Science Professional Certification**, using [Jupyter][jupyter-home], that, in short, is one of the simplest and powerfull tool to create blogs.

Firstly, one must consider the project that must be considered in the last challenge (also called "capestone"): it must contain: 

- The use of dataset, with [Pandas] [pandas-home] funcions, specially assembled from different sources;

- [Beautiful Soap] [beautifulsoap-home] application to extract information;

- [Foursquare] [foursquare-home] for location service;

- Finally, any kind of strategy to define a solution to a specific problem.

# Problem definition

**CONTEXTUALIZATION**

This case is based on a real case: One year ago, we decided to move to Canada. That time we needed to decide what city should be the better choice between two options: Toronto and Ottawa. Of course, several factors can be considered, including objective or subjetive ones. Science cannot deal with the last kind, but if we consider numerical values under the first aspect, maybe it is possible to have a diagnosis about the better choice.

In this work, I am using DATA SCIENCE, in order to colaborate with the decision that, in fact, we have made before. This tool can reinforce that when we consider personal aspects (most related with family and quality of life), our final decision was in fact correct.

**BACKGROUND**

This problem is a very common situation when we consider the doubt related with the best destiny for immigrant families that need to consider different aspects. The first problem is related with the factors that must be considered in order to get the result to the original problem.

In this situation, getting trusted and updated information is crucial. The second problem is that you need to have similar data from both options: having a detailed data of only one city is not fair and sometimes it is a challenge.

# Data description

**DATA SOURCE**

*Source 1: FourSquare*

As mentioned, similar data must be obtained. In fact, using the information from the laboratories in this specialization, I have got details about one city (**Toronto**). It should be relatively easy to have similar information about the other destiny (**Ottawa**).

In fact, only changing the information about the city, the libraries and services from **Foursquare** (https://api.foursquare.com/) were enough to get the complementary information about the Capital City, considering the main Venues obtained from the free service. It was the first criterion used in this comparison exercise.

*Source 2: Versus*

The second criterion was chosen by the information recorded at **Versus** (https://versus.com/en/ottawa-vs-toronto), when we have several indicators in order to compare both cities. However, in order to have a fair situation, both aspects: advantages from both cities were considered. In this situation, it was necessary to combine two tables in a single one, with permutation of some elements, in order to get a whole picture.

**USE OF THE INFORMATION**

If I just used the information without any particular criteria, I just have an equilibrium of the both cities. It looks like correct, but, in fact, it is not and the reason is because for this specific situation, some indicators are more relevant than others.

For instance, does not matter that both cities can have 100 venues in a specific perimeter: what is relevant is what of these places are considered relevant to my situation (family). At the same way, it is unfair that the 16 indicators collected in the Versus page are considered with the same weight: some of them are more important than others.

Only after this kind of preparation of the data, is that I could perform the analysis.

**DATA DESCRIPTION**

*Source 1: FourSquare*

After request the service for Venues (https://api.foursquare.com/v2/venues/explore), it is peformed a filter in order to use only the following information:

    - Venue name;
    - Venue category;
    - Venue location (latitude / longitude).

The category was specially important because it is used to mark some venues as important ("top") or not. It is also shown in the map of both cities.

*Source 2: Versus*

From this page, I used all the 16 initial indicators:

    - Population density;
    - Monthly public transport ticket;
    - Unemployment rate;
    - Cost of one-bedroom ap. in the center;
    - Younger population;
    - Income inequalities;
    - Landmarks from UNESCO World Heritage list;
    - Average minimum temperature;
    - Higher average maximum temperature;
    - Higher average temperature;
    - Million inhabitants;
    - Billion higher gross domestic product (GDP);
    - Higher average salary;
    - 2 more big sport facilities;
    - Cost of one single transportation ticket;
    - Billionaires from annual Forbes.

However, as mentioned above, only some of them were considered really important to our final decision.

# Methodology (Solution)

## General view

The methodology can be resumed in the following steps:

1. General preparation: When libraries, importation and functions are defined;

2. Data gathering and analysis: Using Foursquare and Versus information;

3. Conclusion: Combining the results in order to decide the best city.

Some aditional notes (detailed in the Solution description below):

A. It is important to see that the information had to be prepared and combined from the Versus data;

B. Method to extract numerical information from Versus was also implemented, in order to adjust the output of BeautifulSoup

## Step-by-step solution

### Step 1: General Preparation

#### Step_1.1: The first step is to install packages, specially related to pandas (dataframes) and geolocalization

{% highlight python %}
'''
# Uncomment to perform package installations
! pip install pandas
! pip install geopy
! pip install folium
'''
{% endhighlight %}

#### Step_1.2: The importation of the packages that will be used in this lab.

{% highlight python %}
# General frame libraries
import pandas as pd
# Soup libraries
from bs4 import BeautifulSoup
from pandas.compat import StringIO
import json # library to handle JSON files
import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe
# Maps library
import folium
from geopy.geocoders import Nominatim # module to convert an address into latitude and longitude values
from IPython.display import Image 
from IPython.core.display import HTML 
# Plot library
import numpy as np
import matplotlib.pyplot as plt
{% endhighlight %}

#### Step_1.3: General configuration (that will be used by functions).

Pandas and Matplot configuration:

{% highlight python %}
# Pandas configuration
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)
# Matplot inline
%matplotlib inline
{% endhighlight %}

Foursquare personal credentials (should be hiden for security):

{% highlight python %}
# Define Foursquare Credentials and Versions
CLIENT_ID = '' # your Foursquare ID
CLIENT_SECRET = '' # your Foursquare Secret
VERSION = '20181102' # Foursquare API version
{% endhighlight %}

Parameters to get Venues from Foursquare API:

{% highlight python %}
# Plot definition
LIMIT = 100
radius = 10000 #500 (Changed in order to get more venues from each city)
geolocator = Nominatim(user_agent="TO_explorer")
{% endhighlight %}

#### Step_1.4: In this part, all funcions are defined

Categories of Venue from Foursquere list response:

{% highlight python %}
# function that extracts the category of the venue
def get_category_type(row):
    try:
        categories_list = row['categories']
    except:
        categories_list = row['venue.categories']
    if len(categories_list) == 0:
        return None
    else:
        return categories_list[0]['name']
{% endhighlight %}

Method to get *venues* according a nearby location (in both cases, downtown of the cities):

{% highlight python %}
# Function definition
def getNearbyVenues(names, latitudes, longitudes, radius=500):
    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
        ##print(name)
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}'.format(
                CLIENT_ID, CLIENT_SECRET, lat, lng, VERSION, radius, LIMIT)
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        # return only relevant information for each nearby venue
        venues_list.append([(name, lat, lng, v['venue']['name'], 
            v['venue']['location']['lat'], v['venue']['location']['lng'], 
            v['venue']['categories'][0]['name']) for v in results])
    nearby_venues = pd.DataFrame([item for venue_list in 
                    venues_list for item in venue_list])
    nearby_venues.columns = ['Neighbourhood', 'Neighbourhood Latitude', 
            'Neighbourhood Longitude', 'Venue', 'Venue Latitude', 
            'Venue Longitude', 'Venue Category']
    return(nearby_venues)
{% endhighlight %}

List of common venues from a location:

{% highlight python %}
# Function to Most Common Vanues
def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    return row_categories_sorted.index.values[0:num_top_venues
{% endhighlight %}

Method to get venues from Foursquare API. The result is a new dataframe:

{% highlight python %}
# Function to return venues from a location
def retVenues(results):
    venues = results['response']['groups'][0]['items']
    nearby_venues = json_normalize(venues) # flatten JSON
    # filter columns
    filtered_columns = ['venue.name', 'venue.categories', 'venue.location.lat', 
      'venue.location.lng']
    nearby_venues = nearby_venues.loc[:, filtered_columns]
    # filter the category for each row
    nearby_venues['venue.categories'] = nearby_venues.apply
                                (get_category_type, axis=1)
    # clean columns
    nearby_venues.columns = [col.split(".")[-1] for col in 
                                nearby_venues.columns]
    return nearby_venues
{% endhighlight %}

Code to extract numerical information from strings. It will be used in the comparison:

{% highlight python %}
# Function to extract numbers from strings (used in comparison)
def retListNum(s):
    newstr = ''.join((ch if ch in '0123456789.' else ' ') for ch in s)
    listOfNumbers = [float(i) for i in newstr.split()]
    return listOfNumbers
{% endhighlight %}

Definition of better categories of venues to be used in the comparison from Foursquare (1st criterion):

{% highlight python %}
# Venues categories top list (used to first analysis)
topList = ['Park', 'Museum', 'Supermarket', 'Bookstore', 'Gym', 
    'Department Store', 'Pharmacy', 'Shopping Mall', 'Pool', 
    'Historic Site', 'Monument / Landmark']
{% endhighlight %}

Definition of better indicators that will be considering in the Versus comparion (2nd criterion):

{% highlight python %}
# Best indicators to comparison
personalCriteria = [0, 2, 7, 12]
{% endhighlight %}

### Step 2: Data Exploration

**Criterion A: Foursquare Service**

#### Step_2.1: Toronto City

##### **Step_2.1.1: Toronto Venues**

{% highlight python %}
# Toronto data
addressToronto = 'Toronto, Ontario, Canada'
locationToronto = geolocator.geocode(addressToronto)
latitudeToronto = locationToronto.latitude
longitudeToronto = locationToronto.longitude
# Geo service for Toronto
urlToronto = 'https://api.foursquare.com/v2/venues/explore?client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}'.format(
    CLIENT_ID, CLIENT_SECRET, latitudeToronto, longitudeToronto, VERSION, radius, LIMIT)
resultsToronto = requests.get(urlToronto).json()
venuesToronto = retVenues(resultsToronto)
venuesToronto.head()
{% endhighlight %}

* **Output:**

Name | Category | Lat | Lng
- | - | - | - |
Downtown Toronto | Neighborhood | 43.653232 | -79.385296
Art Gallery of Ontario | Art Gallery | 43.654003 | -79.392922
Pai | Thai Restaurant | 43.647923 | -79.388579
Byblos Toronto | Mediterranean Restaurant | 43.647615 | -79.388381
Nathan Phillips Square | Plaza | 43.652270 | -79.383516

General information about the Venues from Toronto:

{% highlight python %}
# General information
numVenuesToronto = venuesToronto.shape[0]
print('Toronto has {} venues returned by Foursquare.'.format(numVenuesToronto))
{% endhighlight %}

* **Output:** Toronto has 100 venues returned by Foursquare.

List of the categories of Venues:

{% highlight python %}
# List of unique values
categoriesToronto = venuesToronto['categories'].unique()
numcategoriesToronto = categoriesToronto.shape[0] 
print('Number of categories of this city (Toronto):',numcategoriesToronto)
categoriesToronto
{% endhighlight %}

* **Output:**

Number of categories of this city (Toronto): 60
array(['Neighborhood', 'Art Gallery', 'Thai Restaurant',
       'Mediterranean Restaurant', 'Plaza', 'Concert Hall',
       'American Restaurant', 'Clothing Store', 'Sandwich Place',
       'Speakeasy', 'Coffee Shop', 'Record Shop', 'Theater',
       'Sporting Goods Shop', 'Cosmetics Shop',
       'Vegetarian / Vegan Restaurant', 'French Restaurant',
       'Pizza Place', 'Movie Theater', 'Café', 'Gym', 'Dessert Shop',
       'Mexican Restaurant', 'Restaurant', 'Comedy Club', 'Hotel',
       'Arts & Crafts Store', 'Train Station', 'Scenic Lookout',
       'Brewery', 'Shoe Store', 'Organic Grocery', 'Hostel', 'Diner',
       'Farmers Market', 'Spa', 'Seafood Restaurant',
       'Japanese Restaurant', 'Food Truck', 'Park', 'Dance Studio',
       'Monument / Landmark', 'Museum', 'Supermarket', 'Salad Place',
       'Middle Eastern Restaurant', 'Spanish Restaurant',
       'Italian Restaurant', 'Steakhouse', 'Aquarium', 'Baseball Stadium',
       'Bar', 'Ramen Restaurant', 'Theme Restaurant', 'Bakery',
       'Performing Arts Venue', 'Tapas Restaurant', 'Yoga Studio', 'Lake',
       'Butcher'], dtype=object)

##### **Step2.1.2: Plotting Toronto with Venues**

{% highlight python %}
# Plotting data
strcoordToronto = 'Toronto: {}, {}.'.format(latitudeToronto, longitudeToronto)
map_toronto = folium.Map(location = [latitudeToronto, longitudeToronto], zoom_start = 13)
folium.CircleMarker([latitudeToronto, longitudeToronto], radius = 7, popup = strcoordToronto, color = 'red', 
                    fill = True, fill_color = 'white', fill_opacity = 0.7, parse_html = False).add_to(map_toronto)
bestPlacesToronto = 0
for i in range(numVenuesToronto):
    lat = venuesToronto.iloc[i][2]
    lng = venuesToronto.iloc[i][3]
    label = venuesToronto.iloc[i][1] + ":\n" + venuesToronto.iloc[i][0]
    # Venues in the top category are ploted in blue; otherwise, black
    if (venuesToronto.iloc[i][1] in topList):
        folium.CircleMarker([lat, lng], radius = 4, popup = label, color = 'blue', fill = True, fill_color = 'white', 
                        fill_opacity = 0.3, parse_html = False).add_to(map_toronto)
        bestPlacesToronto+=1
    else:
        folium.CircleMarker([lat, lng], radius = 4, popup = label, color = 'green', fill = True, fill_color = 'white', 
                        fill_opacity = 0.3, parse_html = False).add_to(map_toronto)
print('Number of best places:',bestPlacesToronto)
map_toronto
{% endhighlight %}

* **Output:**

{:refdef: style="text-align: center;"}
![Venues Toronto][venuestor]
{: refdef}

#### Step_2.2: Ottawa City

##### **Step_2.2.1: Ottawa Venues**

{% highlight python %}
# Testing data
addressOttawa = 'Ottawa, Ontario, Canada'
locationOttawa = geolocator.geocode(addressOttawa)
latitudeOttawa = locationOttawa.latitude
longitudeOttawa = locationOttawa.longitude
# Geo service for Toronto
urlOttawa = 'https://api.foursquare.com/v2/venues/explore?client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}'.format(
    CLIENT_ID, CLIENT_SECRET, latitudeOttawa, longitudeOttawa, VERSION, radius, LIMIT)
resultsOttawa = requests.get(urlOttawa).json()
venuesOttawa = retVenues(resultsOttawa)
venuesOttawa.head()
{% endhighlight %}

* **Output:**

Name | Category | Lat | Lng
- | - | - | - |
Confederation Park | Park | 45.422069 | -75.692518
National Arts Centre - Centre National des Arts | Concert Hall | 45.422922 | -75.692996
La Bottega | Deli / Bodega | 45.426894 | -75.691906
Major's Hill Park | Park | 45.427040 | -75.696863
Rideau Canal | Other Great Outdoors | 45.424781 | -75.695129

General information about the Venues from Ottawa:

{% highlight python %}
# General information
numVenuesOttawa = venuesOttawa.shape[0]
print('Ottawa has {} venues returned by Foursquare.'.format(numVenuesOttawa))
{% endhighlight %}

* **Output:**: Ottawa has 100 venues returned by Foursquare.

List of Ottawa categories of venues:

{% highlight python %}
# List of venues
categoriesOttawa = venuesOttawa['categories'].unique()
numcategoriesOttawa = categoriesOttawa.shape[0] 
print('Number of categories of this city (Ottawa):',numcategoriesOttawa)
categoriesOttawa
{% endhighlight %}

* **Output:**

Number of categories of this city (Ottawa): 58
array(['Park', 'Concert Hall', 'Deli / Bodega', 'Other Great Outdoors',
       'Hotel', 'Restaurant', 'Noodle House', 'Fast Food Restaurant',
       'Memorial Site', 'Vegetarian / Vegan Restaurant',
       'New American Restaurant', 'Bakery', 'Ice Cream Shop',
       'Mexican Restaurant', 'Art Gallery', 'Japanese Restaurant',
       'Taco Place', 'Coffee Shop', 'Italian Restaurant',
       'Seafood Restaurant', 'Pub', 'Market', 'Salad Place',
       'Skating Rink', 'Café', 'Burger Joint',
       'Southern / Soul Food Restaurant', 'Tapas Restaurant',
       'Indie Movie Theater', 'Breakfast Spot', 'Grocery Store',
       'Korean Restaurant', 'Cupcake Shop', 'Steakhouse', 'Yoga Studio',
       'Bookstore', 'Middle Eastern Restaurant', 'Bar', 'Pizza Place',
       'Museum', 'Historic Site', 'German Restaurant',
       'Monument / Landmark', 'French Restaurant', 'History Museum',
       'BBQ Joint', 'Plaza', 'Record Shop', 'Bagel Shop', 'Gastropub',
       'Movie Theater', 'Football Stadium', 'Brewery', 'Fish Market',
       'Theme Restaurant', 'Wine Bar', 'Diner', 'Sandwich Place'],
      dtype=object)

##### **Step2.2.2: Plotting Ottawa with Venues**

{% highlight python %}
# Plotting data
strcoordOttawa = 'Ottawa: {}, {}.'.format(latitudeOttawa, longitudeOttawa)
map_ottawa = folium.Map(location = [latitudeOttawa, longitudeOttawa], zoom_start = 13)
folium.CircleMarker([latitudeOttawa, longitudeOttawa], radius = 7, 
                    popup = strcoordOttawa, color = 'red', fill = True, fill_color = 'white',
                    fill_opacity = 0.7, parse_html = False).add_to(map_ottawa)
bestPlacesOttawa = 0
for i in range(numVenuesOttawa):
    lat = venuesOttawa.iloc[i][2]
    lng = venuesOttawa.iloc[i][3]
    label = venuesOttawa.iloc[i][1] + ":\n" + venuesOttawa.iloc[i][0]
    # Venues in the top category are ploted in blue; otherwise, black
    if (venuesOttawa.iloc[i][1] in topList):
        folium.CircleMarker([lat, lng], radius = 4, popup = label, color = 'blue', fill = True, 
                            fill_color = 'white', fill_opacity = 0.3, parse_html = False).
                            add_to(map_ottawa)
        bestPlacesOttawa+=1
    else:
        folium.CircleMarker([lat, lng], radius = 4, popup = label, color = 'green', fill = True, 
                            fill_color = 'white', fill_opacity = 0.3, parse_html = False).
                            add_to(map_ottawa)
print('Number of best places:',bestPlacesOttawa)
map_ottawa
{% endhighlight %}

* **Output:**

{:refdef: style="text-align: center;"}
![Venues Ottawa][venuesott]
{: refdef}

**Criterion B: Versus Comparison**

#### Step_2.3: The second criterium is based on the Versus City comparison.

##### **Step_2.3.1: Reading information from Versus Site.**

{% highlight python %}
# Reading source - Versus
urlAdd = 'https://versus.com/en/ottawa-vs-toronto'
source = requests.get(urlAdd).text
soup = BeautifulSoup(source, 'html.parser')
listtab = []
for table in soup.find_all('div',{'class':'tldr'}):
    strtab = ''
    row1 = ';';
    for em in table.find_all('em',{'class':'value'}):
        row1 = row1+','+em.text
    strtab = strtab + row1+'\n'
    row2=';';
    for em in table.find_all('em',{'class':'otherValue'}):
        row2 = row2+','+em.text
    strtab = strtab + row2+'\n'
    listtab.append(strtab)
# Creating head for the future tables
head0='PopDensity,MonthlyPubTrans,UnemploymentRate,CostOneBedroom,YoungPop,
        IncomeIneq,Heritage,AvgMinTemp\n'
head1='AvgMaxTemp,HighAvgTemp,MilInhabitants,BilDGP,HighAvgSal,SportsFacility,
        CostSingTrans,ForbesBil\n'
# Creating lists to tables
strnew = listtab[0]
strnew = listtab[0].replace(';,',head0,1)
strnew = strnew.replace(';,','',1)
listtab[0] = strnew
strnew = listtab[1]
strnew = listtab[1].replace(';,',head1,1)
strnew = strnew.replace(';,','',1)
listtab[1] = strnew
{% endhighlight %}

##### **Step_2.3.2: Crating tables to dataframe**

Cities names:

{% highlight python %}
# Table with City Names
strcity = 'City\nOttawa\nToronto'
CompTabCity = pd.read_csv(StringIO(strcity))
CompTabCity
{% endhighlight %}

* **Output:**

| City
| -
| Ottawa
| Toronto

Ottawa advantages list:

{% highlight python %}
# Table 0: Advantages of Ottawa
CompTab0 = pd.read_csv(StringIO(listtab[0]))
CompTab0
{% endhighlight %}

* **Output:**

| PopDensity | MonthlyPubTrans | UnemploymentRate | CostOneBedroom | YoungPop | IncomeIneq | Heritage | AvgMinTem
| - | - | - | - | - | - | - | -
| 1680 people/km² | 81.26$ | 6.3% | 902.6$ | 36.7 years | 0.440 | 1 | 1.5°C
| 4150 people/km² | 105.01$ | 9.9% | 1199.1$ | 36.9 years | 0.441 | 0 | 6.2°C

Toronto advantages list:

{% highlight python %}
# Table 1: Advantages of Toronto
CompTab1 = pd.read_csv(StringIO(listtab[1]))
CompTab1
{% endhighlight %}

* **Output:**

| AvgMaxTemp | HighAvgTemp | MilInhabitants | BilDGP | HighAvgSal | SportsFacility | CostSingTrans | ForbesBil
| - | - | - | - | - | - | - | -
| 11.9°C | 9.1°C | 2.62 million | 270$ billion | 2574.92$ | 3 | 2.41$ | 5
| 10.9°C | 6.2°C | 0.88 million | 40$ billion | 2441.68$ | 1 | 2.62$ | 0

Adjusting Table 1 to maintain the same order (Ottawa x Toronto):

{% highlight python %}
# Temporary table - Reordering the raws in the Toronto advantages
CompTab2 = CompTab1
line0, line1 = CompTab2.iloc[0].copy(), CompTab2.iloc[1].copy()
CompTab2.iloc[0], CompTab2.iloc[1] = line1, line0
CompTab2
{% endhighlight %}

* **Output:**

| AvgMaxTemp | HighAvgTemp | MilInhabitants | BilDGP | HighAvgSal | SportsFacility | CostSingTrans | ForbesBil
| - | - | - | - | - | - | - | -
| 10.9°C | 6.2°C | 0.88 million | 40$ billion | 2441.68$ | 1 | 2.62$ | 0
| 11.9°C | 9.1°C | 2.62 million | 270$ billion | 2574.92$ | 3 | 2.41$ | 5

Merging tables in one final dataset:

{% highlight python %}
# New table marging all previous tables
frames = [CompTabCity, CompTab0, CompTab2]
CompTab = pd.concat(frames, axis=1, sort=False)
{% endhighlight %}

##### **Step_2.3.3: Getting advantages of the cities**

{% highlight python %}
# Preparation of lists of indicators
tam = CompTab.shape[1]
listNum = []
for i in range(tam-1):
    try:
        s1 = CompTab.iloc[0][i+1]
        s2 = CompTab.iloc[1][i+1]
        s = s1 + ' ' + s2
    except:        
        f1 = CompTab.iloc[0][i+1].item()
        s1 = str(f1)
        f2 = CompTab.iloc[1][i+1].item()
        s2 = str(f2)
        s = s1 + ' ' + s2        
    finally:
        newlist = retListNum(s)
        listNum.append(newlist)
{% endhighlight %}

##### **Step_2.3.4: Computing the general indicators**

{% highlight python %}
# Comparison, considering MIN and MAX values
MIN = 0
MAX = 1
listCriteria = [MIN, MIN, MIN, MIN, MIN, MIN, MAX, MIN,
                MAX, MAX, MAX, MAX, MAX, MAX, MIN, MAX]
listDetails = ['Population density', 'Monthly public transport ticket',
               'Unemployment rate', 'Cost of one-bedroom ap. in the center',
               'Younger population', 'Income inequalities', 
               'Landmarks from UNESCO World Heritage list', 
               'Average minimum temperature', 
               'Higher average maximum temperature', 
               'Higher average temperature', 'Million inhabitants', 
               'Billion higher gross domestic product (GDP)', 
               'Higher average salary', '2 more big sport facilities', 
               'Cost of one single transportation ticket',
               'Billionaires from annual Forbes']
tam = len(listCriteria)
contOttawa = 0
contToronto = 0
print('COMPARISON DETAILS [Ottawa x Toronto]')
listWin = []
OTT = 0
TOR = 1
for i in range(tam):
    if (listCriteria[i]==MIN):
        if (listNum[i][0]<listNum[i][1]):
            contOttawa += 1
            print(' * Comparing(',i,'):',listDetails[i],':',listNum[i][0],'<',
                  listNum[i][1], '= Ottawa wins!')
            listWin.append(OTT)
        else:
            contToronto += 1
            print(' * Comparing(',i,'):',listDetails[i],':',listNum[i][0],'<',
                  listNum[i][1], '= Toronto wins!')
            listWin.append(TOR)
    else:
        if (listNum[i][0]>listNum[i][1]):
            contOttawa += 1
            print(' * Comparing(',i,'):',listDetails[i],':',listNum[i][0],'>',
                  listNum[i][1], '= Ottawa wins!')
            listWin.append(OTT)
        else:
            contToronto += 1
            print(' * Comparing(',i,'):',listDetails[i],':',listNum[i][0],'>',
                  listNum[i][1], '= Toronto wins!')
            listWin.append(TOR)
{% endhighlight %}

* **Output:**

COMPARISON DETAILS [Ottawa x Toronto]
 * Comparing( 0 ): Population density : 1680.0 < 4150.0 = Ottawa wins!
 * Comparing( 1 ): Monthly public transport ticket : 81.26 < 105.01 = Ottawa wins!
 * Comparing( 2 ): Unemployment rate : 6.3 < 9.9 = Ottawa wins!
 * Comparing( 3 ): Cost of one-bedroom ap. in the center : 902.6 < 1199.1 = Ottawa wins!
 * Comparing( 4 ): Younger population : 36.7 < 36.9 = Ottawa wins!
 * Comparing( 5 ): Income inequalities : 0.44 < 0.441 = Ottawa wins!
 * Comparing( 6 ): Landmarks from UNESCO World Heritage list : 1.0 > 0.0 = Ottawa wins!
 * Comparing( 7 ): Average minimum temperature : 1.5 < 6.2 = Ottawa wins!
 * Comparing( 8 ): Higher average maximum temperature : 10.9 > 11.9 = Toronto wins!
 * Comparing( 9 ): Higher average temperature : 6.2 > 9.1 = Toronto wins!
 * Comparing( 10 ): Million inhabitants : 0.88 > 2.62 = Toronto wins!
 * Comparing( 11 ): Billion higher gross domestic product (GDP) : 40.0 > 270.0 = Toronto wins!
 * Comparing( 12 ): Higher average salary : 2441.68 > 2574.92 = Toronto wins!
 * Comparing( 13 ): 2 more big sport facilities : 1.0 > 3.0 = Toronto wins!
 * Comparing( 14 ): Cost of one single transportation ticket : 2.62 < 2.41 = Toronto wins!
 * Comparing( 15 ): Billionaires from annual Forbes : 0.0 > 5.0 = Toronto wins!

##### **Step_2.3.5: Choosing the best indicators (personal view)**

{% highlight python %}
pointOtt = 0
pointTor = 0
for i in range(len(personalCriteria)):
    if (listWin[personalCriteria[i]] == OTT):
        pointOtt+=1
    else:
        pointTor+=1
{% endhighlight %}

### Step 3: Comparison

#### Step_3.1: Foursquare Venues

{% highlight python %}
print('* CRITERION 1 ==================================================')
print('Toronto Downtown =',numVenuesToronto,', 
        Ottawa downtown=',numVenuesOttawa)
print('Best places to family: Toronto =',bestPlacesToronto,
        'x Ottawa =',bestPlacesOttawa)
print('===============================================================')
{% endhighlight %}

* **Output:**

CRITERION 1 ====================================================

Toronto Downtown = 100 , Ottawa downtown= 100

Best places to family: Toronto = 10 x Ottawa = 11

===========================================================

#### Step_3.2: Versus comparison

{% highlight python %}
print('* CRITERION 1 ==================================================')
print('Ottawa advantages:',contOttawa,
      'x Toronto advantages:',contToronto)
print('Best indicator ponctuation: Toronto =',
      pointTor,'x Ottawa =',pointOtt)
print('===============================================================')
{% endhighlight %}

* **Output:**

CRITERION 2 ====================================================

Ottawa advantages: 8 x Toronto advantages: 8

Best indicator ponctuation: Toronto = 1 x Ottawa = 3

===========================================================

#### Step_3.3: Final Comparison

{% highlight python %}
arrayCities = np.array([[bestPlacesToronto, bestPlacesOttawa, 
              pointTor, pointOtt]], np.int32)
dfCities = pd.DataFrame(arrayCities, columns=['bestPlaceTor', 
              'bestPlaceOtt', 'pointTor', 'pointOtt'])
fig, axes = plt.subplots(nrows=1, ncols=2)
dfCities[['bestPlaceTor', 'bestPlaceOtt']].plot(
              ax=axes[0],kind='bar', title='Best Places')
dfCities[['pointTor', 'pointOtt']].plot(ax=axes[1], 
              kind='bar', title='Criteria Points')
{% endhighlight %}

* **Output:**

{:refdef: style="text-align: center;"}
![Final comparison][compgraph]
{: refdef}

# Conclusion

Both cities (Ottawa and Toronto) have similar advantages / disadvantages.

However, under a *specific point of view*, the factors (most related with family benefits) can help to diferentiate the two cities and we can get a different perspective of them.

For this reason, according aspects that our family considers important, such as best places and best criteria ponctuation, **Ottawa** was considered better.

These are some reasons why, considering personal preferences, our family decided to move to Ottawa, Capital City. And in fact, we are loving this city.


[jupyter-home]: https://jupyter.org/
[pandas-home]: https://pandas.pydata.org/
[foursquare-home]: https://foursquare.com/
[beautifulsoap-home]: https://www.crummy.com/software/BeautifulSoup/

[logo]: https://images.youracclaim.com/images/60f2e1e1-1b74-4dc0-a24b-cd08b460c12d/Applied%2BData%2BScience%2BCapstone.png "Badge"
[venuesott]: https://raw.githubusercontent.com/paulobmsousa/courseraibmdatascience/master/venuesott.png
[venuestor]: https://raw.githubusercontent.com/paulobmsousa/courseraibmdatascience/master/venuestor.png
[compgraph]: https://raw.githubusercontent.com/paulobmsousa/courseraibmdatascience/master/finalcomp.png
[devblog2]: https://raw.githubusercontent.com/paulobmsousa/courseraibmdatascience/master/devcorner2.png
