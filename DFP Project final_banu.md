

```python
import requests
from bs4 import BeautifulSoup as bs
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime
from collections import defaultdict
```

## Read Fresh Vegetables data from USDA website into csv and then load into data frame


```python
url = 'https://www.ers.usda.gov/webdocs/DataFiles/54529/SandU%20Fresh.xlsx?v=43189'
r = requests.get(url)
with open("fresh.csv",'wb') as f:
    f.write(r.content)
```


```python
# Load fresh.csv into dataframe
# fresh.csv has 32 sheets for 32 vegetables. Hence loading all the the sheets into one single dataframe
column_names = ['Vegetable','Year','Production','Imports','Supply','Exports','Domestic_Availability','Per_Capita_Availability','Current_Dollars','Constant_2009_Dollars']
fresh_df = pd.DataFrame(columns=column_names)
for i in range(1,31):
    temp_df= pd.read_excel("fresh.csv", sheet_name=i)
    if temp_df.shape[1] == 9: # to remove those sheets which have more than 9 columns
        # Get the vegetable name and insert as the first column of the dataframe
        col1 =temp_df.columns[0]
        beg_pos = 15
        end_pos = col1.find(':')
        veg_name = col1[beg_pos:end_pos]
        veg_name = veg_name.replace(', all uses','')
        veg_name = veg_name.replace('fresh ','')
        temp_df.insert(loc=0, column='Vegetable', value=veg_name) 
        
        # now rename the columns and concatenate the DF
        temp_df.columns = column_names
        fresh_df = pd.concat([fresh_df,temp_df.iloc[4:51]], ignore_index  = True) # only rows between 4 and 51 are relevant
```


```python
fresh_df.shape
```




    (965, 10)




```python
# Write the dataframe data to excel file
writer = pd.ExcelWriter('all_vegetables.xlsx', engine='xlsxwriter')
fresh_df.to_excel(writer, sheet_name='Sheet1')
writer.save()
```


```python
fresh_df.head()
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
      <th>Vegetable</th>
      <th>Year</th>
      <th>Production</th>
      <th>Imports</th>
      <th>Supply</th>
      <th>Exports</th>
      <th>Domestic_Availability</th>
      <th>Per_Capita_Availability</th>
      <th>Current_Dollars</th>
      <th>Constant_2009_Dollars</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>artichokes</td>
      <td>1971</td>
      <td>79.2</td>
      <td>26.1</td>
      <td>105.3</td>
      <td>--</td>
      <td>105.3</td>
      <td>0.507076</td>
      <td>9.72</td>
      <td>40.5608</td>
    </tr>
    <tr>
      <th>1</th>
      <td>artichokes</td>
      <td>1972</td>
      <td>71</td>
      <td>46.8</td>
      <td>117.8</td>
      <td>--</td>
      <td>117.8</td>
      <td>0.56123</td>
      <td>11.6</td>
      <td>46.3907</td>
    </tr>
    <tr>
      <th>2</th>
      <td>artichokes</td>
      <td>1973</td>
      <td>60</td>
      <td>43.5</td>
      <td>103.5</td>
      <td>--</td>
      <td>103.5</td>
      <td>0.488417</td>
      <td>14.5</td>
      <td>54.9951</td>
    </tr>
    <tr>
      <th>3</th>
      <td>artichokes</td>
      <td>1974</td>
      <td>70.2</td>
      <td>37.8</td>
      <td>108</td>
      <td>--</td>
      <td>108</td>
      <td>0.505017</td>
      <td>17.3</td>
      <td>60.2074</td>
    </tr>
    <tr>
      <th>4</th>
      <td>artichokes</td>
      <td>1975</td>
      <td>73.4</td>
      <td>32.4</td>
      <td>105.8</td>
      <td>--</td>
      <td>105.8</td>
      <td>0.489876</td>
      <td>16.1</td>
      <td>51.2821</td>
    </tr>
  </tbody>
</table>
</div>



### Web scrape the food price index data


```python
page = requests.get('https://ycharts.com/indicators/agriculture_index_world_bank')
page = page.content
soup = bs(page, 'html.parser')
dataTableBox = soup.find('div', {"id": "dataTableBox"})
col1 = []
col2 = []
# get the data from table into lists
for tr in dataTableBox.find_all('tr')[2:]:
    cols = tr.find_all('td')
    if(len(cols) == 0 ): # for empty div comming in between
        continue
#     print(datetime.strptime(cols[0].text, '%m. %d, %y'))
    col1.append(cols[0].text)
    col2.append(float(cols[1].text.strip()))

# Create dataframe from the lists of data
pd1_input_list = {'Month':col1,'Price_Index':col2}
df1 = pd.DataFrame(pd1_input_list)
# Get the year in separate column and get the average price_indices foe each year
df1['Year'] = df1.Month.str.slice(-4)
# Write the dataframe data to excel file
writer = pd.ExcelWriter('monthly_price_index.xlsx', engine='xlsxwriter')
df1.to_excel(writer, sheet_name='Sheet1')
writer.save()
```


```python
df1
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
      <th>Month</th>
      <th>Price_Index</th>
      <th>Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>March 31, 2018</td>
      <td>90.37</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Feb. 28, 2018</td>
      <td>88.92</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jan. 31, 2018</td>
      <td>87.52</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dec. 31, 2017</td>
      <td>85.38</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Nov. 30, 2017</td>
      <td>85.96</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Oct. 31, 2017</td>
      <td>85.59</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sept. 30, 2017</td>
      <td>86.33</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Aug. 31, 2017</td>
      <td>85.30</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>8</th>
      <td>July 31, 2017</td>
      <td>87.57</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>9</th>
      <td>June 30, 2017</td>
      <td>87.05</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>10</th>
      <td>May 31, 2017</td>
      <td>88.35</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>11</th>
      <td>April 30, 2017</td>
      <td>86.90</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>12</th>
      <td>March 31, 2017</td>
      <td>88.26</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Feb. 28, 2017</td>
      <td>90.08</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Jan. 31, 2017</td>
      <td>90.15</td>
      <td>2017</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Dec. 31, 2016</td>
      <td>87.92</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Nov. 30, 2016</td>
      <td>88.44</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Oct. 31, 2016</td>
      <td>87.71</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Sept. 30, 2016</td>
      <td>88.87</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Aug. 31, 2016</td>
      <td>89.36</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>20</th>
      <td>July 31, 2016</td>
      <td>90.74</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>21</th>
      <td>June 30, 2016</td>
      <td>92.98</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>22</th>
      <td>May 31, 2016</td>
      <td>90.09</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>23</th>
      <td>April 30, 2016</td>
      <td>87.08</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>24</th>
      <td>March 31, 2016</td>
      <td>84.50</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Feb. 29, 2016</td>
      <td>82.51</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Jan. 31, 2016</td>
      <td>82.01</td>
      <td>2016</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Dec. 31, 2015</td>
      <td>85.39</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Nov. 30, 2015</td>
      <td>85.65</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Oct. 31, 2015</td>
      <td>86.78</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Sept. 30, 2015</td>
      <td>85.84</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Aug. 31, 2015</td>
      <td>87.52</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>32</th>
      <td>July 31, 2015</td>
      <td>90.80</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>33</th>
      <td>June 30, 2015</td>
      <td>90.16</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>34</th>
      <td>May 31, 2015</td>
      <td>90.10</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>35</th>
      <td>April 30, 2015</td>
      <td>90.45</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>36</th>
      <td>March 31, 2015</td>
      <td>90.75</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Feb. 28, 2015</td>
      <td>93.36</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Jan. 31, 2015</td>
      <td>94.67</td>
      <td>2015</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Dec. 31, 2014</td>
      <td>96.81</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Nov. 30, 2014</td>
      <td>98.28</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Oct. 31, 2014</td>
      <td>98.01</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Sept. 30, 2014</td>
      <td>98.42</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Aug. 31, 2014</td>
      <td>102.07</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>44</th>
      <td>July 31, 2014</td>
      <td>103.16</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>45</th>
      <td>June 30, 2014</td>
      <td>105.16</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>46</th>
      <td>May 31, 2014</td>
      <td>107.18</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>47</th>
      <td>April 30, 2014</td>
      <td>107.21</td>
      <td>2014</td>
    </tr>
    <tr>
      <th>48</th>
      <td>March 31, 2014</td>
      <td>108.00</td>
      <td>2014</td>
    </tr>
  </tbody>
</table>
</div>



### Download the data from url into csv

### Getting Data from API:


```python
import requests
import sys
import json

key = 'GlanxFCOEPxiiEXJK37dPGQjnfM0wFfrZf2uuWti'
url = 'https://api.nal.usda.gov/ndb/nutrients/?format=json&api_key=GlanxFCOEPxiiEXJK37dPGQjnfM0wFfrZf2uuWti&nutrients=208&nutrients=203&nutrients=204&nutrients=205&nutrients=291&nutrients=269&nutrients=211&nutrients=212&fg=1100'
response = requests.get(url)
x = response.json()
# df3 = pd.io.json.json_normalize(x['report']['foods'])
# df3.columns = df3.columns.map(lambda x: x.split(".")[-1])

# grab the nutrient information and vegetable name information from nested dict structure from json
my_data = x['report']['foods']
nut_info = [nut for veggie in my_data for nut in veggie['nutrients']] # list of nutrient info
veg = [veggie['name'] for veggie in my_data for nut in veggie['nutrients']] # list of vegetable names - each name is repeated 8 times fro 8 nutrients

#Create dictionary to get just the nutrient value in 'gm' 
gm_dict = {}
gm_dict = defaultdict(list)
for item in nut_info:
    gm_dict['gm'].append(item['gm'])
# create the data frame of the nutrient info - each nutrient values is in each row
nutrients_df_raw = pd.DataFrame(gm_dict)

# get the nutrients in a column format by reshaping the dataframe - total 8 nutrients, so 8 columns are created

nutrients_df_raw = pd.DataFrame(np.reshape(nutrients_df_raw.values,(150,8)), 
                    columns=['Glucose','Fructose','Protein','Sugars','fat','Carbohydrate' , 'Energy', 'Fiber'])
nutrients_df_raw['Veg_full_name'] = veg[0::8] # since each name is repeated 8 times, so grab every eighth name

```


```python
nutrients_df_raw
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
      <th>Glucose</th>
      <th>Fructose</th>
      <th>Protein</th>
      <th>Sugars</th>
      <th>fat</th>
      <th>Carbohydrate</th>
      <th>Energy</th>
      <th>Fiber</th>
      <th>Veg_full_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.08</td>
      <td>0.12</td>
      <td>3.99</td>
      <td>0.2</td>
      <td>0.69</td>
      <td>2.1</td>
      <td>23</td>
      <td>1.9</td>
      <td>Alfalfa seeds, sprouted, raw</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.24</td>
      <td>0.02</td>
      <td>2.89</td>
      <td>0.99</td>
      <td>0.34</td>
      <td>11.39</td>
      <td>51</td>
      <td>5.7</td>
      <td>Artichokes, (globe or french), cooked, boiled,...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.24</td>
      <td>0.02</td>
      <td>2.89</td>
      <td>0.99</td>
      <td>0.34</td>
      <td>11.95</td>
      <td>53</td>
      <td>5.7</td>
      <td>Artichokes, (globe or french), cooked, boiled,...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.42</td>
      <td>0.79</td>
      <td>2.4</td>
      <td>1.3</td>
      <td>0.22</td>
      <td>4.11</td>
      <td>22</td>
      <td>2</td>
      <td>Asparagus, cooked, boiled, drained</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.42</td>
      <td>0.79</td>
      <td>2.4</td>
      <td>1.3</td>
      <td>0.22</td>
      <td>4.11</td>
      <td>22</td>
      <td>2</td>
      <td>Asparagus, cooked, boiled, drained, with salt</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.65</td>
      <td>1</td>
      <td>2.2</td>
      <td>1.88</td>
      <td>0.12</td>
      <td>3.88</td>
      <td>20</td>
      <td>2.1</td>
      <td>Asparagus, raw</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.29</td>
      <td>0.33</td>
      <td>1.12</td>
      <td>0.78</td>
      <td>0.46</td>
      <td>4.32</td>
      <td>22</td>
      <td>1.9</td>
      <td>Beans, snap, green, canned, no salt added, dra...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.63</td>
      <td>0.62</td>
      <td>1.05</td>
      <td>1.44</td>
      <td>0.41</td>
      <td>4.19</td>
      <td>21</td>
      <td>1.9</td>
      <td>Beans, snap, green, canned, regular pack, drai...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.83</td>
      <td>1.36</td>
      <td>1.98</td>
      <td>2.6</td>
      <td>0.41</td>
      <td>6.98</td>
      <td>33</td>
      <td>3.4</td>
      <td>Beans, snap, green, frozen, all styles, microw...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.92</td>
      <td>1.04</td>
      <td>1.79</td>
      <td>2.21</td>
      <td>0.21</td>
      <td>7.54</td>
      <td>33</td>
      <td>2.6</td>
      <td>Beans, snap, green, frozen, all styles, unprep...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1.44</td>
      <td>1.45</td>
      <td>2.31</td>
      <td>3.22</td>
      <td>0.5</td>
      <td>6.41</td>
      <td>33</td>
      <td>3.4</td>
      <td>Beans, snap, green, microwaved</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1.51</td>
      <td>1.39</td>
      <td>1.83</td>
      <td>3.26</td>
      <td>0.22</td>
      <td>6.97</td>
      <td>31</td>
      <td>2.7</td>
      <td>Beans, snap, green, raw</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.28</td>
      <td>0.2</td>
      <td>0.73</td>
      <td>6.53</td>
      <td>0.09</td>
      <td>7.14</td>
      <td>30</td>
      <td>1.2</td>
      <td>Beets, canned, regular pack, solids and liquids</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.23</td>
      <td>0.24</td>
      <td>3.83</td>
      <td>0.62</td>
      <td>0.52</td>
      <td>3.12</td>
      <td>25</td>
      <td>2.8</td>
      <td>Broccoli raab, cooked</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0.1</td>
      <td>0.17</td>
      <td>3.17</td>
      <td>0.38</td>
      <td>0.49</td>
      <td>2.85</td>
      <td>22</td>
      <td>2.7</td>
      <td>Broccoli raab, raw</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.49</td>
      <td>0.74</td>
      <td>2.38</td>
      <td>1.39</td>
      <td>0.41</td>
      <td>7.18</td>
      <td>35</td>
      <td>3.3</td>
      <td>Broccoli, cooked, boiled, drained, with salt</td>
    </tr>
    <tr>
      <th>16</th>
      <td>0.49</td>
      <td>0.74</td>
      <td>2.38</td>
      <td>1.39</td>
      <td>0.41</td>
      <td>7.18</td>
      <td>35</td>
      <td>3.3</td>
      <td>Broccoli, cooked, boiled, drained, without salt</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0.75</td>
      <td>0.83</td>
      <td>2.81</td>
      <td>1.35</td>
      <td>0.29</td>
      <td>4.78</td>
      <td>26</td>
      <td>3</td>
      <td>Broccoli, frozen, chopped, unprepared</td>
    </tr>
    <tr>
      <th>18</th>
      <td>0.49</td>
      <td>0.68</td>
      <td>2.82</td>
      <td>1.7</td>
      <td>0.37</td>
      <td>6.64</td>
      <td>34</td>
      <td>2.6</td>
      <td>Broccoli, raw</td>
    </tr>
    <tr>
      <th>19</th>
      <td>0.81</td>
      <td>0.93</td>
      <td>3.38</td>
      <td>2.2</td>
      <td>0.3</td>
      <td>8.95</td>
      <td>43</td>
      <td>3.8</td>
      <td>Brussels sprouts, raw</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1.62</td>
      <td>1.16</td>
      <td>1.27</td>
      <td>2.79</td>
      <td>0.06</td>
      <td>5.51</td>
      <td>23</td>
      <td>1.9</td>
      <td>Cabbage, common, cooked, boiled, drained, with...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1.62</td>
      <td>1.16</td>
      <td>1.27</td>
      <td>2.79</td>
      <td>0.06</td>
      <td>5.51</td>
      <td>23</td>
      <td>1.9</td>
      <td>Cabbage, cooked, boiled, drained, without salt</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1.67</td>
      <td>1.45</td>
      <td>1.28</td>
      <td>3.2</td>
      <td>0.1</td>
      <td>5.8</td>
      <td>25</td>
      <td>2.5</td>
      <td>Cabbage, raw</td>
    </tr>
    <tr>
      <th>23</th>
      <td>1.42</td>
      <td>1.2</td>
      <td>1.51</td>
      <td>3.32</td>
      <td>0.09</td>
      <td>6.94</td>
      <td>29</td>
      <td>2.6</td>
      <td>Cabbage, red, cooked, boiled, drained, with salt</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1.42</td>
      <td>1.2</td>
      <td>1.51</td>
      <td>3.32</td>
      <td>0.09</td>
      <td>6.94</td>
      <td>29</td>
      <td>2.6</td>
      <td>Cabbage, red, cooked, boiled, drained, without...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1.74</td>
      <td>1.48</td>
      <td>1.43</td>
      <td>3.83</td>
      <td>0.16</td>
      <td>7.37</td>
      <td>31</td>
      <td>2.1</td>
      <td>Cabbage, red, raw</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1.04</td>
      <td>1</td>
      <td>0.64</td>
      <td>4.76</td>
      <td>0.13</td>
      <td>8.24</td>
      <td>35</td>
      <td>2.9</td>
      <td>Carrots, baby, raw</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0.28</td>
      <td>0.26</td>
      <td>0.59</td>
      <td>2.46</td>
      <td>0.14</td>
      <td>5.36</td>
      <td>23</td>
      <td>1.8</td>
      <td>Carrots, canned, no salt added, solids and liq...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>0.4</td>
      <td>0.36</td>
      <td>0.76</td>
      <td>3.45</td>
      <td>0.18</td>
      <td>8.22</td>
      <td>35</td>
      <td>3</td>
      <td>Carrots, cooked, boiled, drained, with salt</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0.4</td>
      <td>0.36</td>
      <td>0.76</td>
      <td>3.45</td>
      <td>0.18</td>
      <td>8.22</td>
      <td>35</td>
      <td>3</td>
      <td>Carrots, cooked, boiled, drained, without salt</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>120</th>
      <td>0.52</td>
      <td>0.55</td>
      <td>0.5</td>
      <td>1.07</td>
      <td>0.3</td>
      <td>2.41</td>
      <td>12</td>
      <td>1</td>
      <td>Pickles, cucumber, dill, reduced sodium</td>
    </tr>
    <tr>
      <th>121</th>
      <td>9.17</td>
      <td>8.81</td>
      <td>0.58</td>
      <td>18.27</td>
      <td>0.41</td>
      <td>21.15</td>
      <td>91</td>
      <td>1</td>
      <td>Pickles, cucumber, sweet (includes bread and b...</td>
    </tr>
    <tr>
      <th>122</th>
      <td>0.76</td>
      <td>0.56</td>
      <td>6.08</td>
      <td>1.79</td>
      <td>14.76</td>
      <td>27.81</td>
      <td>268</td>
      <td>3.3</td>
      <td>Potato pancakes</td>
    </tr>
    <tr>
      <th>123</th>
      <td>0.19</td>
      <td>0</td>
      <td>2.13</td>
      <td>0.27</td>
      <td>9.05</td>
      <td>27.29</td>
      <td>192</td>
      <td>2</td>
      <td>Potato puffs, frozen, oven-heated</td>
    </tr>
    <tr>
      <th>124</th>
      <td>0.16</td>
      <td>0</td>
      <td>1.93</td>
      <td>0.28</td>
      <td>8.71</td>
      <td>24.8</td>
      <td>178</td>
      <td>2.3</td>
      <td>Potato puffs, frozen, unprepared</td>
    </tr>
    <tr>
      <th>125</th>
      <td>0.44</td>
      <td>0.34</td>
      <td>2.5</td>
      <td>1.18</td>
      <td>0.13</td>
      <td>21.15</td>
      <td>93</td>
      <td>2.2</td>
      <td>Potatoes, baked, flesh and skin, with salt</td>
    </tr>
    <tr>
      <th>126</th>
      <td>0.44</td>
      <td>0.34</td>
      <td>2.5</td>
      <td>1.18</td>
      <td>0.13</td>
      <td>21.15</td>
      <td>93</td>
      <td>2.2</td>
      <td>Potatoes, baked, flesh and skin, without salt</td>
    </tr>
    <tr>
      <th>127</th>
      <td>0.34</td>
      <td>0.29</td>
      <td>1.87</td>
      <td>0.91</td>
      <td>0.1</td>
      <td>20.13</td>
      <td>87</td>
      <td>1.8</td>
      <td>Potatoes, boiled, cooked in skin, flesh, witho...</td>
    </tr>
    <tr>
      <th>128</th>
      <td>0.33</td>
      <td>0.28</td>
      <td>1.71</td>
      <td>0.89</td>
      <td>0.1</td>
      <td>20.01</td>
      <td>86</td>
      <td>1.8</td>
      <td>Potatoes, boiled, cooked without skin, flesh, ...</td>
    </tr>
    <tr>
      <th>129</th>
      <td>0.31</td>
      <td>0.26</td>
      <td>2.05</td>
      <td>0.82</td>
      <td>0.09</td>
      <td>17.49</td>
      <td>77</td>
      <td>2.1</td>
      <td>Potatoes, flesh and skin, raw</td>
    </tr>
    <tr>
      <th>130</th>
      <td>0.18</td>
      <td>0</td>
      <td>2.75</td>
      <td>0.37</td>
      <td>5.48</td>
      <td>25.55</td>
      <td>158</td>
      <td>2</td>
      <td>Potatoes, french fried, all types, salt added ...</td>
    </tr>
    <tr>
      <th>131</th>
      <td>0.1</td>
      <td>0</td>
      <td>2.24</td>
      <td>0.2</td>
      <td>4.66</td>
      <td>24.81</td>
      <td>147</td>
      <td>1.9</td>
      <td>Potatoes, french fried, all types, salt added ...</td>
    </tr>
    <tr>
      <th>132</th>
      <td>0.1</td>
      <td>0</td>
      <td>2.24</td>
      <td>0.2</td>
      <td>4.66</td>
      <td>24.81</td>
      <td>147</td>
      <td>1.9</td>
      <td>Potatoes, french fried, all types, salt not ad...</td>
    </tr>
    <tr>
      <th>133</th>
      <td>0.11</td>
      <td>0</td>
      <td>2.66</td>
      <td>0.28</td>
      <td>5.22</td>
      <td>28.71</td>
      <td>168</td>
      <td>2.6</td>
      <td>Potatoes, french fried, all types, salt not ad...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>0.07</td>
      <td>0</td>
      <td>2.34</td>
      <td>0.21</td>
      <td>4.99</td>
      <td>23.96</td>
      <td>146</td>
      <td>2</td>
      <td>Potatoes, french fried, crinkle or regular cut...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>0.12</td>
      <td>0</td>
      <td>2.51</td>
      <td>0.29</td>
      <td>5.13</td>
      <td>27.5</td>
      <td>161</td>
      <td>2.3</td>
      <td>Potatoes, french fried, crinkle or regular cut...</td>
    </tr>
    <tr>
      <th>136</th>
      <td>0.12</td>
      <td>0</td>
      <td>2.16</td>
      <td>0.2</td>
      <td>6.24</td>
      <td>25.59</td>
      <td>163</td>
      <td>2.3</td>
      <td>Potatoes, french fried, shoestring, salt added...</td>
    </tr>
    <tr>
      <th>137</th>
      <td>0.1</td>
      <td>0</td>
      <td>2.9</td>
      <td>0.31</td>
      <td>6.76</td>
      <td>31.66</td>
      <td>194</td>
      <td>2.8</td>
      <td>Potatoes, french fried, shoestring, salt added...</td>
    </tr>
    <tr>
      <th>138</th>
      <td>0.1</td>
      <td>0</td>
      <td>2.19</td>
      <td>0.2</td>
      <td>3.39</td>
      <td>23.51</td>
      <td>130</td>
      <td>1.9</td>
      <td>Potatoes, french fried, steak fries, salt adde...</td>
    </tr>
    <tr>
      <th>139</th>
      <td>0.1</td>
      <td>0</td>
      <td>2.57</td>
      <td>0.25</td>
      <td>3.76</td>
      <td>26.98</td>
      <td>148</td>
      <td>2.6</td>
      <td>Potatoes, french fried, steak fries, salt adde...</td>
    </tr>
    <tr>
      <th>140</th>
      <td>0.26</td>
      <td>0</td>
      <td>2.65</td>
      <td>0.27</td>
      <td>11.59</td>
      <td>28.51</td>
      <td>219</td>
      <td>3.2</td>
      <td>Potatoes, hash brown, frozen, plain, prepared,...</td>
    </tr>
    <tr>
      <th>141</th>
      <td>0.64</td>
      <td>0.52</td>
      <td>3</td>
      <td>1.49</td>
      <td>12.52</td>
      <td>35.11</td>
      <td>265</td>
      <td>3.2</td>
      <td>Potatoes, hash brown, home-prepared</td>
    </tr>
    <tr>
      <th>142</th>
      <td>0.61</td>
      <td>0.45</td>
      <td>3.24</td>
      <td>1.16</td>
      <td>10.3</td>
      <td>33.99</td>
      <td>242</td>
      <td>3.6</td>
      <td>Potatoes, hash brown, refrigerated, prepared, ...</td>
    </tr>
    <tr>
      <th>143</th>
      <td>0.54</td>
      <td>0.36</td>
      <td>1.75</td>
      <td>0.91</td>
      <td>0.08</td>
      <td>19.16</td>
      <td>84</td>
      <td>1.8</td>
      <td>Potatoes, hash brown, refrigerated, unprepared</td>
    </tr>
    <tr>
      <th>144</th>
      <td>0.89</td>
      <td>0.98</td>
      <td>8.34</td>
      <td>3.36</td>
      <td>0.41</td>
      <td>81.17</td>
      <td>354</td>
      <td>6.6</td>
      <td>Potatoes, mashed, dehydrated, flakes without m...</td>
    </tr>
    <tr>
      <th>145</th>
      <td>0.92</td>
      <td>1.01</td>
      <td>8.22</td>
      <td>3.47</td>
      <td>0.54</td>
      <td>85.51</td>
      <td>372</td>
      <td>7.1</td>
      <td>Potatoes, mashed, dehydrated, granules without...</td>
    </tr>
    <tr>
      <th>146</th>
      <td>0.11</td>
      <td>0.12</td>
      <td>1.77</td>
      <td>1.61</td>
      <td>5.13</td>
      <td>10.87</td>
      <td>97</td>
      <td>0.8</td>
      <td>Potatoes, mashed, dehydrated, prepared from fl...</td>
    </tr>
    <tr>
      <th>147</th>
      <td>0.16</td>
      <td>0.18</td>
      <td>2.13</td>
      <td>1.74</td>
      <td>4.8</td>
      <td>16.13</td>
      <td>116</td>
      <td>1.3</td>
      <td>Potatoes, mashed, dehydrated, prepared from gr...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>0.31</td>
      <td>0.25</td>
      <td>1.91</td>
      <td>1.48</td>
      <td>0.57</td>
      <td>17.57</td>
      <td>83</td>
      <td>1.5</td>
      <td>Potatoes, mashed, home-prepared, whole milk added</td>
    </tr>
    <tr>
      <th>149</th>
      <td>0.29</td>
      <td>0.24</td>
      <td>1.86</td>
      <td>1.43</td>
      <td>4.22</td>
      <td>16.81</td>
      <td>113</td>
      <td>1.5</td>
      <td>Potatoes, mashed, home-prepared, whole milk an...</td>
    </tr>
  </tbody>
</table>
<p>150 rows × 9 columns</p>
</div>



### Join Fresh DF and Price index dataframe


```python
pi_by_year = df1.groupby('Year', as_index=False)['Price_Index'].mean() 
pi_by_year['Year'] = pi_by_year['Year'].astype(int) # convert year to int type
# merge fresh_df and pi_by_year on Year column values
final_df = pd.merge(fresh_df, pi_by_year, on="Year",how='left')

```


```python
final_df
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
      <th>Vegetable</th>
      <th>Year</th>
      <th>Production</th>
      <th>Imports</th>
      <th>Supply</th>
      <th>Exports</th>
      <th>Domestic_Availability</th>
      <th>Per_Capita_Availability</th>
      <th>Current_Dollars</th>
      <th>Constant_2009_Dollars</th>
      <th>Price_Index</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>artichokes</td>
      <td>1971</td>
      <td>79.2</td>
      <td>26.1</td>
      <td>105.3</td>
      <td>--</td>
      <td>105.3</td>
      <td>0.507076</td>
      <td>9.72</td>
      <td>40.5608</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>artichokes</td>
      <td>1972</td>
      <td>71</td>
      <td>46.8</td>
      <td>117.8</td>
      <td>--</td>
      <td>117.8</td>
      <td>0.56123</td>
      <td>11.6</td>
      <td>46.3907</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>artichokes</td>
      <td>1973</td>
      <td>60</td>
      <td>43.5</td>
      <td>103.5</td>
      <td>--</td>
      <td>103.5</td>
      <td>0.488417</td>
      <td>14.5</td>
      <td>54.9951</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>artichokes</td>
      <td>1974</td>
      <td>70.2</td>
      <td>37.8</td>
      <td>108</td>
      <td>--</td>
      <td>108</td>
      <td>0.505017</td>
      <td>17.3</td>
      <td>60.2074</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>artichokes</td>
      <td>1975</td>
      <td>73.4</td>
      <td>32.4</td>
      <td>105.8</td>
      <td>--</td>
      <td>105.8</td>
      <td>0.489876</td>
      <td>16.1</td>
      <td>51.2821</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>artichokes</td>
      <td>1976</td>
      <td>80.6</td>
      <td>42.9</td>
      <td>123.5</td>
      <td>4.6</td>
      <td>118.9</td>
      <td>0.545325</td>
      <td>14.4</td>
      <td>43.4796</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>artichokes</td>
      <td>1977</td>
      <td>71.3</td>
      <td>34.5</td>
      <td>105.8</td>
      <td>3.8</td>
      <td>102</td>
      <td>0.463133</td>
      <td>19.3</td>
      <td>54.8716</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>artichokes</td>
      <td>1978</td>
      <td>52.5</td>
      <td>58.5</td>
      <td>111</td>
      <td>3.3</td>
      <td>107.7</td>
      <td>0.48386</td>
      <td>27</td>
      <td>71.7265</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>artichokes</td>
      <td>1979</td>
      <td>87.3</td>
      <td>54</td>
      <td>141.3</td>
      <td>4</td>
      <td>137.3</td>
      <td>0.610073</td>
      <td>27.7</td>
      <td>67.9755</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>artichokes</td>
      <td>1980</td>
      <td>79.2</td>
      <td>58.5</td>
      <td>137.7</td>
      <td>4</td>
      <td>133.7</td>
      <td>0.587109</td>
      <td>34.7</td>
      <td>78.1092</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>artichokes</td>
      <td>1981</td>
      <td>111.6</td>
      <td>65.4</td>
      <td>177</td>
      <td>5.1</td>
      <td>171.9</td>
      <td>0.747502</td>
      <td>32</td>
      <td>65.8816</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>artichokes</td>
      <td>1982</td>
      <td>116.3</td>
      <td>79.2</td>
      <td>195.5</td>
      <td>4.4</td>
      <td>191.1</td>
      <td>0.82304</td>
      <td>32.1</td>
      <td>62.2262</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>artichokes</td>
      <td>1983</td>
      <td>89.2</td>
      <td>86.1</td>
      <td>175.3</td>
      <td>3.2</td>
      <td>172.1</td>
      <td>0.734506</td>
      <td>38.25</td>
      <td>71.3313</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>artichokes</td>
      <td>1984</td>
      <td>109.9</td>
      <td>120</td>
      <td>229.9</td>
      <td>4.3</td>
      <td>225.6</td>
      <td>0.954525</td>
      <td>30.65</td>
      <td>55.2004</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>artichokes</td>
      <td>1985</td>
      <td>133.8</td>
      <td>116.1</td>
      <td>249.9</td>
      <td>6</td>
      <td>243.9</td>
      <td>1.02279</td>
      <td>28.6</td>
      <td>49.911</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>artichokes</td>
      <td>1986</td>
      <td>105.2</td>
      <td>125.4</td>
      <td>230.6</td>
      <td>5.5</td>
      <td>225.1</td>
      <td>0.935379</td>
      <td>34.45</td>
      <td>58.9312</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>artichokes</td>
      <td>1987</td>
      <td>121.7</td>
      <td>124.8</td>
      <td>246.5</td>
      <td>5.4</td>
      <td>241.1</td>
      <td>0.992982</td>
      <td>30.2</td>
      <td>50.3762</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>artichokes</td>
      <td>1988</td>
      <td>124.6</td>
      <td>106.2</td>
      <td>230.8</td>
      <td>6.7</td>
      <td>224.1</td>
      <td>0.914615</td>
      <td>29.35</td>
      <td>47.3021</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>artichokes</td>
      <td>1989</td>
      <td>129.6</td>
      <td>117.702</td>
      <td>247.302</td>
      <td>6.3</td>
      <td>241.002</td>
      <td>0.974368</td>
      <td>26.5</td>
      <td>41.1108</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>artichokes</td>
      <td>1990</td>
      <td>111.4</td>
      <td>106.666</td>
      <td>218.066</td>
      <td>6.5</td>
      <td>211.566</td>
      <td>0.845815</td>
      <td>29.5</td>
      <td>44.1319</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>artichokes</td>
      <td>1991</td>
      <td>120.6</td>
      <td>94.7021</td>
      <td>215.302</td>
      <td>5.7</td>
      <td>209.602</td>
      <td>0.826856</td>
      <td>29.65</td>
      <td>42.9281</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>artichokes</td>
      <td>1992</td>
      <td>110.4</td>
      <td>120.276</td>
      <td>230.676</td>
      <td>4.9</td>
      <td>225.776</td>
      <td>0.878869</td>
      <td>39.4</td>
      <td>55.7726</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>artichokes</td>
      <td>1993</td>
      <td>101.2</td>
      <td>128.117</td>
      <td>229.317</td>
      <td>4.5</td>
      <td>224.817</td>
      <td>0.863834</td>
      <td>50.8</td>
      <td>70.2385</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>artichokes</td>
      <td>1994</td>
      <td>130</td>
      <td>209.488</td>
      <td>339.488</td>
      <td>5.9</td>
      <td>333.588</td>
      <td>1.2663</td>
      <td>56.4</td>
      <td>76.3555</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>artichokes</td>
      <td>1995</td>
      <td>92</td>
      <td>159.608</td>
      <td>251.608</td>
      <td>5.1</td>
      <td>246.508</td>
      <td>0.924784</td>
      <td>75.7</td>
      <td>100.39</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>artichokes</td>
      <td>1996</td>
      <td>89</td>
      <td>182.51</td>
      <td>271.51</td>
      <td>4.69747</td>
      <td>266.812</td>
      <td>0.989414</td>
      <td>73.5</td>
      <td>95.7243</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>artichokes</td>
      <td>1997</td>
      <td>93</td>
      <td>187.328</td>
      <td>280.328</td>
      <td>4.7657</td>
      <td>275.562</td>
      <td>1.00971</td>
      <td>79.5</td>
      <td>101.798</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>artichokes</td>
      <td>1998</td>
      <td>87.3</td>
      <td>243.474</td>
      <td>330.774</td>
      <td>4.98168</td>
      <td>325.793</td>
      <td>1.17992</td>
      <td>70.6</td>
      <td>89.4305</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>artichokes</td>
      <td>1999</td>
      <td>112.5</td>
      <td>264.062</td>
      <td>376.562</td>
      <td>7.6638</td>
      <td>368.898</td>
      <td>1.32082</td>
      <td>67</td>
      <td>83.6757</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>artichokes</td>
      <td>2000</td>
      <td>101.2</td>
      <td>255.531</td>
      <td>356.731</td>
      <td>6.08242</td>
      <td>350.648</td>
      <td>1.24174</td>
      <td>60.3</td>
      <td>73.6345</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>935</th>
      <td>tomatoes</td>
      <td>1988</td>
      <td>3588.9</td>
      <td>816.8</td>
      <td>4405.7</td>
      <td>254.201</td>
      <td>4124.5</td>
      <td>16.8332</td>
      <td>27.1</td>
      <td>43.6759</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>936</th>
      <td>tomatoes</td>
      <td>1989</td>
      <td>3596.2</td>
      <td>867.907</td>
      <td>4464.11</td>
      <td>298.025</td>
      <td>4166.08</td>
      <td>16.8434</td>
      <td>33.2</td>
      <td>51.5048</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>937</th>
      <td>tomatoes</td>
      <td>1990</td>
      <td>3380</td>
      <td>795.857</td>
      <td>4175.86</td>
      <td>293.056</td>
      <td>3882.8</td>
      <td>15.523</td>
      <td>27.4</td>
      <td>40.9904</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>938</th>
      <td>tomatoes</td>
      <td>1991</td>
      <td>3398.8</td>
      <td>795.492</td>
      <td>4194.29</td>
      <td>300.282</td>
      <td>3894.01</td>
      <td>15.3614</td>
      <td>31.7</td>
      <td>45.8961</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>939</th>
      <td>tomatoes</td>
      <td>1992</td>
      <td>3903.3</td>
      <td>432.167</td>
      <td>4335.47</td>
      <td>367.481</td>
      <td>3967.99</td>
      <td>15.446</td>
      <td>35.8</td>
      <td>50.6766</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>940</th>
      <td>tomatoes</td>
      <td>1993</td>
      <td>3666.3</td>
      <td>922.4</td>
      <td>4588.7</td>
      <td>345.831</td>
      <td>4242.87</td>
      <td>16.3027</td>
      <td>31.7</td>
      <td>43.8299</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>941</th>
      <td>tomatoes</td>
      <td>1994</td>
      <td>3738.7</td>
      <td>872.973</td>
      <td>4611.67</td>
      <td>340.749</td>
      <td>4270.92</td>
      <td>16.2124</td>
      <td>27.4</td>
      <td>37.0947</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>942</th>
      <td>tomatoes</td>
      <td>1995</td>
      <td>3409.8</td>
      <td>1368.91</td>
      <td>4778.71</td>
      <td>289.228</td>
      <td>4489.48</td>
      <td>16.8425</td>
      <td>25.5</td>
      <td>33.8169</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>943</th>
      <td>tomatoes</td>
      <td>1996</td>
      <td>3363.4</td>
      <td>1625.14</td>
      <td>4988.54</td>
      <td>295.441</td>
      <td>4693.1</td>
      <td>17.4033</td>
      <td>28.1</td>
      <td>36.5966</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>944</th>
      <td>tomatoes</td>
      <td>1997</td>
      <td>3424.84</td>
      <td>1636.84</td>
      <td>5061.68</td>
      <td>341.677</td>
      <td>4720.01</td>
      <td>17.295</td>
      <td>31.7</td>
      <td>40.5911</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>945</th>
      <td>tomatoes</td>
      <td>1998</td>
      <td>3525.6</td>
      <td>1868.02</td>
      <td>5393.62</td>
      <td>286.324</td>
      <td>5107.29</td>
      <td>18.497</td>
      <td>35.2</td>
      <td>44.5886</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>946</th>
      <td>tomatoes</td>
      <td>1999</td>
      <td>4026.9</td>
      <td>1633.05</td>
      <td>5659.95</td>
      <td>334.35</td>
      <td>5325.6</td>
      <td>19.068</td>
      <td>25.8</td>
      <td>32.2214</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>947</th>
      <td>tomatoes</td>
      <td>2000</td>
      <td>4162</td>
      <td>1609.39</td>
      <td>5771.39</td>
      <td>410.353</td>
      <td>5361.03</td>
      <td>18.9849</td>
      <td>30.7</td>
      <td>37.4889</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>948</th>
      <td>tomatoes</td>
      <td>2001</td>
      <td>4061.1</td>
      <td>1815.64</td>
      <td>5876.74</td>
      <td>397.936</td>
      <td>5478.8</td>
      <td>19.2031</td>
      <td>30</td>
      <td>35.8141</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>949</th>
      <td>tomatoes</td>
      <td>2002</td>
      <td>4289.3</td>
      <td>1894.87</td>
      <td>6184.17</td>
      <td>332.341</td>
      <td>5851.83</td>
      <td>20.3115</td>
      <td>31.6</td>
      <td>37.1529</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>950</th>
      <td>tomatoes</td>
      <td>2003</td>
      <td>3888.4</td>
      <td>2071.14</td>
      <td>5959.54</td>
      <td>314.23</td>
      <td>5645.32</td>
      <td>19.4117</td>
      <td>37.5</td>
      <td>43.2257</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>951</th>
      <td>tomatoes</td>
      <td>2004</td>
      <td>4169.6</td>
      <td>2054.2</td>
      <td>6223.8</td>
      <td>369.269</td>
      <td>5854.53</td>
      <td>19.9498</td>
      <td>37.4</td>
      <td>41.9602</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>952</th>
      <td>tomatoes</td>
      <td>2005</td>
      <td>4196.9</td>
      <td>2098.11</td>
      <td>6295.01</td>
      <td>326.507</td>
      <td>5968.51</td>
      <td>20.1512</td>
      <td>41.6</td>
      <td>45.2218</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>953</th>
      <td>tomatoes</td>
      <td>2006</td>
      <td>4041.1</td>
      <td>2187.88</td>
      <td>6228.98</td>
      <td>317.75</td>
      <td>5911.23</td>
      <td>19.7703</td>
      <td>43.7</td>
      <td>46.0883</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>954</th>
      <td>tomatoes</td>
      <td>2007</td>
      <td>3795.6</td>
      <td>2361.07</td>
      <td>6156.67</td>
      <td>355.69</td>
      <td>5800.98</td>
      <td>19.2083</td>
      <td>34.8</td>
      <td>35.7528</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>955</th>
      <td>tomatoes</td>
      <td>2008</td>
      <td>3554.6</td>
      <td>2460.57</td>
      <td>6015.17</td>
      <td>372.272</td>
      <td>5642.9</td>
      <td>18.5136</td>
      <td>45.3</td>
      <td>45.6488</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>956</th>
      <td>tomatoes</td>
      <td>2009</td>
      <td>3775.5</td>
      <td>2622.62</td>
      <td>6398.12</td>
      <td>375.589</td>
      <td>6022.53</td>
      <td>19.5893</td>
      <td>40.4</td>
      <td>40.4</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>957</th>
      <td>tomatoes</td>
      <td>2010</td>
      <td>3253.8</td>
      <td>3378.56</td>
      <td>6632.36</td>
      <td>266.214</td>
      <td>6366.14</td>
      <td>20.5514</td>
      <td>48.2</td>
      <td>47.6233</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>958</th>
      <td>tomatoes</td>
      <td>2011</td>
      <td>3508.14</td>
      <td>3287.12</td>
      <td>6795.26</td>
      <td>252.57</td>
      <td>6542.69</td>
      <td>20.961</td>
      <td>36.1</td>
      <td>34.981</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>959</th>
      <td>tomatoes</td>
      <td>2012</td>
      <td>3412.84</td>
      <td>3377.84</td>
      <td>6790.68</td>
      <td>258.665</td>
      <td>6532.01</td>
      <td>20.7735</td>
      <td>30.5</td>
      <td>29.0024</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>960</th>
      <td>tomatoes</td>
      <td>2013</td>
      <td>3253.83</td>
      <td>3389.54</td>
      <td>6643.37</td>
      <td>241.335</td>
      <td>6402.03</td>
      <td>20.213</td>
      <td>44.6</td>
      <td>41.7112</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>961</th>
      <td>tomatoes</td>
      <td>2014</td>
      <td>3386.37</td>
      <td>3435.79</td>
      <td>6822.16</td>
      <td>248.74</td>
      <td>6573.42</td>
      <td>20.5945</td>
      <td>41.5</td>
      <td>38.185</td>
      <td>102.430000</td>
    </tr>
    <tr>
      <th>962</th>
      <td>tomatoes</td>
      <td>2015</td>
      <td>3342.62</td>
      <td>3468.35</td>
      <td>6810.97</td>
      <td>214.182</td>
      <td>6596.79</td>
      <td>20.5423</td>
      <td>46.3</td>
      <td>42.1779</td>
      <td>89.289167</td>
    </tr>
    <tr>
      <th>963</th>
      <td>tomatoes</td>
      <td>2016</td>
      <td>2830.93</td>
      <td>3938.33</td>
      <td>6769.26</td>
      <td>186.894</td>
      <td>6582.36</td>
      <td>20.3568</td>
      <td>42.5</td>
      <td>38.1363</td>
      <td>87.684167</td>
    </tr>
    <tr>
      <th>964</th>
      <td>tomatoes</td>
      <td>2017</td>
      <td>2845.42</td>
      <td>3943.63</td>
      <td>6789.06</td>
      <td>188.245</td>
      <td>6600.81</td>
      <td>20.2749</td>
      <td>37.3</td>
      <td>32.9688</td>
      <td>87.243333</td>
    </tr>
  </tbody>
</table>
<p>965 rows × 11 columns</p>
</div>



### Find the common vegetables in Nutrient DF and Final DF and then merge the two DF 


```python
raw_names_nut = [ item for item in nutrients_df_raw['Veg_full_name'] if item.find('raw') > 0]
# find nutrient information for all the vegetables that are present in the fresh vegetables DF
nutrients_df_raw['Vegetable'] = ''
for veg in final_df['Vegetable'].unique():
    veggie = veg + ', raw'
    raw_veggie = [rnn for rnn in raw_names_nut if veggie.lower() in rnn.lower()]
    if len(raw_veggie) > 0:
        true_idx = nutrients_df_raw['Veg_full_name'] == raw_veggie[0]
        nutrients_df_raw['Vegetable'][true_idx] = veg
# Merge nutrients dataframe and Final DF
final_df = pd.merge(final_df, nutrients_df_raw, on="Vegetable", how = 'left')

# Write the dataframe data to excel file
writer = pd.ExcelWriter('final_data.xlsx', engine='xlsxwriter')
final_df.to_excel(writer, sheet_name='Sheet1')
writer.save()
```


```python
final_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 965 entries, 0 to 964
    Data columns (total 20 columns):
    Vegetable                  965 non-null object
    Year                       965 non-null object
    Production                 937 non-null object
    Imports                    937 non-null object
    Supply                     937 non-null object
    Exports                    937 non-null object
    Domestic_Availability      937 non-null object
    Per_Capita_Availability    937 non-null object
    Current_Dollars            937 non-null object
    Constant_2009_Dollars      937 non-null object
    Price_Index                84 non-null float64
    Glucose                    309 non-null object
    Fructose                   309 non-null object
    Protein                    309 non-null object
    Sugars                     309 non-null object
    fat                        309 non-null object
    Carbohydrate               309 non-null object
    Energy                     309 non-null object
    Fiber                      309 non-null object
    Veg_full_name              309 non-null object
    dtypes: float64(1), object(19)
    memory usage: 158.3+ KB
    


```python
final_df = final_df[np.isfinite(final_df['Price_Index'])]
#final_df.set_printoptions(max_rows=500)
# final_df
#.isfinite is used to remove all the Nan values
```


```python
imports = final_df.groupby('Vegetable')['Imports'].sum() 
imports.plot.bar(figsize=(20,10))
plt.title('Total Imports per vegetable')
plt.xticks(rotation=20)
plt.ylabel('Imports')
plt.show()

# Grouping by the vegetable imports
# xticks (rotation = 20), rotates the text 20 degrees
```


![png](output_21_0.png)


# Total Imports per vegetable
The graph shows that tomatoes, cucumbers, potatoes and squash have the highest imports 


```python
price = final_df.groupby('Vegetable')['Current_Dollars'].sum() 
price.plot.bar(figsize=(20,10))
plt.title('Current price of the vegetables')
plt.xticks(rotation=20)
plt.ylabel('Price Current Dollars')
plt.show()

#Grouping by the vegetable imports
# xticks (rotation = 20), rotates the text 20 degrees 
```


![png](output_23_0.png)


# Total Imports per vegetable vs Price Current dollars
The graph shows that asparagus has the highest price since it is mostly imported


```python
pd.options.mode.chained_assignment = None
#The above statement is used to deal with the below warning
#SettingWithCopyWarning: A value is trying to be set on a copy of a slice from a DataFrame.
final_df['prod_imp'] = final_df.eval('Production>Imports')
#Creating a new dataframe by evaluating if Production>imports
#The idea is to display the current dollars for vegetables where the production is greater than the imports
```


```python
Product_greater_imports_true = final_df.loc[final_df['prod_imp']== True]
```


```python
#Product_greater_imports_true['Price_Index'] 
pd.options.mode.chained_assignment = None
Product_greater_imports_true['Current_Dollars_num'] = pd.to_numeric(Product_greater_imports_true['Current_Dollars'])
# using pd.to_numeric to change the type of Current_Dollars from Object to number
Product_greater_imports_true_plot = Product_greater_imports_true.groupby(['Vegetable'], as_index = False)['Current_Dollars_num'].mean()
# grouping by the vegetable Current_dollars and taking the mean of the data 
Product_greater_imports_true_plot.plot.bar(figsize=(20,10))
Product_greater_imports_true_plot.sort_values('Current_Dollars_num', ascending=True)['Current_Dollars_num'].plot.bar(x=Product_greater_imports_true_plot['Vegetable'], stacked=True)
plt.title('Current Dollars for Vegetables where production is greater than imports')
plt.xticks(rotation=40)
plt.ylabel('Price Current Dollars')
plt.show()
```


![png](output_27_0.png)


# Current Dollars for Vegetables where production is greater than imports
The graph shows that for vegetables whose production is greater than imports, the average current price is $40 dollars


```python
final_df['protein_sugars_ratio'] = final_df['Protein'] / final_df['Sugars']
#final_df['protein_sugars_ratio']
#The idea is to display the vegetable dataframe where the ratio of protein content is greater than the sugars
```


```python
ratio_greater_protein_current = final_df.loc[final_df['protein_sugars_ratio']> 1]
#getting the vegeatables whose protein content is greater than sugars
```


```python
pd.options.mode.chained_assignment = None
ratio_greater_protein_current['Current_Dollars_num'] = pd.to_numeric(ratio_greater_protein_current['Current_Dollars'])
# using pd.to_numeric to change the type of Current_Dollars from Object to number
```


```python
Price_vegetable_greater_protein = ratio_greater_protein_current.groupby(['Vegetable'])['Current_Dollars_num'].mean()
#grouping by the vegetable Current_dollars and taking the mean of the data 
Price_vegetable_greater_protein.plot.bar(figsize=(20,10))
plt.title('Current Dollars for Vegetables where protein content is greater than sugars')
plt.xticks(rotation=20)
plt.ylabel('Price Current Dollars')
plt.show()
```


![png](output_32_0.png)


# Current Dollars for Vegetables where protein content is greater than sugars
The graph shows that only four vegetables have protein content greater than sugar, but it doesn't necessarily mean they are expensive


```python
final_df['Current_Dollars_num'] = pd.to_numeric(final_df['Current_Dollars'])
#final_df
```


```python
pd.options.mode.chained_assignment = None
Product_greater_imports_true['Current_Dollars_num'] = pd.to_numeric(Product_greater_imports_true['Current_Dollars'])
```


```python
Product_greater_imports_true_plot = Product_greater_imports_true.groupby(['Vegetable'], as_index = False)['Current_Dollars_num'].mean()
#grouping by the vegetable Current_dollars and taking the mean of the data 
Product_greater_imports_true_plot.plot.bar(figsize=(20,10))
Product_greater_imports_true_plot.sort_values('Current_Dollars_num', ascending=True)['Current_Dollars_num'].plot.bar(x=Product_greater_imports_true_plot['Vegetable'], stacked=True)
plt.title('Current Dollars for Vegetables where production is greater than imports')
plt.xticks(rotation=40)
plt.ylabel('Price Current Dollars')
plt.show()

```


![png](output_36_0.png)


# Current Dollars for Vegetables where production is greater than imports
The graph shows that for vegeatables with production greater than imports, the average price is greater than $45


```python
# final_df[:10]
pmi_data = final_df.loc[final_df['Vegetable'].isin(['artichokes'])]
pmi_data2 = final_df.loc[final_df['Vegetable'].isin(['asparagus'])]
pmi_data3 = final_df.loc[final_df['Vegetable'].isin(['kale'])]
pmi_data4 = final_df.loc[final_df['Vegetable'].isin(['mustard greens'])]
pmi_data5 = final_df.loc[final_df['Vegetable'].isin(['spinach'])]
pmi_data6 = final_df.loc[final_df['Vegetable'].isin(['potatoes'])] 
pmi_data7 = final_df.loc[final_df['Vegetable'].isin(['celery'])]
pmi_data8 = final_df.loc[final_df['Vegetable'].isin(['sweet corn'])]
pmi_data9 = final_df.loc[final_df['Vegetable'].isin(['collard greens'])]
pmi_data10 = final_df.loc[final_df['Vegetable'].isin(['leaf & romanie lettuce'])]
# df.loc[df['B'].isin(['one','three'])]
plt.figure(figsize=(20,10))
plt.plot(pmi_data['Year'], pmi_data['Constant_2009_Dollars'], label = 'Artichokes')
plt.plot(pmi_data2['Year'], pmi_data2['Constant_2009_Dollars'], label = 'Asparagus')
plt.plot(pmi_data3['Year'], pmi_data3['Constant_2009_Dollars'], label = 'Kale')
plt.plot(pmi_data4['Year'], pmi_data4['Constant_2009_Dollars'], label = 'Mustard Greens')
plt.plot(pmi_data5['Year'], pmi_data5['Constant_2009_Dollars'], label = 'Spinach')
plt.plot(pmi_data6['Year'], pmi_data6['Constant_2009_Dollars'], label = 'Potatoes')
plt.plot(pmi_data7['Year'], pmi_data7['Constant_2009_Dollars'], label = 'Celery')
plt.plot(pmi_data8['Year'], pmi_data8['Constant_2009_Dollars'], label = 'Sweet Corn')
plt.plot(pmi_data9['Year'], pmi_data9['Constant_2009_Dollars'], label = 'Collard Greens')
plt.plot(pmi_data10['Year'], pmi_data10['Constant_2009_Dollars'], label = 'Lettuce')
plt.plot(pmi_data2['Year'], pmi_data2['Price_Index'],label = 'Price Index')
plt.ylabel('Price in $ per cwt', size = 30)
plt.xlabel('Year', size = 30)
plt.xticks(range(2014, 2018))
plt.legend()

```




    <matplotlib.legend.Legend at 0x263f2d90c18>




![png](output_38_1.png)


 # Comparing the vegeatables price per cwt to the Market Price Index
 The graph shows that only asparagus is greater than market price index. 
