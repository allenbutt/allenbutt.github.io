---
layout: splash
title: "NFL Arrests Visualization Project"
subtitle: "Fan Arrests vs Player Arrests"
date: 2022-06-13 00:00:00 -0000
#background: 
---

## NFL Arrests Visualization Project
We have a dataset of the number of fans arrested at each NFL game from 2011 to 2015, and another of the number of NFL players arrested each game during that same time period. The intention of this visualization project was to look for potential trends/correlations among these datasets.

## Visualizations
<iframe src="https://public.tableau.com/views/projectWIP3/TheStory?:language=en-US&:display_count=n&:origin=viz_share_link:showVizHome=no&:embed=true"
width="1200" height="800"></iframe>

## Dataset 1 - Prepping the Fans Arrests Data


```python
import pandas as pd
import re
import numpy as np
data = pd.read_csv("nfl_arrests_2011-2015.csv", encoding = 'unicode_escape')
```


```python
#Fix missing data in OT_flag and turn it into a numeric variable
data.fillna({'OT_flag':0}, inplace=True)
data['OT_flag'] = data['OT_flag'].replace(['OT'],1)
data["OT_flag"]=pd.to_numeric(data["OT_flag"])

#Update "division_game" into numeric as well
data['division_game'] = data['division_game'].replace(['n'],0)
data['division_game'] = data['division_game'].replace(['y'],1)
data["division_game"]=pd.to_numeric(data["division_game"])
data.head()
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
      <th>season</th>
      <th>week_num</th>
      <th>day_of_week</th>
      <th>gametime_local</th>
      <th>home_team</th>
      <th>away_team</th>
      <th>home_score</th>
      <th>away_score</th>
      <th>OT_flag</th>
      <th>arrests</th>
      <th>division_game</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2011</td>
      <td>1</td>
      <td>Sunday</td>
      <td>1:15:00 PM</td>
      <td>Arizona</td>
      <td>Carolina</td>
      <td>28</td>
      <td>21</td>
      <td>0</td>
      <td>5.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2011</td>
      <td>4</td>
      <td>Sunday</td>
      <td>1:05:00 PM</td>
      <td>Arizona</td>
      <td>New York Giants</td>
      <td>27</td>
      <td>31</td>
      <td>0</td>
      <td>6.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2011</td>
      <td>7</td>
      <td>Sunday</td>
      <td>1:05:00 PM</td>
      <td>Arizona</td>
      <td>Pittsburgh</td>
      <td>20</td>
      <td>32</td>
      <td>0</td>
      <td>9.0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2011</td>
      <td>9</td>
      <td>Sunday</td>
      <td>2:15:00 PM</td>
      <td>Arizona</td>
      <td>St. Louis</td>
      <td>19</td>
      <td>13</td>
      <td>1</td>
      <td>6.0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2011</td>
      <td>13</td>
      <td>Sunday</td>
      <td>2:15:00 PM</td>
      <td>Arizona</td>
      <td>Dallas</td>
      <td>19</td>
      <td>13</td>
      <td>1</td>
      <td>3.0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Some observations have missing data--they should be dropped from the dataframe.
data = data[data['arrests'].notna()]
```


```python
#Some games were played in London and so have missing data.
#We can impute the missing values taking the mean of the arrests of the same year for that team.

#Create a function to help with this process
def imputeLondon(year, home, away, homescore, awayscore, OT, division):
    new = data[(data['home_team'] == home) & (data['season'] == year )]["arrests"].mean()
    data.loc[len(data)] = np.array([year,0,0,0,home,away,homescore,awayscore,OT,new,division])
    return;
```


```python
#Use the function to fill in the missing data with imputed values.
imputeLondon(2013, "Arizona", "Houston", 30, 9, 0, 0)
imputeLondon(2013, "Jacksonville", "San Francisco", 10, 42, 0, 0)
imputeLondon(2014, "Jacksonville", "Dallas", 17, 32, 0, 0)
imputeLondon(2015, "Jacksonville", "Buffalo", 34, 31, 0, 0)
imputeLondon(2015, "Kansas City", "Detroit", 45, 10, 0, 0)
imputeLondon(2015, "Miami", "New York Jets", 14, 27, 0, 1)
imputeLondon(2014, "Oakland", "Miami", 14, 38, 0, 0)
imputeLondon(2014, "Oakland", "Denver", 17, 41, 0, 1)
imputeLondon(2014, "Oakland", "Kansas City", 24, 20, 0, 0)
imputeLondon(2011, "Tampa Bay", "Chicago", 18, 24, 0, 0)
```


```python
#Three teams had a missing year of data--we can impute this data by taking the mean of the existing years.
def imputeYear(year, home, away, homescore, awayscore, OT, division):
    new = data[(data['home_team'] == home)]["arrests"].mean()
    data.loc[len(data)] = np.array([year,0,0,0,home,away,homescore,awayscore,OT,pd.to_numeric(new),division])
    data["arrests"]=pd.to_numeric(data["arrests"]) #kept getting type errors without brute-forcing it
    return;

imputeYear(2012, "Baltimore", "Cincinnati", 44, 13, 0, 1)
imputeYear(2012, "Baltimore", "New England", 31, 30, 0, 0)
imputeYear(2012, "Baltimore", "Cleveland", 23, 16, 0, 1)
imputeYear(2012, "Baltimore", "Dallas", 31, 29, 0, 1)
imputeYear(2012, "Baltimore", "Oakland", 55, 20, 0, 0)
imputeYear(2012, "Baltimore", "Pittsburgh", 20, 23, 0, 1)
imputeYear(2012, "Baltimore", "Denver", 17, 34, 0, 0)
imputeYear(2012, "Baltimore", "New York Giants", 33, 14, 0, 0)

imputeYear(2015, "Chicago", "Green Bay", 23, 31, 0, 1)
imputeYear(2015, "Chicago", "Arizona", 23, 48, 0, 0)
imputeYear(2015, "Chicago", "Oakland", 22, 20, 0, 0)
imputeYear(2015, "Chicago", "Minnesota", 20, 23, 0, 1)
imputeYear(2015, "Chicago", "Denver", 15, 17, 0, 0)
imputeYear(2015, "Chicago", "San Francisco", 20, 26, 1, 0)
imputeYear(2015, "Chicago", "Washington", 21, 24, 0, 0)
imputeYear(2015, "Chicago", "Detroit", 20, 24, 0, 1)

imputeYear(2011, "Miami", "New England", 24, 38, 0, 1)
imputeYear(2011, "Miami", "Houston", 13, 23, 0, 0)
imputeYear(2011, "Miami", "Denver", 15, 18, 1, 0)
imputeYear(2011, "Miami", "Washington", 2, 9, 0, 0)
imputeYear(2011, "Miami", "Buffalo", 35, 8, 0, 1)
imputeYear(2011, "Miami", "Oakland", 34, 14, 0, 0)
imputeYear(2011, "Miami", "Philadelphia", 10, 26, 0, 0)
imputeYear(2011, "Miami", "New York Jets", 19, 17, 0, 1)
```


```python
#For one of the visualizations, we need to sort the dataset according to team, then year, then week--then an index will need
#to be added to keep things properly sorted.
data = data.sort_values(by = ["home_team","season", "week_num"])

data["Index_num"] = 0
for snuh in range(0,len(data)):
    data.iat[snuh,11] = snuh
```


```python
#Export Dataframe to CSV
data.to_csv(r'nfl_arrests.csv', index = False)
```
Dataset 1 is properly formatted and can be exported as a CSV for use in Tableau.
## Dataset 2 - NFL Player Arrests


```python
#New Dataset, NFL Player Arrests
data2 = pd.read_csv("nfl_player_arrests.csv", encoding = 'unicode_escape')
#Check out the data, look for Missing Data
data2.head()
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
      <th>DATE</th>
      <th>TEAM</th>
      <th>NAME</th>
      <th>POS</th>
      <th>CASE</th>
      <th>CATEGORY</th>
      <th>DESCRIPTION</th>
      <th>OUTCOME</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/13/2020</td>
      <td>DEN</td>
      <td>Melvin Gordon</td>
      <td>RB</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Suspected of drunk driving, speeding in Denver.</td>
      <td>Resolution undetermined.</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10/3/2020</td>
      <td>PIT</td>
      <td>Jarron Jones</td>
      <td>OT</td>
      <td>Arrested</td>
      <td>Domestic violence</td>
      <td>Charged with aggravated assault, strangulation...</td>
      <td>Resolution undetermined.</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9/11/2020</td>
      <td>TEN</td>
      <td>Isaiah Wilson</td>
      <td>OT</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Pulled over, accused of drunken driving near N...</td>
      <td>Resolution undetermined.</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8/25/2020</td>
      <td>CIN</td>
      <td>Mackensie Alexander</td>
      <td>CB</td>
      <td>Arrested</td>
      <td>Battery</td>
      <td>Accused of hitting a man in the face in Collie...</td>
      <td>Resolution undetermined.</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8/7/2020</td>
      <td>WAS</td>
      <td>Derrius Guice</td>
      <td>RB</td>
      <td>Arrested</td>
      <td>Domestic violence</td>
      <td>Accused of strangulation, assault and property...</td>
      <td>Resolution undetermined. Team released him sam...</td>
    </tr>
  </tbody>
</table>
</div>




```python
#We need to standardize Team Names--importing a new csv file with two columns to help ease the transition
data3 = pd.read_csv("nfl_names_conversion.csv", encoding = 'unicode_escape')
data3.head()
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
      <th>Team_Name</th>
      <th>Team_City</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARI</td>
      <td>Arizona</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BAL</td>
      <td>Baltimore</td>
    </tr>
    <tr>
      <th>2</th>
      <td>CAR</td>
      <td>Carolina</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CHI</td>
      <td>Chicago</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CIN</td>
      <td>Cincinnati</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Loop through each row in this small dataset, and change obervations in data2 that match "Team Name" to "Team City".
#Also add a new column to data3 that selects 1 for items that were matched. This will allow us to delete all observations
#with teams outside of our dataset easily.

data2["Found"] = 0
for meh in range(0,len(data3)):
    teamname = data3["Team_Name"][meh]
    for bleh in range(0,len(data2)):
        if data2.iloc[bleh]['TEAM'] == teamname:
            data2.iat[bleh,8] = 1
            data2.iat[bleh,1] = data3.iloc[meh]["Team_City"]

data2.drop(data2[data2['Found'] == 0].index, inplace = True) 
data2.head(20)
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
      <th>DATE</th>
      <th>TEAM</th>
      <th>NAME</th>
      <th>POS</th>
      <th>CASE</th>
      <th>CATEGORY</th>
      <th>DESCRIPTION</th>
      <th>OUTCOME</th>
      <th>Found</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10/13/2020</td>
      <td>Denver</td>
      <td>Melvin Gordon</td>
      <td>RB</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Suspected of drunk driving, speeding in Denver.</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10/3/2020</td>
      <td>Pittsburgh</td>
      <td>Jarron Jones</td>
      <td>OT</td>
      <td>Arrested</td>
      <td>Domestic violence</td>
      <td>Charged with aggravated assault, strangulation...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>9/11/2020</td>
      <td>Tennessee</td>
      <td>Isaiah Wilson</td>
      <td>OT</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Pulled over, accused of drunken driving near N...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8/25/2020</td>
      <td>Cincinnati</td>
      <td>Mackensie Alexander</td>
      <td>CB</td>
      <td>Arrested</td>
      <td>Battery</td>
      <td>Accused of hitting a man in the face in Collie...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>8/7/2020</td>
      <td>Washington</td>
      <td>Derrius Guice</td>
      <td>RB</td>
      <td>Arrested</td>
      <td>Domestic violence</td>
      <td>Accused of strangulation, assault and property...</td>
      <td>Resolution undetermined. Team released him sam...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>7/14/2020</td>
      <td>Houston</td>
      <td>Kenny Stills</td>
      <td>WR</td>
      <td>Arrested</td>
      <td>Disorderly conduct</td>
      <td>Accused of felony intimidation in Louisville a...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6/27/2020</td>
      <td>Arizona</td>
      <td>Jermiah Braswell</td>
      <td>WR</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Accused of driving while intoxiated after his ...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>6/15/2020</td>
      <td>New York Giants</td>
      <td>Aldrick Rosas</td>
      <td>K</td>
      <td>Arrested</td>
      <td>Hit-and-run</td>
      <td>Accused of fleeing the scene of a collision at...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>5/19/2020</td>
      <td>Green Bay</td>
      <td>Montravius Adams</td>
      <td>DE</td>
      <td>Arrested</td>
      <td>Drugs</td>
      <td>Pulled over near Perry, Ga., accused of mariju...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>5/16/2020</td>
      <td>New York Giants</td>
      <td>Deandre Baker</td>
      <td>CB</td>
      <td>Surrendered</td>
      <td>Armed robbery</td>
      <td>Accused of being involved in armed robbery of ...</td>
      <td>Dropped by prosecutors.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>5/16/2020</td>
      <td>Seattle</td>
      <td>Quinton Dunbar</td>
      <td>CB</td>
      <td>Surrendered</td>
      <td>Armed robbery</td>
      <td>Accused of being involved in armed robbery of ...</td>
      <td>Dropped by prosecutors.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>5/16/2020</td>
      <td>Washington</td>
      <td>Cody Latimer</td>
      <td>WR</td>
      <td>Arrested</td>
      <td>Gun</td>
      <td>Accused of felony discharge of a weapon in Eng...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>4/28/2020</td>
      <td>Kansas City</td>
      <td>Bashaud Breeland</td>
      <td>CB</td>
      <td>Arrested</td>
      <td>Drugs</td>
      <td>Accused of marijuana possession, resisting arr...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>3/11/2020</td>
      <td>Dallas</td>
      <td>Ventell Bryant</td>
      <td>WR</td>
      <td>Arrested</td>
      <td>DUI</td>
      <td>Pulled over in Tampa, accused of driving drunk...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>3/5/2020</td>
      <td>New York Jets</td>
      <td>Quinnen Williams</td>
      <td>DE</td>
      <td>Arrested</td>
      <td>Gun</td>
      <td>Accused of criminal possession of pistol at La...</td>
      <td>Resolution undetermined.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1/17/2020</td>
      <td>New England</td>
      <td>Joejuan Williams</td>
      <td>CB</td>
      <td>Arrested</td>
      <td>Drugs</td>
      <td>Pulled over for speeding in Nashville, accused...</td>
      <td>Pleaded no contest to simple possession. Diver...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1/11/2020</td>
      <td>New England</td>
      <td>Julian Edelman</td>
      <td>WR</td>
      <td>Arrested</td>
      <td>Vandalism</td>
      <td>Accused of jumping on the hood of a Mercedes i...</td>
      <td>Charge dropped.</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>12/29/2019</td>
      <td>Miami</td>
      <td>Xavien Howard</td>
      <td>CB</td>
      <td>Arrested</td>
      <td>Domestic violence</td>
      <td>Police in Davie, Fla., say he pushed his fianc...</td>
      <td>Dropped after woman declined to proceed with c...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>21</th>
      <td>12/20/2019</td>
      <td>Pittsburgh</td>
      <td>Kameron Kelly</td>
      <td>S</td>
      <td>Arrested</td>
      <td>Disorderly conduct</td>
      <td>Accused of making threats and resisting arrest...</td>
      <td>Resolution undetermined. Team released him sam...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>12/3/2019</td>
      <td>Dallas</td>
      <td>Antwaun Woods</td>
      <td>DT</td>
      <td>Arrested</td>
      <td>Drugs</td>
      <td>Pulled over for speeding in Frisco, Texas, and...</td>
      <td>Resolution undetermined</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Export Dataframe to CSV
data2.to_csv(r'nfl_players.csv', index = False)
```
Both datasets are prepped to be used in Tableau.