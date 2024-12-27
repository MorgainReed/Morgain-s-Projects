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

**Process**
1. We must aggregate the prices of all menu items each customer has bought to get the sum of their spending.
2. We must join two tables here: sales and menu, so we can extract the price of the products each customer bought from the product id that is in the sales table.
