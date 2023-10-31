# 🍕 Case Study #2 - Pizza Runner

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
