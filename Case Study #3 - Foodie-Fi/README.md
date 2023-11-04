# 🥑 Case Study #3: Foodie-Fi

<img src="https://github.com/Ereh11/8-Week-SQL-Challenge/blob/main/Resources/case3.1.png" width="500" height="520" alt="image">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***

## Business Task
Danny and his friends launched a new startup Foodie-Fi and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world.

This case study focuses on using subscription style digital data to answer important business questions on customer journey, payments, and business performances.

## Entity Relationship Diagram

![image](https://github.com/Ereh11/8-Week-SQL-Challenge/blob/main/Resources/case3.2.png)

***

## Question and Solution


## 🎞️ A. Customer Journey

Here is the 8 sample customers provided in the sample subscriptions table.

**Table: Sample of `subscriptions` table**

<img width="261" src="https://github.com/Ereh11/8-Week-SQL-Challenge/blob/main/Resources/case3.3.png">

- I write a brief description about each customer’s journey.

**Description:**

```sql
SELECT s.customer_id,
	   s.plan_id, 
       start_date
FROM foodie_fi.subscriptions s
INNER JOIN foodie_fi.plans p
USING(plan_id)
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY s.customer_id;
```

| customer_id | plan_id | start_date               |
| ----------- | ------- | ------------------------ |
| 1           | 1       | 2020-08-08T00:00:00.000Z |
| 1           | 0       | 2020-08-01T00:00:00.000Z |
| 2           | 0       | 2020-09-20T00:00:00.000Z |
| 2           | 3       | 2020-09-27T00:00:00.000Z |
| 11          | 4       | 2020-11-26T00:00:00.000Z |
| 11          | 0       | 2020-11-19T00:00:00.000Z |
| 13          | 1       | 2020-12-22T00:00:00.000Z |
| 13          | 0       | 2020-12-15T00:00:00.000Z |
| 13          | 2       | 2021-03-29T00:00:00.000Z |
| 15          | 2       | 2020-03-24T00:00:00.000Z |
| 15          | 0       | 2020-03-17T00:00:00.000Z |
| 15          | 4       | 2020-04-29T00:00:00.000Z |
| 16          | 3       | 2020-10-21T00:00:00.000Z |
| 16          | 0       | 2020-05-31T00:00:00.000Z |
| 16          | 1       | 2020-06-07T00:00:00.000Z |
| 18          | 2       | 2020-07-13T00:00:00.000Z |
| 18          | 0       | 2020-07-06T00:00:00.000Z |
| 19          | 2       | 2020-06-29T00:00:00.000Z |
| 19          | 0       | 2020-06-22T00:00:00.000Z |
| 19          | 3       | 2020-08-29T00:00:00.000Z |

***

## B. Data Analysis Questions

### 1. How many customers has Foodie-Fi ever had?

To determine the count of unique customers for Foodie-Fi, I utilize the `COUNT()` function wrapped around `DISTINCT`.

```sql
SELECT COUNT(DISTINCT customer_id) AS Number_Of_Customers
FROM foodie_fi.subscriptions;
```

**Answer:**

| Number_Of_Customers |
| ----------------- |
| 1000              |

- Foodie-Fi has 1,000 unique customers.

### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

```sql
SELECT DATE_PART('MONTH', start_date) AS Month,
	   COUNT(customer_id) AS Monthly_Distribution
FROM foodie_fi.subscriptions
INNER JOIN foodie_fi.plans
USING(plan_id)
WHERE plan_name = 'trial'
GROUP BY DATE_PART('MONTH', start_date)
ORDER BY Month;
```

**Answer:**

| month | monthly_distribution |
| ----- | -------------------- |
| 1     | 88                   |
| 2     | 68                   |
| 3     | 94                   |
| 4     | 81                   |
| 5     | 88                   |
| 6     | 79                   |
| 7     | 89                   |
| 8     | 88                   |
| 9     | 87                   |
| 10    | 79                   |
| 11    | 75                   |
| 12    | 84                   |

- March has the highest number of trial plans, and February has the lowest number.

### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.

````sql
SELECT p.plan_id,
	   p.plan_name,
	   COUNT(s.customer_id) AS Number_Of_Events
       
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
USING(plan_id)
WHERE s.start_date >= '2021-01-01'
GROUP BY p.plan_id, p.plan_name
ORDER BY plan_id, number_of_events DESC;
````

**Answer:**

| plan_id | plan_name     | number_of_events |
| ------- | ------------- | ---------------- |
| 1       | basic monthly | 8                |
| 2       | pro monthly   | 60               |
| 3       | pro annual    | 63               |
| 4       | churn         | 71               |

### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```sql
SELECT Churned_Customers,
	   ROUND(Churned_Customers / Total * 100, 1) AS Churned_Percentage
FROM 
(
	SELECT SUM(CASE WHEN s.plan_id = 4 THEN 1.0 ELSE 0.0 END) AS Churned_Customers,
  		   COUNT(DISTINCT s.customer_id) * 1.0 AS Total
	FROM foodie_fi.subscriptions AS s
) AS TP;
```

**Answer:**

| churned_customers | churned_percentage |
| ----------------- | ------------------ |
| 307.0             | 30.7               |

- 307 customers have churned. This represents approximately 30.7% of the overall customers.

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

```sql
WITH CTE AS 
(

  SELECT s.customer_id,
  		 plan_name,
  		 LEAD(plan_name) OVER(PARTITION BY s.customer_id ORDER BY start_date) AS Ld
  FROM foodie_fi.subscriptions AS s
  INNER JOIN foodie_fi.plans AS p
  USING(plan_id)
)

SELECT ChurnAfterTrial,
	   ROUND(ChurnAfterTrial / (SELECT COUNT(DISTINCT customer_id) * 1.0 FROM foodie_fi.subscriptions) * 100) AS ChurnAfterTrialPercentage
FROM
(
	SELECT SUM(CASE WHEN plan_name = 'trial' AND Ld = 'churn' THEN 1 ELSE 0 END) AS ChurnAfterTrial
 	FROM CTE
) AS TP;
	   
```

**Answer:**

| churnaftertrial | churnaftertrialpercentage |
| --------------- | ------------------------- |
| 92              | 9                         |


- 92 customers churned after the initial free trial period, representing approximately 9% of the overall customers.

