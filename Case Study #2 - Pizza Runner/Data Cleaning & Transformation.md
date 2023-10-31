# 🍕 Case Study #2 - Pizza Runner

## 🔧 Data Cleaning & Transformation

### 🔨 Table: customer_orders

Looking at the `customer_orders` table below, we can see that there are
- In the `exclusions` column, there are missing/blank spaces and null values. 
- In the `extras` column, there are missing/blank spaces and null values.

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       | null   | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       | null   | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       | null   | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

We clean the table as following:
- Create a temporary table with all the columns
- Remove all missing/blank spaces values in `exlusions` and `extras` columns and replace with null.

````sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
	  WHEN (exclusions = '' OR exclusions LIKE 'null') THEN NULL
	  ELSE exclusions
	  END AS exclusions,
  CASE
	  WHEN (extras = '' OR extras LIKE 'null') THEN NULL
	  ELSE extras
	  END AS extras,
	order_time
FROM pizza_runner.customer_orders;
`````

This is how the clean `customers_orders_temp` table looks like after cleaning.

| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        | null       |    null| 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        | null       |    null| 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        | null       |    null| 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        | null       |    null| 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |    null| 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |    null| 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |    null| 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        | null       | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        | null       |    null| 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        | null       | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        | null       |    null| 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        | null       |    null| 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

***

### 🔨 Table: runner_orders

Looking at the `runner_orders` table below, we can see that there are
- In the `pickup_time` column, there are null values. 
- In the `distance` column, there are "km" after the number of distance
- In the `duration` column, there are "minutes", "minute", and "mins" after the number of duration. 
- In the `cancellation` column, there are missing/ blank spaces.

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

We clean the table as following:
- Create a temporary table with all the columns
- In `distance` column, remove "km".
- In `duration` column, remove "minutes", "minute", "mins".
- In `cancellation` column, remove missing/ blank spaces and replace with blank space null.

````sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT 
  order_id, 
  runner_id,  
  CASE
	  WHEN (pickup_time LIKE 'null' OR pickup_time = '') THEN NULL
	  ELSE pickup_time
	  END AS pickup_time,
  CASE
	  WHEN (distance LIKE 'null' OR distance = '') THEN NULL
	  WHEN distance LIKE '%km' THEN TRIM('km' from distance)
	  ELSE distance 
    END AS distance,
  CASE
	  WHEN (duration LIKE 'null' OR duration = '') THEN NULL
	  WHEN duration LIKE '%mins' THEN TRIM('mins' from duration)
	  WHEN duration LIKE '%minute' THEN TRIM('minute' from duration)
	  WHEN duration LIKE '%minutes' THEN TRIM('minutes' from duration)
	  ELSE duration
	  END AS duration,
  CASE
	  WHEN (cancellation LIKE 'null' OR cancellation = '') THEN NULL
	  ELSE cancellation
	  END AS cancellation
FROM pizza_runner.runner_orders;
````

Then, we alter the `pickup_time`, `distance` and `duration` columns to the correct data type.

````sql
ALTER TABLE runner_orders_temp
ALTER COLUMN pickup_time TYPE TIMESTAMP USING(CAST(pickup_time AS TIMESTAMP)),
ALTER COLUMN distance TYPE FLOAT USING(CAST(distance AS FLOAT)),
ALTER COLUMN duration TYPE INT USING(CAST(duration AS INT));
````

This is how the clean `runner_orders_temp` table looks like after cleaning.

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |    null                     |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |    null                   |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |   null                      |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |    null                     |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |    null                     |
| 6        | 3         |           null          |   null       |    null      | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |  null                       |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |  null                       |
| 9        | 2         |      null               |  null        |    null      | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |   null                      |

***

