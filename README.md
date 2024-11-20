# Basic-Data-Analysis-pandas-

Importing Libraries
The analysis will be done using the following libraries : 

Pandas:  This library helps to load the data frame in a 2D array format and has multiple functions to perform analysis tasks in one go.
Numpy: Numpy arrays are very fast and can perform large computations in a very short time.
Matplotlib / Seaborn: This library is used to draw visualizations.

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

dataset.shape
(1156, 7)

dataset['PURPOSE'].fillna("NOT", inplace=True)
Changing the START_DATE and END_DATE to the date_time format so that further it can be use to do analysis.

dataset['START_DATE'] = pd.to_datetime(dataset['START_DATE'], 
                                       errors='coerce')
dataset['END_DATE'] = pd.to_datetime(dataset['END_DATE'], 
                                     errors='coerce')

Splitting the START_DATE to date and time column and then converting the time into four different categories i.e. Morning, Afternoon, Evening, Night


from datetime import datetime

dataset['date'] = pd.DatetimeIndex(dataset['START_DATE']).date
dataset['time'] = pd.DatetimeIndex(dataset['START_DATE']).hour

#changing into categories of day and night
dataset['day-night'] = pd.cut(x=dataset['time'],
                              bins = [0,10,15,19,24],
                              labels = ['Morning','Afternoon','Evening','Night'])
Once we are done with creating new columns, we can now drop rows with null values.

dataset.dropna(inplace=True)

It is also important to drop the duplicates rows from the dataset. To do that, refer the code below.


1
dataset.drop_duplicates(inplace=True)
Data Visualization
In this section, we will try to understand and compare all columns.

Let’s start with checking the unique values in dataset of the columns with object datatype.


1
obj = (dataset.dtypes == 'object')
2
object_cols = list(obj[obj].index)
3
​
4
unique_values = {}
5
for col in object_cols:
6
  unique_values[col] = dataset[col].unique().size
7
unique_values
Output : 

{'CATEGORY': 2, 'START': 108, 'STOP': 112, 'PURPOSE': 7, 'date': 113}
Now, we will be using matplotlib and seaborn library for countplot the CATEGORY and PURPOSE columns.


1
plt.figure(figsize=(10,5))
2
​
3
plt.subplot(1,2,1)
4
sns.countplot(dataset['CATEGORY'])
5
plt.xticks(rotation=90)
6
​
7
plt.subplot(1,2,2)
8
sns.countplot(dataset['PURPOSE'])
9
plt.xticks(rotation=90)
Output : 

Uber Rides Data Analysis using Python
 

Let’s do the same for time column, here we will be using the time column which we have extracted above.


1
sns.countplot(dataset['day-night'])
2
plt.xticks(rotation=90)
Output : 

Uber Rides Data Analysis using Python
 

Now, we will be comparing the two different categories along with the PURPOSE of the user.


1
plt.figure(figsize=(15, 5))
2
sns.countplot(data=dataset, x='PURPOSE', hue='CATEGORY')
3
plt.xticks(rotation=90)
4
plt.show()
Output : 

Uber Rides Data Analysis using Python
 

Insights from the above count-plots : 
Most of the rides are booked for business purpose.
Most of the people book cabs for Meetings and Meal / Entertain purpose.
Most of the cabs are booked in the time duration of 10am-5pm (Afternoon).
As we have seen that CATEGORY and PURPOSE columns are two very important columns. So now we will be using OneHotEncoder to categories them.


1
from sklearn.preprocessing import OneHotEncoder
2
object_cols = ['CATEGORY', 'PURPOSE']
3
OH_encoder = OneHotEncoder(sparse=False, handle_unknown='ignore')
4
OH_cols = pd.DataFrame(OH_encoder.fit_transform(dataset[object_cols]))
5
OH_cols.index = dataset.index
6
OH_cols.columns = OH_encoder.get_feature_names_out()
7
df_final = dataset.drop(object_cols, axis=1)
8
dataset = pd.concat([df_final, OH_cols], axis=1)
9
​
10
# This code is modified by Susobhan Akhuli
After that, we can now find the correlation between the columns using heatmap.


1
# Select only numerical columns for correlation calculation
2
numeric_dataset = dataset.select_dtypes(include=['number'])
3
​
4
sns.heatmap(numeric_dataset.corr(), 
5
            cmap='BrBG', 
6
            fmt='.2f', 
7
            linewidths=2, 
8
            annot=True)
9
​
10
# This code is modified by Susobhan Akhuli
Output :

downlo
heatmap

Insights from the heatmap:
Business and Personal Category are highly negatively correlated, this have already proven earlier. So this plot, justifies the above conclusions.
There is not much correlation between the features.
Now, as we need to visualize the month data. This can we same as done before (for hours). 



1
dataset['MONTH'] = pd.DatetimeIndex(dataset['START_DATE']).month
2
month_label = {1.0: 'Jan', 2.0: 'Feb', 3.0: 'Mar', 4.0: 'April',
3
               5.0: 'May', 6.0: 'June', 7.0: 'July', 8.0: 'Aug',
4
               9.0: 'Sep', 10.0: 'Oct', 11.0: 'Nov', 12.0: 'Dec'}
5
dataset["MONTH"] = dataset.MONTH.map(month_label)
6
​
7
mon = dataset.MONTH.value_counts(sort=False)
8
​
9
# Month total rides count vs Month ride max count
10
df = pd.DataFrame({"MONTHS": mon.values,
11
                   "VALUE COUNT": dataset.groupby('MONTH',
12
                                                  sort=False)['MILES'].max()})
13
​
14
p = sns.lineplot(data=df)
15
p.set(xlabel="MONTHS", ylabel="VALUE COUNT")
Output :

dow
lineplot

Insights from the above plot : 
The counts are very irregular.
Still its very clear that the counts are very less during Nov, Dec, Jan, which justifies the fact that  time winters are there in Florida, US.
Visualization for days data.


1
dataset['DAY'] = dataset.START_DATE.dt.weekday
2
day_label = {
3
    0: 'Mon', 1: 'Tues', 2: 'Wed', 3: 'Thus', 4: 'Fri', 5: 'Sat', 6: 'Sun'
4
}
5
dataset['DAY'] = dataset['DAY'].map(day_label)

1
day_label = dataset.DAY.value_counts()
2
sns.barplot(x=day_label.index, y=day_label);
3
plt.xlabel('DAY')
4
plt.ylabel('COUNT')
Output :

dixj
barplot

Now, let’s explore the MILES Column .

We can use boxplot to check the distribution of the column.


1
sns.boxplot(dataset['MILES'])
Output :

downlo
boxplot(dataset[‘MILES’])

As the graph is not clearly understandable. Let’s zoom in it for values lees than 100.



1
sns.boxplot(dataset[dataset['MILES']<100]['MILES'])
Output :

downloa
boxplot(dataset[dataset[‘MILES’]<100][‘MILES’])

It’s bit visible. But to get more clarity we can use distplot for values less than 40.


1
sns.distplot(dataset[dataset['MILES']<40]['MILES'])
Output :

download
distplot

Insights from the above plots :
Most of the cabs booked for the distance of 4-5 miles.
Majorly people chooses cabs for the distance of 0-20 miles.
For distance more than 20 miles cab counts is nearly negligible.

