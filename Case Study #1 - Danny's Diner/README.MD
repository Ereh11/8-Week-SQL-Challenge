# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="1000" height="500">

## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***

## Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

***

## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

***

## Question and Solution

**1. What is the total amount each customer spent at the restaurant?**

````sql
SELECT
      customer_id,
      SUM(price) AS Total_Spent
FROM dannys_diner.menu
INNER JOIN dannys_diner.sales
USING(product_id)
GROUP BY customer_id
ORDER BY customer_id; 
````
#### Answer:
| customer_id | Total_Spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

**2. How many days has each customer visited the restaurant?**

````sql
SELECT
	customer_id,
  	COUNT(DISTINCT order_date) AS Number_visited
FROM  dannys_diner.sales
GROUP BY customer_id;
````
#### Answer:
| customer_id | Number_visited |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***

**3. What was the first item from the menu purchased by each customer?**

````sql
WITH Rank_orders AS 
(
	SELECT
          s.customer_id,
          m.product_name,
  	DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS Rank
	FROM  dannys_diner.sales AS s
	INNER JOIN dannys_diner.menu AS m
	ON s.product_id = m.product_id
)

SELECT 	customer_id,
        product_name
FROM Rank_orders
WHERE Rank = 1
GROUP BY customer_id, product_name
````
#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first order has two orders curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.
***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

````sql
SELECT  product_name,
        COUNT(product_id) AS Number_OF_Purchases
FROM dannys_diner.menu AS m
INNER JOIN dannys_diner.sales AS s
USING(product_id)
GROUP BY product_name
ORDER BY Number_OF_Purchases DESC
LIMIT 1;
````
#### Answer:
| product_name | Number_OF_Purchases | 
| ----------- | ----------- |
| ramen       | 8 |


- Most purchased item on the menu is ramen with 8 times!

***

**5. Which item was the most popular for each customer?**

````sql
WITH Ranked_Popular_Item AS
(
  SELECT  customer_id,
	 	  product_name,
		  COUNT(product_id) AS Number_OF_Purchase,
  		  DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_id) DESC) AS Rank
  
  FROM dannys_diner.menu AS m
  INNER JOIN dannys_diner.sales AS s
  USING(product_id)
  GROUP BY customer_id, product_name
  ORDER BY customer_id, Number_OF_Purchase DESC
)


SELECT customer_id,
       product_name,
       Number_OF_Purchase

FROM Ranked_Popular_Item
WHERE Rank = 1;
````
#### Answer:
| customer_id | product_name | Number_OF_Purchase |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A's favourite item is ramen.
- Customer B enjoys all items on the menu (sushi, curry, ramen).
- Customer C's favourite item is ramen.

***

**6. Which item was purchased first by the customer after they became a member?**

```sql
WITH First_Item AS
(
  SELECT  m.product_name,
  	  mb.customer_id,
	  DENSE_RANK() OVER(PARTITION BY mb.customer_id ORDER BY order_date) AS Rank
  
  FROM dannys_diner.members AS mb
  INNER JOIN dannys_diner.sales AS s
  USING(customer_id)
  INNER JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
  WHERE order_date > join_date
)

SELECT  customer_id,
		product_name
FROM First_Item
WHERE Rank = 1;
```
#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order as is ramen.
- Customer B's first order as is sushi.

***

**7. Which item was purchased just before the customer became a member?**

````sql
WITH First_Item_Before_Join AS
(
  SELECT  m.product_name,
  	  mb.customer_id,
	  DENSE_RANK() OVER(PARTITION BY mb.customer_id ORDER BY order_date DESC) AS Rank
  
  FROM dannys_diner.members AS mb
  INNER JOIN dannys_diner.sales AS s
  ON mb.customer_id = s.customer_id AND s.order_date < mb.join_date
  INNER JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id

)

SELECT  customer_id,
	product_name
FROM First_Item_Before_Join
WHERE Rank = 1;
````
#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Customer A have two as first orders sushi and curry.
- Customer B's first order as is sushi.

***

**8. What is the total items and amount spent for each member before they became a member?**

```sql
SELECT  mb.customer_id,
	COUNT(s.product_id) AS Total_items,
  	SUM(m.price) AS Amount_Spent
FROM dannys_diner.members AS mb
INNER JOIN dannys_diner.sales AS s
ON mb.customer_id = s.customer_id AND s.order_date < mb.join_date
INNER JOIN dannys_diner.menu AS m
ON s.product_id = m.product_id
GROUP BY mb.customer_id
ORDER BY customer_id
```
#### Answer:
| customer_id | Total_items | Amount_Spent |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?**

```sql
SELECT customer_id,
	SUM(
         	CASE WHEN product_name = 'sushi' THEN price * 20
         	ELSE price * 10 END
       	  ) AS Total_Points
       
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
USING(product_id)
GROUP BY customer_id
ORDER BY customer_id;
```
#### Answer:
| customer_id | Total_Points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is 860 points.
- Total points for Customer B is 940 points.
- Total points for Customer C is 360 points.

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customers A and B have at the end of January?**

```sql
SELECT mb.customer_id,
	SUM(
         	CASE WHEN s.order_date BETWEEN join_date AND join_date + INTERVAL '6 days' THEN price * 20
         	WHEN product_name = 'sushi' THEN price * 20
         	ELSE price * 10 END
       	  ) AS Total_Points
       
FROM dannys_diner.sales AS s
INNER JOIN dannys_diner.menu AS m
ON s.product_id = m.product_id
INNER JOIN dannys_diner.members AS mb
ON s.customer_id = mb.customer_id
WHERE LEFT(CAST(order_date AS VARCHAR), 7) = '2021-01'

GROUP BY mb.customer_id
ORDER BY mb.customer_id;

```

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1370 points.
- Total points for Customer B is 820 points.

***
## BONUS QUESTIONS

**Join All The Things**

**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

```sql
SELECT s.customer_id,
	   s.order_date,
	   m.product_name,
       m.price,
       CASE
       		WHEN mb.join_date <= s.order_date THEN 'Y'
    		ELSE 'N' 
       END AS member
FROM dannys_diner.sales AS s
LEFT JOIN dannys_diner.members AS mb
ON mb.customer_id = s.customer_id
INNER JOIN dannys_diner.menu AS m
ON s.product_id = m.product_id
ORDER BY mb.customer_id, s.order_date, m.product_name;
```
 
#### Answer: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

***
**Rank All The Things**

**Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.**

```sql
WITH Add_Ranking AS
(
  SELECT s.customer_id,
         s.order_date,
         m.product_name,
         m.price,
         CASE
              WHEN mb.join_date <= s.order_date THEN 'Y'
              ELSE 'N' 
         END AS member
  FROM dannys_diner.sales AS s
  LEFT JOIN dannys_diner.members AS mb
  ON mb.customer_id = s.customer_id
  INNER JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
  ORDER BY mb.customer_id, s.order_date, m.product_name
 )
 
 SELECT *,
        CASE 
        	WHEN member = 'N' THEN null
            ELSE  DENSE_RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
        END AS ranking
 FROM Add_Ranking
```
#### Answer:
| customer_id | order_date               | product_name | price | member | ranking |
| ----------- | ------------------------ | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |  null   |
| A           | 2021-01-01T00:00:00.000Z | sushi        | 10    | N      |  null   |
| A           | 2021-01-07T00:00:00.000Z | curry        | 15    | Y      | 1       |
| A           | 2021-01-10T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01T00:00:00.000Z | curry        | 15    | N      |  null   |
| B           | 2021-01-02T00:00:00.000Z | curry        | 15    | N      |  null   |
| B           | 2021-01-04T00:00:00.000Z | sushi        | 10    | N      |  null   |
| B           | 2021-01-11T00:00:00.000Z | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16T00:00:00.000Z | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01T00:00:00.000Z | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |  null   |
| C           | 2021-01-01T00:00:00.000Z | ramen        | 12    | N      |  null   |
| C           | 2021-01-07T00:00:00.000Z | ramen        | 12    | N      |  null   |

***
