# Bellabeat-Capstone-Project
This capstone project is a part of the 'Google Data Analytics' professional certificate.

## Business Case
Bellabeat is a high-tech company that manufactures health-focused smart products. Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. The Bellabeat app provides users with health data related to their activity, sleep, stress, menstrual cycle, and mindfulness habits. This data can help users better understand their current habits and make healthy decisions. The Bellabeat app connects to their line of smart wellness products.

The main focus of this case is to analyze smart devices fitness data and determine how it could help unlock new growth opportunities for Bellabeat. We will focus on one of Bellabeat’s products: Bellabeat app.

## Data Analysis Process
- Ask
- Prepare
- Process
- Analyze
- Share
- Act

### 1. Ask
Identify trends in how consumers use non-Bellabeat smart devices to apply insights into Bellabeat’s marketing strategy.

**Stakeholders:**
- Urška Sršen - Bellabeat cofounder and Chief Creative Officer
- Sando Mur - Bellabeat cofounder and key member of Bellabeat executive team
- Bellabeat Marketing Analytics team

### 2. Prepare
#### 2.1 Dataset
**Link:** [FitBit Fitness Tracker Data](https://www.kaggle.com/datasets/arashnic/fitbit)
This dataset is stored in Kaggle and was made available through Mobius.

#### 2.2 Information about dataset
These datasets were generated by respondents to a distributed survey via Amazon Mechanical Turk between 03.12.2016-05.12.2016. Thirty eligible Fitbit users consented to the submission of personal tracker data, including minute-level output for physical activity, heart rate, and sleep monitoring. Variation between output represents use of different types of Fitbit trackers and individual tracking behaviors / preferences.

Verifying the metadata of our dataset we can confirm it is open-source.

#### 2.3 Details about dat (All CSV files)
- **dailyActivity_merged**
  - Daily Activity over 31 days of 33 users. Tracking daily: Steps, Distance, Intensities, Calories
- **sleepDay_merged**
  - Daily sleep logs, tracked by: Total count of sleeps a day, Total minutes, Total Time in Bed

### 3. Process
I will use Python and its popular data analysis libraries.

#### 3.1 Importing necessary libraries
- pandas
- numpy
- matplotlib
- seaborn

```
#Importing libraries

import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
%matplotlib inline
```
#### 3.2 Importing datasets

For our analysis we are going to use the 2 CSV files.
- daily_activity
- daily_sleep

```
#importing csv files into dataframes

daily_activity = pd.read_csv(r"D:\PROJECTS\Google DA Capstone\Fitabase Data 4.12.16-5.12.16\For Project\dailyActivity.csv")
daily_sleep = pd.read_csv(r"D:\PROJECTS\Google DA Capstone\Fitabase Data 4.12.16-5.12.16\For Project\sleepDay.csv")
```

#### 3.3 Data Cleaning & Preparation

- **Case consistency**

```
#Turning column names to lowercase for consistency

daily_activity.columns = daily_activity.columns.str.lower()
daily_sleep.columns = daily_sleep.columns.str.lower()
```
- **Removing some unnessary columns**

```
###Removing unncessary columns

#In daily_activity df 'totalDistance' and 'tranckerdistance' seems to have the same data. Remove 'trackerdistance' column

daily_activity.drop(columns='trackerdistance', inplace= True)


#Drop LoggedActivitiesDistance from daily_activity

daily_activity.drop(columns='loggedactivitiesdistance', inplace = True)


#SedentaryActiveDistance doesn't add any value and is pretty much useless so we will remove it

daily_activity.drop(columns='sedentaryactivedistance', inplace = True)
```
- **Round column to 2 decimal places**

```
#Changing 'totaldistance' to 2 decimal places

daily_activity['totaldistance'] = daily_activity['totaldistance'].round(decimals= 2)
```

- **Identifying number of users**

Verifying number of users in datasets
```
daily_activity['id'].nunique()
```
33

People who used their devices daily.
```
daily_sleep['id'].nunique()
```
24

People who used their devices even during sleeping.
```
daily_activity['id'].nunique() - daily_sleep['id'].nunique()
```
9

People who used their devices daily but probably took them off during before their sleep.
- **Removing Duplicates**
```
#Removing duplicates

daily_sleep.drop_duplicates(inplace= True)

#check

sum(daily_sleep.duplicated())
```
0

- **Changing data format**

Changing 'activitydate' & 'sleepday' columns to datetime format
```
daily_activity['activitydate'] = pd.to_datetime(daily_activity['activitydate'])
daily_sleep['sleepday'] = pd.to_datetime(daily_sleep['sleepday'])
```

Since we will merge the 'daily_activity' & 'daily_sleep' dataframes we want to maintain date format consistency, 
so we will ignore the time part of 'sleepday' column in 'daily_sleep' dataframe

```
daily_sleep['sleepday'] = pd.to_datetime(daily_sleep['sleepday']).dt.date
```

We have to change 'sleepday' column format again and rename it.
```
daily_sleep['sleepday'] = pd.to_datetime(daily_sleep['sleepday'])

daily_sleep.rename(columns= {'sleepday':'activitydate'}, inplace= True)
```

- **Merging Dataframes**
```
#Merging 'daily_activity' and 'daily_sleep' datasets

df_activity_sleep = pd.merge(left = daily_activity, right = daily_sleep, on = ['id', 'activitydate'], how = "inner")
```

### 4. Analyze & Share

Group By

```
#group by id for 'df_activity_sleep'

groupby_id_activity_sleep = df_activity_sleep.groupby(by='id')
```

```
#group by id for 'daily_activity' df

groupby_id_daily_activity = daily_activity.groupby(by= 'id')
```

```
#group by 'id' for 'daily_sleep'

groupby_id_daily_sleep = daily_sleep.groupby(by='id')
```

Creating a dataframe 'avg_data' with some average datas of each category as per 'id' and then resetting the index

```
avg_data = groupby_id_activity_sleep.mean()

#resetting the index

avg_data.reset_index(inplace=True)

avg_data
```
We want to find the average number of steps walked daily by each user and then plot that into a chart.

```
#changing 'id' to string type for plotting bar chart

avg_data['id']= avg_data['id'].astype('str')

#adjust figure size, label font size, and axis font size

plt.figure(figsize=(10,10))
plt.rc('axes', titlesize=15)
plt.rc('axes', labelsize=15)
plt.rc('xtick', labelsize=10)
plt.rc('ytick', labelsize=10)

#Horizontal Barchart and values Sorted by 'totalsteps' in ascending order

plt.barh(avg_data['id'], avg_data['totalsteps'].sort_values(ascending=True))

#labeling

plt.xlabel('Avg Steps')
plt.ylabel('ID')
plt.title('Avg Steps walked daily per user')

plt.show()
```
![This is an image]())

**User Classification**

```
#adding a conditional column 'user_type' based on average total steps per user

conditions = [
    (avg_data['totalsteps'] >= 10000),
    (avg_data['totalsteps'] >= 7500) & (avg_data['totalsteps'] < 10000),
    (avg_data['totalsteps'] >= 5000) & (avg_data['totalsteps'] < 7500),
    (avg_data['totalsteps'] < 5000)
]

values = ['very active', 'fairly active', 'lightly active', 'sedentary']

avg_data['user_type'] = np.select(conditions, values)

#round the dataframe values to 2 decimal places

avg_data = avg_data.round(2)

avg_data
```
Making a new datafer for **User Classification**

```
df_user_type = avg_data[['id','user_type']]

#proportion of each user_type value to total in % into a dataframe

user_type_percentage = df_user_type['user_type'].value_counts(normalize= True)*100

user_type_percentage = pd.DataFrame(data= user_type_percentage)


#round to 2 decimal places

user_type_percentage = user_type_percentage.round(2)

user_type_percentage

#reset the index

user_type_percentage.reset_index(inplace= True)

#rename the columns properly

user_type_percentage.rename(columns={'index':'user_type',
                                    'user_type': 'perc_total'}, inplace= True)
                                    
user_type_percentage
```
