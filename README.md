# BellaBeat-Analysis-Using-SQL-and-PowerBI

- In this case study I will be working as a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused products for women. I will follow  the Key steps Of the data
analysis Process.
 1. Ask
 2. Prepare
 3. Process
 4. Analyze
 5. Share
 6. Act

Data Source: https://www.kaggle.com/datasets/arashnic/fitbit 

## Background: 

Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart
device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The
insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat executive team along with your high-level recommendations for Bellabeat’s marketing strategy.


# Ask

Sršen asks you to analyze smart device usage data in order to gain insight into how consumers use non-Bellabeat smart
devices. She then wants you to select one Bellabeat product to apply these insights to in your presentation. These questions
will guide your analysis:

1. What are some trends in smart device usage?
2. How could these trends apply to Bellabeat customers?
3. How could these trends help influence Bellabeat marketing strategy?
# Prepare

- Where is your data stored?
- Ans: The data used in this analysis is sourced from Fitbit Fitness Tracker Data. CC0: Public Domain, dataset made available through Mobius and is made available on Kaggle.
- How is the data organized? Is it in long or wide format?
- Ans: It is in both long and wide format since it involves time I choose to work with long format
- Are there issues with bias or credibility in this data? Does your data ROCCC?
- Ans : The Dataset primarily focuses on fitbit users who not use bellabeat smart devices. As a result, the findings and insights drawn from this dataset should be interpreted within the context of Fitbit users specifically. 
- How are you addressing licensing, privacy, security, and accessibility?
- Ans: The dataset used in this analysis is publicly available and released under the CC0: Public Domain license, which means it can be used for various purposes, including research and analysis
- How did you verify the data’s integrity?
- Ans: I conducted data profiling to identify missing values, outliers, and inconsistencies. Additionally, we cross-verified the data with domain knowledge and compared it with expected patterns
- How does it help you answer your question?
- This dataset provides valuable insights into the physical activity, heart rate, and sleep monitoring patterns of Fitbit users. By analyzing this data, we aim to answer specific research questions related to health and activity patterns during the specified time frame. The dataset allows us to explore trends and correlations that can contribute to a better understanding of fitness behaviors and health outcomes. We then could translate them in the buisness world to aid bellabeat's marketing strategy!
- Are there any problems with the data?
- The data is relatively small when we talk about the number of users who tracked other features such as sleep,heartrate and weight. The duration of daily activity data is for 31 days which is fine. The dataset actually came abit preprocessed so we had to deal with minimal preprocessing!


# Process 

Here we will use excel to merge the hours data to hours activity. 
- In order to maintain the integrity we use this steps :
-  Removing the nulls or missing values 
-  checking for any duplicate values and removing them 
-  Transforming the data types to correct ones

```
ALTER TABLE daily_activity
ALTER COLUMN TotalDistance float; 

ALTER TABLE daily_activity
ALTER COLUMN LoggedActivitiesDistance float;

ALTER TABLE daily_activity
ALTER COLUMN VeryActiveDistance float;

ALTER TABLE daily_activity
ALTER COLUMN ModeratelyActiveDistance float;

ALTER TABLE daily_activity
ALTER COLUMN LightActiveDistance float;

ALTER TABLE daily_activity
ALTER COLUMN SedentaryActiveDistance float; 

ALTER TABLE sleepDay
ALTER COLUMN TotalSleepRecords int;

ALTER TABLE hourly_activity
ALTER COLUMN ActivityHour datetime2; 

SELECT [Id], [days_tracked], [TotalSleepRecords], [TotalMinutesAsleep], [TotalTimeInBed]
FROM [Bellabeat].[dbo].[sleepDay]
GROUP BY [Id], [days_tracked], [TotalSleepRecords], [TotalMinutesAsleep], [TotalTimeInBed]
HAVING COUNT(*) > 1; 

````

- changing to appropriate data type is necessary while transforming the data.  we check for duplicates in the sleepday Table using this query and remove those duplicates.
we see that there are duplicate so we will remove those rows.

![Screenshot (208)](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/9f573a16-7ee9-4d5b-8fc8-a852405daeb6)


```
WITH Duplicates AS (
    SELECT
        Id,
        days_tracked,
        TotalSleepRecords,
        TotalMinutesAsleep,
        TotalTimeInBed,
        ROW_NUMBER() OVER (PARTITION BY Id, days_tracked, TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed ORDER BY (SELECT 0)) AS RowNum
    FROM
        [Bellabeat].[dbo].[sleepDay]
)
DELETE FROM Duplicates
WHERE RowNum > 1; 
```
- We have duplicates in minutes sleep table so we will remove duplicate rows.

```
SELECT [Id], [date], [quality], [logId]
FROM [Bellabeat].[dbo].[minuteSleep]
GROUP BY [Id], [date], [quality], [logId]
HAVING COUNT(*) > 1;

WITH Duplicates AS (
    SELECT
        [Id],
        [date],
        [quality],
        [logId],
        ROW_NUMBER() OVER (PARTITION BY [Id], [date], [quality], [logId] ORDER BY (SELECT 0)) AS RowNum
    FROM
        [Bellabeat].[dbo].[minuteSleep]
)
DELETE FROM Duplicates
WHERE RowNum > 1; 
```
# Transforming Date Columns 

- While Processing the Data from the Excel to the SQL Server we get errors if the date format is not recognized properly. so I saved the date with varchar and later I can trasform the date to the required type for analysis.

```
    update
    [Bellabeat].[dbo].[daily_activity] 
    SET ActivityDate= FORMAT(CONVERT(datetime2, ActivityDate, 101), 'MM/dd/yyyy');
````
- Set format type using the datetime2 which is useful for detecting the date types.
- We add extra column day of week under daily activity
```
Alter Table [Bellabeat].[dbo].[daily_activity] 
ADD day_of_week nvarchar(50);
Update [Bellabeat].[dbo].[daily_activity] 
SET day_of_week = DATENAME(DW, ActivityDate)
````
- Adding total sleep columns in daily activity
```
Alter Table [Bellabeat].[dbo].[daily_activity]
ADD total_minutes_sleep int,
total_time_in_bed int;
````
  
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/30906016-a7ab-4796-b8d6-95fc9d6f726c) 

Converting To date Time 
 ```

update
    [Bellabeat].[dbo].[sleepDay] 
    SET days_tracked=FORMAT(CONVERT(datetime2, days_tracked, 101), 'MM/dd/yyyy');
````

- We will join daily activity and sleep day
```
update
    [Bellabeat].[dbo].[sleepDay] 
    SET days_tracked=FORMAT(CONVERT(datetime2, days_tracked, 101), 'MM/dd/yyyy');
````
```
- We will add sleep records to dailyactivity
UPDATE [Bellabeat].[dbo].[daily_activity]
Set total_minutes_sleep = temp2.TotalMinutesAsleep,
total_time_in_bed = temp2.TotalTimeInBed 
From [Bellabeat].[dbo].[daily_activity] as temp1
Full Outer Join [Bellabeat].[dbo].[sleepDay] as temp2
````

- Change the date to the new Format
```
Alter table [Bellabeat].[dbo].[daily_activity]
Add date_new date;
Update [Bellabeat].[dbo].[daily_activity]
 SET date_new=FORMAT(CONVERT(datetime2, date_new, 101), 'MM/dd/yyyy');
on temp1.id = temp2.id and temp1.ActivityDate = temp2.days_tracked;
````

- Split date and time for hourly calories
```
Alter Table [Bellabeat].[dbo].[Calories]
ADD time_new int, date_new DATE;
Update [Bellabeat].[dbo].[Calories]
Set time_new = DATEPART(hh, ActivityHour);
Update [Bellabeat].[dbo].[Calories]
Set date_new = CAST(ActivityHour AS DATE);
````
- We will add date and time to the intensities table
```
Alter Table [Bellabeat].[dbo].[Intensities]
ADD time_new int, date_new DATE;
````
```
Update [Bellabeat].[dbo].[Intensities]
Set time_new = DATEPART(hh, ActivityHour);
Update [Bellabeat].[dbo].[Intensities]
Set date_new = CAST(ActivityHour AS DATE);
````
- We will do the same for steps table
```
lter Table [Bellabeat].[dbo].[Steps]
ADD time_new int, date_new DATE;
Update [Bellabeat].[dbo].[Steps]
Set time_new = DATEPART(hh, ActivityHour);
Update [Bellabeat].[dbo].[Steps]
Set date_new = CAST(ActivityHour AS DATE);
````
- In new table we merge hour calories,hour intensitties,hour steps
```
Create table hourly_data_merge(
id numeric(18,0),
date_new nvarchar(50),
time_new int,
calories numeric(18,0),
total_intensity numeric(18,0),
average_intensity float,
step_total numeric (18,0)
);
````
- Insert the  merge multiple table into one table
```
Insert Into hourly_data_merge(
id, date_new, time_new, calories, total_intensity, average_intensity, step_total)
(SELECT 
temp1.Id, temp1.date_new, temp1.time_new, 
temp1.Calories, temp2.TotalIntensity, temp2.AverageIntensity, temp3.StepTotal
From [Bellabeat].[dbo].[Calories] AS temp1
Inner Join [Bellabeat].[dbo].[Intensities] AS temp2
ON temp1.Id = temp2.Id and temp1.date_new = temp2.date_new and temp1.time_new = temp2.time_new 
Inner Join [Bellabeat].[dbo].[Steps] AS temp3
ON temp1.Id = temp3.Id and temp1.date_new = temp3.date_new and temp1.time_new = temp3.time_new);
````
- Remove the Duplicates
```
SELECT sum(duplicates) as total_duplicates
FROM (SELECT id, time_new, calories, total_intensity, average_intensity, step_total, Count(*) as duplicates
      FROM hourly_data_merge
      GROUP BY id, time_new, calories, total_intensity, average_intensity, step_total
      HAVING Count(*) > 1) AS temp;
```

# Analysis 

## Time Spent On Activity Per Day 
```
Select Distinct Id, SUM(SedentaryMinutes) as sedentary_mins,
SUM(LightlyActiveMinutes) as lightly_active_mins,
SUM(FairlyActiveMinutes) as fairly_active_mins, 
SUM(VeryActiveMinutes) as very_active_mins
From [Bellabeat].[dbo].[daily_activity]
where total_time_in_bed IS NOT NULL
Group by Id
````
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/5fae10be-3f48-4b6b-8ed9-eb98b23f85ac) 

- We will upload this query data to powerBI

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/0dbb2431-78fb-4d67-9115-1f19e668b669) 

## Daily Average Analysis

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/21a8402c-d56b-4998-b861-e541d93d6bbb) 

## Active Duration And Calories Burned 
```
Select Id,
SUM(TotalSteps) as total_steps,
SUM(VeryActiveMinutes) as total_very_active_mins,
Sum(FairlyActiveMinutes) as total_fairly_active_mins,
SUM(LightlyActiveMinutes) as total_lightly_active_mins,
SUM(Calories) as total_calories
From [Bellabeat].[dbo].[daily_activity]
Group By Id
````
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/bdb78adb-4529-4061-8158-10daa2069c13) 

- we can see there is strong correlation between active mins and calories burned.

## Sleep And Calorie Comparison 

```
Select temp1.Id, SUM(TotalMinutesAsleep) as total_sleep_min,
SUM(TotalTimeInBed) as total_time_inbed_min,
SUM(Calories) as calories
From [Bellabeat].[dbo].[daily_activity] as temp1
Inner Join [Bellabeat].[dbo].[sleepDay]as temp2
ON temp1.Id = temp2.Id and temp1.ActivityDate = temp2.days_tracked
Group By temp1.Id
````
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/467cd7dd-745f-4335-911c-c38f97fb4960) 

- we see that people who get good amount of sleep having burned good amount of calories but if it is more it is showing the  opposite affects.

## Avg Met By Month And Day 

- What is a MET?

A MET is a ratio of your working metabolic rate relative to your resting metabolic rate. Metabolic rate is the rate of energy expended per unit of time. It’s one way to describe the intensity of an exercise or activity.

- One MET is the energy you spend sitting at rest — your resting or basal metabolic rate. So, an activity with a MET value of 4 means you’re exerting four times the energy than you would if you were sitting still.

- To put it in perspective, a brisk walk at 3 or 4 miles per hour has a value of 4 METs. Jumping rope, which is a more vigorous activity, has a MET value of 12.3.

##  SUMMARY
METs = metabolic equivalents.
One MET is defined as the energy you use when you’re resting or sitting still.
An activity that has a value of 4 METs means you’re exerting four times the energy than you would if you were sitting still.

- How are METs calculated?
To better understand METs, it’s helpful to know a little about how your body uses energy.

The cells in your muscles use oxygen to help create the energy needed to move your muscles. One MET is approximately 3.5 milliliters of oxygen consumed per kilogram (kg) of body weight per minute.

- So, for example, if you weigh 160 pounds (72.5 kg), you consume about 254 milliliters of oxygen per minute while you’re at rest (72.5 kg x 3.5 mL).

- Energy expenditure may differ from person to person based on several factors, including your age and fitness level. For example, a young athlete who exercises daily won’t need to expend the same amount of energy during a brisk walk as an older, sedentary person.

- For most healthy adults, MET values can be helpful in planning an exercise regimen, or at least gauging how much you’re getting out of your workout routine.

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/d7671546-0307-4e57-874c-e831b3c40422) 


# Share 

- We divide users into each category :
      1. Fairly Active
      2. Lightly Active
      3. Sedentary
      4. Very Active
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/b4e1b682-5b40-4cd1-851d-9a9a6b74d065)

- we see there are 9 people each in fairly active and less active which is more compared to other categories. since it represents less amount of data of 2 months it only has few people recorded the data with the tracker.


## Steps based on Each Category  

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/57d13c58-d06a-4cf7-a8f1-360c091168a0) 

- It is self Explainable that Avg of total steps is more for the very active person 12487 avg steps whereas fairly active person taking 8000 steps

## Calories Burned For Each Category 

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/2fc940e5-7d03-4659-b870-9fa263be1d35) 

- Though there is some similarity between the steps and the category but there is not much differnce in the calories burned we can see that very active person burning the daily calories of the 2549 whereas the fairly active person burning the calories 2464 there is few calories difference so fairly active person is having some other factors that can impact on the calories.

## Total Intensity By Year, Month and day wise 

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/152ab61f-49eb-477e-a747-26331a7f701f) 

- we can see  that saturdays in both months April and May 2016 having contant intensity when compared to other months

## Steps Based On Day of the Week:

![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/d530accc-424c-46fa-ba56-213a4ff221f9)  

- Sunday being the less Steps taken and saturday being the more amount of steps taken having Avg steps of 344.

## Steps Based on Time:
![image](https://github.com/tiru18324/BellaBeat-Analysis-Using-SQL-and-PowerBI/assets/71921628/e1bf02d5-eb70-4c70-9533-0fb7814ab879) 

- Time from 10 am there is more steps recording which tells people start moving from 10 AM and at afternoon, evening more steps are being recorded.




# Act: 

- Based on the users activity here are some recommendations for Bellabeat's marketing strategy:

## Focus on User Engagement During Peak Activity Hours:

Users tend to be more active and record higher step counts during the late morning and afternoon hours. Bellabeat could tailor marketing messages or reminders during these peak activity times to encourage users to stay active. 

## Promote Sleep Quality and its Impact on Calories Burned:

Highlight the correlation between good sleep quality and the calories burned. Bellabeat could incorporate features or content in their product that supports and promotes healthy sleep habits. This could include sleep tracking features, guided relaxation, or sleep improvement challenges.

## Differentiated Marketing for Various Activity Levels:

Users exhibit different activity levels, categorized as sedentary, lightly active, fairly active, and very active. Bellabeat can tailor marketing campaigns or product features for each category. For instance, offering personalized challenges, rewards, or tips based on the user's activity level. 

## Weekday vs. Weekend Strategies:

Users show variations in activity levels based on the day of the week. Bellabeat could design specific promotions, challenges, or content for weekends to encourage more activity. For instance, weekend challenges or social features that allow users to share their weekend activities. 

## Educational Content on METs and Fitness Intensity:

Since METs are an essential metric for understanding fitness intensity, Bellabeat could provide educational content within their app or website. This could help users better interpret their activity data and make informed decisions about their fitness routines 

## Continuous User Feedback and Iteration:

Establish a system for gathering user feedback on the effectiveness of new features, challenges, or marketing campaigns. Use this feedback to iterate and continuously improve the user experience, ensuring that Bellabeat remains aligned with user needs and preferences. 




















