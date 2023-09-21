# CLOTHING-COMPANY

Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!
The CEO of this trendy fashion company has asked me to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

## Available Data
2 Datasets are used here to analyse and bring business solutions 

### Product Details
product_details includes all information about the entire range that Clothing brand sells in their store.

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/d1d8d4b7-4b1c-4829-8ad5-5bf737bb6dad)
### Product Sales
Product sales contains product level information for all the transactions made for the brand including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp.

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/c39152f6-42df-4ad5-b510-097707711380)

## 📚 Table of Contents
- [Case Study Questions](#case-study-questions)
- [Product Analysis](#product-analysis)

***
## Case Study Questions

The following questions can be considered key business questions and metrics that the Clothing brand team requires for their monthly reports.

## High Level Sales Analysis
### 1. What was the total quantity sold for all products?

```sql
SELECT
product_name, 
SUM(qty) AS Total_qty_sold
FROM balanced_tree_clothing_company.`product details` AS a JOIN 
balanced_tree_clothing_company.productsales AS b 
ON a.product_id = b.prod_id
GROUP BY prod_id
ORDER BY Total_qty_sold DESC
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/b906b2d1-4a5a-4c33-b6f8-3f97f4eed0e7)

### 2.	What is the total generated revenue for all products before discounts?

```sql
SELECT
product_name, 
SUM(b.price)*SUM(qty) AS Total_generated_revenue
FROM balanced_tree_clothing_company.`product details` AS a JOIN 
balanced_tree_clothing_company.productsales AS b 
ON a.product_id = b.prod_id
GROUP BY prod_id
ORDER BY Total_generated_revenue DESC
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/830909ed-62db-4caa-aec8-b74b7a1feb83)

### 3.	What was the total discount amount for all products?

```sql
SELECT 
product_name, 
ROUND(SUM(b.price*b.qty*b.discount/100),0) AS Total_discount
FROM balanced_tree_clothing_company.`product details` AS a JOIN 
balanced_tree_clothing_company.productsales AS b 
ON a.product_id = b.prod_id
GROUP BY product_name
ORDER BY Total_discount DESC

```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/a2cfc70d-1ab2-4e2c-a480-07335c721521)

***
## Transaction Analysis

### 1.	How many unique transactions were there?
```sql
SELECT 
COUNT(DISTINCT(txn_id)) AS unique_transactions
FROM balanced_tree_clothing_company.productsales
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/04359513-ab33-4602-a89e-3959b35ef7ea)

### 2.	What is the average unique products purchased in each transaction?
```sql
SELECT
  ROUND(COUNT(prod_id) / COUNT(distinct txn_id),0) AS avg_number_of_product_in_order
FROM
  balanced_tree_clothing_company.productsales
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/e815e238-106b-439f-a109-19cd05129d04)

### 3.	What are the 25th, 50th and 75th percentile values for the revenue per transaction?

```sql
WITH CTE as
(SELECT *, 
       percent_rank() OVER (ORDER BY Total_revenue) AS percent_ranks
FROM (SELECT txn_id, SUM(price*qty) AS Total_revenue
FROM balanced_tree_clothing_company.productsales
GROUP BY txn_id
ORDER BY Total_revenue DESC) AS abc)

SELECT
DISTINCT first_value(Total_revenue) over( order by case when percent_ranks <= 0.25 then percent_ranks end desc) AS _25percentile,
first_value(Total_revenue) over( order by case when percent_ranks <= 0.50 then percent_ranks end desc) AS _50percentile,
first_value(Total_revenue) over( order by case when percent_ranks <= 0.75 then percent_ranks end desc) AS _75percentile
FROM CTE
```
![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/78c60903-7799-4d3f-b054-e172d378c585)

### 4.	What is the average discount value per transaction?

```sql
WITH cte AS
(SELECT
txn_id, SUM(qty*price*(discount/100)) AS Total_disc_per_transaction
FROM balanced_tree_clothing_company.productsales
GROUP BY txn_id
ORDER BY Total_disc_per_transaction DESC)

SELECT
ROUND(SUM(Total_disc_per_transaction)/COUNT(txn_id),0) AS Avg_disc_value_per_transaction
FROM cte

```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/38dcbb5f-8296-438c-aa71-a36cd0c3c569)

### 5.	What is the percentage split of all transactions for members vs non-members?

```sql
WITH cte AS (SELECT 
SUM(CASE WHEN member = 't' THEN Transaction_count  END) AS t,
SUM(CASE WHEN member = 'f' THEN Transaction_count  END) AS f
FROM (SELECT member, COUNT(*) AS Transaction_count
FROM balanced_tree_clothing_company.productsales
GROUP BY member) AS abc)

SELECT CONCAT(t/(t+f)*100,' %') AS Member_transactions , CONCAT(f/(t+f)*100, ' %') AS Non_Member_transactions
FROM cte
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/2d1de765-145e-4867-a234-ed641ab65c62)

### 6.	What is the average revenue for member transactions and non-member transactions?

```sql
WITH revenue_cte AS (
  SELECT
    member,
  	txn_id,
    SUM(price*qty*(100-discount)/100) AS revenue
  FROM balanced_tree_clothing_company.productsales
  GROUP BY member, txn_id
)

SELECT
	member,
  ROUND(AVG(revenue),2) AS avg_revenue
FROM revenue_cte
GROUP BY member;
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/b0da0daa-5b0f-414c-a608-d7ec45540834)

## Product Analysis

### 1.	What are the top 3 products by total revenue before discount?

```sql
SELECT product_name, Total_revenue
FROM (SELECT prod_id, 
SUM(qty*price) AS Total_revenue
FROM balanced_tree_clothing_company.productsales
GROUP BY prod_id
ORDER BY Total_revenue DESC
LIMIT 3) AS a JOIN balanced_tree_clothing_company.`product details` AS b 
ON a.prod_id = b.product_id

```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/8b317c33-7cb0-4145-8223-597ec4f9cc46)

### 2.	What is the total quantity, revenue and discount for each segment?

```sql
WITH pd AS (SELECT product_id, segment_name, segment_id
FROM balanced_tree_clothing_company.`product details`),
ps AS 
(SELECT prod_id, qty, price, discount
FROM balanced_tree_clothing_company.productsales)

SELECT segment_id, segment_name, SUM(qty) AS Tot_qty, ROUND(SUM((qty*price)*((100-discount)/100)),0) AS Tot_revenue, 
      ROUND(SUM((qty*price)*(discount/100)),0) AS Tot_discount
FROM pd JOIN ps
ON pd.product_id = ps.prod_id
GROUP BY segment_id, segment_name
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/8e2086f5-480a-4ef7-8f44-fcc3bd241813)

### 3.	What is the top selling product for each segment?

```sql
WITH top_selling_cte AS ( 
  SELECT 
    product.segment_id, product.segment_name, product.product_id,
    product.product_name,
    SUM(sales.qty) AS total_quantity,
    RANK() OVER (PARTITION BY segment_id ORDER BY SUM(sales.qty) DESC) AS ranking
  FROM balanced_tree_clothing_company.productsales AS sales
  JOIN balanced_tree_clothing_company.`product details` AS product
    ON sales.prod_id = product.product_id
  GROUP BY 
    product.segment_id, product.segment_name, product.product_id, product.product_name
)

SELECT 
  segment_id, segment_name, product_id, product_name, total_quantity
FROM top_selling_cte
WHERE ranking = 1;
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/edd18b91-fd1c-4d52-86e4-e8e5e01d8d6e)

### 4.	What is the total quantity, revenue and discount for each category?

```sql
SELECT 
  product.segment_id,
  product.segment_name, 
  SUM(sales.qty) AS total_quantity,
  SUM(sales.qty * sales.price) AS total_revenue,
  SUM((sales.qty * sales.price) * sales.discount/100) AS total_discount
FROM balanced_tree_clothing_company.productsales AS sales
JOIN balanced_tree_clothing_company.`product details` AS product
	ON sales.prod_id = product.product_id
GROUP BY product.segment_id, product.segment_name;
```
![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/b1054bd6-5645-449e-b771-207749d844d2)

### 5.	What is the top selling product for each category?

```sql
WITH top_selling_cte AS
( 
  SELECT product.category_id, product.category_name, product.product_id,product.product_name,
        SUM(sales.qty) AS total_quantity,
        RANK() OVER (PARTITION BY product.category_id ORDER BY SUM(sales.qty) DESC) AS ranking
    FROM balanced_tree_clothing_company.productsales AS sales
    JOIN balanced_tree_clothing_company.`product details` AS product
    ON sales.prod_id = product.product_id
    GROUP BY 
    product.category_id, product.category_name, product.product_id, product.product_name
)

SELECT
category_id,category_name,product_id,product_name,total_quantity
FROM top_selling_cte
WHERE ranking = 1;

```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/f094af66-1d8b-4e17-b030-32ad76222ef3)

### 6.	What is the percentage split of revenue by product for each segment?

```sql
SELECT
  segment_name,
  product_name,
  ROUND(100 *(SUM(qty * s.price)/ SUM(SUM(qty * s.price)) OVER(PARTITION BY segment_name)),1) AS percent_of_revenue
FROM balanced_tree_clothing_company.productsales AS s
  JOIN balanced_tree_clothing_company.`product details` AS pd 
  ON s.prod_id = pd.product_id
GROUP BY segment_name, product_name
ORDER BY 1,3 DESC
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/f00904c0-53a6-4ca9-a5a6-1074bb4d3802)

### 7.	What is the percentage split of total revenue by category?

```sql
WITH cte AS
(SELECT 
SUM(CASE WHEN category_name = 'Mens' THEN Total_revenue END) AS Mens,
SUM(CASE WHEN category_name = 'Womens' THEN Total_revenue END) AS Womens
FROM (SELECT category_name, SUM(qty*s.price*(100-discount)/100) AS Total_revenue
FROM
  balanced_tree_clothing_company.productsales AS s
  JOIN balanced_tree_clothing_company.`product details` AS pd ON s.prod_id = pd.product_id
GROUP BY category_name
ORDER BY 1) AS abc)

SELECT
ROUND((Mens/(Mens+Womens))*100,1) AS Men_percent_of_revenue, 
ROUND((Womens/(Mens+Womens))*100,1) AS Women_percent_of_revenue
FROM cte

```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/cb968fb6-e61d-45fd-9f7b-9fe9e98679cd)

### 8.	What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)

```sql
WITH cte AS (SELECT prod_id, COUNT(txn_id) AS Total_txn
FROM balanced_tree_clothing_company.productsales
GROUP BY prod_id)

, cte2 AS(SELECT *, SUM(Total_txn) OVER(ORDER BY Total_txn ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS Sum_Total_Txn
FROM cte)

,cte3 AS 
(SELECT product_id, product_name
FROM balanced_tree_clothing_company.`product details`)

SELECT product_name, ROUND((Total_txn/Sum_Total_Txn)*100.0,2) AS total_transaction_penetration_for_each_product
FROM cte2 JOIN cte3 
ON cte2.prod_id = cte3.product_id
ORDER BY total_transaction_penetration_for_each_product DESC
```

![image](https://github.com/habyphilipose/CLOTHING-COMPANY/assets/31076902/d3fe23b4-28d1-4bfc-86b9-fb9bbf62e9af)
