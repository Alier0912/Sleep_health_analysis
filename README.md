# Sleep_health_analysis





```sql
CREATE TABLE sleep_dataset
(person_id INT PRIMARY KEY,
gender VARCHAR(100),
age INT,
occupation VARCHAR(100),
sleep_duration FLOAT,
quality_of_sleep INT,
physical_activity_level INT,
stress_level INT,
bmi_category VARCHAR(100),
blood_pressure VARCHAR(100),
heart_rate INT,
daily_steps INT,
sleep_disorder VARCHAR(100)
);
```

-- E.D.A
```sql
SELECT * FROM sleep_dataset
```
-- Check for missing values and nulls
```sql
SELECT * FROM sleep_dataset
WHERE person_id IS NULL
OR gender IS NULL
OR age IS NULL
OR occupation IS NULL
OR sleep_duration IS NULL
OR quality_of_sleep IS NULL
OR physical_activity_level IS NULL
OR stress_level IS NULL
OR bmi_category IS NULL
OR blood_pressure IS NULL
OR heart_rate IS NULL
OR daily_steps IS NULL
OR sleep_disorder IS NULL;
```
*our dataset has no null values*

**ANALYSIS**

1. Sleep Patterns & Quality
a)What is the average sleep duration across the dataset?
```sql
SELECT
     AVG(sleep_duration) AS average_sleep_duration
     FROM sleep_dataset
```
b)How does sleep duration vary by age group?
```sql
SELECT
       CASE 
       WHEN age between 10 AND 27 THEN 'Gen Z'
         WHEN age between 28 AND 43 THEN 'Millennials'
         WHEN age between 44 AND 59 THEN 'Gen X'
         WHEN age between 60 AND 75 THEN 'Baby Boomers'
         ELSE 'Older Adults' END AS age_group,
         ROUND(AVG(sleep_duration::integer),2) AS average_sleep_duration
FROM sleep_dataset
GROUP BY age_group
```


c)Is there a difference in sleep duration between genders?
```sql
WITH gender_sleep AS 
(
SELECT 
       gender,
         ROUND(AVG(sleep_duration::integer),2) AS average_sleep_duration
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC
)
SELECT 
      MAX(CASE WHEN gender = 'Female' THEN average_sleep_duration END) -
        MAX(CASE WHEN gender = 'Male' THEN average_sleep_duration END) AS sleep_duration_difference
FROM gender_sleep
```
d)What percentage of people get "healthy" sleep (e.g., 7–9 hours)?
```sql
SELECT
      ROUND(COUNT(CASE WHEN sleep_duration BETWEEN 7 AND 9 THEN 1 END) * 100.0 / COUNT(*),2)
       AS healthy_sleep_percentage
FROM sleep_dataset
```
e)How does sleep quality correlate with sleep duration?
```sql
SELECT
    CORR(sleep_duration, quality_of_sleep) AS correlation_coefficient
FROM sleep_dataset
```
f)Are there specific age groups with notably poor/good sleep quality?
```sql
WITH age_sleep_quality AS 
(
SELECT
      *,
    CASE 
        WHEN age BETWEEN 10 AND 27 THEN 'Gen Z'
        WHEN age BETWEEN 28 AND 43 THEN 'Millennials'
        WHEN age BETWEEN 44 AND 59 THEN 'Gen X'
        WHEN age BETWEEN 60 AND 75 THEN 'Baby Boomers'
        ELSE 'Older Adults' END AS age_group,
    ROUND(AVG(quality_of_sleep::integer),2) AS average_sleep_quality,
    CASE
        WHEN AVG(quality_of_sleep::integer) < 5 THEN 'Poor Sleep '
        WHEN AVG(quality_of_sleep::integer) BETWEEN 5 AND 7 THEN 'Average Sleep '
        ELSE 'Good Sleep ' END AS sleep_quality_category,
        COUNT(*) AS number_of_people,
        ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(),2)  AS percentage_of_group
FROM sleep_dataset
group by person_id
) 
SELECT
    age_group,
    sleep_quality_category
FROM age_sleep_quality
WHERE sleep_quality_category = 'Poor Sleep ' OR sleep_quality_category = 'Good Sleep '
```

**2. Lifestyle & Occupation Impact**
a)Which occupations have the highest/lowest average sleep duration?
```sql
SELECT
    occupation,
    ROUND(AVG(sleep_duration::integer),2) AS average_sleep_duration
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC
```
b)Do high-stress jobs correlate with poorer sleep quality?
```sql
SELECT
     occupation,
     ROUND(AVG(stress_level),2) AS average_stress_level,
     ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
```
C)How does physical activity level affect sleep quality?
```sql
SELECT
    physical_activity_level,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
    FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
d)Is there a relationship between daily step count and sleep quality?
```sql
WITH daily_steps_sleep 
AS 
(
SELECT
    daily_steps,
    age,
    quality_of_sleep,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality,
    CASE
       WHEN age BETWEEN 10 AND 27 THEN 'Gen Z'
       WHEN age BETWEEN 28 AND 43 THEN 'Millennials'
       WHEN age BETWEEN 44 AND 59 THEN 'Gen X'
       WHEN age BETWEEN 60 AND 75 THEN 'Baby Boomers'
       ELSE 'Older Adults' END AS age_group
FROM sleep_dataset
GROUP BY 1,2,3
)
SELECT
    age_group,
    ROUND(AVG(daily_steps),2) AS average_daily_steps,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM daily_steps_sleep
GROUP BY 1
ORDER BY 2 DESC, 2 DESC
```

e)Do people with sedentary occupations report more sleep disorders?
```sql
SELECT
       occupation,
     sleep_disorder
FROM sleep_dataset
```
**3. Health Metrics & Sleep**
f)How does BMI category (Underweight/Normal/Overweight/Obese) relate to sleep quality?
```sql
SELECT
    bmi_category,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
g)Do individuals with higher blood pressure report more sleep disorders?
```sql
WITH blood_pressure_sleep AS 
(
SELECT
     *,
    CASE
    WHEN split_part(blood_pressure, '/', 1)::int >= 180 OR split_part(blood_pressure, '/', 2)::int >= 120 THEN 'Hypertensive Crisis'
    WHEN split_part(blood_pressure, '/', 1)::int >= 140 OR split_part(blood_pressure, '/', 2)::int >= 90 THEN 'Hypertension Stage 2'
    WHEN split_part(blood_pressure, '/', 1)::int >= 130 OR split_part(blood_pressure, '/', 2)::int >= 80 THEN 'Hypertension Stage 1'
    WHEN split_part(blood_pressure, '/', 1)::int >= 120 AND split_part(blood_pressure, '/', 2)::int < 80 THEN 'Elevated'
    WHEN split_part(blood_pressure, '/', 1)::int < 120 AND split_part(blood_pressure, '/', 2)::int < 80 THEN 'Normal'
    ELSE 'Unclassified'
  END AS bp_category
FROM sleep_dataset
)
SELECT
     bp_category,
        ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality,
        COUNT(sleep_disorder) AS number_of_sleep_disorders
FROM blood_pressure_sleep
WHERE sleep_disorder IN ('Insomnia', 'Sleep Apnea', 'Restless Legs Syndrome')
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```

h)Is there a link between resting heart rate and sleep quality?
```sql
SELECT
    CASE
        WHEN heart_rate < 60 THEN 'Low'
        WHEN heart_rate BETWEEN 60 AND 100 THEN 'Normal'
        ELSE 'High' END AS heart_rate_category,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```

i)Does heart rate vary significantly between those with/without sleep disorders?
```sql
SELECT
     sleep_disorder,
        ROUND(AVG(heart_rate),2) AS average_heart_rate
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
**4. Stress & Mental Health**
j)How does self-reported stress level correlate with sleep quality?
```sql
SELECT
      stress_level,
        ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
k)Are high-stress individuals more likely to have sleep disorders (e.g., insomnia)?
```sql
SELECT
      sleep_disorder,
        ROUND(AVG(stress_level),2) AS average_stress_level,
        ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
l)Does physical activity mitigate the impact of stress on sleep?
```sql
SELECT
    physical_activity_level,
    ROUND(AVG(stress_level),2) AS average_stress_level,
    ROUND(AVG(quality_of_sleep),2) AS average_sleep_quality
FROM sleep_dataset
GROUP BY 1
ORDER BY 2 DESC, 1 DESC
```
