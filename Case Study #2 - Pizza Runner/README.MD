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
## 🍝 Solution - A. Pizza Metrics

### 1. How many pizzas were ordered?

````sql
SELECT COUNT(*) AS number_of_ordered_pizza
FROM customer_orders_temp;
````

**Answer:**

| number_of_ordered_pizza |
| ----------------------- |
|       14                |

- Total of orderes is 14.

### 2. How many unique customer orders were made?

````sql
SELECT COUNT(DISTINCT order_id) AS unique_order
FROM customer_orders_temp;
````

**Answer:**

| unique_order |
| ------------ |
| 10           |

- There are 10 unique customer orders.

### 3. How many successful orders were delivered by each runner?

````sql
SELECT runner_id, 
       COUNT(*) AS Number_Successful_orders
FROM runner_orders_temp
WHERE distance IS NOT NULL AND distance != 0
GROUP BY runner_id
ORDER BY runner_id;
````

**Answer:**

| runner_id | number_successful_orders |
| --------- | ------------------------ |
| 1         | 4                        |
| 2         | 3                        |
| 3         | 1                        |

- Runner 1 has 4 successful delivered orders.
- Runner 2 has 3 successful delivered orders.
- Runner 3 has 1 successful delivered order.

### 4. How many of each type of pizza was delivered?

````sql
SELECT pizza_id, 
       COUNT(*) AS Number_Delivered_Pizza
FROM runner_orders_temp 
INNER JOIN customer_orders_temp
USING(order_id)
WHERE distance IS NOT NULL AND distance != 0
GROUP BY pizza_id
ORDER BY pizza_id;
````

**Answer:**

| pizza_id | number_delivered_pizza |
| -------- | ---------------------- |
| 1        | 9                      |
| 2        | 3                      |

- There are 9 delivered Meatlovers pizzas
- There are 3 delivered Vegetarian pizzas.

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
SELECT customer_id, 
	   pizza_name,
       COUNT(pizza_name) AS Number_Ordered_Pizza
FROM customer_orders_temp
INNER JOIN pizza_runner.pizza_names
USING(pizza_id)
GROUP BY pizza_name, customer_id
ORDER BY customer_id;
````

**Answer:**

| customer_id | pizza_name | number_ordered_pizza |
| ----------- | ---------- | -------------------- |
| 101         | Meatlovers | 2                    |
| 101         | Vegetarian | 1                    |
| 102         | Meatlovers | 2                    |
| 102         | Vegetarian | 1                    |
| 103         | Meatlovers | 3                    |
| 103         | Vegetarian | 1                    |
| 104         | Meatlovers | 3                    |
| 105         | Vegetarian | 1                    |

- Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 102 ordered 2 Meatlovers pizzas and 2 Vegetarian pizzas.
- Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 1 Meatlovers pizza.
- Customer 105 ordered 1 Vegetarian pizza.



### 6. What was the maximum number of pizzas delivered in a single order?

````sql
SELECT MAX(Number_Ordered_Pizza) AS Max_Pizza_delivered
FROM
(
  SELECT order_id,
         COUNT(pizza_id) AS Number_Ordered_Pizza
  FROM customer_orders_temp
  INNER JOIN runner_orders_temp
  USING(order_id)
  WHERE distance IS NOT NULL AND distance != 0
  GROUP BY order_id
) AS TP;
````

**Answer:**

| max_pizza_delivered |
| ------------------- |
| 3                   |

- Maximum number of pizza delivered in a single order is 3 pizzas.

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT customer_id,
	   SUM(
           CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
           ELSE 0 END
           ) AS Number_Changeed_Pizza,
           
       SUM(
           CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1
           ELSE 0 END 
           )AS Number_No_Changeed_Pizza 
       
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE distance IS NOT NULL AND distance != 0
GROUP BY customer_id;
````

**Answer:**

| customer_id | number_changeed_pizza | number_no_changeed_pizza |
| ----------- | --------------------- | ------------------------ |
| 101         | 0                     | 2                        |
| 103         | 3                     | 0                        |
| 104         | 2                     | 1                        |
| 105         | 1                     | 0                        |
| 102         | 0                     | 3                        |

- Customer 101 and 102 have no changes.
- Customer 103, 104 and 105 requested at least 1 change (extra or exclusion topping) on their pizza.

### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT COUNT(pizza_id) AS Delivered_Pizza_With_Exclu_And_Extras
FROM customer_orders_temp
INNER JOIN runner_orders_temp
USING(order_id)
WHERE (distance IS NOT NULL AND distance != 0)
	  AND exclusions IS NOT NULL AND extras IS NOT NULL;
````

**Answer:**

| delivered_pizza_with_exclu_and_extras |
| ------------------------------------- |
| 1                                     |

- Only 1 pizza delivered that had both extras and exclusion.

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT DATE_PART('HOUR', order_time) AS Hour,
	   COUNT(pizza_id) AS Total_Ordered_Pizza
FROM customer_orders_temp
GROUP BY DATE_PART('HOUR', order_time)
ORDER BY Hour;
````

**Answer:**

| hour | total_ordered_pizza |
| ---- | ------------------- |
| 11   | 1                   |
| 13   | 3                   |
| 18   | 3                   |
| 19   | 1                   |
| 21   | 3                   |
| 23   | 3                   |

### 10. What was the volume of orders for each day of the week?

````sql
SELECT TO_CHAR(order_time, 'Day') AS Day,
	   COUNT(order_id) AS Total_Orders
FROM customer_orders_temp
GROUP BY TO_CHAR(order_time, 'Day');
````

**Answer:**

| day       | total_orders |
| --------- | ------------ |
| Saturday  | 5            |
| Thursday  | 3            |
| Friday    | 1            |
| Wednesday | 5            |

- There are 5 pizzas ordered on Wednesday and Saturday.
- There are 3 pizzas ordered on Thursday.
- There is 1 pizza ordered on Friday.

***

