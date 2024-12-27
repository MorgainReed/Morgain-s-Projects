# 🍜 Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

## Relationship Diagram
![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

## Solutions
Please note the SQL dialect used here is PostgreSQL 13 and DB Fiddle is used to write and execute these queries

## 1. What is the total amount each customer spent at the restaurant?
````sql
SELECT
    sales.customer_id,
    SUM(menu.price) AS total_spent
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY 
	sales.customer_id
ORDER BY sales.customer_id ASC;
````

**Result**
| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**Process**
- We must join two tables here: sales and menu, so we can extract the price of the products each customer bought from the product id that is in the sales table.
- We must aggregate the prices of all menu items each customer has bought to get the sum of their spending.
- Insert an ORDER BY to make this more logical to read.

## 2. How many days has each customer visited the restaurant?
````sql
SELECT 
    sales.customer_id, 
    COUNT(DISTINCT sales.order_date) AS Days_Visited
FROM sales
GROUP BY sales.customer_id
ORDER BY sales.customer_id ASC;
````

**Result**
| customer_id | days_visited |
| ----------- | ----------- |
|      A      |     4       |
|      B      |     6       |
|      C      |     2       |

**Process**
- For each customer we want to know how many unique days they have visited, so we must use both **COUNT** and **DISTINCT**, in order to not count any visits on the same day twice.



## 3. What was the first item from the menu purchased by each customers
````sql
SELECT
    s.customer_id,
    m.product_name
FROM sales s
JOIN (
    SELECT
	customer_id,
	MIN(order_date) as earliest_order_date
    FROM sales
    GROUP BY customer_id
)
   AS earliest_orders
ON s.customer_id = earliest_orders.customer_id
AND s.order_date = earliest_orders.earliest_order_date
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id ASC;
````

**Result**
| customer_id | product_name |
| ----------- | ------------ |
|      A      |     curry    |
|      A      |     sushi    |
|      B      |     currry   |
|      C      |     ramen    |

**Process**
- We need a subquery here to grab the earliest date that a customer is entered into the **sales** table and groups this by customer_id. We alias this as "earliest orders"
- The main query joins this subquery to the sales table **AND** filters this to only the dates matching the earliest order date.
- We then want this to be easily readable, so there is a second **JOIN** to find the name of the product from the menu table using the product_id from the sales table.
- We then group by the customer_id and product_name so we only get one answer per customer on their earliest order date.
**We get two rows for customer A as they have ordered both the curry ans sushi on their first date of purchase. The order_date column in the sales table is not a timestamp, only a date, so we cannot conclude that either one of these was the first order, so we must leave both in as both answers are correct.**


## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
## 6. Which item was the most popular for each customer?
## 7. Which item was purchased first by the customer after they became a member?
## 8. Which item was purchased just before the customer became a member?
## 9. What is the total items and amount spent for each member before they became a member?
## 10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
## 11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?








   
