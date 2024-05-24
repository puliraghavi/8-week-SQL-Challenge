# # Case Study #7 - Balanced Tree Clothing Co.

![7](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/d1edd99a-80e7-44a8-bd0f-7df2c0ef2f3c)

## 📚Contents
- [Problem Statement](#problem-statement)
- [Datasets](#datasets)
- [Question and Solution](#question-and-solution)

***
## Problem Statement
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer! Danny, the CEO of this trendy fashion company has asked to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

## Datasets
For this case study there is a total of 4 datasets for this case study
 1. balanced_tree.product_details
 2. balanced_tree.sales
 3. balanced_tree.product_hierarchy
 4. balanced_tree.product_prices

# Question and Solution

## A. High Level Sales Analysis

**1. What was the total quantity sold for all products?**
````sql
select p.product_name as Product,
sum(s.qty) as Total_units_sold
from balanced_tree.sales s
left join balanced_tree.product_details p
on s.prod_id = p.product_id
group by 1
````

**2.What is the total generated revenue for all products before discounts?**
````sql
select p.product_name, sum(s.price)*sum(s.qty)
from balanced_tree.product_details p
join balanced_tree.sales s
on s.prod_id = p.product_id
group by 1
````

**3.What was the total discount amount for all products?**
````sql
select p.product_name, sum(s.qty*s.price*s.discount/100) as total_discount
from balanced_tree.product_details p
join balanced_tree.sales s
on s.prod_id = p.product_id
group by 1
````

## B. Transaction Analysis

**1. How many unique transactions were there?**
````sql
select count(distinct txn_id) 
from balanced_tree.sales s
````

**2. What is the average unique products purchased in each transaction?**
````sql
select round(avg(total_quantity)) as avg_unique_products
from(
SELECT 
    txn_id, 
    SUM(qty) AS total_quantity
  FROM balanced_tree.sales
  GROUP BY txn_id) as new
````

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**


**4. What is the average discount value per transaction?**
````sql
select round(avg(avg_discount),2) 
from (
select 
  txn_id, 
  sum(qty*price*discount/100) as     avg_discount
  from balanced_tree.sales
group by 1) as new
````

**5.What is the percentage split of all transactions for members vs non-members?**
````sql
with txn_cte as (
  select member, count(distinct(txn_id)) as txns
  from balanced_tree.sales
  group by 1
)
  select *,
  round(100*txns/(select sum(txns) from txn_cte),1)
  from txn_cte
  group by member, txns
````

**6.What is the average revenue for member transactions and non-member transactions?**
````sql
with cte1 as (  
select member, txn_id, sum(price*qty) as rev
  from balanced_tree.sales 
  group by 1,2
  )
  select member, round(avg(rev),2)
  from cte1 
  group by 1
````

  ## C. Product Analysis
  
**1. What are the top 3 products by total revenue before discount?**
````sql
select p.product_name, sum(s.price*s.qty) as rev
from balanced_tree.product_details p
left join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1
order by 2 desc
limit 3
````

**2. What is the total quantity, revenue and discount for each segment?**
````sql
select p.segment_id, 
p.segment_name, 
sum(s.qty), 
sum(s.price*s.qty) as rev,
sum(s.price*s.qty*(s.discount)/100) as discount
from balanced_tree.product_details p
join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1,2
order by 1
````

**3. What is the top selling product for each segment?**
````sql
with cte2 as (
select
p.segment_id,
p.segment_name,
p.product_id,
p.product_name,
sum(s.qty)as units_sold,
rank()over(partition by segment_id order by sum(s.qty)desc) as ranking
from balanced_tree.product_details p 
inner join balanced_tree.sales s 
on p.product_id = s.prod_id  
group by 1,2,3,4
           )
  select *
  from cte2 
  where ranking = 1
````

**4. What is the total quantity, revenue and discount for each category?**
````sql
select p.category_id, 
p.category_name, 
sum(s.qty), 
sum(s.price*s.qty) as rev,
sum(s.price*s.qty*(s.discount)/100) as discount
from balanced_tree.product_details p
join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1,2
order by 1
````

**5. What is the top selling product for each category?**
````sql
with cte as( 
  select p.category_id,
  p.category_name,
  p.product_id,
  p.product_name,
  sum(s.qty) as units_sold,
  rank()over(partition by p.category_id order by sum(s.qty) desc) as ranking
  from balanced_tree.product_details p
  join balanced_tree.sales s
  on p.product_id = s.prod_id
  group by 1,2,3,4
  )
  select *
  from cte
  where ranking =1
````

**6. What is the percentage split of revenue by product for each segment?**
````sql
select p.segment_name,
p.product_name,
sum(s.price*s.qty) as rev,
round(100 *(sum(s.qty * s.price)/ sum(sum(s.qty * s.price)) over(partition by segment_name)),1)
AS percent_of_revenue
from balanced_tree.product_details p
left join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1,2
order by segment_name,product_name, percent_of_revenue desc
````

**7. What is the percentage split of revenue by segment for each category?**
````sql
select p.segment_name,
p.category_name,
sum(s.price*s.qty) as rev,
round(100 *(sum(s.qty * s.price)/ sum(sum(s.qty * s.price)) over(partition by category_name)),1)
AS percent_of_revenue
from balanced_tree.product_details p
left join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1,2
order by p.segment_name,category_name, percent_of_revenue desc
````

**8. What is the percentage split of total revenue by category?**
````sql
select
p.category_name,
sum(s.price*s.qty) as rev,
round(100 *(sum(s.qty * s.price)/ sum(sum(s.qty * s.price))over()),1)
AS percent_of_revenue
from balanced_tree.product_details p
left join balanced_tree.sales s
on p.product_id = s.prod_id
group by 1
order by category_name
````

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**


**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

## D. Reporting Challenge
  





