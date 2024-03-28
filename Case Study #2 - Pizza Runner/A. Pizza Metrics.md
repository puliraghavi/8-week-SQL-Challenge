## Data Cleaning

Before writing the SQL queries, the table was investigated and cleaned accordingly in the following steps

**Cleaning - customer_orders table**
````sql
UPDATE customer_orders
SET exclusions = CASE WHEN exclusions = '' THEN NULL 
                 WHEN exclusions = 'null' THEN NULL 
                 ELSE exclusions END;
UPDATE customer_orders
SET extras = CASE WHEN extras = '' THEN NULL 
             WHEN extras = 'null' THEN NULL 
             ELSE extras END;
````

**Cleaning -  runner_orders table**
````sql
update runner_orders
set pickup_time =case 
     when pickup_time = "null" then null 
     else pickup_time end,
distance= case 
	when distance like '%km' then trim("km" from distance) 
    when distance = "null" then null
    else distance end,
duration = case 
	when duration like "%minute" then trim("minute" from duration)
    when duration like "%mins" then trim("mins" from duration)
	when duration like "%minutes" then trim("minutes" from duration)
    when duration like "null" then null else duration end,
cancellation = case
    when cancellation = "null" then null else cancellation end;
update runner_orders
set cancellation= case
when cancellation = '' then null else cancellation end;
````

**Data manipulation - pizza recipes**
````sql
CREATE TEMPORARY TABLE pizza_recipes_clean as (
SELECT p.pizza_id,
       REPLACE(SUBSTRING_INDEX(SUBSTRING_INDEX(p.toppings, 
              ',', numbers.n), ',', -1), ' ', '') AS toppings
FROM (select 1 AS n UNION ALL
  select 2 UNION ALL
        select 3 UNION ALL
        select 4 UNION ALL
        select 5 UNION ALL
        select 6 UNION ALL
        select 7 UNION ALL
        select 8 ) AS numbers
INNER JOIN pizza_recipes p 
ON CHAR_LENGTH(p.toppings) - CHAR_LENGTH(REPLACE(p.toppings, 
          ',', '')) >= numbers.n-1
ORDER BY p.pizza_id, n);
````

**Data Manipulation - changing datatypes**
````sql
alter table runner_orders
 modify column pickup_time datetime null,
 modify column distance decimal(5,1) null,
 modify column duration int null;
````
***

 ## A. Pizza Metrics
**1.How many pizzas were ordered?**
````sql
select count(order_id) as Total_orders
 from customer_orders;
````

**2.How many unique customer orders were made?**
````sql
select count(distinct order_id) as unique_orders
from customer_orders;
````

**3.How many successful orders were delivered by each runner?**
````sql
select runner_id, count(order_id) 
from runner_orders
where cancellation is null 
group by 1;
````

**4.How many of each type of pizza was delivered?**
````sql
select pizza_names.pizza_name, 
count(customer_orders.pizza_id)
from customer_orders
join runner_orders
using (order_id)
join pizza_names
using (pizza_id)
where runner_orders.cancellation is null
group by 1;
````

**5.How many Vegetarian and Meatlovers were ordered by each customer?**
````sql
select customer_id, pizza_names.pizza_name,
count(customer_orders.pizza_id) as pizza_count
from customer_orders
join pizza_names
using(pizza_id)
group by customer_orders.customer_id,pizza_names.pizza_name
order by customer_orders.customer_id;
````

**6.What was the maximum number of pizzas delivered in a single order?**
````sql
select runner_orders.order_id, 
count(customer_orders.pizza_id) as TotalDelivered
from runner_orders
join customer_orders
using (order_id)
where runner_orders.cancellation is null
group by runner_orders.order_id
order by TotalDelivered desc
limit 1;
````

**7.For each customer, how many delivered pizzas had 
at least 1 change and how many had no changes?**
````sql
select customer_orders.customer_id, 
sum(case
  when (exclusions is not null) or (extras is not null) then 1
        else 0 end )as AtleastOneChange,
sum(case 
  when (exclusions is null ) and (extras is null ) then 1
        else 0 end ) as NoChange
from customer_orders
left join runner_orders
using (order_id)
where runner_orders.distance is not null
group by customer_orders.customer_id;
````

**8.How many pizzas were delivered that had both exclusions 
and extras?**
````sql
select customer_orders.customer_id,
count(customer_orders.order_id) as Both_Exclusion_Extra
from customer_orders
join runner_orders
using (order_id)
where (customer_orders.exclusions is not null) 
  and (customer_orders.extras is not null)
  and (runner_orders.cancellation is null)
group by customer_orders.customer_id;
````

**9.What was the total volume of pizzas ordered for each hour of the day?**
````sql
select hour(order_time) as Hour_, 
count(order_id) as Total_orders
from customer_orders
group by Hour_
order by Hour_;
````

**10.What was the volume of orders for each day of the week?**
````sql
select dayname(order_time) as Day_, 
count(order_id) as Total_orders
from customer_orders
group by Day_
order by Total_orders desc;
````

