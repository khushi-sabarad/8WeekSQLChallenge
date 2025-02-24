# Case Study #1 - Danny's Diner🍜
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="400" height="410">

 ***

## Introduction
Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: 🍣sushi, 🍛 curry and 🍜ramen.

## Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. This deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers. 

**`ERD(Entity Relationship Diagram)` of the 3 datasets Danny shared:** 

<img width="500" alt="case1 ERD" src="https://github.com/khushi-sabarad/8-Week-SQL-Challenge/assets/71957748/3ae86d1c-6be2-497f-8334-f6e9a1328b4e">

[Click here](https://8weeksqlchallenge.com/case-study-1/) to learn more about the case study in detail.

***
## Questions with Solutions
(Tool used: MYSQL 8.0 Command Line Client) 

1. What is the total amount each customer spent at the restaurant?
```sql
SELECT s.customer_id, SUM(m.price) AS total_amount_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```
- `SELECT` specifies the columns to retrieve from the database.
- `SUM`, an `aggregate function`, is used to calculate the total amount spent by each customer.
- `ALIAS` is used to give tables/columns temporary names, making the query more concise and readable. In this case, 'sales' and 'menu' are aliased as 's' and 'm', respectively. Additionally, 'SUM(m.price)' is aliased as 'total_amount_spent'.
-  `JOINS`: The default join in SQL is an `INNER JOIN`, which retrieves records that have matching values in both tables being joined based on a specific condition.
-  `GROUP BY` groups rows that have the same values, in this case, grouping by customer_id.


The total amount spent at the restaurant by:
  - Customer A is $76
  - Customer B is $74
  - Customer C is $36

    
***

2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS days_visited
FROM sales
GROUP BY customer_id;
```
- `COUNT`(DISTINCT order_date): This function calculates the total number of unique days each customer visited the diner. By using `DISTINCT`, it ensures that if a customer visits the diner multiple times on the same day, it counts as one visit for that day instead of being counted multiple times.

Number of days visited by:
  - Customer A is 4
  - Customer B is 6
  - Customer C is 2

***

3. What was the first item from the menu purchased by each customer?
```sql
SELECT s.customer_id, 
  (SELECT m.product_name
  FROM sales s_sub
  JOIN menu m ON s_sub.product_id = m.product_id
  WHERE s_sub.customer_id = s.customer_id
  ORDER BY s_sub.order_date
  LIMIT 1) AS first_item
FROM (SELECT DISTINCT customer_id FROM sales) as s;
```
- Main Query [SELECT s.customer_id, (...) AS first_item]:  
Selects the customer ID and an alias representing the first purchased item for each customer.

- Subquery [(SELECT DISTINCT customer_id FROM sales) as s]:   
  Creates a subquery to select unique customer IDs from the sales table and aliases it as 's', without this, there will be multiple rows for each customer.
  
- Nested Subquery [(SELECT m.product_name .....LIMIT 1)]:  
  Within each row of the main query, this nested subquery selects the product name of the first purchase for the corresponding customer ID.
  - `WHERE` filters out rows based on specific conditions
  - `ORDER BY` sorts the resulting set in an order based on a specific column/expression. The default is ascending order.
  - `LIMIT 1` restricts the result to only the first row, which represents the first item purchased by the customer.
  
Note: As the dataset lacks specific DateTime information, I am assuming that the first item ordered corresponds to the entry in the first row of the dataset. Here, Customer A placed orders for both curry and sushi on day 1, each order is counted as a separate visit, and the item listed first in the order is considered the first ordered item.

The first item, from the menu, purchased by:
  - Customer A was Sushi
  - Customer B was Curry
  - Customer C was Ramen

***

4. What is the most purchased item on the menu and how many times did all customers purchase it?
```sql
SELECT m.product_name AS most_purchased_item,
    COUNT(*) AS purchase_count,
    CONCAT(COUNT(*), '/', (SELECT COUNT(*) FROM sales)) AS purchase_fraction
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_count DESC
LIMIT 1;
```
- `COUNT(*)` returns the number of rows. Each row represents a purchase.
- to get a deeper understanding, I used `CONCAT` to concatenate the purchase_count with the total number of purchases in the sales table, displaying it as a fraction.
- I ordered the results in descending order to prioritize the most purchased item and then applied LIMIT 1 to display only the top row.

The most purchased item is Ramen, bought 8 out of 15 times.

***

5. Which item was the most popular for each customer?
```sql
SELECT customer_id,
    GROUP_CONCAT(most_popular_item) AS most_popular_items,
    max_purchase_count
FROM
    ( SELECT s.customer_id, m.product_name AS most_popular_item,
      COUNT(*) AS purchase_count,
      MAX(COUNT(*)) OVER (PARTITION BY s.customer_id) AS max_purchase_count
    FROM sales s
    JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
    ) AS subquery
WHERE purchase_count=max_purchase_count
GROUP BY customer_id, max_purchase_count;
```
- `GROUP_CONCAT()` combines the most popular items purchased by each customer into a single string. Here, Customer B has ordered all three items twice. Instead of having three separate rows for one customer, I've utilized GROUP_CONCAT() to condense the information. It's important to note that using GROUP_CONCAT() in database storage may violate `database normalization` rules. However, for presentation purposes, it enables us to display the data in a single, more readable row.

I am assuming that the customer's most popular item would be the one they ordered the most. 
  - The most popular item is Ramen, which was bought three times each by both Customer A and C.
  - Customer B bought all three items twice, indicating either a lack of specific preference or an equal liking for all items

***

6. Which item was purchased first by the customer after they became a member?
```sql
SELECT s.customer_id,
    mem.join_date,
    s.order_date,
    m.product_name
FROM  sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON mem.customer_id = s.customer_id
WHERE s.order_date = (
        SELECT MIN(order_date)
        FROM sales
        WHERE customer_id = s.customer_id
          AND order_date > mem.join_date )
ORDER BY s.customer_id;

```
Customers A & B took membership in the diner in January 2021.
After they became members, the first item bought by:
  - Customer A was Ramen
  - Customer B was Sushi
***
7. Which item was purchased just before the customer became a member?
```sql
SELECT s.customer_id,
    mem.join_date,
    s.order_date,
    m.product_name
FROM  sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON mem.customer_id = s.customer_id
WHERE s.order_date = (
        SELECT MAX(order_date)
        FROM sales
        WHERE customer_id = s.customer_id
          AND order_date < mem.join_date )
ORDER BY s.customer_id;
```

Just before becoming members:
  - Customer A bought Curry
  - Customer B bought Sushi
  
***
8. What are the total items and amount spent for each member before they became a member?
```sql
SELECT 
    s.customer_id,
    GROUP_CONCAT(menu.product_name ORDER BY s.order_date DESC) AS ordered_items,
    COUNT(s.product_id) AS total_items, 
    SUM(menu.price) AS total_sales
FROM sales s
JOIN members m ON s.customer_id = m.customer_id
JOIN menu ON s.product_id = menu.product_id
WHERE s.order_date < m.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

Before they became members:
  - Customer A bought 2 items, curry & sushi, spending $25
  - Customer B bought 3 items, curry twice & sushi, spending $40

***
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT s.customer_id,
        SUM(IF(m.product_name = 'sushi', (m.price * 2) * 10, m.price * 10)) AS total_points
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

- `IF()` checks if the product name is sushi, if it is, it multiplies the price by 20 (since each $1 spent = 10 points, and sushi has a 2x points multiplier); if it's not sushi, it multiplies the price by 10.

  - Customer A has 860 points
  - Customer B has 940 points
  - Customer C has 360 points

***
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of January?

```sql
WITH cte_dates AS (
    SELECT *,
           DATE_ADD(join_date, INTERVAL 7 DAY) AS join_week,
           LAST_DAY('2021-01-31') AS jan_end
    FROM members
)
SELECT
    s.customer_id,
    SUM(
        CASE
            WHEN m.product_name = 'sushi' OR s.order_date BETWEEN cte.join_date AND cte.join_week THEN m.price * 20
            ELSE m.price * 10
        END
    ) AS points_january
FROM cte_dates cte
JOIN sales s ON cte.customer_id = s.customer_id
JOIN menu m ON m.product_id = s.product_id
WHERE s.order_date < cte.jan_end
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
- `Common Table Expression (CTE)` is like creating a temporary table but only for the duration of the query. It allows us to define a named subquery using a `WITH` clause at the beginning of the query, which we can later refer to within the main query, making complex queries more readable and easier to manage. Once the query execution is complete, the CTE is discarded.
  Here, 'cte_dates' selects all information from the 'members' table and adds two more columns: join_week (represents the last day of the first week after a customer becomes a member) and jan_end (represents the end of January)

- `SUM()` calculates the total points based on the conditions specified in the `CASE` statement:
    - If the product_name is 'sushi' `OR` the order date is within the first week after joining, it multiplies the price by 20. Otherwise, it multiplies the price by 10.
  

Assuming the points are not spent, by the end of January:
  - Customer A has 1370 points
  - Customer B has 940 points

***
## Bonus Questions
Generate some basic datasets so Danny's team can easily inspect the data without needing to use SQL.

**1. Join All The Things:** Generate a table with customer_id, order_date, product_name, price, member (Y/N)
```sql
SELECT
    s.customer_id,
    s.order_date,
    menu.product_name,
    menu.price,
    CASE
        WHEN m.join_date  <= s.order_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM sales s
LEFT JOIN members m ON s.customer_id = m.customer_id
JOIN menu ON s.product_id = menu.product_id
ORDER BY
    s.customer_id,
    s.order_date,
    menu.product_name;
```
- `LEFT JOIN` ensures that all records from the sales table (the left table in the join) are included in the result set, regardless of whether there is a matching record in the "members" table (the right table in the join). If there is no matching record in the members table, the result set will include all records from the sales table and the corresponding columns from the "members" table will be NULL.

***
**2. Rank All The Things:** Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases. Hence, he expects null ranking values for the records when customers are not yet part of the loyalty program.

```sql
WITH member_data AS (
  SELECT
    s.customer_id,
    s.order_date,
    menu.product_name,
    menu.price,
    CASE
        WHEN m.join_date  <= s.order_date THEN 'Y'
        ELSE 'N'
    END AS member
FROM sales s
LEFT JOIN members m ON s.customer_id = m.customer_id
JOIN menu ON s.product_id = menu.product_id
ORDER BY
    s.customer_id,
    s.order_date,
    menu.product_name
)

SELECT *, 
  CASE
    WHEN member = 'N' then NULL
    ELSE RANK () OVER (
      PARTITION BY customer_id, member ORDER BY order_date) END AS ranking
FROM member_data;
```
- All columns are selected from the defined CTE (member_data) and an additional column for ranking is added.
- `Window functions` operate on a set of rows, called a "window," within the result set of a query. Unlike aggregate functions like SUM() or AVG(), which collapse multiple rows into a single value, window functions return a value for each row based on a specific calculation over the window of rows. Here, the `RANK()` is a window function.
    - The RANK() function assigns a rank to each row based on its position within a partition, which is defined by the `PARTITION BY` clause. Within each partition, rows are ordered by the `ORDER BY` clause. Ties in ranking result in the same rank for affected rows, with the next row receiving the next sequential rank.

- If the "member" column is 'N', indicating that the customer was not a member at the time of the order, we assign NULL to the ranking column.

***
Let's connect on [LinkedIn!](https://www.linkedin.com/in/khushi-sabarad/)🤝
