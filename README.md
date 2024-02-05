# -*- coding: utf-8 -*-
"""NewYork-hourseprice1.ipynb

Automatically generated by Colaboratory.

Original file is located at
    https://colab.research.google.com/drive/1qScDheDcygKoqFHmKldP4xlFtFjHxeO1
"""

* Import Libraries

import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

import seaborn as sns

from pandas.api.types import is_string_dtype, is_numeric_dtype

from scipy import stats

from sklearn.model_selection import train_test_split, cross_val_score, KFold, GridSearchCV, RandomizedSearchCV

from sklearn.neighbors import KNeighborsClassifier, KNeighborsRegressor

from sklearn.linear_model import LogisticRegression, LinearRegression, Ridge, Lasso, RidgeCV, LassoCV

from sklearn.tree import DecisionTreeClassifier, DecisionTreeRegressor

from sklearn.metrics import accuracy_score, mean_squared_error as MSE, classification_report, confusion_matrix

from sklearn.ensemble import RandomForestRegressor, RandomForestClassifier, GradientBoostingClassifier, GradientBoostingRegressor

from sklearn.preprocessing import StandardScaler, LabelEncoder

from sklearn.pipeline import Pipeline

"""## 1. Import Data"""

* Read the first 5 rows of dataset

df = pd.read_csv('/content/drive/MyDrive/Python/Data sets/NY-House-Dataset.csv')
df.head(5)

* Lower the column names to call easier

df.columns = ['brokertitle', 'type', 'price', 'beds', 'bath', 'property_sqft', 'address', 'state', 'main_address', 'admin_area_lev2', 'locality', 'sublocality', 'street_name', 'long_name','formatted_add', 'lat', 'long']

"""## 2. Insights from the Dataset

###2.1. Shape of dataset
"""

* Check the number of observations and features


df.shape

"""###2.2. Last 5 rows ""'

"""# Extract last 5 rows

df.tail(5)"""

"""###<h3>2.3. Count of non-Null values and data types of each feature</h3>"""

* Extract the count of non-null values for each feature as well as its data type

df.info()

"""###<h3>2.4. Satatistics summary</h3>"""

*Check the statistics summary of entire dataset

df.describe(include = 'all')

"""###<h3>2.5. Handling missing values</h3>"""

* The count of missing values

missing_count = df.isnull().sum()


* The count of all values

value_count = df.isnull().count()

* The percentage of missing values

missing_percentage = round(missing_count/value_count * 100, 2)

* Create a dataframe

missing_df = pd.DataFrame({'count': missing_count, 'percentage': missing_percentage})

print(missing_df)

* Visualize the percentage of missing values:

missing_chart = missing_df.plot.bar(y='percentage')
for index, percentage in enumerate(missing_percentage):
  missing_chart.text(index, percentage, str(percentage) + '%')

"""There is no missing values in this dataset.

###<h3>2.6. Handling dupplicated values</h3>
"""

* Check the duplicate rows

df[df.duplicated()]

"""There are 214 duplicated rows needed to remove as bad effect to our training model later."""

* Remove duplicates

df = df.drop_duplicates()

"""###<h3>2.7. Handling Incorrect Values and Outliers</h3>"""

* Investigate 'price'

df.sort_values(by='price', ascending = True).head(10)

"""Seem the price of *2000 Dollar* to *5000 Dollar* for a house in New York is impossible and unfounded. So we will remove 3 smallest values of 'price' in dataset."""

* Remove 3 smallest values of 'price'

df_sorted = df.sort_values(by='price', ascending = True)

df = df_sorted.iloc[3:]

* Loop for ploting Outliers:

df_num = df[['price', 'beds', 'bath', 'property_sqft']]

for feature in df_num.columns:
  plt.figure(figsize =(8,6))
  sns.boxplot(x=df_num[feature], color = 'lightgreen')
  plt.show()

* Remove 2 largest values of 'price'

df_sorted = df.sort_values(by='price', ascending = False)

df = df_sorted.iloc[2:]

df.sort_values(by='beds', ascending = False).head(5)

* Remove 3 largest values of 'beds'

df_sorted = df.sort_values(by='beds', ascending = False)

df = df_sorted.iloc[2:]

df.sort_values(by='bath', ascending = False).head(5)

* Remove 3 largest values of 'bath'

df_sorted = df.sort_values(by='bath', ascending = False)
df = df_sorted.iloc[1:]
df.sort_values(by='property_sqft', ascending = False).head(5)

"""## 3. Feature Engineering

Our dataset contains 17 features as checked above. This part will dive onto each of these features and see how relevant they are.

###<h3>3.1. Brokertitle</h3>

* Remove 'Brokered by'
"""

# Transform 'brokertitle'
df['brokertitle'] = df['brokertitle'].str.replace('Brokered by', '')
print(df['brokertitle'])

"""* Transform into numeric variable for further used in the correlation analysis"""

# Transform into numeric variable
df['broker_len'] = df['brokertitle'].apply(len)
df.head(3)

"""###<h3>3.2. Address</h3>

Transform into numeric variable for further used in the correlation analysis.
"""

# Transform into numeric variable
df['formatted_add_len'] = df['formatted_add'].apply(len)
df.head(3)

"""###<h3>3.3. Price</h3>"""

# Transform 'price' to Million Dollar
df['price_mil'] = df['price']/1000000
df.drop('price', axis=1, inplace = True)
df.head(5)

"""###<h3>3.4. Investigating and Transforming 'street' and 'locality'</h3>"""

df[['street_name', 'state', 'locality']].head(5)

# Unique values of 'street_name'
df['street_name'].unique()

"""Because there are large amount of unique of street name. So that it will be removed from this analysis."""

# Unique values of 'state'
df['state'].unique()

"""* State"""

# Extract state name only
df['city'] = df['state'].str.split(',').str[0]
df['city'] = df['city'].str.strip()
df['city'].unique()

"""There are large amount of unique of street name. So that it will be removed from this analysis after used for correcting values in 'locality'"""

# Unique values of 'locality'
df['locality'].unique()

df['locality'].value_counts()

# Inspect value 'United State' in 'locality' with respective value in 'city'
rows_with_united_states = df[df['locality'] == 'United States']
rows_with_united_states['city'].unique()

valid_cities = ['Brooklyn', 'Queens', 'Staten Island', 'New York', 'Howard Beach', 'Saint Albans', 'Bronx']


# Update 'locality' column for rows where 'city' is in the valid_cities array
df.loc[(df['city'].isin(valid_cities)) & (df['locality'] == 'United States'), 'locality'] = df['city']

df['locality'].unique()

# Correct values
df['locality'].replace('New York', 'Manhattan', inplace = True)
df['locality'].replace('New York County', 'Manhattan', inplace = True)
df['locality'].replace('Queens County', 'Queens', inplace = True)
df['locality'].replace('The Bronx', 'Bronx', inplace = True)
df['locality'].replace('Bronx County', 'Bronx', inplace = True)
df['locality'].replace('Kings County', 'Brooklyn', inplace = True)
df['locality'].replace('Richmond County', 'Staten Island', inplace = True)
df['locality'].replace('Howard Beach', 'Queens', inplace = True)
df['locality'].replace('Saint Albans', 'Queens', inplace = True)
df['locality'].replace('Flatbush', 'Brooklyn', inplace = True)
df['locality'].value_counts()

# Encode the 'locality'
# Get dummies 'locality'
local_dummies = pd.get_dummies(df['locality'], drop_first = True)
# Concatenate type_dummies with original df
df1 = pd.concat([df, local_dummies], axis = 1)

"""###<h3>3.5. Type</h3>"""

# Create dummy dataframe for 'type' feature
type_dummies = pd.get_dummies(df['type'], drop_first = True)
# Concatenate type_dummies with original df
df1 = pd.concat([df1, type_dummies], axis = 1)
df1.columns

"""###<h3>3.6. Beds</h3>

###<h3>3.7. Bath and Property_sqft</h3>
"""

# Round the 'bath' and 'property_sqft'
df1[['bath', 'property_sqft']] = df1[['bath', 'property_sqft']].round(0)

"""###<h3>3.8. Trimming out and Dividing dataset</h3>"""

# Remove unnecessary columns
df1 = df1.drop(['address', 'state','main_address', 'admin_area_lev2', 'sublocality', 'street_name','long_name', 'formatted_add', 'city'], axis=1)
df1.head(5)

"""The remaining variables are categorized into numerical and categorical, since univariate analysis and multivariate analysis require different approaches to handle different data types."""

# Populate the list of numeri attributes and categorical attributes
num_df1 = []
cat_df1 = []

for column in df1.columns:
  if is_numeric_dtype(df1[column]):
    num_df1.append(column)
  elif is_string_dtype(df1[column]):
    cat_df1.append(column)

print(num_df1)
print(cat_df1)

"""## 4. Univariate Analysis


"""

for column in df1.columns:
  plt.figure(column)
  plt.title(column)
  if is_numeric_dtype(df1[column]):
    df1[column].plot(kind = 'hist')
  elif is_string_dtype(df[column]):
    # Show only the top 10 value count of each categorical feature
    df1[column].value_counts()[:10].plot(kind = 'bar')

"""### 4.1. Price Analysis

1. What is the average price of houses?
2. What is the median price of houses?
4. How does the distribution of house prices?
"""

# The average price
print('The average price of houses is ', df1['price_mil'].mean(), 'million Dollars')

#The median price
print('The median price of houses is ', df1['price_mil'].median(), 'million Dollars')

#The minimum price
print('The mimimum price of houses is ', df1['price_mil'].min(), 'million Dollars')

#The maximum price
print('The maximum price of houses is ', df1['price_mil'].max(), 'million Dollars')

# The distribution of price range
df1['price_mil'].plot(kind = 'hist')
plt.title('Price')
plt.show()

"""The price of houses in New York ranges from **49 Thousand Dollars** up to **65 Million Dollars**, and the average price is **1.8 Milliion Dollars**. Most of the houses have the price below **1 Million Dollars**.

### 4.2. Property Size Analysis

1. What is the average size of houses?
2. What is the median size of houses?
4. How does the distribution of house size?
"""

# The average property size
print('The average property size of houses is ', df1['property_sqft'].mean(), 'square feet')

#The median property size
print('The median property size of houses is ', df1['property_sqft'].median(), 'square feet')

#The minimum property size
print('The minimum property size of houses is ', df1['property_sqft'].min(), 'square feet')

#The maximum property size
print('The maximum property size of houses is ', df1['property_sqft'].max(), 'square feet')

# The distribution of property size range
df1['property_sqft'].plot(kind = 'hist')
plt.title('Property Size (Square Feet)')
plt.show()

"""The size of houses range from 230 sq.ft up to 65.5 thousand sq.ft. The average size is 2160 sq.ft. Most of houses in New York City have size lower than 2000 sq.ft.

### 4.3. Broker Analysis

1. What is the distribution of Brokers?
2. Which Broker has most houses for sale?
3. Which Broker has the most expensive average price?
4. Which Broker has the largest average size of sale houses?
"""

# Create the distribution bar chart
broker_counts = df1['brokertitle'].value_counts()[:10]
broker_counts.plot(kind='bar')

# Add annotations (value counts) on top of the bars
for i, count in enumerate(broker_counts):
    plt.text(i, count + 0.1, str(count), ha='center', va='bottom')

# Set chart title and axis labels
plt.title('Distribution of Brokers')
plt.xlabel('Broker Name')
plt.ylabel('Count')

# Display the plot
plt.show()

# Brokertitle grouped by average price and value counts
broker_count_mean = df1.groupby('brokertitle')['price_mil'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as Broker with more than 2 values
broker_count_mean[broker_count_mean['count']>2].sort_values('mean', ascending = False)

# Brokertitle grouped by average property_sqft and value counts
broker_count_mean = df1.groupby('brokertitle')['property_sqft'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as Broker with more than 2 values
broker_count_mean[broker_count_mean['count'] > 2].sort_values('mean', ascending = False)

"""**COMPASS** is the broker have the most houses on sale with 435 houses. While the broker **Garfield, Leslie J. & Co. Inc** has only 14 houses on sale but get highest average price as $9.6 Million for each house. **Link Ny Realty** is recorded as the broker selling largest houses with average size of 8375 sq.ft.

### 4.4. Type Analysis

1. What is the distribution of house types?
2. Which house type is the most popular?
3. Which house type has the most expensive average price?
4. Which house types is the the largest average size?
"""

# Create the distribution bar chart
type_counts = df1['type'].value_counts()
type_counts.plot(kind='bar')

# Add annotations (value counts) on top of the bars
for i, count in enumerate(type_counts):
    plt.text(i, count + 0.1, str(count), ha='center', va='bottom')

# Set chart title and axis labels
plt.title('Distribution of House Types')
plt.xlabel('House Type')
plt.ylabel('Count')

# Display the plot
plt.show()

# Types grouped by average price and value counts
type_count_mean = df1.groupby('type')['price_mil'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as type with more than 2 value counts
type_count_mean[type_count_mean['count']>2].sort_values('mean', ascending = False)

# Types grouped by average property_sqft and value counts
type_count_mean = df1.groupby('type')['property_sqft'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as type with more than 2 value counts
type_count_mean[type_count_mean['count']>2].sort_values('mean', ascending = False)

"""**Co-op for sale** is the type having the most sales with 1388 houses. While **Townhouse for sale** has the highest price as $6.4 Million for each house and also is the largest house with average size of 3887 sq.ft.

### 4.5. Bed Count Analysis

1. What is the distribution of Bed Counts?
2. Which kind of bed count is the most popular?
3. Which bed count has the most expensive average price?
4. Which bed count has the the largest average size?
"""

# Create the distribution bar chart
bed_counts = df1['beds'].value_counts()
bed_counts.plot(kind='bar')

# Add annotations (value counts) on top of the bars
for i, count in enumerate(bed_counts):
    plt.text(i, count + 0.1, str(count), ha='center', va='bottom')

# Set chart title and axis labels
plt.title('Distribution of Bed Counts')
plt.xlabel('Number of Bedrooms')
plt.ylabel('Count')

# Display the plot
plt.show()

# Number of Beds in dataset
print(sorted(df1['beds'].unique()))

# Bed Counts grouped by average price and value counts
bed_count_mean = df1.groupby('beds')['price_mil'].agg(['count', 'mean']).reset_index().sort_values('mean', ascending = False)
print(bed_count_mean)

# Bed Counts grouped by average property size and value counts
bed_count_mean = df1.groupby('beds')['property_sqft'].agg(['count', 'mean']).reset_index().sort_values('mean', ascending = False)
print(bed_count_mean)

df1[(df1['beds'] == 40) | (df1['beds'] == 30)]

"""The number of bedrooms ranges from 1 to 40. The most popular house is with **3 bedrooms** accounted for 1395 houses. The **Multi-family-home** type with **40 bedrooms** is recored as the most expensive house - **$5.9 Million*** for each; and the one with **30 bedrooms** is the one having largest size as ***18400 sq.ft*** for each house. However, there is only 1 value of each these types recorded in this dataset, leading the lack of credibility of this result.

### 4.6. Bath Count Analysis

1. What is the distribution of Bath Counts?
2. Which kind of bath count is the most popular?
3. Which bath count has the most expensive average price?
4. Which bath count has the the largest average size?
"""

# Create the distribution bar chart
bath_counts = df1['bath'].value_counts()
bath_counts.plot(kind='bar')

# Add annotations (value counts) on top of the bars
for i, count in enumerate(bath_counts):
    plt.text(i, count + 0.1, str(count), ha='center', va='bottom')

# Set chart title and axis labels
plt.title('Distribution of Bath Counts')
plt.xlabel('Number of Bathrooms')
plt.ylabel('Count')

# Display the plot
plt.show()

# Number of Baths in dataset
print(sorted(df1['bath'].unique()))

# Bath Counts grouped by average price and value counts
bath_count_mean = df1.groupby('bath')['price_mil'].agg(['count', 'mean']).reset_index().sort_values('mean', ascending = False)
print(bath_count_mean)

# Bath Counts grouped by average property size and value counts
bath_count_mean = df1.groupby('bath')['property_sqft'].agg(['count', 'mean']).reset_index().sort_values('mean', ascending = False)
print(bath_count_mean)

df1[(df1['bath'] == 13) | (df1['bath'] == 24)]

"""The number of bathrooms ranges from 0 to 24. The most popular house is the 2-bath house accounted for 1880 houses. The **13-bath house**, known as Townhouse type, has the most expensive average price - **$29.9 Million**; and the Multi-family home with 24 bathrooms ths the house having largest size as 18936sq.ft. however, there is only 1 value of each of these types recorded in this dataset, leading the lack of credibility of this result.

### 4.7. Locality Analysis

1. What is the distribution of localities?
2. Which locality is the most popular?
3. Which locality has the most expensive average price?
4. Which locality has the the largest average size?
"""

# Create the distribution bar chart
loc_counts = df1['locality'].value_counts()
loc_counts.plot(kind='bar')

# Add annotations (value counts) on top of the bars
for i, count in enumerate(loc_counts):
    plt.text(i, count + 0.1, str(count), ha='center', va='bottom')

# Set chart title and axis labels
plt.title('Distribution of Localities')
plt.xlabel('Locality')
plt.ylabel('Count')

# Display the plot
plt.show()

# Localities grouped by average price and value counts
loc_count_mean = df1.groupby('locality')['price_mil'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as Locality with more than 2 value counts
loc_count_mean[loc_count_mean['count']>2].sort_values('mean', ascending = False)

# Localities grouped by average property sizeand value counts
loc_count_mean = df1.groupby('locality')['property_sqft'].agg(['count', 'mean']).reset_index().sort_values('count', ascending = False)
# Filter as Locality with more than 2 value counts
loc_count_mean[loc_count_mean['count']>2].sort_values('mean', ascending = False)

"""Known as a financial district and iconic landmarks, Mahattan is shown as the most popular borough with 3324 houses for sale and the highest average price of houses - **$2.3 Million**. Additionally, it also has the largest size as average of 2434 for each house. Although Bronx is well known as a rich historical borough, it is recorded as the borough having lowest average price of houses (340 Thousand Dollars), and smallest size (1210 sq.ft) for each house based on this dataset. And Staten Island is the least popular area with only 67 houses for sale.

## 5. Multicariate Analysis

### 5.1. Numerical vs Numerical
"""

# The correatiob matrix and heatmap
correlation = df.corr()
sns.heatmap(correlation, cmap = 'GnBu', annot = True)

correlation = df1.corr()
sns.heatmap(correlation, cmap = 'GnBu', annot = True)

# Pairplot
sns.pairplot(df, height = 2.5)

"""### 5.2. Categorical vs. Categorical"""

for i in range(0, len(cat_df1)):
  primary_cat = cat_df1[i]
  for j in range(0, len(cat_df1)):
    secondary_cat = cat_df1[j]
    if secondary_cat != primary_cat:
      plt.figure(figsize = (15, 15))
      chart = sns.countplot(
          data = df,
          x = primary_cat,
          hue = secondary_cat,
          palette = 'GnBu',
          order = df[primary_cat].value_counts().iloc[:5].index
      )
      plt.show()

"""### 5.3. Categorical vs. Numerical"""

for i in range(0, len(cat_df1)):
  cat = cat_df1[i]
  for j in range(0, len(num_df1)):
    num = num_df1[j]
    plt.figure(figsize = (15, 15))
    sns.boxplot(x = cat, y = num, data = df1, palette = 'GnBu')

# Compute correlation matrix
corr_matrix = df1.corr()
print(corr_matrix)

# Generate heatmap
plt.figure(figsize=(12, 8))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', square=True)

