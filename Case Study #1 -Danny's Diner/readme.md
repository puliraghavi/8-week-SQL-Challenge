# 🍜 Case Study #1: Danny's Diner 

![1](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/b98b0710-fc64-4fc7-b035-dfec6a0b557b)
Link to the case study - https://8weeksqlchallenge.com/case-study-1/

## 📚Contents
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***
## Problem Statement
Danny seriously loves Japanese food so he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

***

 ## Entity Relationship Diagram
![image](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/6289984c-a5f1-466d-9fd8-a58f9e202cf9)

***

## Datasets
- Sales: This table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.
- Menu: Maps the product_id to the actual product_name and price of each menu item.
- Members:Tells us when a customer_id has joined the beta version of the Danny’s Diner loyalty program.

***

## 1. Case Study Questions and Solutions
I have studied the case and solved the following questions using MySQL. In the solutions i used various joins, cte expressions ,window fns, etc


**1. What is the total amount each customer spent at the restaurant?**
````sql
select s.customer_id, 
       sum(m.price) as total_amt
from sales s
join menu m
using (product_id)
group by customer_id
order by customer_id ;
````
#### Result:

**2.How many days has each customer visited the restaurant?**
````sql
select customer_id, 
       count(distinct order_date) as Days_count
from sales
group by 1;
````

**3.What was the first item from the menu purchased by each customer?**
````sql
with first_item_purchased as (
select s.customer_id, 
       m.product_name,
       rank () over (partition by customer_id 
               order by order_date) as rankk
from sales s
join menu m
using (product_id)
)
select * from first_item_purchased 
where rankk = 1;
````

**4.What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
select m.product_name, 
       count(s.product_id) as total_count
from menu m
join sales s
using (product_id)
group by 1
order by total_count desc
limit 1;
````
**5.Which item was the most popular for each customer?**
````sql
with most_popular as (
select s.customer_id,
       m.product_name,
       count(s.product_id) as orders_count,
       dense_Rank() over (partition by s.customer_id 
                    order by count(s.customer_id) desc) as rankk
from sales s
join menu m
using (product_id)
group by 1,2
)
select customer_id,
       product_name, 
       orders_count
from most_popular
where rankk =1;
````
**6.Which item was purchased first by the customer after they became a member?**
````sql
with after_join_date as(
select s.customer_id,
       s.order_date, 
       mem.join_date,
       m.product_name,
       dense_rank() over (partition by s.customer_id 
                    order by s.order_Date asc) as rankk
from sales s
left join members mem
using (customer_id)
join menu m
using (product_id)
where s.order_date >= mem.join_date
)
select customer_id,
	   order_date,
       join_date,
       product_name
from after_join_date
where rankk = 1;
````
**7.Which item was purchased just before the customer became a member?**
````sql
with before_join_date as (
select s.customer_id,
       s.order_date, 
       mem.join_Date, 
       dense_rank() over (partition by s.customer_id 
                    order by s.order_Date desc) as rankk,
       m.product_name
from sales s
left join members mem 
using (customer_id)
join menu m 
using (product_id)
where s.order_date < mem.join_date
)
select customer_id,
       order_date,
       join_Date,
       product_name
from before_join_date
where rankk = 1;
````
**8.What is the total items and amount spent for each member before they became a member?**
````sql
select s.customer_id,
count(s.customer_id) as total_orders,
sum(m.price) as amount_spent
from sales s
left join members mem 
using (customer_id)
join menu m
using (product_id)
where s.order_Date < mem.join_date
group by 1
order by 1;
````
**9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - 
how many points would each customer have?**
````sql
with cte1 as(
select s.customer_id,
       case 
         when m.product_name = "sushi" then price*20 
         else price*10 end as points
from sales s
join menu m 
using (product_id)
)
select customer_id,
       sum(points) as total_points
from cte1
group by 1;
````

**10.In the first week after a customer joins the program 
(including their join date) they earn 2x points on all items, not just sushi -
how many points do customer A and B have at the end of January?**
````sql
with total_points as (
select customer_id,
       join_date,
       date_add(join_Date, interval 6 day) as after_a_week,
       last_day("2021-01-31") as end_of_jan
from members
)
select s.customer_id,
       sum(case
         when m.product_name = "sushi" then m.price*2*10
         when s.order_date between join_date and after_a_week
		    then m.price*2*10
		 else m.price * 10 end) as total_points
from total_points t
join sales s
    on t.customer_id = s.customer_id
join menu m
    using (product_id)
where (t.join_date <= s.order_date)
       and (s.order_date <= t.end_of_jan)
group by 1
order by 1;
````

## 2.Bonus questions
**Join All The Things**
**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**
````sql
create table new_table as (
select s.customer_id,
       s.order_date,
       m.product_name,
       m.price,
       case 
         when mm.join_date <= s.order_date then "Y" 
         else "N" end as `member`
from sales s
join menu m
     using (product_id)
left join members mm
     using (customer_id)
order by customer_id,order_date);
````

**Rank All The Things**
**Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.**
````sql
with ranking_cte as (
 select s.customer_id,
       s.order_date,
       m.product_name,
       m.price,    
       case 
         when mm.join_date <= s.order_date then "Y" 
         else "N" end as `member`
from sales s
join menu m
     using (product_id)
left join members mm
     using (customer_id)
order by customer_id,order_date
)
select *,
       case when `member` = "Y"
			then dense_rank() over (partition by customer_id, `member`
				 order by order_date asc) 
			else null end as ranking
from ranking_cte
order by customer_id
````




