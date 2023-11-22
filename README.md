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
cars = pd.read_csv("C:/Users/jpatchigolla/Downloads/practical_application_II_starter/data/vehicles.csv")
```

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
6. Dropped model as there is a type column

 ```python
cars_data.drop(columns = ['id', 'region', 'paint_color', 'model', 'size', 'manufacturer','state'], inplace= True)
cars_data.shape
```
    (420393, 11)


 ### Data Cleansing

**VIN**

Presence/absence of VIN is what is required to appraise the car correctly. Hence converted the VIN into a numeric column by assigning 1 for all the rows where VIN is present and assigned 0 where VIN is missing. Converted the column into integer column using pd.to_numeric() funciton.

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/04757e8c-5c0a-4c28-92b4-4ebd01abf344)


#### Cylinders

Removed the string value " cylinders" in cylinders column values. Converted the data into integers. Calculated the mean value of the column and it's 6. So, assigned 6 to the missing and 'other' cylinder data. Converted this column also into numeric

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/7f99a29b-e39e-42ca-9c59-d407ea8de8a0)


#### Drive 

"drive" column has values for forward drive differently spelled. Some have 'fwd' and some have '4wd'. Replaced '4wd' with 'fwd'. For all the missing values, randomly assigned one of the 'fwd' or 'rwd'.

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/2a529acc-9813-45c7-a116-851b726c304f)


#### Missing values in "odometer" column are assigned the mean value of the data in that column.
#### Missing values in "transmission", "fuel" and "title_status" columns are assinged randomly one of the categorical values they have
#### Dropped the rows having missing values in 'condition' and 'type' columns

Now the shape of the dataset is (217393, 11)



#### Transform the Data



Converted all the categorical columns into numeric columns by getting dummies using pd.get_dummies() function and concating the resulting columns with the main dataframe


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
      <th>price</th>
      <td>1.000000</td>
      <td>-0.002019</td>
      <td>0.004839</td>
      <td>0.001125</td>
      <td>-0.003452</td>
      <td>0.000835</td>
      <td>-0.000291</td>
      <td>-0.000284</td>
      <td>-0.000245</td>
      <td>-0.000504</td>
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
      <td>-0.004497</td>
      <td>0.018059</td>
      <td>-0.022060</td>
      <td>0.004841</td>
      <td>0.016634</td>
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
      <td>0.085570</td>
      <td>-0.010220</td>
      <td>-0.022186</td>
      <td>-0.016634</td>
      <td>-0.071858</td>
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
      <td>-0.004962</td>
      <td>0.000053</td>
      <td>0.004870</td>
      <td>0.007142</td>
      <td>-0.005671</td>
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
      <td>0.065711</td>
      <td>-0.020758</td>
      <td>0.023579</td>
      <td>0.038524</td>
      <td>-0.079085</td>
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
      <td>-0.004497</td>
      <td>0.085570</td>
      <td>-0.004962</td>
      <td>0.065711</td>
      <td>1.000000</td>
      <td>-0.396369</td>
      <td>-0.304670</td>
      <td>-0.276856</td>
      <td>-0.642233</td>
      <td>...</td>
      <td>-0.018127</td>
      <td>0.152883</td>
      <td>-0.010652</td>
      <td>-0.003748</td>
      <td>-0.104780</td>
      <td>-0.002259</td>
      <td>0.009448</td>
      <td>-0.062816</td>
      <td>-0.000192</td>
      <td>0.078138</td>
    </tr>
    <tr>
      <th>title_status_lien</th>
      <td>-0.000291</td>
      <td>0.018059</td>
      <td>-0.010220</td>
      <td>0.000053</td>
      <td>-0.020758</td>
      <td>-0.396369</td>
      <td>1.000000</td>
      <td>-0.007076</td>
      <td>-0.006430</td>
      <td>-0.014916</td>
      <td>...</td>
      <td>-0.003201</td>
      <td>-0.065293</td>
      <td>0.013508</td>
      <td>0.010274</td>
      <td>-0.000862</td>
      <td>0.015470</td>
      <td>0.004182</td>
      <td>0.015021</td>
      <td>0.000481</td>
      <td>-0.031237</td>
    </tr>
    <tr>
      <th>title_status_missing</th>
      <td>-0.000284</td>
      <td>-0.022060</td>
      <td>-0.022186</td>
      <td>0.004870</td>
      <td>0.023579</td>
      <td>-0.304670</td>
      <td>-0.007076</td>
      <td>1.000000</td>
      <td>-0.004942</td>
      <td>-0.011465</td>
      <td>...</td>
      <td>0.025486</td>
      <td>-0.059331</td>
      <td>-0.020141</td>
      <td>-0.004457</td>
      <td>0.028424</td>
      <td>-0.001320</td>
      <td>-0.002557</td>
      <td>0.019191</td>
      <td>0.002397</td>
      <td>-0.023416</td>
    </tr>
    <tr>
      <th>title_status_parts only</th>
      <td>-0.000245</td>
      <td>0.004841</td>
      <td>-0.016634</td>
      <td>0.007142</td>
      <td>0.038524</td>
      <td>-0.276856</td>
      <td>-0.006430</td>
      <td>-0.004942</td>
      <td>1.000000</td>
      <td>-0.010418</td>
      <td>...</td>
      <td>-0.003235</td>
      <td>-0.061981</td>
      <td>-0.018165</td>
      <td>-0.001760</td>
      <td>0.030283</td>
      <td>-0.001457</td>
      <td>-0.004749</td>
      <td>0.017615</td>
      <td>0.003141</td>
      <td>-0.021117</td>
    </tr>
    <tr>
      <th>title_status_rebuilt</th>
      <td>-0.000504</td>
      <td>0.016634</td>
      <td>-0.071858</td>
      <td>-0.005671</td>
      <td>-0.079085</td>
      <td>-0.642233</td>
      <td>-0.014916</td>
      <td>-0.011465</td>
      <td>-0.010418</td>
      <td>1.000000</td>
      <td>...</td>
      <td>-0.002851</td>
      <td>-0.090815</td>
      <td>0.027540</td>
      <td>0.001678</td>
      <td>0.004331</td>
      <td>-0.002390</td>
      <td>-0.009296</td>
      <td>0.044069</td>
      <td>-0.002060</td>
      <td>-0.050426</td>
    </tr>
    <tr>
      <th>title_status_salvage</th>
      <td>-0.000442</td>
      <td>-0.017098</td>
      <td>-0.044123</td>
      <td>0.010309</td>
      <td>-0.047662</td>
      <td>-0.469496</td>
      <td>-0.010904</td>
      <td>-0.008381</td>
      <td>-0.007616</td>
      <td>-0.017668</td>
      <td>...</td>
      <td>0.028774</td>
      <td>-0.057748</td>
      <td>-0.003026</td>
      <td>0.000622</td>
      <td>0.171827</td>
      <td>-0.003541</td>
      <td>-0.005727</td>
      <td>0.032653</td>
      <td>-0.000663</td>
      <td>-0.036695</td>
    </tr>
    <tr>
      <th>transmission_automatic</th>
      <td>0.001447</td>
      <td>-0.265815</td>
      <td>-0.067613</td>
      <td>0.161034</td>
      <td>-0.335677</td>
      <td>-0.112233</td>
      <td>0.041167</td>
      <td>0.025831</td>
      <td>0.029663</td>
      <td>0.081589</td>
      <td>...</td>
      <td>0.048510</td>
      <td>-0.502472</td>
      <td>0.154162</td>
      <td>0.024347</td>
      <td>0.015554</td>
      <td>0.077998</td>
      <td>-0.056833</td>
      <td>0.238759</td>
      <td>-0.003872</td>
      <td>-0.340646</td>
    </tr>
    <tr>
      <th>transmission_manual</th>
      <td>-0.000866</td>
      <td>-0.263296</td>
      <td>-0.100912</td>
      <td>0.052379</td>
      <td>-0.144292</td>
      <td>-0.014278</td>
      <td>0.007027</td>
      <td>0.006455</td>
      <td>-0.005012</td>
      <td>0.003193</td>
      <td>...</td>
      <td>0.074618</td>
      <td>-0.060157</td>
      <td>0.012108</td>
      <td>0.017537</td>
      <td>0.017079</td>
      <td>0.042864</td>
      <td>-0.015256</td>
      <td>0.037490</td>
      <td>-0.007673</td>
      <td>-0.071434</td>
    </tr>
    <tr>
      <th>transmission_other</th>
      <td>-0.001088</td>
      <td>0.412943</td>
      <td>0.122159</td>
      <td>-0.196120</td>
      <td>0.426526</td>
      <td>0.125473</td>
      <td>-0.046926</td>
      <td>-0.030477</td>
      <td>-0.028730</td>
      <td>-0.087588</td>
      <td>...</td>
      <td>-0.088764</td>
      <td>0.559846</td>
      <td>-0.168561</td>
      <td>-0.034504</td>
      <td>-0.025007</td>
      <td>-0.103817</td>
      <td>0.067587</td>
      <td>-0.270513</td>
      <td>0.007951</td>
      <td>0.395005</td>
    </tr>
    <tr>
      <th>drive_fwd</th>
      <td>0.000349</td>
      <td>0.076469</td>
      <td>-0.208634</td>
      <td>0.028038</td>
      <td>-0.055118</td>
      <td>-0.029142</td>
      <td>0.006941</td>
      <td>-0.000621</td>
      <td>-0.005600</td>
      <td>0.038498</td>
      <td>...</td>
      <td>-0.001966</td>
      <td>-0.124486</td>
      <td>0.032572</td>
      <td>0.007898</td>
      <td>0.003533</td>
      <td>-0.016911</td>
      <td>-0.029786</td>
      <td>0.060731</td>
      <td>0.043081</td>
      <td>-0.073761</td>
    </tr>
    <tr>
      <th>drive_rwd</th>
      <td>-0.000349</td>
      <td>-0.076469</td>
      <td>0.208634</td>
      <td>-0.028038</td>
      <td>0.055118</td>
      <td>0.029142</td>
      <td>-0.006941</td>
      <td>0.000621</td>
      <td>0.005600</td>
      <td>-0.038498</td>
      <td>...</td>
      <td>0.001966</td>
      <td>0.124486</td>
      <td>-0.032572</td>
      <td>-0.007898</td>
      <td>-0.003533</td>
      <td>0.016911</td>
      <td>0.029786</td>
      <td>-0.060731</td>
      <td>-0.043081</td>
      <td>0.073761</td>
    </tr>
    <tr>
      <th>type_SUV</th>
      <td>-0.001744</td>
      <td>-0.005255</td>
      <td>-0.062797</td>
      <td>0.037542</td>
      <td>-0.066917</td>
      <td>-0.030435</td>
      <td>0.018828</td>
      <td>0.016501</td>
      <td>0.015882</td>
      <td>0.012818</td>
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
      <td>0.007853</td>
      <td>-0.002911</td>
      <td>-0.000147</td>
      <td>-0.002834</td>
      <td>-0.005138</td>
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
      <td>0.003505</td>
      <td>-0.000818</td>
      <td>0.001831</td>
      <td>-0.005884</td>
      <td>-0.003961</td>
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
      <td>0.009993</td>
      <td>-0.004790</td>
      <td>-0.007187</td>
      <td>-0.008060</td>
      <td>-0.005466</td>
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
      <td>0.012047</td>
      <td>-0.008567</td>
      <td>-0.008783</td>
      <td>-0.008048</td>
      <td>-0.002931</td>
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
      <td>-0.001527</td>
      <td>-0.004807</td>
      <td>-0.008949</td>
      <td>-0.008473</td>
      <td>0.014064</td>
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
      <td>-0.004085</td>
      <td>0.002639</td>
      <td>0.003567</td>
      <td>-0.000745</td>
      <td>-0.001465</td>
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
      <td>0.032169</td>
      <td>-0.009944</td>
      <td>0.000070</td>
      <td>-0.000900</td>
      <td>-0.031476</td>
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
      <td>0.056060</td>
      <td>-0.018120</td>
      <td>-0.022376</td>
      <td>-0.023295</td>
      <td>-0.033374</td>
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
      <td>-0.012242</td>
      <td>-0.013622</td>
      <td>-0.005492</td>
      <td>-0.007568</td>
      <td>0.026425</td>
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
      <td>-0.033942</td>
      <td>0.032683</td>
      <td>0.015465</td>
      <td>0.020025</td>
      <td>0.008529</td>
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
      <td>-0.031219</td>
      <td>0.014415</td>
      <td>0.019740</td>
      <td>0.026162</td>
      <td>0.005011</td>
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
      <td>0.003114</td>
      <td>-0.007028</td>
      <td>-0.000002</td>
      <td>0.005305</td>
      <td>-0.002231</td>
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
      <td>-0.135623</td>
      <td>0.059453</td>
      <td>0.061867</td>
      <td>0.071776</td>
      <td>0.078567</td>
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
      <td>-0.018127</td>
      <td>-0.003201</td>
      <td>0.025486</td>
      <td>-0.003235</td>
      <td>-0.002851</td>
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
      <td>0.152883</td>
      <td>-0.065293</td>
      <td>-0.059331</td>
      <td>-0.061981</td>
      <td>-0.090815</td>
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
      <td>-0.010652</td>
      <td>0.013508</td>
      <td>-0.020141</td>
      <td>-0.018165</td>
      <td>0.027540</td>
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
      <td>-0.003748</td>
      <td>0.010274</td>
      <td>-0.004457</td>
      <td>-0.001760</td>
      <td>0.001678</td>
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
      <td>-0.104780</td>
      <td>-0.000862</td>
      <td>0.028424</td>
      <td>0.030283</td>
      <td>0.004331</td>
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
      <td>-0.002259</td>
      <td>0.015470</td>
      <td>-0.001320</td>
      <td>-0.001457</td>
      <td>-0.002390</td>
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
      <td>0.009448</td>
      <td>0.004182</td>
      <td>-0.002557</td>
      <td>-0.004749</td>
      <td>-0.009296</td>
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
      <td>-0.062816</td>
      <td>0.015021</td>
      <td>0.019191</td>
      <td>0.017615</td>
      <td>0.044069</td>
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
      <td>-0.000192</td>
      <td>0.000481</td>
      <td>0.002397</td>
      <td>0.003141</td>
      <td>-0.002060</td>
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
      <td>0.078138</td>
      <td>-0.031237</td>
      <td>-0.023416</td>
      <td>-0.021117</td>
      <td>-0.050426</td>
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

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/a29bf07f-fa56-4443-97f3-8ef751062d37)


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

#### Linear Regression on dataset with Polynomial Features for degrees 1,2,3,4

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/e4ba4840-0075-471b-a0c5-3181a8909e33)


    Degree       4.000000e+00
    Train_MSE    1.047832e+14
    Test_MSE     2.324209e+11
    Name: 3, dtype: float64



From the above observation, when the model is applied on PolynomialFeatures for degree 4, the mean squared error on the development data set is the lowest. But the processing time is quite high around 10 minutes because of 105 features under consideration.

So, now set to explore the dataset by selecting 5 or 7 features using SequientialSelector() function and applying Linear Regression on them.

#### Linear Regression with Sequential Feature selection (# 5)

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/f137c721-6d10-4480-982d-54388d8492a5)


#### Linear Regression with Sequential Feature selection (# 7)

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/a760621e-4df6-4a4e-aae1-e40aa1f31b5e)


#### As we know using Sequential Feature selection allows us to predict the price, but we will not be able to derive inferences as our model is applied on very limited features.
#### So, Now applied the Ridge Regularization model on the entire dataset and calculated mean_squared_error for various values of alpha hyperparameter

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/5eebac4f-be63-4d14-a6ba-c2c9838153f0)


#### Plotted the alpha values with corresponding Train MSE and Test MSEs
    
![png](output_31_2.png)
    
    
![png](output_32_2.png)
    


Above shows that for alpha value of 10000, the mean squared error on the development data set is the lowest.

#### Now used the GridSearchCV to determine the best alpha value for the dataset


    Best Model  Pipeline(steps=[('scaler', StandardScaler()), ('ridge', Ridge(alpha=10000.0))])
    Ridge Train MSE:  104776688144975.28
    Ridge Test MSE:  236839115163.52
    
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
      <td>-7540.526506</td>
    </tr>
    <tr>
      <th>1</th>
      <td>cylinders</td>
      <td>47059.048228</td>
    </tr>
    <tr>
      <th>2</th>
      <td>odometer</td>
      <td>6500.711638</td>
    </tr>
    <tr>
      <th>3</th>
      <td>VIN</td>
      <td>-49099.272845</td>
    </tr>
    <tr>
      <th>4</th>
      <td>title_status_clean</td>
      <td>3395.030661</td>
    </tr>
    <tr>
      <th>5</th>
      <td>title_status_lien</td>
      <td>-2214.056855</td>
    </tr>
    <tr>
      <th>6</th>
      <td>title_status_missing</td>
      <td>784.098228</td>
    </tr>
    <tr>
      <th>7</th>
      <td>title_status_parts only</td>
      <td>1123.178266</td>
    </tr>
    <tr>
      <th>8</th>
      <td>title_status_rebuilt</td>
      <td>-3439.469304</td>
    </tr>
    <tr>
      <th>9</th>
      <td>title_status_salvage</td>
      <td>-1533.931164</td>
    </tr>
    <tr>
      <th>10</th>
      <td>transmission_automatic</td>
      <td>9101.882468</td>
    </tr>
    <tr>
      <th>11</th>
      <td>transmission_manual</td>
      <td>-8500.811949</td>
    </tr>
    <tr>
      <th>12</th>
      <td>transmission_other</td>
      <td>-5283.957798</td>
    </tr>
    <tr>
      <th>13</th>
      <td>drive_fwd</td>
      <td>4347.633417</td>
    </tr>
    <tr>
      <th>14</th>
      <td>drive_rwd</td>
      <td>-4347.633417</td>
    </tr>
    <tr>
      <th>15</th>
      <td>type_SUV</td>
      <td>-20036.498613</td>
    </tr>
    <tr>
      <th>16</th>
      <td>type_bus</td>
      <td>-5824.378367</td>
    </tr>
    <tr>
      <th>17</th>
      <td>type_convertible</td>
      <td>-7604.916233</td>
    </tr>
    <tr>
      <th>18</th>
      <td>type_coupe</td>
      <td>-4545.599283</td>
    </tr>
    <tr>
      <th>19</th>
      <td>type_hatchback</td>
      <td>2506.976942</td>
    </tr>
    <tr>
      <th>20</th>
      <td>type_mini-van</td>
      <td>-10817.662930</td>
    </tr>
    <tr>
      <th>21</th>
      <td>type_offroad</td>
      <td>-5001.729276</td>
    </tr>
    <tr>
      <th>22</th>
      <td>type_other</td>
      <td>7512.680950</td>
    </tr>
    <tr>
      <th>23</th>
      <td>type_pickup</td>
      <td>75229.329654</td>
    </tr>
    <tr>
      <th>24</th>
      <td>type_sedan</td>
      <td>-8225.294303</td>
    </tr>
    <tr>
      <th>25</th>
      <td>type_truck</td>
      <td>-32831.366398</td>
    </tr>
    <tr>
      <th>26</th>
      <td>type_van</td>
      <td>-7201.239923</td>
    </tr>
    <tr>
      <th>27</th>
      <td>type_wagon</td>
      <td>-1353.678023</td>
    </tr>
    <tr>
      <th>28</th>
      <td>condition_excellent</td>
      <td>15024.796783</td>
    </tr>
    <tr>
      <th>29</th>
      <td>condition_fair</td>
      <td>-11560.924537</td>
    </tr>
    <tr>
      <th>30</th>
      <td>condition_good</td>
      <td>-7965.871390</td>
    </tr>
    <tr>
      <th>31</th>
      <td>condition_like new</td>
      <td>-5189.901535</td>
    </tr>
    <tr>
      <th>32</th>
      <td>condition_new</td>
      <td>-1777.939542</td>
    </tr>
    <tr>
      <th>33</th>
      <td>condition_salvage</td>
      <td>-3647.760745</td>
    </tr>
    <tr>
      <th>34</th>
      <td>fuel_diesel</td>
      <td>20413.683296</td>
    </tr>
    <tr>
      <th>35</th>
      <td>fuel_electric</td>
      <td>1368.497511</td>
    </tr>
    <tr>
      <th>36</th>
      <td>fuel_gas</td>
      <td>-7393.533491</td>
    </tr>
    <tr>
      <th>37</th>
      <td>fuel_hybrid</td>
      <td>-1563.271007</td>
    </tr>
    <tr>
      <th>38</th>
      <td>fuel_other</td>
      <td>-5296.927238</td>
    </tr>
  </tbody>
</table>
</div>



#### Plotted the coefficients for the different features

    
![png](output_38_0.png)
    


### Evaluation


After executing different Linear and Ridge regression models on our Cars dataset, and calculating mean squared errors on Training and Development sets, tabulated the below table to help compare and identify the models that are best fit for our cars dataset.

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/dedaa896-68e8-41e1-8423-02267a6eb50d)

This table clearly shows the Linear Regression has the lowest mean squared error on the development set when we use SequentialFeature selector and selected 5 features. These 5 features are Car type - Bus, offroad and Car Condition - Fair/New/Salvage

When we select 7 features, the mean squared error is a bit high and the two new features added are title_status_parts only and fuel_electric

So, overall, the model sees that Car Type, Condition, Title Status and Fuel (Electric/gas/..) as the primary influencing factors of the price. 

Also, the Ridge Regression with alpha = 10000 has the lowest mean squared error on the development set predictions.

### Recommendations and Next Steps:
We were able to explore, analyze and apply model only on subset of features. Dataset has many missing values.
Less than 10% of the given data has all the information. Data is missing for one feature or the other in 90% of the data set.
Especially some important columns. One of them is Size. The size can be a very big factor in determining the price of the car. But as 75% of datapoints has Size data missing, we had to drop that column. 

As mentioned in the above Exploratory Data Analysis section, we were able to fill in few missing values with mean values, random categorical values etc. and thus were able to utilize 50% of the dataset for analysis. 

So, as next steps, we should go back and collect the data that's missing and re-run our analysis to determine effectively the factors customers would like to look at while purchasing an used car. With good dataset we should be able to predict precisely the price of an used car.


### Deployment

Now that we've settled on our models and findings, it is time to deliver the information to the client.  You should organize your work as a basic report that details your primary findings.  Keep in mind that your audience is a group of used car dealers interested in fine tuning their inventory.

### Findings Report

From the Linear Regression model performed after selecting the best 5 features that influence the pricing of the used car, the below plot is drawn. This clearly shows that these 5 features have a strong negative weight. Which means the presence of these features in the car will make the car less expensive. So, customers also will avoid buying the cars with these features.

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/8779ec22-e919-42df-a1e5-5e47a441088b)


When we want to look at all the features and not just few, Ridge Regularization model helps. Below is the plot that shows the Coefficients plotted against the Features that clearly shows how each feature positively or negatively impacts the price.

From the plot, it's inferred that -

People prefer to buy Electric or Diesel fuel cars in Excellent condition and are not very old. 
Mostly customers prefer hatch back or pickup vehicles with a forward wheel drive and an automatic transmission. 
They look for a higher power engine which has more cylinders and also drove certain miles along with title status Clean. They are also ok if the title is missing. 

![image](https://github.com/jyothiknj/Used-Cars-Dataset/assets/35855780/030f1ffc-6071-447b-9e5a-eeb0f62e82fb)

