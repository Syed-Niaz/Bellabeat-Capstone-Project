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

**Group By**

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

**User Type - Avg Steps daily Visualization**

We want to find the average number of steps walked daily by each user and then plot that into a chart.

```
#changing 'id' to string type for plotting bar chart

avg_data['id']= avg_data['id'].astype('str')
```


```
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
![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/Avg%20Steps%20daily.png)

**User Classification - Avg Steps**

We will classify users based on the average number of steps they walked daily. The following benchmarks are set for classification:

- Sedentary - Less than 5000 steps a day
- Lightly Active - Between 5000 and 7499 steps a day
- Fairly Active - Between 7500 and 9999 steps a day
- Very Active - 10000 steps or more a day

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
Making a new dataframe 'user_type_percentage' based on **User Classification**

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

**User Classification - Avg Steps Visualization**

```
#USER CATEGORY- Pie Chart of the users categorized by their activity level which is measured by their avg daily steps

#formating fig size and label

plt.figure(figsize=(10,10))
plt.rc('axes', titlesize=15)


#Pie Chart

plt.pie(user_type_percentage['perc_total'], labels= user_type_percentage['user_type'], autopct='%1.1f%%')

plt.title('User Type %')

plt.legend()

plt.show()
```

![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/User%20Type%20-%20Avg%20Steps.png)

**Correlations**

```
#CORRELATION between steps & calories

plt.figure(figsize=(10,10))


#Scatter Plot with Seaborn

sns.regplot(x= 'totalsteps', y= 'calories', data = daily_activity, scatter_kws= {"color": "blue"}, line_kws= {"color": "red"})

plt.title('Total Steps VS Calories')

plt.show()
```

![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/Total%20Steps%20VS%20Calories.png)

```
#CORRELATION between steps & sleep

plt.figure(figsize=(10,10))

#Scatter Plot with seaborn

sns.regplot(x= 'totalsteps', y= 'totalminutesasleep', data = df_activity_sleep, scatter_kws= {"color": "blue"}, line_kws= {"color": "red"})

plt.title('Total Steps VS Total Minutes Slept')
plt.show()
```

![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/Total%20Steps%20VS%20Total%20Minutes%20Slept.png)

**Device Usage**

```
#We will get number of days device was worn but with 1 less day

days_worn = groupby_id_daily_activity['activitydate'].max() - groupby_id_daily_activity['activitydate'].min()
```
Create a 'days_worn' dataframe
```
days_worn = pd.DataFrame(days_worn)

#resetting index

days_worn.reset_index(inplace=True)

#first converting 'activitydate' col value into str so we can use split to separate 'days' term

days_worn['activitydate'] = days_worn['activitydate'].astype('str')

#first converting 'activitydate' col value into str so we can use split to separate 'days' term

days_worn['activitydate'] = days_worn['activitydate'].astype('str')

#spilting away 'days' term from the column

x= days_worn['activitydate'].str.split(' ', expand= True)

#adding the separated column to our 'days_worn' df

days_worn['activitydate']= x[0]

days_worn['activitydate'] = days_worn['activitydate'].astype('int')

#Add the 1 day to fix the final 'days_worn' df

days_worn['activitydate'] = days_worn['activitydate']+1

#renaming column appropriately

days_worn.rename(columns={'activitydate': 'days'}, inplace= True)

days_worn
```

```
#adding a conditional column 'use' based on days device was used/worn per user

conditions = [
    (days_worn['days'] >= 21),
    (days_worn['days'] >= 11) & (days_worn['days'] <= 20),
    (days_worn['days'] <= 10)
]

values = ['high use', 'moderate use', 'low use']

days_worn['use'] = np.select(conditions, values)

days_worn

```
**User Classification - Device Usage**

```
days_worn_perc = days_worn['use'].value_counts(normalize=True)*100

days_worn_perc = pd.DataFrame(days_worn_perc)

days_worn_perc= round(days_worn_perc, 2)

days_worn_perc.reset_index(inplace=True)

days_worn_perc.rename(columns={'index':'use_level',
                                    'user_type': 'perc_total'}, inplace= True)

days_worn_perc
```

**User Classification - Device Usage Visualization**

```
#USER CATEGORY- Pie Chart of the users categorized by their activity level which is measured by their avg daily steps

#formating fig size and label

plt.figure(figsize=(8,8))
plt.rc('axes', titlesize=15)

#Pie Chart

plt.pie(days_worn_perc['use'], labels= days_worn_perc['use_level'], autopct='%1.1f%%')

plt.title('User Type %')

plt.legend()

plt.show()
```
![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/User%20Type%20-%20Device%20Usage.png)

**User Sleep Pattern**

```
#putting it in a df and resetting the index

avg_sleep= groupby_id_daily_sleep.mean()

avg_sleep.reset_index(inplace= True)

avg_sleep
```

```
#Let's see how long user's are in bed when they are not asleep
#could indicate bad sleeping habits

avg_sleep['time_to_fall_asleep'] = avg_sleep['totaltimeinbed'] - avg_sleep['totalminutesasleep']

#change 'id' dtype to str so it can be used as a label

avg_sleep['id']= avg_data['id'].astype('str')

avg_sleep
```

**Users' Sleep Pattern**

```
#change 'id' dtype to str so it can be used as a label

avg_sleep['id']= avg_data['id'].astype('str')
```

```
#formating the figuresize and font

plt.figure(figsize=(17,12))
plt.rc('axes', titlesize=15)
plt.rc('axes', labelsize=15)
plt.rc('xtick', labelsize=12)
plt.xticks(rotation='vertical')

#Barchart

plt.bar(x=avg_sleep['id'], height=avg_sleep['totaltimeinbed'], label= "Time in Bed", )
plt.bar(x=avg_sleep['id'], height=avg_sleep['totalminutesasleep'], label= "Avg min Slept", )
plt.bar(x=avg_sleep['id'], height=avg_sleep['time_to_fall_asleep'], label= "min to Fall Asleep")

plt.title('Sleep Pattern')
plt.ylabel('Minutes')

plt.legend()
plt.show()
```
![This is an image](https://github.com/Syed-Niaz/Bellabeat-Capstone-Project/blob/main/Users'%20Sleep%20Pattern.png)
