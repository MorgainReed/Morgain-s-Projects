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
- Firstly, we know that only two columns and one row are needed here: the most popular product and how many times it has been purchased, so we only need select the product name and an aggregate function to count the purchase times of 
  each product.
- We need to make this easy to read, so we will join the menu and the sales table to get the name of the product instead of the product_id.
- To easily find the most purchased item, we can order our result descending by the count of purchases and then use **LIMIT** to restrict the reult to just the first row of results, giving us the most purchased item.


## 5. Which item was the most popular for each customer?
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
- I tend to like to start simple with my queries, look at the data, and then make the query more complex if it is needed. Here, my simplistic thinking is not going to cut it, as 1. Customers may have more than one favorite item, and 2. 
  One customer may come more frequently than another, resulting in a higher purchase count, therefore if I simply order by 'times_purchased' and limit the results to 3, I could come back with just the top 3 dishes purchased by one 
  customer! We need to write a more complex query that will find customers' favorites more dynamically. We will use a **CTE** and the **DENSE RANK() Window Function** for this.


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

## 6. Which item was purchased first by the customer after they became a member?
````SQL
  WITH member_value AS (
  SELECT
  	members.customer_id,
        sales.product_id,
        ROW_NUMBER() OVER (
	      PARTITION BY members.customer_id
	      ORDER BY sales.order_date)
        AS member_row
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND members.join_date < sales.order_date
    )
 
 SELECT 
    member_value.customer_id,
    menu.product_name
 FROM member_value
 JOIN dannys_diner.menu
 ON member_value.product_id = menu.product_id
 WHERE member_row = 1;
````
**Result**
| customer_id | product_name  | 
| ------------| ------------- |
|     A       |    sushi      | 
|     B       |    curry      | 

- Cusomer A purchased sushi right after becoming a member.
- Customer B purchased curry right after becoming a member.
  
**Process**
- Here, our CTE is selecting the columns we need and then joining the sales table with the members table, grabbing only the sales made after the customer has become a member.
- The Window Function **Row_Number()** finds the row number of each purchase, partitioned by the customer_id and ordered by sale date, which then allows us to later find the purchases made only after the customer becomes a member.
- The outer query grabs the customer_id and product_id, joining the CTE and the menu tables to return product_id, and then the WHERE statement filters for only the first purchase after the customer becomes a memeber.

## 7. Which item was purchased just before the customer became a member?
````SQL
WITH purchase_before_member AS (
  SELECT
	members.customer_id,
        sales.product_id,
        ROW_NUMBER() OVER (
	      PARTITION BY members.customer_id
	      ORDER BY sales.order_date DESC)
        AS desc_purchase_rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND members.join_date > sales.order_date
    )
 
 SELECT 
    purchase_before_member.customer_id,
    menu.product_name
 FROM purchase_before_member
 JOIN dannys_diner.menu
 ON purchase_before_member.product_id = menu.product_id
 WHERE desc_purchase_rank = 1;
````
**Result**
| customer_id | product_name  | 
| ------------| ------------- |
|     B       |    sushi      | 
|     A       |    sushi      | 

- Both customer A and B bought sushi before they joined as members.

**Process**
- This query is very similar to query #6, so the process is nearly the same, but we are looking for the purchases made before the member join date instead of after.
- Here, our CTE is selecting the columns we need and then joining the sales table with the members table, grabbing only the sales made before the customer has become a member.
- The Window Function **Row_Number()** finds the row number of each purchase, partitioned by the customer_id and in descending order by date so at the top of our list (i.e. rank #1) will be the last purchase before the customer was 
  converted to a member.
- The outer query grabs the customer_id and product_id, joining the CTE and the menu tables to return product_id, and then the WHERE statement filters for only the first purchase before the customer becomes a memeber.

## 8. What is the total items and amount spent for each member before they became a member?
````SQL
SELECT
   sales.customer_id
  ,COUNT(menu.product_name) AS total_items
  ,SUM(menu.price) AS total_spent

FROM dannys_diner.sales
INNER JOIN dannys_diner.members
   ON   sales.customer_id = members.customer_id
   AND  sales.order_date < members.join_date

INNER JOIN dannys_diner.menu
   ON sales.product_id = menu.product_id

GROUP BY sales.customer_id
ORDER BY sales.customer_id;
````
**Result**
| customer_id | total_items | total_spent |
| ------------| ----------- | ----------- |
|     A       |      2      |      25     |
|     B       |      3      |      40     |

- Customer A bought two items totaling $25 before becoming a member.
-  Customer B bought 3 items totaling $40 before becoming a member.

**Process**
- Select the columns we need, which here is just the customer ID, count of the total items each customer bought, and the total they spent. We use simple aggregate functions **COUNT** and **SUM** here to accomplish this.
- We then need to filter down to only purchases made before the customer joined as a member. In order to do this, we join the members table with the sales table on customer_id and ensure the purchase date returned is before thier 
  membership join date.
- In order to return values from the menu table, we also need to join the sales table and menu table based on product_id.
- We then want to group and order by customer_id to get the summarized table which is out end product.
 
## 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````SQL
WITH points_table AS (
SELECT 
     menu.product_id
    ,CASE 
     WHEN menu.product_name = 'sushi' THEN menu.price * 20
     ELSE menu.price * 10 
     END
  	 AS points
 FROM dannys_diner.menu
 )
  
 SELECT 
  	sales.customer_id
  	,sum(points_table.points)
 FROM points_table

 INNER JOIN dannys_diner.sales
   ON sales.product_id = points_table.product_id

 GROUP BY sales.customer_id
 ORDER BY sales.customer_id;
````

**Result**
| customer_id |   points  |
| ------------| --------- |
|     A       |    860    |
|     B       |    940    |
|     C       |    360    |

- Customer A has 860 points.
- Customer B has 940 points.
- Customer C has 360 points.

**Process**
- First, lets create a CTE for points to sum up how many points each customer has collected. It is important to remember that we must use the 2x multiplier for sushi only, so we use a **CASE WHEN** statement for this. 
- Next, we just need the customer_id and the sum of points associated with each customer, so we select these columns, join the sales table and points_table based on product_id, and group by customer_id.
- We then order by customer_id for readability, and get our results table.

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````SQL
WITH time_period AS (
  SELECT
     members.customer_id
    ,members.join_date
    ,members.join_date + 6 as first_week_ends
    ,DATE_TRUNC ('month', '2021-1-31'::DATE)
  		+ INTERVAL '1 month'
  		- INTERVAL '1 day' AS last_jan_date
  
   FROM dannys_diner.members
   )
 
 SELECT
 	sales.customer_id
    ,SUM(CASE 
         WHEN menu.product_name = 'sushi' THEN menu.price * 20
         WHEN sales.order_date BETWEEN time_period.join_date
			       AND time_period.first_week_ends
			       THEN menu.price * 20
         ELSE menu.price * 10
         END) AS points
  FROM dannys_diner.sales
  
  INNER JOIN time_period
    ON sales.customer_id = time_period.customer_id
    AND sales.order_date >= time_period.join_date
    AND sales.order_date <= time_period.last_jan_date
  
  INNER JOIN dannys_diner.menu
  	ON sales.product_id = menu.product_id
  
  GROUP BY sales.customer_id
  ORDER BY sales.customer_id;
  ````
**Result**
| customer_id |   points  |
| ------------| --------- |
|     A       |    1020   |
|     B       |    320    |

- Customer A collected 1020 points in January.
- Customer B collected 320 points in January.

**Process**
- First, we have a specific time period to look at, so we should create a CTE for these dates. We do this by selecting customer_ID, their join date, adding 6 days to their join date to get the last day of the 'double 
  points period', and then use the DATE_TRUNC window function to find and return the last day of January 2021. This completes the dates we need and therefore what we need in our time_period CTE.
- Next, we select customer_id and the sum of the points these customers have collected. We need to specify how many points are collected in certain cases, so we use CASE WHEN to define these. We must define that sushi always gets 
  double points, everything gets double points during the first week of membership, and outside of that first week of membership, everything but sushi gets 1x points.
- We then have to join our tables to filter down results. The first join puts together the sales table and the time_period CTE based on customer_id, sales.order_date must be greater than or equal to the member's join_date, and less 
  than or equal to the last day of January.
- We then join the sales table to the menu based on product_id so that we can use the price column to calculate points, as we did in the CASE WHEN statement.
- Finally, we group and order by customer_id to return the sum of points and maximize readability of the table.










   
