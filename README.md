# What drives the price of a car?

![](images/kurt.jpeg)

**OVERVIEW**

In this application, we explored an used cars dataset from kaggle. This is a subset from the original dataset that contained information on 3 million used cars. The provided dataset contains information on 426K cars to ensure speed of processing.  Our goal is to understand what factors make a car more or less expensive.  And also provide clear recommendations to the Used Car Dealership owners as to what consumers value in a used car.

### Goal: 

To understand what factors make a car more or less expensive and provide clear recommendations to your client (an used car dealership) as to what consumers value in a used car.
    

The dataset provides data about various features of the cars like number of cylinders in the car, model of the car, manufacturer, drive type (forward drive/rearward dive/all wheel drive),number of miles the car drove etc.
By exploring the data, we need to determine,
1. How each feature is correlated to the price and if they negatively or positively impact the pricing of the car?
2. If we have limited data, like data only for few features, how the price will be impacted?
3. Are there any features that help us determine if we can price the car lower or higher?

This information should help the car dealer to understand what features the consumers are looking for in a car when they want to buy an used car


### Data Understanding


- Read the csv file
- Performed Feature Analysis by understanding the features that may impact the price. Gathered the below points from various sources in internet

    - Region, state - Supply and demand for certain vehicle types varies geographically, which is why used car prices vary across cities and states,” said iSeeCars Executive Analyst Karl Brauer. “States with temperate climates tend to have more lower-priced sedans, while mountainous and harsh climate areas prefer trucks and SUVs.

    - VIN - By entering your License Plate or VIN, you'll get a more accurate appraisal that may raise your car's value.

    - Transmission - A vehicle with a bad transmission has the additional option of “being repaired.” However, as stated before, transmission repairs can be costly. While the option to sell or trade-in your car is on the table, keep in mind that its value is negatively impacted by a bad transmission

    - Cylinder - Generally, the more cylinders your engine has, the more power is produced. Most cars have a 4, 6, or 8 cylinder engine. 

    - Odometer - Cars with more miles on the odometer are generally worth less, so you'll be able to get them cheaper. Opposingly, cars with fewer miles on the odometer usually hold a higher value, barring any major mechanical or electrical failures, which will make them slightly more expensive.

- Also, identified that almost all types of used cars are available in different regions/states. 

- Imported the different libraries in Python to get started with the exploratory data analysis


```python
cars.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 426880 entries, 0 to 426879
    Data columns (total 18 columns):
     #   Column        Non-Null Count   Dtype  
    ---  ------        --------------   -----  
     0   id            426880 non-null  int64  
     1   region        426880 non-null  object 
     2   price         426880 non-null  int64  
     3   year          425675 non-null  float64
     4   manufacturer  409234 non-null  object 
     5   model         421603 non-null  object 
     6   condition     252776 non-null  object 
     7   cylinders     249202 non-null  object 
     8   fuel          423867 non-null  object 
     9   odometer      422480 non-null  float64
     10  title_status  418638 non-null  object 
     11  transmission  424324 non-null  object 
     12  VIN           265838 non-null  object 
     13  drive         296313 non-null  object 
     14  size          120519 non-null  object 
     15  type          334022 non-null  object 
     16  paint_color   296677 non-null  object 
     17  state         426880 non-null  object 
    dtypes: float64(2), int64(2), object(14)
    memory usage: 58.6+ MB
    



#### Exploratory Data Analysis
1. Looked at the datapoints in the dataset
2. Dataset has used cars from sale from almost 100 years back. Assuming there are very few people who buy cars more than 50 years old, I dropped the rows from the dataset that have cars manufacturing year less than 1970. This resulted in dropping of 5283 rows
3. Dropped the column Id as it doesn't influence price in any way.
4. Dropped the columns manufacturer, region and state as irrespective of these, our goal is to determine what customers would value in a used car. 
5. Dropped the Size column as 75% of the data in the column is missing.
6. Dropping model as there is a type column

    


```python
cars_data = cars.query("year >=1970")
cars_data.shape
```




    (420393, 18)




```python
cars_data.drop(columns = ['id', 'region', 'paint_color', 'model', 'size', 'manufacturer','state'], inplace= True)
cars_data.shape
```

    (420393, 11)



 ### Data Cleansing

**VIN**

Presence/absence of VIN is what is required to appraise the car correctly. Hence converted the VIN into a numeric column by assigning 1 for all the rows where VIN is present and assigned 0 where VIN is missing. Converted the column into integer column using pd.to_numeric() funciton.


```python
cars_data['VIN'].fillna(0, inplace=True)
cars_data.loc[cars_data["VIN"] != 0, "VIN"] = 1
cars_data['VIN']=pd.to_numeric(cars_data['VIN'])
cars_data['VIN'].value_counts()
```

    1    264036
    0    156357
    Name: VIN, dtype: int64



#### Cylinders

Removed the string value " cylinders" in cylinders column values. Converted the data into integers. Calculated the mean value of the column and it's 6. So, assigned 6 to the missing and 'other' cylinder data. Converted this column also into numeric



```python
cars_data['cylinders'] = cars_data.query("cylinders != ''")['cylinders'].str.replace(' cylinders', '')
cars_data['cylinders'].value_counts()
```


    6        93069
    4        76903
    8        69469
    5         1711
    10        1447
    other     1226
    3          644
    12         207
    Name: cylinders, dtype: int64




```python
cars_data.loc[cars_data["cylinders"] =='other', "cylinders"] = '6'
cars_data.loc[cars_data["cylinders"].isnull(), "cylinders"] = '6'
cars_data['cylinders'].value_counts()
cars_data['cylinders']=cars_data['cylinders'].astype(int)
```


#### Drive 

"drive" column has values for forward drive differently spelled. Some have 'fwd' and some have '4wd'. Replaced '4wd' with 'fwd'. For all the missing values, randomly assigned one of the 'fwd' or 'rwd'.


```python
cars_data.loc[cars_data["drive"] =='4wd', "drive"] = 'fwd'
cars_data['drive'].value_counts()

```

    fwd    236320
    rwd     56023
    Name: drive, dtype: int64


```python
cars_data.loc[cars_data["drive"].isnull(), "drive"] = np.random.choice(['fwd','rwd'], cars_data['drive'].isna().sum())
cars_data['drive'].value_counts()
```

    fwd    300358
    rwd    120035
    Name: drive, dtype: int64



#### Missing values in "odometer" column are assigned the mean value of the data in that column.
#### Missing values in "transmission", "fuel" and "title_status" columns are assinged randomly one of the categorical values they have


```python
cars_data.loc[cars_data["odometer"].isnull(), "odometer"] = cars_data['odometer'].mean()
cars_data.loc[cars_data['transmission'].isna(),'transmission']= np.random.choice(['automatic','manual','other'], cars_data['transmission'].isna().sum())
cars_data.loc[cars_data['fuel'].isna(),'fuel']= np.random.choice(['gas','diesel','other','hybrid','electric'], cars_data['fuel'].isna().sum())
cars_data.loc[cars_data['title_status'].isna(),'title_status']= np.random.choice(['clean','rebuilt','salvage','lien','missing','parts only'], cars_data['title_status'].isna().sum())
```

#### Dropped the rows having missing values in 'condition' and 'type' columns


```python
cars_data.dropna(subset=['condition','type'], inplace=True)
```


```python
cars_data.shape
```

    (217393, 11)


#### Transform the Data



Converted all the categorical columns into numeric columns by getting dummies using pd.get_dummies() function and concating the resulting columns with the main dataframe


```python
dummies = pd.get_dummies(cars_data[['title_status', 'transmission','drive', 'type', 'condition', 'fuel']])
dummies.head()
```

```python
cars_data_dummies = pd.concat([cars_data,dummies], axis=1).drop(columns = ['title_status', 'transmission','drive', 'type', 'condition', 'fuel'])
cars_data_dummies.head(10)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>price</th>
      <th>year</th>
      <th>cylinders</th>
      <th>odometer</th>
      <th>VIN</th>
      <th>title_status_clean</th>
      <th>title_status_lien</th>
      <th>title_status_missing</th>
      <th>title_status_parts only</th>
      <th>title_status_rebuilt</th>
      <th>...</th>
      <th>condition_fair</th>
      <th>condition_good</th>
      <th>condition_like new</th>
      <th>condition_new</th>
      <th>condition_salvage</th>
      <th>fuel_diesel</th>
      <th>fuel_electric</th>
      <th>fuel_gas</th>
      <th>fuel_hybrid</th>
      <th>fuel_other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27</th>
      <td>33590</td>
      <td>2014.0</td>
      <td>8</td>
      <td>57923.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>22590</td>
      <td>2010.0</td>
      <td>8</td>
      <td>71229.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>39590</td>
      <td>2020.0</td>
      <td>8</td>
      <td>19160.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30990</td>
      <td>2017.0</td>
      <td>8</td>
      <td>41124.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>15000</td>
      <td>2013.0</td>
      <td>6</td>
      <td>128000.0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>27990</td>
      <td>2012.0</td>
      <td>8</td>
      <td>68696.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>34590</td>
      <td>2016.0</td>
      <td>6</td>
      <td>29499.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>35000</td>
      <td>2019.0</td>
      <td>6</td>
      <td>43000.0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>29990</td>
      <td>2016.0</td>
      <td>6</td>
      <td>17302.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>38590</td>
      <td>2011.0</td>
      <td>8</td>
      <td>30237.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 40 columns</p>
</div>



Run the correlation function and observe correlations between different columns with price. As per the correlation matrix, it's observed that all the features are very sparsely correlated to the price. So, it indicates the price determination is based on multiple features and not few.


```python
cars_data_dummies.corr()
```





<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>price</th>
      <th>year</th>
      <th>cylinders</th>
      <th>odometer</th>
      <th>VIN</th>
      <th>title_status_clean</th>
      <th>title_status_lien</th>
      <th>title_status_missing</th>
      <th>title_status_parts only</th>
      <th>title_status_rebuilt</th>
      <th>...</th>
      <th>condition_fair</th>
      <th>condition_good</th>
      <th>condition_like new</th>
      <th>condition_new</th>
      <th>condition_salvage</th>
      <th>fuel_diesel</th>
      <th>fuel_electric</th>
      <th>fuel_gas</th>
      <th>fuel_hybrid</th>
      <th>fuel_other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>price</th>
      <td>1.000000</td>
      <td>-0.002019</td>
      <td>0.004839</td>
      <td>0.001125</td>
      <td>-0.003452</td>
      <td>0.000835</td>
      <td>-0.000289</td>
      <td>-0.000284</td>
      <td>-0.000247</td>
      <td>-0.000501</td>
      <td>...</td>
      <td>-0.000252</td>
      <td>-0.001227</td>
      <td>-0.000057</td>
      <td>-0.000134</td>
      <td>-0.000209</td>
      <td>0.002756</td>
      <td>-0.000145</td>
      <td>-0.000928</td>
      <td>-0.000401</td>
      <td>-0.000647</td>
    </tr>
    <tr>
      <th>year</th>
      <td>-0.002019</td>
      <td>1.000000</td>
      <td>-0.103845</td>
      <td>-0.223158</td>
      <td>0.437080</td>
      <td>-0.004821</td>
      <td>0.019730</td>
      <td>-0.024127</td>
      <td>0.005019</td>
      <td>0.018169</td>
      <td>...</td>
      <td>-0.235651</td>
      <td>0.165995</td>
      <td>0.013356</td>
      <td>0.028817</td>
      <td>-0.051230</td>
      <td>-0.075635</td>
      <td>0.039765</td>
      <td>-0.146340</td>
      <td>0.031959</td>
      <td>0.216116</td>
    </tr>
    <tr>
      <th>cylinders</th>
      <td>0.004839</td>
      <td>-0.103845</td>
      <td>1.000000</td>
      <td>0.010819</td>
      <td>0.048857</td>
      <td>0.086675</td>
      <td>-0.012397</td>
      <td>-0.018922</td>
      <td>-0.016378</td>
      <td>-0.072535</td>
      <td>...</td>
      <td>0.016530</td>
      <td>0.120995</td>
      <td>-0.045136</td>
      <td>-0.005159</td>
      <td>-0.010384</td>
      <td>0.134106</td>
      <td>-0.007528</td>
      <td>-0.079901</td>
      <td>-0.078785</td>
      <td>0.035610</td>
    </tr>
    <tr>
      <th>odometer</th>
      <td>0.001125</td>
      <td>-0.223158</td>
      <td>0.010819</td>
      <td>1.000000</td>
      <td>-0.168226</td>
      <td>-0.004680</td>
      <td>-0.001403</td>
      <td>0.005096</td>
      <td>0.006703</td>
      <td>-0.005790</td>
      <td>...</td>
      <td>0.095850</td>
      <td>-0.088411</td>
      <td>0.005035</td>
      <td>-0.013550</td>
      <td>0.029229</td>
      <td>0.075053</td>
      <td>-0.021290</td>
      <td>0.039538</td>
      <td>-0.004725</td>
      <td>-0.097128</td>
    </tr>
    <tr>
      <th>VIN</th>
      <td>-0.003452</td>
      <td>0.437080</td>
      <td>0.048857</td>
      <td>-0.168226</td>
      <td>1.000000</td>
      <td>0.064828</td>
      <td>-0.020518</td>
      <td>0.023369</td>
      <td>0.039554</td>
      <td>-0.079643</td>
      <td>...</td>
      <td>-0.165083</td>
      <td>0.261125</td>
      <td>-0.195139</td>
      <td>-0.034644</td>
      <td>-0.048576</td>
      <td>-0.048501</td>
      <td>0.029541</td>
      <td>-0.165770</td>
      <td>0.013712</td>
      <td>0.230390</td>
    </tr>
    <tr>
      <th>title_status_clean</th>
      <td>0.000835</td>
      <td>-0.004821</td>
      <td>0.086675</td>
      <td>-0.004680</td>
      <td>0.064828</td>
      <td>1.000000</td>
      <td>-0.396068</td>
      <td>-0.303195</td>
      <td>-0.281177</td>
      <td>-0.639552</td>
      <td>...</td>
      <td>-0.017931</td>
      <td>0.153673</td>
      <td>-0.010335</td>
      <td>-0.003677</td>
      <td>-0.104522</td>
      <td>-0.002669</td>
      <td>0.009787</td>
      <td>-0.062614</td>
      <td>-0.000750</td>
      <td>0.078323</td>
    </tr>
    <tr>
      <th>title_status_lien</th>
      <td>-0.000289</td>
      <td>0.019730</td>
      <td>-0.012397</td>
      <td>-0.001403</td>
      <td>-0.020518</td>
      <td>-0.396068</td>
      <td>1.000000</td>
      <td>-0.007069</td>
      <td>-0.006555</td>
      <td>-0.014910</td>
      <td>...</td>
      <td>-0.003240</td>
      <td>-0.065683</td>
      <td>0.013405</td>
      <td>0.010241</td>
      <td>-0.000873</td>
      <td>0.014709</td>
      <td>0.005526</td>
      <td>0.015517</td>
      <td>-0.000376</td>
      <td>-0.031285</td>
    </tr>
    <tr>
      <th>title_status_missing</th>
      <td>-0.000284</td>
      <td>-0.024127</td>
      <td>-0.018922</td>
      <td>0.005096</td>
      <td>0.023369</td>
      <td>-0.303195</td>
      <td>-0.007069</td>
      <td>1.000000</td>
      <td>-0.005018</td>
      <td>-0.011414</td>
      <td>...</td>
      <td>0.025608</td>
      <td>-0.058973</td>
      <td>-0.020086</td>
      <td>-0.004445</td>
      <td>0.028513</td>
      <td>-0.000655</td>
      <td>-0.004320</td>
      <td>0.018903</td>
      <td>0.002982</td>
      <td>-0.023351</td>
    </tr>
    <tr>
      <th>title_status_parts only</th>
      <td>-0.000247</td>
      <td>0.005019</td>
      <td>-0.016378</td>
      <td>0.006703</td>
      <td>0.039554</td>
      <td>-0.281177</td>
      <td>-0.006555</td>
      <td>-0.005018</td>
      <td>1.000000</td>
      <td>-0.010585</td>
      <td>...</td>
      <td>-0.003540</td>
      <td>-0.063442</td>
      <td>-0.018519</td>
      <td>-0.001872</td>
      <td>0.029651</td>
      <td>-0.000400</td>
      <td>-0.004834</td>
      <td>0.018364</td>
      <td>-0.000076</td>
      <td>-0.021528</td>
    </tr>
    <tr>
      <th>title_status_rebuilt</th>
      <td>-0.000501</td>
      <td>0.018169</td>
      <td>-0.072535</td>
      <td>-0.005790</td>
      <td>-0.079643</td>
      <td>-0.639552</td>
      <td>-0.014910</td>
      <td>-0.011414</td>
      <td>-0.010585</td>
      <td>1.000000</td>
      <td>...</td>
      <td>-0.002768</td>
      <td>-0.090880</td>
      <td>0.027753</td>
      <td>0.001716</td>
      <td>0.004364</td>
      <td>-0.001700</td>
      <td>-0.008841</td>
      <td>0.043320</td>
      <td>-0.001479</td>
      <td>-0.050328</td>
    </tr>
    <tr>
      <th>title_status_salvage</th>
      <td>-0.000447</td>
      <td>-0.018619</td>
      <td>-0.045987</td>
      <td>0.011117</td>
      <td>-0.046306</td>
      <td>-0.471497</td>
      <td>-0.010992</td>
      <td>-0.008415</td>
      <td>-0.007804</td>
      <td>-0.017750</td>
      <td>...</td>
      <td>0.028368</td>
      <td>-0.058137</td>
      <td>-0.003422</td>
      <td>0.000530</td>
      <td>0.170673</td>
      <td>-0.004040</td>
      <td>-0.006950</td>
      <td>0.032552</td>
      <td>0.001936</td>
      <td>-0.036945</td>
    </tr>
    <tr>
      <th>transmission_automatic</th>
      <td>0.001447</td>
      <td>-0.265807</td>
      <td>-0.067665</td>
      <td>0.161036</td>
      <td>-0.335654</td>
      <td>-0.112285</td>
      <td>0.042854</td>
      <td>0.026578</td>
      <td>0.030773</td>
      <td>0.081075</td>
      <td>...</td>
      <td>0.048505</td>
      <td>-0.502520</td>
      <td>0.154153</td>
      <td>0.024508</td>
      <td>0.015553</td>
      <td>0.077990</td>
      <td>-0.056837</td>
      <td>0.238780</td>
      <td>-0.003877</td>
      <td>-0.340664</td>
    </tr>
    <tr>
      <th>transmission_manual</th>
      <td>-0.000866</td>
      <td>-0.263343</td>
      <td>-0.100968</td>
      <td>0.052374</td>
      <td>-0.144416</td>
      <td>-0.014755</td>
      <td>0.006097</td>
      <td>0.007415</td>
      <td>-0.004870</td>
      <td>0.003083</td>
      <td>...</td>
      <td>0.074656</td>
      <td>-0.060046</td>
      <td>0.012146</td>
      <td>0.017549</td>
      <td>0.017088</td>
      <td>0.042711</td>
      <td>-0.015251</td>
      <td>0.037555</td>
      <td>-0.007660</td>
      <td>-0.071411</td>
    </tr>
    <tr>
      <th>transmission_other</th>
      <td>-0.001088</td>
      <td>0.412918</td>
      <td>0.122227</td>
      <td>-0.196108</td>
      <td>0.426536</td>
      <td>0.125763</td>
      <td>-0.048233</td>
      <td>-0.031746</td>
      <td>-0.029970</td>
      <td>-0.086988</td>
      <td>...</td>
      <td>-0.088766</td>
      <td>0.559819</td>
      <td>-0.168565</td>
      <td>-0.034677</td>
      <td>-0.025008</td>
      <td>-0.103725</td>
      <td>0.067585</td>
      <td>-0.270557</td>
      <td>0.007949</td>
      <td>0.394994</td>
    </tr>
    <tr>
      <th>drive_fwd</th>
      <td>0.000349</td>
      <td>0.077971</td>
      <td>-0.207515</td>
      <td>0.027856</td>
      <td>-0.053814</td>
      <td>-0.025607</td>
      <td>0.006667</td>
      <td>-0.006952</td>
      <td>-0.003048</td>
      <td>0.038556</td>
      <td>...</td>
      <td>-0.000802</td>
      <td>-0.121844</td>
      <td>0.032210</td>
      <td>0.007144</td>
      <td>0.001138</td>
      <td>-0.017891</td>
      <td>-0.029612</td>
      <td>0.060360</td>
      <td>0.041734</td>
      <td>-0.072092</td>
    </tr>
    <tr>
      <th>drive_rwd</th>
      <td>-0.000349</td>
      <td>-0.077971</td>
      <td>0.207515</td>
      <td>-0.027856</td>
      <td>0.053814</td>
      <td>0.025607</td>
      <td>-0.006667</td>
      <td>0.006952</td>
      <td>0.003048</td>
      <td>-0.038556</td>
      <td>...</td>
      <td>0.000802</td>
      <td>0.121844</td>
      <td>-0.032210</td>
      <td>-0.007144</td>
      <td>-0.001138</td>
      <td>0.017891</td>
      <td>0.029612</td>
      <td>-0.060360</td>
      <td>-0.041734</td>
      <td>0.072092</td>
    </tr>
    <tr>
      <th>type_SUV</th>
      <td>-0.001744</td>
      <td>-0.005255</td>
      <td>-0.062797</td>
      <td>0.037542</td>
      <td>-0.066917</td>
      <td>-0.030613</td>
      <td>0.018767</td>
      <td>0.014745</td>
      <td>0.019659</td>
      <td>0.013142</td>
      <td>...</td>
      <td>-0.006171</td>
      <td>-0.156411</td>
      <td>0.030377</td>
      <td>0.006722</td>
      <td>0.001694</td>
      <td>-0.096090</td>
      <td>-0.026418</td>
      <td>0.129659</td>
      <td>-0.032214</td>
      <td>-0.073499</td>
    </tr>
    <tr>
      <th>type_bus</th>
      <td>-0.000146</td>
      <td>-0.028665</td>
      <td>0.027232</td>
      <td>0.005403</td>
      <td>-0.030587</td>
      <td>0.007881</td>
      <td>-0.002919</td>
      <td>-0.000131</td>
      <td>-0.002885</td>
      <td>-0.005123</td>
      <td>...</td>
      <td>0.021428</td>
      <td>-0.008276</td>
      <td>-0.005302</td>
      <td>0.051744</td>
      <td>-0.001847</td>
      <td>0.064167</td>
      <td>-0.001450</td>
      <td>-0.032467</td>
      <td>-0.005045</td>
      <td>-0.003630</td>
    </tr>
    <tr>
      <th>type_convertible</th>
      <td>-0.000492</td>
      <td>-0.164692</td>
      <td>0.024079</td>
      <td>-0.012809</td>
      <td>-0.048369</td>
      <td>0.005208</td>
      <td>-0.004883</td>
      <td>0.003103</td>
      <td>-0.004860</td>
      <td>-0.004847</td>
      <td>...</td>
      <td>0.003756</td>
      <td>-0.030189</td>
      <td>0.020001</td>
      <td>0.001564</td>
      <td>-0.000246</td>
      <td>-0.034707</td>
      <td>-0.005507</td>
      <td>0.048099</td>
      <td>-0.016114</td>
      <td>-0.027318</td>
    </tr>
    <tr>
      <th>type_coupe</th>
      <td>-0.000696</td>
      <td>-0.067582</td>
      <td>0.059227</td>
      <td>-0.033775</td>
      <td>0.004886</td>
      <td>0.010331</td>
      <td>-0.007392</td>
      <td>-0.005077</td>
      <td>-0.008276</td>
      <td>-0.006057</td>
      <td>...</td>
      <td>0.003047</td>
      <td>0.046289</td>
      <td>-0.001497</td>
      <td>0.003366</td>
      <td>0.001168</td>
      <td>-0.057978</td>
      <td>-0.016442</td>
      <td>0.038431</td>
      <td>-0.025681</td>
      <td>0.008090</td>
    </tr>
    <tr>
      <th>type_hatchback</th>
      <td>-0.000870</td>
      <td>0.076821</td>
      <td>-0.137877</td>
      <td>-0.033972</td>
      <td>0.049418</td>
      <td>0.011874</td>
      <td>-0.006623</td>
      <td>-0.008448</td>
      <td>-0.007392</td>
      <td>-0.003418</td>
      <td>...</td>
      <td>-0.012846</td>
      <td>0.085990</td>
      <td>-0.023332</td>
      <td>-0.005685</td>
      <td>0.002800</td>
      <td>-0.045492</td>
      <td>0.084190</td>
      <td>-0.098542</td>
      <td>0.136897</td>
      <td>0.082217</td>
    </tr>
    <tr>
      <th>type_mini-van</th>
      <td>-0.000559</td>
      <td>-0.039315</td>
      <td>-0.010687</td>
      <td>0.029181</td>
      <td>-0.074710</td>
      <td>-0.001384</td>
      <td>-0.004839</td>
      <td>-0.008921</td>
      <td>-0.008644</td>
      <td>0.014167</td>
      <td>...</td>
      <td>0.019244</td>
      <td>-0.052432</td>
      <td>0.015980</td>
      <td>0.009269</td>
      <td>-0.001063</td>
      <td>-0.027699</td>
      <td>-0.009512</td>
      <td>0.056331</td>
      <td>-0.014533</td>
      <td>-0.042396</td>
    </tr>
    <tr>
      <th>type_offroad</th>
      <td>-0.000169</td>
      <td>-0.072590</td>
      <td>0.005974</td>
      <td>0.022832</td>
      <td>-0.049833</td>
      <td>-0.004023</td>
      <td>0.002620</td>
      <td>0.003595</td>
      <td>-0.000854</td>
      <td>-0.001438</td>
      <td>...</td>
      <td>0.018797</td>
      <td>-0.015975</td>
      <td>0.010537</td>
      <td>0.001377</td>
      <td>0.010127</td>
      <td>0.000859</td>
      <td>-0.003636</td>
      <td>0.014884</td>
      <td>-0.006123</td>
      <td>-0.015973</td>
    </tr>
    <tr>
      <th>type_other</th>
      <td>-0.000495</td>
      <td>0.140917</td>
      <td>0.022772</td>
      <td>-0.064745</td>
      <td>0.159043</td>
      <td>0.032704</td>
      <td>-0.011562</td>
      <td>0.000423</td>
      <td>-0.002060</td>
      <td>-0.031136</td>
      <td>...</td>
      <td>-0.029624</td>
      <td>0.209956</td>
      <td>-0.064921</td>
      <td>-0.013309</td>
      <td>-0.006949</td>
      <td>-0.033819</td>
      <td>-0.012022</td>
      <td>-0.093268</td>
      <td>0.018943</td>
      <td>0.136725</td>
    </tr>
    <tr>
      <th>type_pickup</th>
      <td>0.007718</td>
      <td>0.047534</td>
      <td>0.231056</td>
      <td>-0.026752</td>
      <td>0.086644</td>
      <td>0.056338</td>
      <td>-0.018202</td>
      <td>-0.022291</td>
      <td>-0.023779</td>
      <td>-0.033218</td>
      <td>...</td>
      <td>0.000846</td>
      <td>0.113278</td>
      <td>-0.056335</td>
      <td>-0.006694</td>
      <td>-0.004234</td>
      <td>0.069635</td>
      <td>-0.026028</td>
      <td>-0.095897</td>
      <td>-0.043757</td>
      <td>0.093007</td>
    </tr>
    <tr>
      <th>type_sedan</th>
      <td>-0.001858</td>
      <td>0.036935</td>
      <td>-0.274973</td>
      <td>-0.008098</td>
      <td>-0.034663</td>
      <td>-0.012757</td>
      <td>-0.013778</td>
      <td>-0.009806</td>
      <td>-0.006743</td>
      <td>0.027645</td>
      <td>...</td>
      <td>-0.011051</td>
      <td>-0.036600</td>
      <td>0.026066</td>
      <td>-0.005474</td>
      <td>-0.000692</td>
      <td>-0.104877</td>
      <td>0.040023</td>
      <td>0.063916</td>
      <td>0.011469</td>
      <td>-0.017875</td>
    </tr>
    <tr>
      <th>type_truck</th>
      <td>-0.000507</td>
      <td>-0.109431</td>
      <td>0.296262</td>
      <td>0.084094</td>
      <td>-0.083346</td>
      <td>-0.033461</td>
      <td>0.032052</td>
      <td>0.017318</td>
      <td>0.018665</td>
      <td>0.007001</td>
      <td>...</td>
      <td>0.035124</td>
      <td>-0.091570</td>
      <td>0.042968</td>
      <td>0.009047</td>
      <td>0.005071</td>
      <td>0.359964</td>
      <td>-0.022955</td>
      <td>-0.109007</td>
      <td>-0.035186</td>
      <td>-0.105692</td>
    </tr>
    <tr>
      <th>type_van</th>
      <td>-0.000492</td>
      <td>-0.001321</td>
      <td>0.040496</td>
      <td>0.008566</td>
      <td>0.005922</td>
      <td>-0.033860</td>
      <td>0.021568</td>
      <td>0.021484</td>
      <td>0.022250</td>
      <td>0.006097</td>
      <td>...</td>
      <td>0.012801</td>
      <td>-0.016460</td>
      <td>0.007308</td>
      <td>-0.001591</td>
      <td>0.001336</td>
      <td>-0.005577</td>
      <td>-0.010753</td>
      <td>0.029031</td>
      <td>-0.016787</td>
      <td>-0.023096</td>
    </tr>
    <tr>
      <th>type_wagon</th>
      <td>-0.000622</td>
      <td>0.011722</td>
      <td>-0.081084</td>
      <td>0.000397</td>
      <td>0.040666</td>
      <td>0.003168</td>
      <td>-0.004472</td>
      <td>0.005316</td>
      <td>0.001165</td>
      <td>-0.003041</td>
      <td>...</td>
      <td>-0.003346</td>
      <td>0.007740</td>
      <td>-0.016837</td>
      <td>-0.002677</td>
      <td>-0.001241</td>
      <td>-0.019161</td>
      <td>-0.004440</td>
      <td>0.006426</td>
      <td>0.055022</td>
      <td>-0.014872</td>
    </tr>
    <tr>
      <th>condition_excellent</th>
      <td>0.001400</td>
      <td>-0.104467</td>
      <td>-0.103251</td>
      <td>0.057761</td>
      <td>-0.103937</td>
      <td>-0.136695</td>
      <td>0.059925</td>
      <td>0.061424</td>
      <td>0.073627</td>
      <td>0.078486</td>
      <td>...</td>
      <td>-0.121645</td>
      <td>-0.812133</td>
      <td>-0.226220</td>
      <td>-0.048333</td>
      <td>-0.034936</td>
      <td>0.035111</td>
      <td>-0.032595</td>
      <td>0.190688</td>
      <td>0.003103</td>
      <td>-0.257801</td>
    </tr>
    <tr>
      <th>condition_fair</th>
      <td>-0.000252</td>
      <td>-0.235651</td>
      <td>0.016530</td>
      <td>0.095850</td>
      <td>-0.165083</td>
      <td>-0.017931</td>
      <td>-0.003240</td>
      <td>0.025608</td>
      <td>-0.003540</td>
      <td>-0.002768</td>
      <td>...</td>
      <td>1.000000</td>
      <td>-0.154434</td>
      <td>-0.043018</td>
      <td>-0.009191</td>
      <td>-0.006643</td>
      <td>0.018578</td>
      <td>-0.009899</td>
      <td>0.032712</td>
      <td>-0.012358</td>
      <td>-0.047217</td>
    </tr>
    <tr>
      <th>condition_good</th>
      <td>-0.001227</td>
      <td>0.165995</td>
      <td>0.120995</td>
      <td>-0.088411</td>
      <td>0.261125</td>
      <td>0.153673</td>
      <td>-0.065683</td>
      <td>-0.058973</td>
      <td>-0.063442</td>
      <td>-0.090880</td>
      <td>...</td>
      <td>-0.154434</td>
      <td>1.000000</td>
      <td>-0.287198</td>
      <td>-0.061362</td>
      <td>-0.044354</td>
      <td>-0.044989</td>
      <td>0.033240</td>
      <td>-0.232739</td>
      <td>0.000092</td>
      <td>0.316221</td>
    </tr>
    <tr>
      <th>condition_like new</th>
      <td>-0.000057</td>
      <td>0.013356</td>
      <td>-0.045136</td>
      <td>0.005035</td>
      <td>-0.195139</td>
      <td>-0.010335</td>
      <td>0.013405</td>
      <td>-0.020086</td>
      <td>-0.018519</td>
      <td>0.027753</td>
      <td>...</td>
      <td>-0.043018</td>
      <td>-0.287198</td>
      <td>1.000000</td>
      <td>-0.017092</td>
      <td>-0.012355</td>
      <td>0.009859</td>
      <td>0.002630</td>
      <td>0.065480</td>
      <td>0.001046</td>
      <td>-0.090203</td>
    </tr>
    <tr>
      <th>condition_new</th>
      <td>-0.000134</td>
      <td>0.028817</td>
      <td>-0.005159</td>
      <td>-0.013550</td>
      <td>-0.034644</td>
      <td>-0.003677</td>
      <td>0.010241</td>
      <td>-0.004445</td>
      <td>-0.001872</td>
      <td>0.001716</td>
      <td>...</td>
      <td>-0.009191</td>
      <td>-0.061362</td>
      <td>-0.017092</td>
      <td>1.000000</td>
      <td>-0.002640</td>
      <td>0.001167</td>
      <td>0.002220</td>
      <td>0.013358</td>
      <td>-0.000066</td>
      <td>-0.018078</td>
    </tr>
    <tr>
      <th>condition_salvage</th>
      <td>-0.000209</td>
      <td>-0.051230</td>
      <td>-0.010384</td>
      <td>0.029229</td>
      <td>-0.048576</td>
      <td>-0.104522</td>
      <td>-0.000873</td>
      <td>0.028513</td>
      <td>0.029651</td>
      <td>0.004364</td>
      <td>...</td>
      <td>-0.006643</td>
      <td>-0.044354</td>
      <td>-0.012355</td>
      <td>-0.002640</td>
      <td>1.000000</td>
      <td>-0.001104</td>
      <td>-0.001599</td>
      <td>0.012080</td>
      <td>0.000170</td>
      <td>-0.014014</td>
    </tr>
    <tr>
      <th>fuel_diesel</th>
      <td>0.002756</td>
      <td>-0.075635</td>
      <td>0.134106</td>
      <td>0.075053</td>
      <td>-0.048501</td>
      <td>-0.002669</td>
      <td>0.014709</td>
      <td>-0.000655</td>
      <td>-0.000400</td>
      <td>-0.001700</td>
      <td>...</td>
      <td>0.018578</td>
      <td>-0.044989</td>
      <td>0.009859</td>
      <td>0.001167</td>
      <td>-0.001104</td>
      <td>1.000000</td>
      <td>-0.016072</td>
      <td>-0.509126</td>
      <td>-0.027061</td>
      <td>-0.074615</td>
    </tr>
    <tr>
      <th>fuel_electric</th>
      <td>-0.000145</td>
      <td>0.039765</td>
      <td>-0.007528</td>
      <td>-0.021290</td>
      <td>0.029541</td>
      <td>0.009787</td>
      <td>0.005526</td>
      <td>-0.004320</td>
      <td>-0.004834</td>
      <td>-0.008841</td>
      <td>...</td>
      <td>-0.009899</td>
      <td>0.033240</td>
      <td>0.002630</td>
      <td>0.002220</td>
      <td>-0.001599</td>
      <td>-0.016072</td>
      <td>1.000000</td>
      <td>-0.159047</td>
      <td>-0.008454</td>
      <td>-0.023309</td>
    </tr>
    <tr>
      <th>fuel_gas</th>
      <td>-0.000928</td>
      <td>-0.146340</td>
      <td>-0.079901</td>
      <td>0.039538</td>
      <td>-0.165770</td>
      <td>-0.062614</td>
      <td>0.015517</td>
      <td>0.018903</td>
      <td>0.018364</td>
      <td>0.043320</td>
      <td>...</td>
      <td>0.032712</td>
      <td>-0.232739</td>
      <td>0.065480</td>
      <td>0.013358</td>
      <td>0.012080</td>
      <td>-0.509126</td>
      <td>-0.159047</td>
      <td>1.000000</td>
      <td>-0.267801</td>
      <td>-0.738394</td>
    </tr>
    <tr>
      <th>fuel_hybrid</th>
      <td>-0.000401</td>
      <td>0.031959</td>
      <td>-0.078785</td>
      <td>-0.004725</td>
      <td>0.013712</td>
      <td>-0.000750</td>
      <td>-0.000376</td>
      <td>0.002982</td>
      <td>-0.000076</td>
      <td>-0.001479</td>
      <td>...</td>
      <td>-0.012358</td>
      <td>0.000092</td>
      <td>0.001046</td>
      <td>-0.000066</td>
      <td>0.000170</td>
      <td>-0.027061</td>
      <td>-0.008454</td>
      <td>-0.267801</td>
      <td>1.000000</td>
      <td>-0.039247</td>
    </tr>
    <tr>
      <th>fuel_other</th>
      <td>-0.000647</td>
      <td>0.216116</td>
      <td>0.035610</td>
      <td>-0.097128</td>
      <td>0.230390</td>
      <td>0.078323</td>
      <td>-0.031285</td>
      <td>-0.023351</td>
      <td>-0.021528</td>
      <td>-0.050328</td>
      <td>...</td>
      <td>-0.047217</td>
      <td>0.316221</td>
      <td>-0.090203</td>
      <td>-0.018078</td>
      <td>-0.014014</td>
      <td>-0.074615</td>
      <td>-0.023309</td>
      <td>-0.738394</td>
      <td>-0.039247</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
<p>40 rows × 40 columns</p>
</div>



#### Created a pipeline with FeatureUnion function to create PolynomialFeatures for numeric columns and concatenate them with the dummies columns data.



<style>#sk-container-id-2 {color: black;}#sk-container-id-2 pre{padding: 0;}#sk-container-id-2 div.sk-toggleable {background-color: white;}#sk-container-id-2 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-2 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-2 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-2 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-2 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-2 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-2 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-2 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-2 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-2 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-2 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-2 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-2 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-2 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-2 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-2 div.sk-item {position: relative;z-index: 1;}#sk-container-id-2 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-2 div.sk-item::before, #sk-container-id-2 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-2 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-2 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-2 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-2 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-2 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-2 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-2 div.sk-label-container {text-align: center;}#sk-container-id-2 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-2 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-2" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>Pipeline(steps=[(&#x27;features&#x27;,
                 FeatureUnion(transformer_list=[(&#x27;num&#x27;,
                                                 Pipeline(steps=[(&#x27;extract&#x27;,
                                                                  ColumnExtractor(columns=[&#x27;year&#x27;,
                                                                                           &#x27;cylinders&#x27;,
                                                                                           &#x27;odometer&#x27;,
                                                                                           &#x27;VIN&#x27;])),
                                                                 (&#x27;poly&#x27;,
                                                                  PolynomialFeatures())])),
                                                (&#x27;cat_var&#x27;,
                                                 ColumnExtractor(columns=[&#x27;title_status_clean&#x27;,
                                                                          &#x27;title_status_lien&#x27;,
                                                                          &#x27;title_status_missing&#x27;,
                                                                          &#x27;title_status_parts &#x27;
                                                                          &#x27;only&#x27;,
                                                                          &#x27;title_status_rebuilt&#x27;,
                                                                          &#x27;title_sta...
                                                                          &#x27;transmission_manual&#x27;,
                                                                          &#x27;transmission_other&#x27;,
                                                                          &#x27;drive_fwd&#x27;,
                                                                          &#x27;drive_rwd&#x27;,
                                                                          &#x27;type_SUV&#x27;,
                                                                          &#x27;type_bus&#x27;,
                                                                          &#x27;type_convertible&#x27;,
                                                                          &#x27;type_coupe&#x27;,
                                                                          &#x27;type_hatchback&#x27;,
                                                                          &#x27;type_mini-van&#x27;,
                                                                          &#x27;type_offroad&#x27;,
                                                                          &#x27;type_other&#x27;,
                                                                          &#x27;type_pickup&#x27;,
                                                                          &#x27;type_sedan&#x27;,
                                                                          &#x27;type_truck&#x27;,
                                                                          &#x27;type_van&#x27;,
                                                                          &#x27;type_wagon&#x27;,
                                                                          &#x27;condition_excellent&#x27;,
                                                                          &#x27;condition_fair&#x27;,
                                                                          &#x27;condition_good&#x27;,
                                                                          &#x27;condition_like &#x27;
                                                                          &#x27;new&#x27;,
                                                                          &#x27;condition_new&#x27;,
                                                                          &#x27;condition_salvage&#x27;, ...]))]))])</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-6" type="checkbox" ><label for="sk-estimator-id-6" class="sk-toggleable__label sk-toggleable__label-arrow">Pipeline</label><div class="sk-toggleable__content"><pre>Pipeline(steps=[(&#x27;features&#x27;,
                 FeatureUnion(transformer_list=[(&#x27;num&#x27;,
                                                 Pipeline(steps=[(&#x27;extract&#x27;,
                                                                  ColumnExtractor(columns=[&#x27;year&#x27;,
                                                                                           &#x27;cylinders&#x27;,
                                                                                           &#x27;odometer&#x27;,
                                                                                           &#x27;VIN&#x27;])),
                                                                 (&#x27;poly&#x27;,
                                                                  PolynomialFeatures())])),
                                                (&#x27;cat_var&#x27;,
                                                 ColumnExtractor(columns=[&#x27;title_status_clean&#x27;,
                                                                          &#x27;title_status_lien&#x27;,
                                                                          &#x27;title_status_missing&#x27;,
                                                                          &#x27;title_status_parts &#x27;
                                                                          &#x27;only&#x27;,
                                                                          &#x27;title_status_rebuilt&#x27;,
                                                                          &#x27;title_sta...
                                                                          &#x27;transmission_manual&#x27;,
                                                                          &#x27;transmission_other&#x27;,
                                                                          &#x27;drive_fwd&#x27;,
                                                                          &#x27;drive_rwd&#x27;,
                                                                          &#x27;type_SUV&#x27;,
                                                                          &#x27;type_bus&#x27;,
                                                                          &#x27;type_convertible&#x27;,
                                                                          &#x27;type_coupe&#x27;,
                                                                          &#x27;type_hatchback&#x27;,
                                                                          &#x27;type_mini-van&#x27;,
                                                                          &#x27;type_offroad&#x27;,
                                                                          &#x27;type_other&#x27;,
                                                                          &#x27;type_pickup&#x27;,
                                                                          &#x27;type_sedan&#x27;,
                                                                          &#x27;type_truck&#x27;,
                                                                          &#x27;type_van&#x27;,
                                                                          &#x27;type_wagon&#x27;,
                                                                          &#x27;condition_excellent&#x27;,
                                                                          &#x27;condition_fair&#x27;,
                                                                          &#x27;condition_good&#x27;,
                                                                          &#x27;condition_like &#x27;
                                                                          &#x27;new&#x27;,
                                                                          &#x27;condition_new&#x27;,
                                                                          &#x27;condition_salvage&#x27;, ...]))]))])</pre></div></div></div><div class="sk-serial"><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-7" type="checkbox" ><label for="sk-estimator-id-7" class="sk-toggleable__label sk-toggleable__label-arrow">features: FeatureUnion</label><div class="sk-toggleable__content"><pre>FeatureUnion(transformer_list=[(&#x27;num&#x27;,
                                Pipeline(steps=[(&#x27;extract&#x27;,
                                                 ColumnExtractor(columns=[&#x27;year&#x27;,
                                                                          &#x27;cylinders&#x27;,
                                                                          &#x27;odometer&#x27;,
                                                                          &#x27;VIN&#x27;])),
                                                (&#x27;poly&#x27;,
                                                 PolynomialFeatures())])),
                               (&#x27;cat_var&#x27;,
                                ColumnExtractor(columns=[&#x27;title_status_clean&#x27;,
                                                         &#x27;title_status_lien&#x27;,
                                                         &#x27;title_status_missing&#x27;,
                                                         &#x27;title_status_parts &#x27;
                                                         &#x27;only&#x27;,
                                                         &#x27;title_status_rebuilt&#x27;,
                                                         &#x27;title_status_salvage&#x27;,
                                                         &#x27;transmission_a...
                                                         &#x27;transmission_manual&#x27;,
                                                         &#x27;transmission_other&#x27;,
                                                         &#x27;drive_fwd&#x27;,
                                                         &#x27;drive_rwd&#x27;,
                                                         &#x27;type_SUV&#x27;, &#x27;type_bus&#x27;,
                                                         &#x27;type_convertible&#x27;,
                                                         &#x27;type_coupe&#x27;,
                                                         &#x27;type_hatchback&#x27;,
                                                         &#x27;type_mini-van&#x27;,
                                                         &#x27;type_offroad&#x27;,
                                                         &#x27;type_other&#x27;,
                                                         &#x27;type_pickup&#x27;,
                                                         &#x27;type_sedan&#x27;,
                                                         &#x27;type_truck&#x27;,
                                                         &#x27;type_van&#x27;,
                                                         &#x27;type_wagon&#x27;,
                                                         &#x27;condition_excellent&#x27;,
                                                         &#x27;condition_fair&#x27;,
                                                         &#x27;condition_good&#x27;,
                                                         &#x27;condition_like new&#x27;,
                                                         &#x27;condition_new&#x27;,
                                                         &#x27;condition_salvage&#x27;, ...]))])</pre></div></div></div><div class="sk-parallel"><div class="sk-parallel-item"><div class="sk-item"><div class="sk-label-container"><div class="sk-label sk-toggleable"><label>num</label></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-8" type="checkbox" ><label for="sk-estimator-id-8" class="sk-toggleable__label sk-toggleable__label-arrow">ColumnExtractor</label><div class="sk-toggleable__content"><pre>ColumnExtractor(columns=[&#x27;year&#x27;, &#x27;cylinders&#x27;, &#x27;odometer&#x27;, &#x27;VIN&#x27;])</pre></div></div></div><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-9" type="checkbox" ><label for="sk-estimator-id-9" class="sk-toggleable__label sk-toggleable__label-arrow">PolynomialFeatures</label><div class="sk-toggleable__content"><pre>PolynomialFeatures()</pre></div></div></div></div></div></div></div></div><div class="sk-parallel-item"><div class="sk-item"><div class="sk-label-container"><div class="sk-label sk-toggleable"><label>cat_var</label></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-10" type="checkbox" ><label for="sk-estimator-id-10" class="sk-toggleable__label sk-toggleable__label-arrow">ColumnExtractor</label><div class="sk-toggleable__content"><pre>ColumnExtractor(columns=[&#x27;title_status_clean&#x27;, &#x27;title_status_lien&#x27;,
                         &#x27;title_status_missing&#x27;, &#x27;title_status_parts only&#x27;,
                         &#x27;title_status_rebuilt&#x27;, &#x27;title_status_salvage&#x27;,
                         &#x27;transmission_automatic&#x27;, &#x27;transmission_manual&#x27;,
                         &#x27;transmission_other&#x27;, &#x27;drive_fwd&#x27;, &#x27;drive_rwd&#x27;,
                         &#x27;type_SUV&#x27;, &#x27;type_bus&#x27;, &#x27;type_convertible&#x27;,
                         &#x27;type_coupe&#x27;, &#x27;type_hatchback&#x27;, &#x27;type_mini-van&#x27;,
                         &#x27;type_offroad&#x27;, &#x27;type_other&#x27;, &#x27;type_pickup&#x27;,
                         &#x27;type_sedan&#x27;, &#x27;type_truck&#x27;, &#x27;type_van&#x27;, &#x27;type_wagon&#x27;,
                         &#x27;condition_excellent&#x27;, &#x27;condition_fair&#x27;,
                         &#x27;condition_good&#x27;, &#x27;condition_like new&#x27;,
                         &#x27;condition_new&#x27;, &#x27;condition_salvage&#x27;, ...])</pre></div></div></div></div></div></div></div></div></div></div></div></div>



### MODELING

Now the pipeline is ready to create the final dataset, decided to apply below unsupervised learning models on the data to derive needed inferences.

Unsupervised Learning
- Linear Regression 
        - All the features
        - Sequential Feature Selection
 - Ridge Reularization (Grid search for hyper parameter)
        

Before applying a model, split the dataset into train and development(test) datasets.

Calculated mean_squared_errors for each hyperparameter and compared to determine the best model for the dataset

Plotted the coefficients against the hyperparameter values and observer which features have high/low weightage and also positive/negative weightage in determining the price of the used car.


```python
# Linear Regression on dataset with Polynomial Features for degrees 1,2,3,4

y = cars_data['price']

mses_train=[]
mses_test = []
coef = []
degrees = [1,2,3,4]
for p in range(1, 5):
    pipe2nvars.set_params(features__num__poly__degree=p)
    res = pipe2nvars.fit_transform(pd.get_dummies(cars_data))
    print('polynomial degree: {}; shape: {}'.format(p, res.shape))

    X = pd.DataFrame(res)#, columns = pipe2nvars.named_steps['poly'].get_feature_names_out())
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state=42)
    
    linreg = LinearRegression(fit_intercept = False)
    linreg.fit(X_train,y_train)
    mse_train = mean_squared_error(linreg.predict(X_train), y_train)
    mse_test = mean_squared_error(linreg.predict(X_test), y_test)
    mses_train.append(mse_train)
    mses_test.append(mse_test)
    coef.append(linreg.coef_)
    

print("Train MSEs: ", mses_train)
print("Test MSEs: ", mses_test)




```

    polynomial degree: 1; shape: (217393, 40)
    polynomial degree: 2; shape: (217393, 50)
    polynomial degree: 3; shape: (217393, 70)
    polynomial degree: 4; shape: (217393, 105)
    Train MSEs:  [104776617480808.67, 104986703962074.77, 104779748702913.72, 104783228983206.36]
    Test MSEs:  [238784716554.31058, 474318258526.46466, 239522331280.9014, 232420927948.23767]
    


```python
error_df = pd.DataFrame(list(zip(degrees, mses_train, mses_test)), columns=['Degree','Train_MSE','Test_MSE'])
error_df
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
      <th>Degree</th>
      <th>Train_MSE</th>
      <th>Test_MSE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1.047766e+14</td>
      <td>2.387847e+11</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>1.049867e+14</td>
      <td>4.743183e+11</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>1.047797e+14</td>
      <td>2.395223e+11</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>1.047832e+14</td>
      <td>2.324209e+11</td>
    </tr>
  </tbody>
</table>
</div>




```python
best_degree = error_df.sort_values("Test_MSE").iloc[0,:]
best_degree
```




    Degree       4.000000e+00
    Train_MSE    1.047832e+14
    Test_MSE     2.324209e+11
    Name: 3, dtype: float64



From the above observation, when the model is applied on PolynomialFeatures for degree 4, the mean squared error on the development data set is the lowest. But the processing time is quite high because of 105 features under consideration.

So, now set to explore the dataset by selecting 5 or 7 features using SequientialSelector() function and applying Linear Regression on them.


```python
cars_data_dummies.head()
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
      <th>price</th>
      <th>year</th>
      <th>cylinders</th>
      <th>odometer</th>
      <th>VIN</th>
      <th>title_status_clean</th>
      <th>title_status_lien</th>
      <th>title_status_missing</th>
      <th>title_status_parts only</th>
      <th>title_status_rebuilt</th>
      <th>...</th>
      <th>condition_fair</th>
      <th>condition_good</th>
      <th>condition_like new</th>
      <th>condition_new</th>
      <th>condition_salvage</th>
      <th>fuel_diesel</th>
      <th>fuel_electric</th>
      <th>fuel_gas</th>
      <th>fuel_hybrid</th>
      <th>fuel_other</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27</th>
      <td>33590</td>
      <td>2014.0</td>
      <td>8</td>
      <td>57923.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>22590</td>
      <td>2010.0</td>
      <td>8</td>
      <td>71229.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>29</th>
      <td>39590</td>
      <td>2020.0</td>
      <td>8</td>
      <td>19160.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>30990</td>
      <td>2017.0</td>
      <td>8</td>
      <td>41124.0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>15000</td>
      <td>2013.0</td>
      <td>6</td>
      <td>128000.0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>...</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 40 columns</p>
</div>




```python
#Linear Regression with Sequential Feature selection (# 5)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state = 42)
selector = SequentialFeatureSelector(LinearRegression(), n_features_to_select=5)
best_features = selector.fit_transform(X_train, y_train)
best_features_df = pd.DataFrame(best_features, columns = selector.get_feature_names_out())

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('column_selector', selector),
    ('linreg', LinearRegression())])
pipe.fit(X_train, y_train)
train_preds = pipe.predict(X_train)
test_preds = pipe.predict(X_test)
train_mse = mean_squared_error(y_train, train_preds)
test_mse = mean_squared_error(y_test, test_preds)
linreg_feature_select_coef = pipe.named_steps['linreg'].coef_

print(f'Train MSE: {train_mse: .2f}')
print(f'Test MSE: {test_mse: .2f}')
pipe
```

    Train MSE:  104792879965223.89
    Test MSE:  222297198453.25
    




<style>#sk-container-id-5 {color: black;}#sk-container-id-5 pre{padding: 0;}#sk-container-id-5 div.sk-toggleable {background-color: white;}#sk-container-id-5 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-5 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-5 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-5 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-5 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-5 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-5 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-5 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-5 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-5 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-5 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-5 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-5 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-5 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-5 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-5 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-5 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-5 div.sk-item {position: relative;z-index: 1;}#sk-container-id-5 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-5 div.sk-item::before, #sk-container-id-5 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-5 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-5 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-5 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-5 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-5 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-5 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-5 div.sk-label-container {text-align: center;}#sk-container-id-5 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-5 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-5" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>Pipeline(steps=[(&#x27;scaler&#x27;, StandardScaler()),
                (&#x27;column_selector&#x27;,
                 SequentialFeatureSelector(estimator=LinearRegression(),
                                           n_features_to_select=5)),
                (&#x27;linreg&#x27;, LinearRegression())])</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-20" type="checkbox" ><label for="sk-estimator-id-20" class="sk-toggleable__label sk-toggleable__label-arrow">Pipeline</label><div class="sk-toggleable__content"><pre>Pipeline(steps=[(&#x27;scaler&#x27;, StandardScaler()),
                (&#x27;column_selector&#x27;,
                 SequentialFeatureSelector(estimator=LinearRegression(),
                                           n_features_to_select=5)),
                (&#x27;linreg&#x27;, LinearRegression())])</pre></div></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-21" type="checkbox" ><label for="sk-estimator-id-21" class="sk-toggleable__label sk-toggleable__label-arrow">StandardScaler</label><div class="sk-toggleable__content"><pre>StandardScaler()</pre></div></div></div><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-22" type="checkbox" ><label for="sk-estimator-id-22" class="sk-toggleable__label sk-toggleable__label-arrow">column_selector: SequentialFeatureSelector</label><div class="sk-toggleable__content"><pre>SequentialFeatureSelector(estimator=LinearRegression(), n_features_to_select=5)</pre></div></div></div><div class="sk-parallel"><div class="sk-parallel-item"><div class="sk-item"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-23" type="checkbox" ><label for="sk-estimator-id-23" class="sk-toggleable__label sk-toggleable__label-arrow">estimator: LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-24" type="checkbox" ><label for="sk-estimator-id-24" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div></div></div></div><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-25" type="checkbox" ><label for="sk-estimator-id-25" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div></div></div>




```python
plt.rcdefaults()

plt.figure(figsize=(10,5))

plt.barh(best_features_df.columns.to_list(),linreg_feature_select_coef,0.3 ,align='edge',color="#D3B4B4", label="Linear Regression")
plt.grid(linewidth=0.2)
plt.xlabel("Coefficient")
plt.ylabel("Predictors")
plt.legend(loc='best')
plt.title
plt.xlim(-5000,3500)
plt.show()
```


    
![png](output_44_0.png)
    



```python
#Linear Regression with Sequential Feature selection (# 7)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state = 42)
selector = SequentialFeatureSelector(LinearRegression(), n_features_to_select=7)
best_features = selector.fit_transform(X_train, y_train)
best_features_df = pd.DataFrame(best_features, columns = selector.get_feature_names_out())

pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('column_selector', selector),
    ('linreg', LinearRegression())])
pipe.fit(X_train, y_train)
train_preds = pipe.predict(X_train)
test_preds = pipe.predict(X_test)
train_mse = mean_squared_error(y_train, train_preds)
test_mse = mean_squared_error(y_test, test_preds)
### END SOLUTION

# Answer check
linreg_coefs = pipe.named_steps['linreg'].coef_
print(f'Train MSE: {train_mse: .2f}')
print(f'Test MSE: {test_mse: .2f}')
pipe
```

    Train MSE:  104792867968355.81
    Test MSE:  222307911211.17
    




<style>#sk-container-id-6 {color: black;}#sk-container-id-6 pre{padding: 0;}#sk-container-id-6 div.sk-toggleable {background-color: white;}#sk-container-id-6 label.sk-toggleable__label {cursor: pointer;display: block;width: 100%;margin-bottom: 0;padding: 0.3em;box-sizing: border-box;text-align: center;}#sk-container-id-6 label.sk-toggleable__label-arrow:before {content: "▸";float: left;margin-right: 0.25em;color: #696969;}#sk-container-id-6 label.sk-toggleable__label-arrow:hover:before {color: black;}#sk-container-id-6 div.sk-estimator:hover label.sk-toggleable__label-arrow:before {color: black;}#sk-container-id-6 div.sk-toggleable__content {max-height: 0;max-width: 0;overflow: hidden;text-align: left;background-color: #f0f8ff;}#sk-container-id-6 div.sk-toggleable__content pre {margin: 0.2em;color: black;border-radius: 0.25em;background-color: #f0f8ff;}#sk-container-id-6 input.sk-toggleable__control:checked~div.sk-toggleable__content {max-height: 200px;max-width: 100%;overflow: auto;}#sk-container-id-6 input.sk-toggleable__control:checked~label.sk-toggleable__label-arrow:before {content: "▾";}#sk-container-id-6 div.sk-estimator input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-6 div.sk-label input.sk-toggleable__control:checked~label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-6 input.sk-hidden--visually {border: 0;clip: rect(1px 1px 1px 1px);clip: rect(1px, 1px, 1px, 1px);height: 1px;margin: -1px;overflow: hidden;padding: 0;position: absolute;width: 1px;}#sk-container-id-6 div.sk-estimator {font-family: monospace;background-color: #f0f8ff;border: 1px dotted black;border-radius: 0.25em;box-sizing: border-box;margin-bottom: 0.5em;}#sk-container-id-6 div.sk-estimator:hover {background-color: #d4ebff;}#sk-container-id-6 div.sk-parallel-item::after {content: "";width: 100%;border-bottom: 1px solid gray;flex-grow: 1;}#sk-container-id-6 div.sk-label:hover label.sk-toggleable__label {background-color: #d4ebff;}#sk-container-id-6 div.sk-serial::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: 0;}#sk-container-id-6 div.sk-serial {display: flex;flex-direction: column;align-items: center;background-color: white;padding-right: 0.2em;padding-left: 0.2em;position: relative;}#sk-container-id-6 div.sk-item {position: relative;z-index: 1;}#sk-container-id-6 div.sk-parallel {display: flex;align-items: stretch;justify-content: center;background-color: white;position: relative;}#sk-container-id-6 div.sk-item::before, #sk-container-id-6 div.sk-parallel-item::before {content: "";position: absolute;border-left: 1px solid gray;box-sizing: border-box;top: 0;bottom: 0;left: 50%;z-index: -1;}#sk-container-id-6 div.sk-parallel-item {display: flex;flex-direction: column;z-index: 1;position: relative;background-color: white;}#sk-container-id-6 div.sk-parallel-item:first-child::after {align-self: flex-end;width: 50%;}#sk-container-id-6 div.sk-parallel-item:last-child::after {align-self: flex-start;width: 50%;}#sk-container-id-6 div.sk-parallel-item:only-child::after {width: 0;}#sk-container-id-6 div.sk-dashed-wrapped {border: 1px dashed gray;margin: 0 0.4em 0.5em 0.4em;box-sizing: border-box;padding-bottom: 0.4em;background-color: white;}#sk-container-id-6 div.sk-label label {font-family: monospace;font-weight: bold;display: inline-block;line-height: 1.2em;}#sk-container-id-6 div.sk-label-container {text-align: center;}#sk-container-id-6 div.sk-container {/* jupyter's `normalize.less` sets `[hidden] { display: none; }` but bootstrap.min.css set `[hidden] { display: none !important; }` so we also need the `!important` here to be able to override the default hidden behavior on the sphinx rendered scikit-learn.org. See: https://github.com/scikit-learn/scikit-learn/issues/21755 */display: inline-block !important;position: relative;}#sk-container-id-6 div.sk-text-repr-fallback {display: none;}</style><div id="sk-container-id-6" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>Pipeline(steps=[(&#x27;scaler&#x27;, StandardScaler()),
                (&#x27;column_selector&#x27;,
                 SequentialFeatureSelector(estimator=LinearRegression(),
                                           n_features_to_select=7)),
                (&#x27;linreg&#x27;, LinearRegression())])</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-26" type="checkbox" ><label for="sk-estimator-id-26" class="sk-toggleable__label sk-toggleable__label-arrow">Pipeline</label><div class="sk-toggleable__content"><pre>Pipeline(steps=[(&#x27;scaler&#x27;, StandardScaler()),
                (&#x27;column_selector&#x27;,
                 SequentialFeatureSelector(estimator=LinearRegression(),
                                           n_features_to_select=7)),
                (&#x27;linreg&#x27;, LinearRegression())])</pre></div></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-27" type="checkbox" ><label for="sk-estimator-id-27" class="sk-toggleable__label sk-toggleable__label-arrow">StandardScaler</label><div class="sk-toggleable__content"><pre>StandardScaler()</pre></div></div></div><div class="sk-item sk-dashed-wrapped"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-28" type="checkbox" ><label for="sk-estimator-id-28" class="sk-toggleable__label sk-toggleable__label-arrow">column_selector: SequentialFeatureSelector</label><div class="sk-toggleable__content"><pre>SequentialFeatureSelector(estimator=LinearRegression(), n_features_to_select=7)</pre></div></div></div><div class="sk-parallel"><div class="sk-parallel-item"><div class="sk-item"><div class="sk-label-container"><div class="sk-label sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-29" type="checkbox" ><label for="sk-estimator-id-29" class="sk-toggleable__label sk-toggleable__label-arrow">estimator: LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div><div class="sk-serial"><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-30" type="checkbox" ><label for="sk-estimator-id-30" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div></div></div></div><div class="sk-item"><div class="sk-estimator sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-31" type="checkbox" ><label for="sk-estimator-id-31" class="sk-toggleable__label sk-toggleable__label-arrow">LinearRegression</label><div class="sk-toggleable__content"><pre>LinearRegression()</pre></div></div></div></div></div></div></div>




```python
plt.rcdefaults()

plt.figure(figsize=(10,5))

plt.barh(best_features_df.columns.to_list(),linreg_coefs,0.3 ,align='edge',color="#D3B4B4", label="Linear Regression")
plt.grid(linewidth=0.2)
plt.xlabel("Coefficient")
plt.ylabel("Predictors")
plt.legend(loc='best')
plt.title
plt.xlim(-5000,3500)
plt.show()
```


    
![png](output_46_0.png)
    


#### As we know using Sequential Feature selection allows us to predict the price, but we will not be able to derive inferences as our model is applied on very limited features.
#### So, Now applied the Ridge Regularization model on the entire dataset and calculated mean_squared_error for various values of alpha hyperparameter


```python
# Ridge Regression

y = cars_data_dummies['price']
X = cars_data_dummies.drop(columns = 'price')

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.33, random_state = 42)

ridge_mse_train_list = []
ridge_mse_test_list = []
inv_alphas = []
coefs = []
alphas = 10**np.linspace(-5,4,15)
for alpha in alphas:
    inv_alphas.append(1/alpha)  
    ridge_reg = Ridge(alpha = alpha)
    ridge_reg.fit(X_train, y_train)
    ridge_mse_train = mean_squared_error(ridge_reg.predict(X_train),y_train)
    ridge_mse_test = mean_squared_error(ridge_reg.predict(X_test),y_test)
    ridge_mse_train_list.append(ridge_mse_train)
    ridge_mse_test_list.append(ridge_mse_test)
    coefs.append(ridge_reg.coef_)
    
error_df = pd.DataFrame(list(zip(alphas, ridge_mse_train_list, ridge_mse_test_list)), columns=['Alpha','Train_MSE','Test_MSE'])
#Rescaling all the errors so that they become Z-scores, so interpreting these errors is a bit non-trivial

scale = StandardScaler()
erro_df = pd.DataFrame(scale.fit_transform(error_df), columns = scale.get_feature_names_out())
error_df['Train_MSE'] = error_df['Train_MSE']/1000000000000
error_df['Test_MSE'] = error_df['Test_MSE']/1000000000000
error_df

```

    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=1.90605e-21): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=8.37517e-21): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=3.68003e-20): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=1.617e-19): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=7.10503e-19): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=3.12192e-18): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=1.37175e-17): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    C:\Users\jpatchigolla\AppData\Local\anaconda3\Lib\site-packages\sklearn\linear_model\_ridge.py:211: LinAlgWarning: Ill-conditioned matrix (rcond=6.02722e-17): result may not be accurate.
      return linalg.solve(A, Xy, assume_a="pos", overwrite_a=True).T
    




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
      <th>Alpha</th>
      <th>Train_MSE</th>
      <th>Test_MSE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.000010</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000044</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000193</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000848</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.003728</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.016379</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.071969</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.316228</td>
      <td>104.776617</td>
      <td>0.238783</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1.389495</td>
      <td>104.776617</td>
      <td>0.238780</td>
    </tr>
    <tr>
      <th>9</th>
      <td>6.105402</td>
      <td>104.776618</td>
      <td>0.238770</td>
    </tr>
    <tr>
      <th>10</th>
      <td>26.826958</td>
      <td>104.776618</td>
      <td>0.238726</td>
    </tr>
    <tr>
      <th>11</th>
      <td>117.876863</td>
      <td>104.776623</td>
      <td>0.238546</td>
    </tr>
    <tr>
      <th>12</th>
      <td>517.947468</td>
      <td>104.776664</td>
      <td>0.237873</td>
    </tr>
    <tr>
      <th>13</th>
      <td>2275.845926</td>
      <td>104.776942</td>
      <td>0.235699</td>
    </tr>
    <tr>
      <th>14</th>
      <td>10000.000000</td>
      <td>104.778606</td>
      <td>0.231066</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.plot(alphas,ridge_mse_train_list,'-' )
plt.xlabel("Alphas")
plt.ylabel("Train MSE")
plt.legend(loc='best')
```

    No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
    




    <matplotlib.legend.Legend at 0x1df120bd110>




    
![png](output_49_2.png)
    



```python
plt.plot(alphas,ridge_mse_test_list,'-' )
plt.xlabel("Alphas")
plt.ylabel("Test MSE")
plt.legend(loc='best')
```

    No artists with labels found to put in legend.  Note that artists whose label start with an underscore are ignored when legend() is called with no argument.
    




    <matplotlib.legend.Legend at 0x1df1505b1d0>




    
![png](output_50_2.png)
    


Above shows that for alpha value of 10000, the mean squared error on the development data set is the lowest.

#### Now used the GridSearchCV to determine the best alpha value for the dataset


```python
# Ridge Regression with GridSearch CV

ridge_pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('ridge', Ridge())
])
parameters_to_try = {'ridge__alpha': 10**np.linspace(-5,4,100)}

model_finder = GridSearchCV(estimator = ridge_pipe, param_grid = parameters_to_try,scoring='neg_mean_squared_error')
model_finder.fit(X_train,y_train)
best_model = model_finder.best_estimator_
print("Best Model ", best_model)

```

    Best Model  Pipeline(steps=[('scaler', StandardScaler()), ('ridge', Ridge(alpha=10000.0))])
    


```python
coefs = best_model.named_steps['ridge'].coef_
ridge_train_preds = best_model.predict(X_train)
ridge_test_preds = best_model.predict(X_test)
ridge_train_mse = mean_squared_error(y_train, ridge_train_preds)
ridge_test_mse = mean_squared_error(y_test, ridge_test_preds)

features = X_train.columns.values.tolist()
column_coefs = pd.DataFrame(list(zip(features, coefs)), columns=['Feature','Coefficient'])
print(f'Ridge Train MSE: {ridge_train_mse: .2f}')
print(f'Ridge Test MSE: {ridge_test_mse: .2f}')

print("\nFeature Coefficients for the best model\n")

column_coefs
```

    Ridge Train MSE:  104776688984552.52
    Ridge Test MSE:  236838978361.93
    
    Feature Coefficients for the best model
    
    




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
      <th>Feature</th>
      <th>Coefficient</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>year</td>
      <td>-7547.542306</td>
    </tr>
    <tr>
      <th>1</th>
      <td>cylinders</td>
      <td>47029.774746</td>
    </tr>
    <tr>
      <th>2</th>
      <td>odometer</td>
      <td>6480.564887</td>
    </tr>
    <tr>
      <th>3</th>
      <td>VIN</td>
      <td>-49117.229310</td>
    </tr>
    <tr>
      <th>4</th>
      <td>title_status_clean</td>
      <td>3348.069877</td>
    </tr>
    <tr>
      <th>5</th>
      <td>title_status_lien</td>
      <td>-2219.659930</td>
    </tr>
    <tr>
      <th>6</th>
      <td>title_status_missing</td>
      <td>758.985377</td>
    </tr>
    <tr>
      <th>7</th>
      <td>title_status_parts only</td>
      <td>1361.829639</td>
    </tr>
    <tr>
      <th>8</th>
      <td>title_status_rebuilt</td>
      <td>-3563.394921</td>
    </tr>
    <tr>
      <th>9</th>
      <td>title_status_salvage</td>
      <td>-1476.662705</td>
    </tr>
    <tr>
      <th>10</th>
      <td>transmission_automatic</td>
      <td>9120.780650</td>
    </tr>
    <tr>
      <th>11</th>
      <td>transmission_manual</td>
      <td>-8496.661246</td>
    </tr>
    <tr>
      <th>12</th>
      <td>transmission_other</td>
      <td>-5307.289665</td>
    </tr>
    <tr>
      <th>13</th>
      <td>drive_fwd</td>
      <td>4299.094978</td>
    </tr>
    <tr>
      <th>14</th>
      <td>drive_rwd</td>
      <td>-4299.094978</td>
    </tr>
    <tr>
      <th>15</th>
      <td>type_SUV</td>
      <td>-20028.938338</td>
    </tr>
    <tr>
      <th>16</th>
      <td>type_bus</td>
      <td>-5839.241377</td>
    </tr>
    <tr>
      <th>17</th>
      <td>type_convertible</td>
      <td>-7623.129420</td>
    </tr>
    <tr>
      <th>18</th>
      <td>type_coupe</td>
      <td>-4554.877242</td>
    </tr>
    <tr>
      <th>19</th>
      <td>type_hatchback</td>
      <td>2509.760757</td>
    </tr>
    <tr>
      <th>20</th>
      <td>type_mini-van</td>
      <td>-10820.064383</td>
    </tr>
    <tr>
      <th>21</th>
      <td>type_offroad</td>
      <td>-5004.185887</td>
    </tr>
    <tr>
      <th>22</th>
      <td>type_other</td>
      <td>7495.412589</td>
    </tr>
    <tr>
      <th>23</th>
      <td>type_pickup</td>
      <td>75236.785696</td>
    </tr>
    <tr>
      <th>24</th>
      <td>type_sedan</td>
      <td>-8208.524200</td>
    </tr>
    <tr>
      <th>25</th>
      <td>type_truck</td>
      <td>-32852.080114</td>
    </tr>
    <tr>
      <th>26</th>
      <td>type_van</td>
      <td>-7162.014614</td>
    </tr>
    <tr>
      <th>27</th>
      <td>type_wagon</td>
      <td>-1371.569874</td>
    </tr>
    <tr>
      <th>28</th>
      <td>condition_excellent</td>
      <td>15010.295847</td>
    </tr>
    <tr>
      <th>29</th>
      <td>condition_fair</td>
      <td>-11576.814241</td>
    </tr>
    <tr>
      <th>30</th>
      <td>condition_good</td>
      <td>-7956.007008</td>
    </tr>
    <tr>
      <th>31</th>
      <td>condition_like new</td>
      <td>-5174.195947</td>
    </tr>
    <tr>
      <th>32</th>
      <td>condition_new</td>
      <td>-1770.068960</td>
    </tr>
    <tr>
      <th>33</th>
      <td>condition_salvage</td>
      <td>-3649.725121</td>
    </tr>
    <tr>
      <th>34</th>
      <td>fuel_diesel</td>
      <td>20437.738929</td>
    </tr>
    <tr>
      <th>35</th>
      <td>fuel_electric</td>
      <td>1370.601995</td>
    </tr>
    <tr>
      <th>36</th>
      <td>fuel_gas</td>
      <td>-7399.649017</td>
    </tr>
    <tr>
      <th>37</th>
      <td>fuel_hybrid</td>
      <td>-1542.745939</td>
    </tr>
    <tr>
      <th>38</th>
      <td>fuel_other</td>
      <td>-5315.394301</td>
    </tr>
  </tbody>
</table>
</div>




```python
plt.rcdefaults()

plt.figure(figsize=(16,16))
plt.barh(features,coefs,0.8 ,align='edge',color="#7E7EC0", label="Ridge regularisation")
plt.grid(linewidth=0.2)
plt.xlabel("Coefficient")
plt.ylabel("Features")
plt.legend(loc='best')
plt.title
plt.xlim(-6500,3500)
plt.show()
```


    
![png](output_55_0.png)
    


### Evaluation

With some modeling accomplished, we aim to reflect on what we identify as a high quality model and what we are able to learn from this.  We should review our business objective and explore how well we can provide meaningful insight on drivers of used car prices.  Your goal now is to distill your findings and determine whether the earlier phases need revisitation and adjustment or if you have information of value to bring back to your client.

### MY POINTS

After executing different Linear and Ridge regression models on our Cars dataset, and calculating mean squared errors on Training and Development sets, tabulated the below table to help compare and identify the models that are best fit for our cars dataset.

![image-4.png](attachment:image-4.png)

This table clearly shows the Linear Regression has the lowest mean squared error on the development set when we use SequentialFeature selector and selected 5 features. These 5 features are Car type - Bus, offroad and Car Condition - Fair/New/Salvage

When we select 7 features, the mean squared error is a bit high and the two new features added are title_status_parts only and fuel_electric

So, overall, the model sees that Car Type, Condition, Title Status and Fuel (Electric/gas/..) as the primary influencing factors of the price. 

Also, the Ridge Regression with alpha = 10000 has the lowest mean squared error on the development set predictions.

Next Steps: We were able to explore, analyze and apply model only on subset of features. Dataset has many missing values.
Less than 10% of the given data has all the information. Data is missing for one feature or the other in 90% of the data set.
Especially some important columns. One of them is Size. The size can be a very big factor in determining the price of the car. But as 75% of datapoints has Size data missing, we had to drop that column. 

As mentioned in the above Exploratory Data Analysis section, we were able to fill in few missing values with mean values, random categorical values etc. and thus were able to utilize 50% of the dataset for analysis. 

So, as next steps, we should go back and collect the data that's missing and re-run our analysis to determine effectively the factors customers would like to look at while purchasing an used car. With good dataset we should be able to predict precisely the price of an used car.


### Deployment

Now that we've settled on our models and findings, it is time to deliver the information to the client.  You should organize your work as a basic report that details your primary findings.  Keep in mind that your audience is a group of used car dealers interested in fine tuning their inventory.

### Findings Report

From the Linear Regression model performed after selecting the best 5 features that influence the pricing of the used car, the below plot is drawn. This clearly shows that these 5 features have a strong negative weight. Which means the presence of these features in the car will make the car less expensive. So, customers also will avoid buying the cars with these features.

![image.png](attachment:image.png)






When we want to look at all the features and not just few, Ridge Regularization model helps. Below is the plot that shows the Coefficients plotted against the Features that clearly shows how each feature positively or negatively impacts the price.

From the plot, it's inferred that -

People prefer to buy Electric or Diesel fuel cars in Excellent condition and are not very old. 
Mostly customers prefer hatch back or pickup vehicles with a forward wheel drive and an automatic transmission. 
They look for a higher power engine which has more cylinders and also drove certain miles along with title status Clean. They are also ok if the title is missing. 

![image-2.png](attachment:image-2.png)



```python

```
