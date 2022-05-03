# **1. Exploratory Data Analysis(EDA)**

## **Data Science project lifecycle**

```mermaid
flowchart LR;
ExploratoryDataAnalysis --> FeatureEngineering -->FeatureSelection-->ModelBuilding-->ModelDeployment-->ModelMonitoring;

```

1. Exploratory Data Analysis
2. Feature Engineering
3. Feature Selection
4. Model Building
5. Model Deployment

## **1. Exploratory Data Analysis(EDA):**

### **Steps:**

1. Missing Values
2. Numerical variables
3. Distribution of Numerical Variables
4. Outliers
5. Categorical Variables
6. Cardinality of Categorical Features
7. Relationship between Independent and Dependent Features.
8. Other Methods

---

## **Importing Libraries:**

Importing necessary libraires.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline

# shows all columns
pd.set_option('display.max_columns', None)
```

## **Take a Quick Look at the Data Structure**

```python
#gives the total columns and rows to display
pd.set_option('display.max_columns', None)

df = pd.read_csv('./data/train.csv')

# return tuple of no. of rows and columns
print(f'Shape:{df.shape}')

df.head()
# ------------------------------------------
# displaying high priority results
# Output:
Shape:(1460, 81)
```

```python
# gives columns datatypes and null values count
df.info()

# gives statistics of the dataset
df.describe()
# ------------------------------------------
# displaying high priority results
# Output:
| Column          | Non-Null Count | Dtype  |
| --------------- | -------------- | ------ |
| Restaurant ID   | 9551 non-null  | int64  |
| Restaurant Name | 9551 non-null  | object |
| Country Code    | 9551 non-null  | int64  |
...

| Restaurant ID | Country Code | Longitude   | Latitude    |
| ------------- | ------------ | ----------- | ----------- |
| count         | 9.551000e+03 | 9551.000000 | 9551.000000 |
| mean          | 9.051128e+06 | 18.365616   | 64.126574   |
...
```

### **Merging two datasets:**

Using .merge() you can merge two table like sql joins. Merge on `'Country Code'` column which exist in both tables and do a left join on them.

```python
dfFinal = pd.merge(df, dfCountry, on= 'Country Code', how='left')
```

## **1. Missing values:**

Remove the missing values if its feature has not dependent on the dependent/target variable.

```python
# stores column having atleast 1 null value
# .isna().sum() sum up the null values
featuresWithNa = [feature for feature in df.columns if df[feature].isna().sum()>1]

# mean of null values of each column
for feature in featuresWithNa:
    print(f'{feature}: {round(df[feature].isna().mean(), 4)} % missing values')

# ------------------------------------------
# displaying high priority results
# Output:
LotFrontage: 0.1774 % missing
Alley: 0.9377 % missing
MasVnrType: 0.0055 % missing
...
```

> **Note:** We need to find the relationship between missing values and dependent variable(Sales Price). If missing values and target variable are not related we can simply drop the missing values otherwise keep the missing feature.

```python
for feature in featuresWithNa:
    # create copy to keep original data unchanged
    data = df.copy()

    # replace each null value with 1 and 0 for not null
    data[feature] = np.where(data[feature].isna(), 1, 0)

    # groupby null and not null values and take median of SalePrice of both groups(null and not null)
    data.groupby(feature)['SalePrice'].median().plot.bar()

    plt.title(feature)
    plt.show()

# ------------------------------------------
# displaying high priority results
# Output:
```

![missingValue](https://github.com/nxtbyt/graphics/blob/main/images/miss1.png?raw=true)

Missing data has high prices that shows missing data is important for analysis. Because expensive house prices are missing. So don't drop missing features.

Fill the missing values with **Feature Engineering** techniques.

```python
print(f"total id's: {len(df.Id)}")

# ------------------------------------------
# displaying high priority results
# Output:
1460
```

### **1.1 Visualizing Null Values using Heatmap**

```python
# zomato dataset visualizing null values
sns.heatmap(df.isna(), yticklabels=False, cbar=False, cmap='viridis')
# ------------------------------------------
# displaying high priority results
# Output:
```
![heatMap](https://github.com/nxtbyt/graphics/blob/main/images/miss1.png?raw=true)

## **2. Numerical Variables:**

Numerical features can be used to perform operations on them.

```python
# dtype "O" means object which indicates non-string field values
numericalFeatures = [feature for feature in df.columns if df[feature].dtype != 'O']

# total numerical features
print(f'No. of numerical features: {len(numericalFeatures)}')


df[numericalFeatures].head()
# ------------------------------------------
# displaying high priority results
# Output:
No. of numerical features: 38
```

### **2.1 Temporal Variables**

It is any data with a **time** or **date** associated with it.

```python
# get columns with 'Yr' or 'Year' in it.
yearFeature = [feature for feature in numericalFeatures if 'Yr' in feature or 'Year' in feature]
yearFeature

# get the data of the temporal variables
# unique finds the unique elements of array
for feature in yearFeature:
    print(feature, df[feature].unique())
# ------------------------------------------
# displaying high priority results
# Output:
YearBuilt [2003 1976 2001 1915 2000 ...
YearRemodAdd [2003 1976 2002 ...
GarageYrBlt [2003. 1976. 2001. ...
YrSold [2008 2007 2006 2009 2010]
```

Now plot a graph between year and SalePrice which shows relation between the two.

```python
# find the relation between year variables and SalePrice
for feature in yearFeature:
    df.groupby(feature)['SalePrice'].median().plot()

    plt.xlabel(feature)
    plt.ylabel('SalePrice')
    plt.title(feature)
    plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![yearSale](https://github.com/nxtbyt/graphics/blob/main/images/year2.png?raw=true)

Plot difference between year feature to year sold with Sale Price you can see the no. of years affected by the sale price.

```python
# plot difference between sold price and year feature with SalePrice
for feature in yearFeature:
    if feature != 'YrSold':
        data = df.copy()

        # no. of years
        data[feature] = data['YrSold'] - data[feature]

        # plot year feature vs SalePrice
        plt.scatter(data[feature], data['SalePrice'])
        plt.xlabel(feature)
        plt.ylabel('SalePrice')
        plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![yearFeature](https://github.com/nxtbyt/graphics/blob/main/images/year1.png?raw=true)

### **2.2 Types of Numerical Variables:**

1. Continous Variables
2. Discrete Variables

**2.2.1 Discrete Features:**

Features that have discrete features less than 25.

```python
# features that have discrete features less than 25
discreteFeatures = [feature for feature in numericalFeatures if len(df[feature].unique())
 < 25 and feature not in yearFeature and feature != 'Id']

discreteFeatures
# ------------------------------------------
# displaying high priority results
# Output:
['MSSubClass',
 'OverallQual',
 'OverallCond', ...
```

Comparison of discrete features with target variable.

```python
# discrete features vs SalePrice
for feature in discreteFeatures:
    data = df.copy()
    data.groupby(feature)['SalePrice'].median().plot.bar()

    plt.title("Discrete features vs SalePrice")
    plt.xlabel(feature)
    plt.ylabel("SalePrice")
    plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![discreteFeatures](https://github.com/nxtbyt/graphics/blob/main/images/discrete1.png?raw=true)
**2.2.2 Continous Features:**

We need to find the distribution of the continuous variable either they are gaussian distribution or not. If not we need to convert them into Guassian dist, Log normal or standard normal dist.

```python
# continuous features
continuousFeatures = [feature for feature in numericalFeatures if feature not in discreteFeatures
                      and feature not in yearFeature and feature != 'Id']
print(f'Continuous Feature Count: {len(continuousFeatures)}')
# ------------------------------------------
# displaying high priority results
# Output:
Continuous Feature Count: 16
```

Continuous variables with the sale price comparison.

```python
# For continuous values comparison with the target variable use histograms
# continuous variable distribution

for feature in continuousFeatures:
    data = data.copy()
    data[feature].hist(bins=25)

    plt.title('Continuous Features vs SalePrice')
    plt.xlabel(feature)
    plt.ylabel('SalePrice')
    plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![continuousFeatures](https://github.com/nxtbyt/graphics/blob/main/images/dist1.png?raw=true)

## **3. Distribution of Numerical Variables:**

Continuous data needs to be converted to a distribution to analysis purpose.

```python
# transformations
for feature in continuousFeatures:

    # because log(0) is undefined
    # if 0 exists in the column
    if 0 in df[feature].unique():
        pass

    # if 0 don't exists in the column
    else:
        data = df.copy()
        data[feature] = np.log(data[feature])
        data['SalePrice'] = np.log(data['SalePrice'])

        # scatter plot
        plt.scatter(data[feature], data['SalePrice'])

        plt.title("Tranformed data")
        plt.xlabel(feature)
        plt.ylabel('SalePrice')
        plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![numericalDistribution](https://github.com/nxtbyt/graphics/blob/main/images/dist%202.png?raw=true)

## **4. Outliers:**

The values that are very large or very small compared to all the other related distribution.

```python
# using boxplot to find outliers in continuous data
for feature in continuousFeatures:
    if 0 in df[feature].unique():
        pass
    else:
        data=df.copy()
        data[feature] = np.log(data[feature])
        data['SalePrice'] = np.log(data['SalePrice'])

        # using dataframe builtin method
        data.boxplot(column=feature)
        plt.ylabel(feature)
        plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![outlier](https://github.com/nxtbyt/graphics/blob/main/images/outlier1.png?raw=true)

## **5. Categorical Variables:**

The features that are str or object type.

How many different types or categories of variables exists in a specific column.

```python
# storing all categorical features
categoricalFeatures = [feature for feature in df.columns if df[feature].dtype == 'O']
print(f"No.of categorical features: {len(categoricalFeatures)}")
# ------------------------------------------
# displaying high priority results
# Output:
No.of categorical features: 43
```

### **Occurence of Values:**

```python
# zomato - counts the occurence of each category in column 'Country'
dfFinal['Country'].value_counts()
# ------------------------------------------
# displaying high priority results
# Output:
India             8652
United States      434
United Kingdom      80
```

## **6. Cardinality of Categorical Features:**

Displaying each categorical feature cardinality.

```python
# find the cardinality of each feature
for feature in categoricalFeatures:
    print(f"Cardinality of {feature}: {len(df[feature].unique())}")
# ------------------------------------------
# displaying high priority results
# Output:
Cardinality of MSZoning: 5
Cardinality of Street: 2
Cardinality of Alley: 3
...
```

Pie plot for occurence of each category inside a column.

> **Observation from Graph:** zomato max records are from India then United Stated and United Kingdom.

Use **.index** for getting index of the dataset and **.values** for getting values of the dataset.

```python
# zomato
# names of each of countries
countryNames = dfFinal['Country'].value_counts().index


# counts the occurence of each category in column 'Country'
countryValues=dfFinal['Country'].value_counts().values

# top 3 with the most countryValues
plt.pie(countryValues[:3], labels=countryNames[:3], autopct='%1.3f%%')

# ------------------------------------------
# displaying high priority results
# Output:
```
![occurence](https://github.com/nxtbyt/graphics/blob/main/images/occurenceFeatures.png?raw=true)

Give size of each group of the feature.

### **Using .groupby().size()**

```python
# gives size of each group
dfFinal.groupby(['Country']).size()
# ------------------------------------------
# displaying high priority results
# Output:
Country
Australia           24
Brazil              60
Canada               4
```

Or using below method gives same results.

### **Using .value_counts()**

```python
# gives size of each group
dfFinal['Country'].value_counts()
# ------------------------------------------
# displaying high priority results
# Output:
Country
Australia           24
Brazil              60
Canada               4
```

### **Grouping Features:**

```python
#zomato
# groupby features for the group size
dfFinal.groupby(['Aggregate rating', 'Rating color', 'Rating text']).size()
# ------------------------------------------
# displaying high priority results
# Output:
Aggregate rating  Rating color  Rating text
0.0               White         Not rated      2148
1.8               Red           Poor              1
1.9               Red           Poor              2
```

## **7. Relationship between Independent and Dependent Variable:**

Displaying relationship between `categorical`(independent variable) and `"Saleprice"`(dependent variable).

```python
# relationship between categorical features vs target variable
for feature in categoricalFeatures:
    data = df.copy()

    # groupby each categorical feature and take median of SalePrice of each group
    data.groupby(feature)['SalePrice'].median().plot.bar()

    plt.title("Categorical Feature vs SalePrice")
    plt.xlabel(feature)
    plt.ylabel("SalePrice")
    plt.show()
# ------------------------------------------
# displaying high priority results
# Output:
```
![categorical1](https://github.com/nxtbyt/graphics/blob/main/images/categorical1.png?raw=true)

![categorical2](https://github.com/nxtbyt/graphics/blob/main/images/categorical2.png?raw=true)
## **Other Methods**

### **Feature Filtering :**

```python
# using dataframe values filtering
# find countries with zero Aggregate Rating
countryWithZero = dfFinal[dfFinal['Aggregate rating'] ==0]['Country'].reset_index()
countryWithZero.groupby('Country').size()
# ------------------------------------------
# displaying high priority results
# Output:
Country
Brazil               5
India             2139
United Kingdom       1
United States        3
```

Or you can also use below method.

```python
# zomato
# using groupby()
# find countries with zero Aggregate Rating
dfFinal.groupby(['Aggregate rating', 'Country']).size().reset_index().head(4)
# ------------------------------------------
# displaying high priority results
# Output:
| Aggregate rating | Country        | 0    |
| ---------------- | -------------- | ---- |
| 0.0              | India          | 2148 |
| 0.0              | Brazil         | 5    |
| 0.0              | United Kingdom | 1    |
| 0.0              | United States  | 3    |
```

```python
# zomato
# countries with online deliveries
dfFinal[['Has Online delivery', 'Country']].groupby(['Has Online delivery', 'Country']).size()
# ------------------------------------------
# displaying high priority results
# Output:
Has Online delivery  Country
No                   Australia           24
                     Brazil              60
                     Canada               4
                     India             6229
...
Yes                  India             2423
                     UAE                 28
```

# **2. Feature Engineering**

1. Missing Values
2. Temporal Variables
3. Transforming Numerical Features to a Distribution
4. Removing Rare Categorical Features
5. Encoding Categorical Features
6. Feature Scaling

---

In the real world projects feature engineering is applied both on training and test data seperately.

Importing necessary libraries.

```python
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import MinMaxScaler
```

**Data Leakage:**
Some training data is passed to test data and vice versa which should be avoided because test data is unseen to the model.

```python
df = pd.read_csv('./data/train.csv')
print(f"Shape of dataset: {df.shape}")
# ------------------------------------------
# displaying high priority results
# Output:
Shape of dataset: (1460, 81)
```

**Train Test Split:**
In real world problems we divide the dataset in train and test data using sklearn `train_test_split`.
This splitting of data reduce:

- Data Leakage
- Overfitting

```python
# splites data, labels in train and test
# train_test_split(data, labels)
# test_size= proportion of the dataset to include in the test split.
# train_size= represent the proportion of the dataset to include in the train split.
# random_state=Controls the shuffling applied to the data before applying the split.

X_train, X_test, y_train, y_test = train_test_split(df, df['SalePrice'], test_size=0.1, random_state=0)
print(f"trainData: {X_train.shape}")
print(f"testData: {X_test.shape}")
# ------------------------------------------
# displaying high priority results
# Output:
trainData: (1314, 81)
testData: (146, 81)
```

## **2.1 Missing Values:**

Handling missing values in the data.

### **Categorical Features:**

Categorical missing values are handled by replacing missing values with some **`new label`**.
e.g `Missing`.

```python
# missing values in categorical features
featuresWithNa = [feature for feature in df.columns if
                  df[feature].isna().sum() > 1 and df[feature].dtype == 'O']

# each feature missing values
for feature in featuresWithNa:
    print(f"{feature} missing: {np.round(df[feature].isna().mean(),3)}%")
# ------------------------------------------
# displaying high priority results
# Output:
Alley missing: 0.938%
MasVnrType missing: 0.005%
BsmtQual missing: 0.025%
...
```

Replacing missing values.

```python
# replace missing values with text 'Missing'
def replaceCatFeatures(df, featuresWithNa):
    data = df.copy()
    data[featuresWithNa] = data[featuresWithNa].fillna('Missing')
    return data

df = replaceCatFeatures(df, featuresWithNa)

# see again missing values in categorical features
df[featuresWithNa].isna().sum()
# ------------------------------------------
# displaying high priority results
# Output:
Alley           0
MasVnrType      0
BsmtQual        0
...
```

### **Numerical Features:**

Numerical missing values are replaced by median or mode because out data has a lot of missing values.

```python
# find numerical features with missing values
numericalFeatures = [feature for feature in df.columns if df[feature].isna().sum() > 1
                     and df[feature].dtype != 'O']
numericalFeatures

# each feature missing value %
for feature in numericalFeatures:
    print(f"{feature} missing: {np.round(df[feature].isna().mean(), 3)} %")
# ------------------------------------------
# displaying high priority results
# Output:
LotFrontage missing: 0.177 %
MasVnrArea missing: 0.005 %
GarageYrBlt missing: 0.055 %
```

Replacing numerical features with median of each column.

```python
for feature in numericalFeatures:
    medianValue = df[feature].median()

    # creating new features for nan
    df[feature + 'nan'] = np.where(df[feature].isna(), 1, 0)

    # fill with median of the column
    df[feature].fillna(medianValue, inplace=True)

df[numericalFeatures].isna().sum()
# ------------------------------------------
# displaying high priority results
# Output:
LotFrontage    0
MasVnrArea     0
GarageYrBlt    0
```

## **2.2 Temporal Features:**

Filling the year values with the diff of `Year Feature` with the `YrSold`.

```python
# numericalFeatures: ['YearBuilt', 'YearRemodAdd', 'GarageYrBlt', 'YrSold']
# take each feature diff with 'YrSold' because its graph is decreasing
# yearFeature[:-1] is equal to ['YearBuilt', 'YearRemodAdd', 'GarageYrBlt']
for feature in ['YearBuilt', 'YearRemodAdd', 'GarageYrBlt']:
    df = df.copy()
    df[feature] = df['YrSold'] - df[feature]
df.head()
# ------------------------------------------
# displaying high priority results
# Output:
| YearBuilt | YearRemodAdd |
|-----------|--------------|
| 5         | 5            |
| 31        | 31           |
| 7         | 6            |
```

Lets see after filling the temporal variables with no. of years compared to **"YrSold"**.

```python
for feature in ['YearBuilt', 'YearRemodAdd', 'GarageYrBlt']:
    print(f'{feature} missing: {df[feature].isna().sum()}')
# ------------------------------------------
# displaying high priority results
# Output:
YearBuilt missing: 0
YearRemodAdd missing: 0
GarageYrBlt missing: 0
```

## **2.3 Transforming Numerical Feature to a Distribution:**

Take that numerical features that don't have 0 in them and convert them to distribution. Here we use **log normal distribution**.
Converting to a distribution ensures to remove `skewness` from the data.

```python
# taking some features to convert them to distribution
numericalFeatures =['LotFrontage', 'LotArea', '1stFlrSF']
for feature in numericalFeatures:
    df[feature] = np.log(df[feature])
# ------------------------------------------
# displaying high priority results
# Output:
| LotFrontage | LotArea  | 1stFlrSF |
| ----------- | -------- | -------- |
| 4.174387    | 9.041922 | 6.752270 |
| 4.382027    | 9.169518 | 7.140453 |
| 4.219508    | 9.328123 | 6.824374 |
```

## **2.4 Removing Rare Categorical Features:**

Remove categorical features that are present less than 1% of the observations.

```python
categoricalFeature = [feature for feature in df.columns if df[feature].dtype == 'O']

for feature in categoricalFeature:

    # grouping each category in a feature and count them and divide
    # each category count by total rows to get percentage of each
    # category in a categorical feature
    tmp = df.groupby(feature)['SalePrice'].count()/len(df)

    # stores the categories whose % are greater than 0.01 that exists in feature
    tmpDf = tmp[tmp > 0.01].index

    # if feature has category name that exists in tmpDf then put that category name in
    # that specific row otherwise replace the category name with 'RareVal'
    df[feature] = np.where(df[feature].isin(tmpDf),df[feature], 'RareVal')
# ------------------------------------------
# displaying high priority results
# Output:
| Id  | MSZoning | HouseStyle |
| --- | -------- | ---------- |
| 94  | RareVal  | RareVal    |
```

## **2.5 Encoding Categorical Features:**

Convert the categorical feature to `int` is important because string name should be converted to int values for processing.

```python
for feature in categoricalFeature:

    # for 'MSZoning' it is: ['RareVal', 'RM', 'RH', 'RL', 'FV']
    labels_ordered=df.groupby(feature)['SalePrice'].mean().sort_values().index

    # {'RareVal': 0, 'RM': 1, 'RH': 2, 'RL': 3, 'FV': 4}
    labels_ordered={k:i for i,k in enumerate(labels_ordered)}

    # replace the category name with the index
    df[feature]=df[feature].map(labels_ordered)
# ------------------------------------------
# displaying high priority results
# Output:
| Id  | MSZoning |
| --- | -------- |
| 1   | 3        |
```

## **2.6 Feature Scaling:**

Now using **MinMaxScaler** to scale data that converts data between `0-1`.

```python
# 'Id' and target variable('SalePrice') don't need to be scaled
featureScale = [feature for feature in df.columns if feature not in ['Id', 'SalePrice']]

# creating scaler object
scaler=MinMaxScaler()
scaler.fit(df[featureScale])

# concatenating the Id and SalePrice with the scaled data
data = pd.concat(df[['Id', 'SalePrice']].reset_index(drop=True),
                 pd.DataFrame(scaler.transform(df[featureScale]), columns=featureScale), axis=1)

# exporting data
data.to_csv('X_train.csv', index=False)
# ------------------------------------------
# displaying high priority results
# Output:
| Id  | SalePrice | MSSubClass | MSZoning | LotFrontage | LotArea  | Street | Alley |
| --- | --------- | ---------- | -------- | ----------- | -------- | ------ | ----- |
| 1   | 208500    | 0.235294   | 0.75     | 0.418208    | 0.366344 | 1.0    | 1.0   |
| 2   | 181500    | 0.000000   | 0.75     | 0.495064    | 0.391317 | 1.0    | 1.0   |
...
```

# **3. Feature Selection**

Importing necessary libraries.

```python
from sklearn.linear_model import Lasso
from sklearn.feature_selection import SelectFromModel
```

Using Lasso regression to select important features for predicting y_train(target variable).

```python
# first select Lasso regression model
# select a alpha(equivalent of penalty)
# bigger alpha lesser the features will be selected

# SelectFromModel select features which coefficients are non-zero

featureSelModel = SelectFromModel(Lasso(alpha=0.005, random_state=0))

# fit on the training data
featureSelModel.fit(X_train, y_train)
# ------------------------------------------
# displaying high priority results
# Output:
SelectFromModel(estimator=Lasso(alpha=0.005, random_state=0))
```

Showing the array that shows important and not important features.

```python
# True indicates feature is important, False tells feature is not important
featureSelModel.get_support()
# ------------------------------------------
# displaying high priority results
# Output:
array([ True,  True,  True,  True,  True,  True,  True,  True,  True,
        True,  True,  True,  True,  True,  True,  True,  True,  True,
        True,  True,  True,  True,  True,  True,  True,  True,  True,
        True,  True,
        ...
```

Selecting features using the predicted array.

```python
# important features predicted by lasso
selectedFeature = X_train.columns[(featureSelModel.get_support())]

# total features
print(f'total features: {len(X_train.columns)}')

# selected feature count
print(f'selected features: {len(selectedFeature)}')

# which features are assigned 0 weights(ignored)
print(f'selected features: {np.sum(featureSelModel.estimator_.coef_ == 0)}')

# selected features
print(f"Selected Features: {selectedFeatures}")
# ------------------------------------------
# displaying high priority results
# Output:
total features: 82
selected features: 82
selected features: 0

Selected Features: Index(['MSSubClass', 'MSZoning', 'LotFrontage',
 'LotArea', 'Street', 'Alley','LotShape', 'LandContour', 'Utilities',
       ...
```
