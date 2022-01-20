<head>
    <p align="center">
    <img src="https://logowik.com/content/uploads/images/bellabeat3809.jpg" alt="Bellabeat" width="200"/>
    </p>
    <h1 align="center">Case Study: Bellabeat</h1>
</head>


# About the company
Bellabeat, a high-tech manufacturer of health-focused products for women. 
Bellabeat is a successful small company, but they have the potential to become a larger player in the global smart device market. 
Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart device fitness data could help unlock new growth opportunities for the company. 

# Questions for the analysis
* What are some trends in smart device usage?
* How could thse trends apply to Bellabeat customers?
* How could thse trends help to influence Bellabeat marketing strategy?

# Business Task
Identify potential opportunities for growth and recommendations for the Bellabeat marketing strategy improvement based on trends in smart device usage.

# SQL Exploration
-- Check to see which column names are shared across tables

```sql
SELECT
  column_name,
  COUNT(table_name)
FROM
  `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
GROUP BY
  1;
```
## Query Result
| column\_name | f0\_ |
| ------------ | ---- |
| Id           | 18   |
| ActivityDay  | 3    |
| StepTotal    | 2    |
| Time         | 1    |
| Value        | 1    |

Seeing this firts 5 rows I found that Id was a common column, let's make sure that it is in every table we have
```sql
SELECT
  table_name,
  SUM(CASE
      WHEN column_name = "Id" THEN 1
    ELSE
    0
  END
    ) AS has_id_column
FROM
  `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
GROUP BY
  1
ORDER BY
  1 ASC;
```
## Query Result
| table\_name                     | has\_id\_column |
| ------------------------------- | --------------- |
| dailyActivity\_merged           | 1               |
| dailyCalories\_merged           | 1               |
| dailyIntensities\_merged        | 1               |
| dailySteps\_merged              | 1               |
| heartrate\_seconds\_merged      | 1               |
| hourlyCalories\_merged          | 1               |
| hourlyIntensities\_merged       | 1               |
| hourlySteps\_merged             | 1               |
| minuteCaloriesNarrow\_merged    | 1               |
| minuteCaloriesWide\_merged      | 1               |
| minuteIntensitiesNarrow\_merged | 1               |
| minuteIntensitiesWide\_merged   | 1               |
| minuteMETsNarrow\_merged        | 1               |
| minuteSleep\_merged             | 1               |
| minuteStepsNarrow\_merged       | 1               |
| minuteStepsWide\_merged         | 1               |
| sleepDay\_merged                | 1               |
| weightLogInfo\_merged           | 1               |


I want to do the analysis based upon daily data, this query go help me to find tables that might be at the day level
```sql
SELECT
  DISTINCT table_name
FROM
  `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
WHERE
  REGEXP_CONTAINS(LOWER(table_name),"day|daily");
```
## Query Result
| table\_name              |
| ------------------------ |
| dailySteps\_merged       |
| dailyActivity\_merged    |
| dailyIntensities\_merged |
| dailyCalories\_merged    |
| sleepDay\_merged         |

Now I have a list of tables I should look at the columns that are shared among the tables
```sql
SELECT
  column_name,
  data_type,
  COUNT(table_name) AS table_count
FROM
  `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
WHERE
  REGEXP_CONTAINS(LOWER(table_name),"day|daily")
GROUP BY
  1,
  2;
```
## Query Result
| column\_name             | data\_type | table\_count |
| ------------------------ | ---------- | ------------ |
| Id                       | INT64      | 5            |
| ActivityDay              | DATE       | 3            |
| StepTotal                | INT64      | 1            |
| ActivityDate             | DATE       | 1            |
| TotalSteps               | INT64      | 1            |
| TotalDistance            | FLOAT64    | 1            |
| TrackerDistance          | FLOAT64    | 1            |
| LoggedActivitiesDistance | FLOAT64    | 1            |
| VeryActiveDistance       | FLOAT64    | 2            |
| ModeratelyActiveDistance | FLOAT64    | 2            |
| LightActiveDistance      | FLOAT64    | 2            |
| SedentaryActiveDistance  | FLOAT64    | 2            |
| VeryActiveMinutes        | INT64      | 2            |
| FairlyActiveMinutes      | INT64      | 2            |
| LightlyActiveMinutes     | INT64      | 2            |
| SedentaryMinutes         | INT64      | 2            |
| Calories                 | INT64      | 2            |
| SleepDay                 | DATE       | 1            |
| TotalSleepRecords        | INT64      | 1            |
| TotalMinutesAsleep       | INT64      | 1            |
| TotalTimeInBed           | INT64      | 1            |

Now I have a list of tables and shared columns among the tables, I should also make certain that the data types align between tables
```sql
SELECT
  column_name,
  table_name,
  data_type
FROM
  `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
WHERE
  REGEXP_CONTAINS(LOWER(table_name),"day|daily")
  AND column_name IN (
  SELECT
    column_name
  FROM
   `bellabeat-334617.Bellabeat.INFORMATION_SCHEMA.COLUMNS`
  WHERE
    REGEXP_CONTAINS(LOWER(table_name),"day|daily")
  GROUP BY
    1
  HAVING
    COUNT(table_name) >=2)
ORDER BY
  1;
```
## Query Results
| column\_name             | table\_name              | data\_type |
| ------------------------ | ------------------------ | ---------- |
| ActivityDay              | dailySteps\_merged       | DATE       |
| ActivityDay              | dailyIntensities\_merged | DATE       |
| ActivityDay              | dailyCalories\_merged    | DATE       |
| Calories                 | dailyActivity\_merged    | INT64      |
| Calories                 | dailyCalories\_merged    | INT64      |
| FairlyActiveMinutes      | dailyActivity\_merged    | INT64      |
| FairlyActiveMinutes      | dailyIntensities\_merged | INT64      |
| Id                       | dailySteps\_merged       | INT64      |
| Id                       | dailyActivity\_merged    | INT64      |
| Id                       | dailyIntensities\_merged | INT64      |
| Id                       | dailyCalories\_merged    | INT64      |
| Id                       | sleepDay\_merged         | INT64      |
| LightActiveDistance      | dailyActivity\_merged    | FLOAT64    |
| LightActiveDistance      | dailyIntensities\_merged | FLOAT64    |
| LightlyActiveMinutes     | dailyActivity\_merged    | INT64      |
| LightlyActiveMinutes     | dailyIntensities\_merged | INT64      |
| ModeratelyActiveDistance | dailyActivity\_merged    | FLOAT64    |
| ModeratelyActiveDistance | dailyIntensities\_merged | FLOAT64    |
| SedentaryActiveDistance  | dailyActivity\_merged    | FLOAT64    |
| SedentaryActiveDistance  | dailyIntensities\_merged | FLOAT64    |
| SedentaryMinutes         | dailyActivity\_merged    | INT64      |
| SedentaryMinutes         | dailyIntensities\_merged | INT64      |
| VeryActiveDistance       | dailyActivity\_merged    | FLOAT64    |
| VeryActiveDistance       | dailyIntensities\_merged | FLOAT64    |
| VeryActiveMinutes        | dailyActivity\_merged    | INT64      |
| VeryActiveMinutes        | dailyIntensities\_merged | INT64      |

Last query with some JOINS for I made the analysis based on dai/daily data
```sql
SELECT
  A.Id,
  A.Calories,
  * EXCEPT(Id,
    Calories,
    ActivityDay,
    SleepDay,
    SedentaryMinutes,
    LightlyActiveMinutes,
    FairlyActiveMinutes,
    VeryActiveMinutes,
    SedentaryActiveDistance,
    LightActiveDistance,
    ModeratelyActiveDistance,
    VeryActiveDistance),
  I.SedentaryMinutes,
  I.LightlyActiveMinutes,
  I.FairlyActiveMinutes,
  I.VeryActiveMinutes,
  I.SedentaryActiveDistance,
  I.LightActiveDistance,
  I.ModeratelyActiveDistance,
  I.VeryActiveDistance
FROM
  `bellabeat-334617.Bellabeat.dailyActivity_merged` A
LEFT JOIN
  `bellabeat-334617.Bellabeat.dailyCalories_merged` C
ON
  A.Id = C.Id
  AND A.ActivityDate=C.ActivityDay
  AND A.Calories = C.Calories
LEFT JOIN
  `bellabeat-334617.Bellabeat.dailyIntensities_merged` I
ON
  A.Id = I.Id
  AND A.ActivityDate=I.ActivityDay
  AND A.FairlyActiveMinutes = I.FairlyActiveMinutes
  AND A.LightActiveDistance = I.LightActiveDistance
  AND A.LightlyActiveMinutes = I.LightlyActiveMinutes
  AND A.ModeratelyActiveDistance = I.ModeratelyActiveDistance
  AND A.SedentaryActiveDistance = I.SedentaryActiveDistance
  AND A.SedentaryMinutes = I.SedentaryMinutes
  AND A.VeryActiveDistance = I.VeryActiveDistance
  AND A.VeryActiveMinutes = I.VeryActiveMinutes
LEFT JOIN
  `bellabeat-334617.Bellabeat.dailySteps_merged` S
ON
  A.Id = S.Id
  AND A.ActivityDate=S.ActivityDay
LEFT JOIN
  `bellabeat-334617.Bellabeat.sleepDay_merged` Sl
ON
  A.Id = Sl.Id
  AND A.ActivityDate=Sl.SleepDay;
```
## Query Results
I limited the results in 10 rows, because the total is 943, so I go show only 10 and made a dashboard in Google Data Studio with the total to show the results.
| Id         | Calories | ActivityDate | TotalSteps | TotalDistance | TrackerDistance | LoggedActivitiesDistance | StepTotal | TotalSleepRecords | TotalMinutesAsleep | TotalTimeInBed | SedentaryMinutes | LightlyActiveMinutes | FairlyActiveMinutes | VeryActiveMinutes | SedentaryActiveDistance | LightActiveDistance | ModeratelyActiveDistance | VeryActiveDistance |
| ---------- | -------- | ------------ | ---------- | ------------- | --------------- | ------------------------ | --------- | ----------------- | ------------------ | -------------- | ---------------- | -------------------- | ------------------- | ----------------- | ----------------------- | ------------------- | ------------------------ | ------------------ |
| 1624580081 | 2690     | 2016-05-01   | 36019      | 28.03000069   | 28.03000069     | 0                        | 36019     |                   |                    |                | 1020             | 171                  | 63                  | 186               | 0.01999999955           | 1.909999967         | 4.190000057              | 21.92000008        |
| 1644430081 | 3226     | 2016-04-14   | 11037      | 8.020000458   | 8.020000458     | 0                        | 11037     |                   |                    |                | 1125             | 252                  | 58                  | 5                 | 0                       | 5.099999905         | 2.559999943              | 0.3600000143       |
| 1644430081 | 3300     | 2016-04-19   | 11256      | 8.180000305   | 8.180000305     | 0                        | 11256     |                   |                    |                | 1099             | 278                  | 58                  | 5                 | 0                       | 5.300000191         | 2.529999971              | 0.3600000143       |
| 1644430081 | 3108     | 2016-04-28   | 9405       | 6.840000153   | 6.840000153     | 0                        | 9405      |                   |                    |                | 1157             | 227                  | 53                  | 3                 | 0                       | 4.309999943         | 2.319999933              | 0.200000003        |
| 1644430081 | 3846     | 2016-04-30   | 18213      | 13.23999977   | 13.23999977     | 0                        | 18213     | 1                 | 124                | 142            | 816              | 402                  | 71                  | 9                 | 0                       | 9.460000038         | 3.140000105              | 0.6299999952       |
| 1644430081 | 3324     | 2016-05-03   | 12850      | 9.340000153   | 9.340000153     | 0                        | 12850     |                   |                    |                | 1115             | 221                  | 94                  | 10                | 0                       | 4.539999962         | 4.090000153              | 0.7200000286       |
| 2022484408 | 2897     | 2016-04-20   | 15112      | 10.67000008   | 10.67000008     | 0                        | 15112     |                   |                    |                | 1053             | 276                  | 63                  | 48                | 0                       | 5.400000095         | 1.929999948              | 3.339999914        |
| 2022484408 | 2709     | 2016-05-09   | 13379      | 9.390000343   | 9.390000343     | 0                        | 13379     |                   |                    |                | 1061             | 297                  | 47                  | 35                | 0                       | 5.639999866         | 1.629999995              | 2.119999886        |
| 2347167796 | 2010     | 2016-04-14   | 10129      | 6.699999809   | 6.699999809     | 0                        | 10129     | 1                 | 445                | 489            | 705              | 206                  | 48                  | 1                 | 0                       | 3.940000057         | 2.74000001               | 0.01999999955      |
| 2347167796 | 2670     | 2016-04-16   | 22244      | 15.07999992   | 15.07999992     | 0                        | 22244     |                   |                    |                | 968              | 268                  | 72                  | 66                | 0                       | 5.53000021          | 4.099999905              | 5.449999809        |


## Dashboard 
After all the exploration and look at some trends in the data, I made the dashboard using Google Data Studio and focused on show how a good sleep helps in calorie burn.
This can help the marketing team to make some ADs about how good is for you to monitor your sleep and the other metrics with the Bellabeat smart device, for a healthier lifestyle.
![Bellabeat Dashboard](/BellaBeat_1.jpg)

