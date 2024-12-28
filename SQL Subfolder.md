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
- Grouping this is crutial as we do not want a **COUNT** for each entry in the sales table.

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
**We get two rows for customer A as they have ordered both the curry and the sushi on their first date of purchase. The order_date column in the sales table is not a timestamp, only a date, so we cannot conclude that either one of these was the first order, therefore we leave both in as both are correct.**


## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
````SQL
SELECT
  menu.product_name
  ,COUNT(sales.product_id)
 FROM sales
 JOIN menu
 ON menu.product_id = sales.product_id
 GROUP BY menu.product_name
 ORDER BY COUNT(sales.product_id) DESC
 LIMIT 1
````
**Result**
| product_name | times_purchased |
| ------------ | --------------- |
|      ramen   |    8            |

**Process**
- Firstly, we know that only two columns and one row are needed here: the most popular product and how many times it has been purchased, so we only need select the product name and an aggregate function to count the purchase times of each product.
- We need to make this easy to read, so we will join the menu and the sales table to get the name of the product instead of the product_id.
- To easily find the most purchased item, we can order our result descending by the count of purchases and then use **LIMIT** to restrict the reult to just the first row of results, giving us the most purchased item.


## 6. Which item was the most popular for each customer?
MY FIRST ATTEMPT: 
````SQL
SELECT
   sales.customer_id,
   menu.product_name,
   COUNT(sales.product_id) AS times_purchased
 FROM sales
 JOIN menu
 ON sales.product_id = menu.product_id
 GROUP BY 
   sales.customer_id,
   menu.product_name
 ORDER BY times_purchased DESC
 LIMIT 3;
````
- I tend to like to start simple with my queries, look at the data, and then make the query more complex if it is needed. Here, my simplistic thinking is not going to cut it, as 1. cutomers may have more than one favorite item, and 2. one customer may come more frequently than another, resulting in a higher purchase count, therefore if I simply order by 'times_purchased' and limit the results to 3, I could come back with just the top 3 dishes purchased by one customer! We need to write a more complex query that will find customers' favorites more dynamically. We will use a **CTE** and the **DENSE RANK() Window Function** for this.


````SQL
WITH ranked_favorites AS (
  SELECT
     sales.customer_id,
     menu.product_name,
     COUNT(sales.product_id) AS times_purchased,
   	DENSE_RANK() OVER (
    PARTITION BY sales.customer_id
    ORDER BY COUNT(sales.product_id) DESC) AS ranks
 FROM menu
 JOIN sales
 ON menu.product_id = sales.product_id
 GROUP BY 
   sales.customer_id,
   menu.product_name
)

  SELECT
     customer_id,
     product_name,
     times_purchased 
  FROM ranked_favorites
  WHERE ranks = 1;
````
**Result**
| customer_id | product_name  |  times_purchased |
| ------------| ------------- | -----------------|
|     A       |    ramen      |    3             |
|     B       |    ramen      |    2             |
|     B       |    curry      |    2             |
|     B       |    sushi      |    2             |
|     C       |    ramen      |    3             |

**Process**
- Here, the CTE joined the tables menu and sales to give us the columns we need, then groups by customer_id and product_name, to return a table with the customer, the products they 
  bought, and a count of how many times they bought these items.
- The Window Function **DENSE RANK()** then assigns a rank to each item based on the number of times it was bought, and **PARTITION BY** allows these ranks to be assigned per customer, instead of over the whole data set.
- It is worth noting that I chose **DENSE RANK()** as it allows for ties, where **RANK()** does not, which is important as customer B has 3 favorite items!

## 7. Which item was purchased first by the customer after they became a member?
## 8. Which item was purchased just before the customer became a member?
## 9. What is the total items and amount spent for each member before they became a member?
## 10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
## 11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?








   
