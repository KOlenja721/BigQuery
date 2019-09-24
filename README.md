# Project 1: Query Project

- In the Query Project, you will get practice with SQL while learning about
  Google Cloud Platform (GCP) and BiqQuery. You'll answer business-driven
  questions using public datasets housed in GCP. To give you experience with
  different ways to use those datasets, you will use the web UI (BiqQuery) and
  the command-line tools, and work with them in jupyter notebooks.

- We will be using the Bay Area Bike Share Trips Data
  (https://cloud.google.com/bigquery/public-data/bay-bike-share). 

#### Problem Statement

- You're a data scientist at Ford GoBike (https://www.fordgobike.com/), the
  company running Bay Area Bikeshare. You are trying to increase ridership, and
  you want to offer deals through the mobile app to do so. What deals do you
  offer though? Currently, your company has three options: a flat price for a
  single one-way trip, a day pass that allows unlimited 30-minute rides for 24
  hours and an annual membership. 

- Through this project, you will answer these questions: 
  * What are the 5 most popular trips that you would call "commuter trips"?
  * What are your recommendations for offers (justify based on your findings)?


---

## Part 1 - Querying Data with BigQuery

### What is Google Cloud?
- Read: https://cloud.google.com/docs/overview/

### Get Going

- Go to https://cloud.google.com/bigquery/
- Click on "Try it Free"
- It asks for credit card, but you get $300 free and it does not autorenew after the $300 credit is used, so go ahead (OR CHANGE THIS IF SOME SORT OF OTHER ACCESS INFO)
- Now you will see the console screen. This is where you can manage everything for GCP
- Go to the menus on the left and scroll down to BigQuery
- Now go to https://cloud.google.com/bigquery/public-data/bay-bike-share 
- Scroll down to "Go to Bay Area Bike Share Trips Dataset" (This will open a BQ working page.)


### Some initial queries
Paste your SQL query and answer the question in a sentence.

- What's the size of this dataset? (i.e., how many trips)


Answer: The size of this dataset is 983648

Query: 

```sql
#standardSQL
SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

- What is the earliest start time and latest end time for a trip?

Answer: 
The earlist start time is 2013-08-29 09:08:00
The latest end time is 2016-08-31 23:48:00

Query:

```sql
#standardSQL
SELECT min(start_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

```sql
#standardSQL
SELECT max(end_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

- How many bikes are there?
Answer: There are 700 bikes

Query:

```sql
#standardSQL
SELECT count(distinct bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```

### Questions of your own
- Make up 3 questions and answer them using the Bay Area Bike Share Trips Data.
- Use the SQL tutorial (https://www.w3schools.com/sql/default.asp) to help you with mechanics.

- Question 1: How many peak hour trips were made on any random day by subscribers each year?
  
  * Answer: There were 1820, 853, 891, 847 and 159 subscriber trips in 2017, 2016, 2015, 2014, 2013 respectively based on the arbitrary week # 35, Wednesday of the week between 6-10 AM and 3-6 PM. Week 35 was picked because it gives the results for the maximum number of years based on the min and max dates.   
  
  * SQL query: 
  
```sql
select subscriber_type, Extract(year from start_date) as trip_year, count(*) as subscriber_trip_count from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where  (subscriber_type = "Subscriber" and (Extract(week from start_date) = 35 and Extract(dayofweek from start_date) = 3) ) and ( (Extract(hour from start_date) > 6 and Extract(hour from start_date) < 10) or (Extract(hour from start_date) > 15 and Extract(hour from start_date) < 19)) group by subscriber_type, trip_year order by trip_year desc
```

- Question 2: How many peak hour trips were made on any random day by customers each year?

* Answer: There were 221,26,39,49 and 117 customer trips in 2017, 2016, 2015, 2014, 2013 respectively based on the arbitrary week # 35, Wednesday of the week between 6-10 AM and 3-6 PM. Again, Week 35 was picked because it gives the results for the maximum number of years based on the min and max dates.   
      
  * SQL query:

```sql
select subscriber_type, Extract(year from start_date) as trip_year, count(*) as customer_trip_count 
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` 
where  (subscriber_type = "Customer" and (Extract(week from start_date) = 35 
and Extract(dayofweek from start_date) = 3) ) and 
( (Extract(hour from start_date) > 6 and Extract(hour from start_date) < 10) or 
(Extract(hour from start_date) > 15 and Extract(hour from start_date) < 19)) 
group by subscriber_type, trip_year order by trip_year desc
```


- Question 3: What are the total number of trips made during the weekend each year?

  * Answer: Total number of weekend trips each year is 77k, 96k, 19k, 34k, 40k and 17k in 2018, 2017, 2016, 2015, 2014, 2013 respectively
  
  * SQL query:
  
```sql
select EXTRACT(YEAR FROM start_date) as year, count(*) as weekend_trips  
from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` 
where Extract(DAYOFWEEK FROM start_date) = 1 or Extract(DAYOFWEEK FROM start_date) = 7 
group by year order by year desc
```



---

## Part 2 - Querying data from the BigQuery CLI - set up 

### What is Google Cloud SDK?
- Read: https://cloud.google.com/sdk/docs/overview

- If you want to go further, https://cloud.google.com/sdk/docs/concepts has
  lots of good stuff.

### Get Going

- Install Google Cloud SDK: https://cloud.google.com/sdk/docs/

- Try BQ from the command line:

  * General query structure

    ```
    bq query --use_legacy_sql=false '
        SELECT count(*)
        FROM
           `bigquery-public-data.san_francisco.bikeshare_trips`'
    ```

### Queries

1. Rerun last week's queries using bq command line tool (Paste your bq
   queries):

- What's the size of this dataset? (i.e., how many trips)

```
bq query --use_legacy_sql=false 'SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
Waiting on bqjob_r5e02d03a9aa1ca0_0000016d61358563_1 ... (0s) Current status: DONE   
+--------+
|  f0_   |
+--------+
| 983648 |
+--------+
```



- What is the earliest start time and latest end time for a trip?

```
bq query --use_legacy_sql=false 'SELECT min(start_date) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
Waiting on bqjob_r5458cf33dc3f6cec_0000016d6137446c_1 ... (4s) Current status: DONE   
+---------------------+
|         f0_         |
+---------------------+
| 2013-08-29 09:08:00 |
+---------------------+
```

```
bq query --use_legacy_sql=false 'SELECT max(end_date) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
Waiting on bqjob_r5f150eb9b9d3b965_0000016d6139c7d7_1 ... (0s) Current status: DONE   
+---------------------+
|         f0_         |
+---------------------+
| 2016-08-31 23:48:00 |
+---------------------+
```

- How many bikes are there?

```
bq query --use_legacy_sql=false 'SELECT count(distinct bike_number) FROM `bigquery-public-data.san_francisco.bikeshare_trips`'
Waiting on bqjob_r36dc0f773e296b4a_0000016d613bc3fb_1 ... (0s) Current status: DONE   
+-----+
| f0_ |
+-----+
| 700 |
+-----+
```


2. New Query (Paste your SQL query and answer the question in a sentence):

- How many trips are in the morning vs in the afternoon?

Answer: There were 171k morning trips and 272k afternoon trips in 2018
Assumption: Morning trips start before 12. Afternoon trips start after 12. We do not worry about end times in this case. 

```
bq query -q  --use_legacy_sql=FALSE 'select  Extract(year from start_date) as trip_year, count(*) as am_trip_count from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where  ( Extract(hour from start_date) < 12 ) group by trip_year order by trip_year desc'
+-----------+---------------+
| trip_year | am_trip_count |
+-----------+---------------+
|      2018 |        171144 |
|      2017 |        195925 |
|      2016 |         91774 |
|      2015 |        149812 |
|      2014 |        133514 |
|      2013 |         37239 |
+-----------+---------------+
```
```
bq query -q  --use_legacy_sql=FALSE 'select  Extract(year from start_date) as trip_year, count(*) as pm_trip_count from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where  ( Extract(hour from start_date) >= 12 ) group by trip_year order by trip_year desc'
+-----------+---------------+
| trip_year | pm_trip_count |
+-----------+---------------+
|      2018 |        272927 |
|      2017 |        323775 |
|      2016 |        118720 |
|      2015 |        196440 |
|      2014 |        192825 |
|      2013 |         63324 |
+-----------+---------------+

```

### Project Questions
Identify the main questions you'll need to answer to make recommendations (list
below, add as many questions as you need).

- Question 1: How many peak hour trips were made on any random day by subscribers each year?

- Question 2:  How many peak hour trips were made on any random day by customers each year?

- Question 3: How many bike trips are used for business (weekdays)?

- Question 4: How many bike trips are used for leisure (weekends)? 

- Question 5: How many males are riding the bikes? 

- Question 6: How many females are riding the bikes?

- Question 7: What is the rate of growth of subscribers year over year?

- Question 8: What is the rate of growth of customers year over year?

- Question 9: What is the rate of growth of male riders?

- Question 10: What is the rate of growth of female riders?


### Answers

Answer at least 4 of the questions you identified above. You can use either
BigQuery or the bq command line tool.  Paste your questions, queries and
answers below.

- Question 1: How many peak hour trips were made on any random day by subscribers each year?

  * Answer: There were 1820, 853, 891, 847 and 159 subscriber trips in 2017, 2016, 2015, 2014, 2013 respectively based on the arbitrary week # 35, Wednesday of the week between 6-10 AM and 3-6 PM. Week 35 was picked because it gives the results for the maximum number of years based on the min and max dates.   
  
  * SQL query:
```  
bq query -q --format=csv --use_legacy_sql=FALSE 'select subscriber_type, Extract(year from start_date) as trip_year, count(*) as subscriber_trip_count from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where  (subscriber_type = "Subscriber" and (Extract(week from start_date) = 35 and Extract(dayofweek from start_date) = 3) ) and ( (Extract(hour from start_date) > 6 and Extract(hour from start_date) < 10) or (Extract(hour from start_date) > 15 and Extract(hour from start_date) < 19)) group by subscriber_type, trip_year order by trip_year desc'
+-----------------+-----------+-----------------------+
| subscriber_type | trip_year | subscriber_trip_count |
+-----------------+-----------+-----------------------+
| Subscriber      |      2017 |                  1820 |
| Subscriber      |      2016 |                   853 |
| Subscriber      |      2015 |                   891 |
| Subscriber      |      2014 |                   847 |
| Subscriber      |      2013 |                   159 |
+-----------------+-----------+-----------------------+
```




- Question 2: How many peak hour trips were made on any random day by customers each year?

      * Answer: There were 221,26,39,49 and 117 customer trips in 2017, 2016, 2015, 2014, 2013 respectively based on the arbitrary week # 35, Wednesday of the week between 6-10 AM and 3-6 PM. Again, Week 35 was picked because it gives the results for the maximum number of years based on the min and max dates.   
      
  * SQL query:

```
bq query -q --format=csv --use_legacy_sql=FALSE 'select subscriber_type, Extract(year from start_date) as trip_year, count(*) as customer_trip_count from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where  (subscriber_type = "Customer" and (Extract(week from start_date) = 35 and Extract(dayofweek from start_date) = 3) ) and ( (Extract(hour from start_date) > 6 and Extract(hour from start_date) < 10) or (Extract(hour from start_date) > 15 and Extract(hour from start_date) < 19)) group by subscriber_type, trip_year order by trip_year desc'
+-----------------+-----------+---------------------+
| subscriber_type | trip_year | customer_trip_count |
+-----------------+-----------+---------------------+
| Customer        |      2017 |                 221 |
| Customer        |      2016 |                  26 |
| Customer        |      2015 |                  39 |
| Customer        |      2014 |                  49 |
| Customer        |      2013 |                 117 |
+-----------------+-----------+---------------------+
```

- Question 3: What are the total number of trips made during the weekend each year?

  * Answer: Total number of weekend trips each year is 77k, 96k, 19k, 34k, 40k and 17k in 2018, 2017, 2016, 2015, 2014, 2013 respectively
  
  * SQL query:
  
```
bq query -q  --use_legacy_sql=FALSE 'select EXTRACT(YEAR FROM start_date) as year, count(*) as weekend_trips  from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where Extract(DAYOFWEEK FROM start_date) = 1 or Extract(DAYOFWEEK FROM start_date) = 7 group by year order by year desc'
+------+---------------+
| year | weekend_trips |
+------+---------------+
| 2018 |         77602 |
| 2017 |         96265 |
| 2016 |         19359 |
| 2015 |         34209 |
| 2014 |         40309 |
| 2013 |         17777 |
+------+---------------+
```

- ...

- Question 4: What are the total number of trips made during the weekday each year?

  * Answer:Total number of weekend trips each year is 366k, 423k, 191k, 312k, 286k and 82k in 2018, 2017, 2016, 2015, 2014, 2013 respectively
  * SQL query:
```
bq query -q  --use_legacy_sql=FALSE 'select EXTRACT(YEAR FROM start_date) as year, count(*) as weekday_trips  from `bigquery-public-data.san_francisco_bikeshare.bikeshare_trips` where Extract(DAYOFWEEK FROM start_date) != 1 and Extract(DAYOFWEEK FROM start_date) != 7 group by year order by year desc'
+------+---------------+
| year | weekday_trips |
+------+---------------+
| 2018 |        366469 |
| 2017 |        423435 |
| 2016 |        191135 |
| 2015 |        312043 |
| 2014 |        286030 |
| 2013 |         82786 |
+------+---------------+
```

---

## Part 3 - Employ notebooks to synthesize query project results

### Get Going

Use JupyterHub on your midsw205 cloud instance to create a new python3 notebook. 


#### Run queries in the notebook 

```
! bq query --use_legacy_sql=FALSE '<your-query-here>'
```

- NOTE: 
- Queries that return over 16K rows will not run this way, 
- Run groupbys etc in the bq web interface and save that as a table in BQ. 
- Query those tables the same way as in `example.ipynb`


#### Report
- Short description of findings and recommendations 
- Add data visualizations to support recommendations 

### Resource: see example .ipynb file 

[Example Notebook](example.ipynb)
