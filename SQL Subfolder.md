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
    SUM(menu.price) AS Total_Spent
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY 
	sales.customer_id
ORDER BY sales.customer_id ASC;
````

**Result**
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

**Process**
1. We must join two tables here: sales and menu, so we can extract the price of the products each customer bought from the product id that is in the sales table.
2. We must aggregate the prices of all menu items each customer has bought to get the sum of their spending.
3. Insert an ORDER BY to make this more logical to read.




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
| customer_id | total_sales |
| ----------- | ----------- |
|      A      |     4       |
|      B      |     6       |
|      C      |     2       |

**Process**
1. For each customer we want to know how many unique days they have visited, so we must use both **COUNT** and **DISTINCT**, in order to not count any visits on the same day twice.



## 3. What was the first item from the menu purchased by each customers
## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
## 6. Which item was the most popular for each customer?
## 7. Which item was purchased first by the customer after they became a member?
## 8. Which item was purchased just before the customer became a member?
## 9. What is the total items and amount spent for each member before they became a member?
## 10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
## 11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?








   
