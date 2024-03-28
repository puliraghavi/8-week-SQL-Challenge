 ## B. Runner and Customer Experience

**1.How many runners signed up for each 1 week period?
(i.e. week starts 2021-01-01)**
````sql
select week(registration_date) as week_, 
count(runner_id) as Total_registrations
from runners
group by week_;
````

**2.What was the average time in minutes it took for each 
runner to arrive at the Pizza Runner HQ to pickup the order?**
````sql
select runner_id, 
round(avg(timestampdiff(minute,order_time,pickup_time)),1) as AvgTime
from runner_orders
join customer_orders
using (order_id)
where cancellation is null or distance !=0 
group by runner_id
order by AvgTime;
````

**3.Is there any relationship between the number of pizzas 
and how long the order takes to prepare?**
````sql
with prep_time as
(select c.order_id, 
	count(c.order_id) as pizza_order, 
    c.order_time, 
    r.pickup_time, 
    round(timestampdiff(minute, c.order_time, r.pickup_time)) as prep_time_minutes
from customer_orders as c
join runner_orders as r
on (order_id)
where r.distance != 0
group by c.order_id, c.order_time, r.pickup_time)
select pizza_order, 
round(avg(prep_time_minutes)) as avg_prep_time
from prep_time
where prep_time_minutes > 1
group by pizza_order;
````

**4.What was the average distance travelled for each customer?**
````sql
select customer_id, round(avg(distance),1) as AvgDistance
from customer_orders
join runner_orders
using (order_id)
where cancellation is null or distance != 0
group by 1;
````

**5.What was the difference between the longest 
and shortest delivery times for all orders?**
````sql
select max(duration) - min(duration) as longest_shortest_diff
from runner_orders;
````

**6.What was the average speed for each runner for each delivery
 and do you notice any trend for these values?**
 ````sql
select runner_id, order_id,
round(distance *60/duration,1) as speedKMH
from runner_orders
where distance != 0
order by runner_id;
````

**7.What is the successful delivery percentage for each runner?**
````sql
with successful_deliveries as
(select runner_id, sum(case when distance != 0 then 1 else 0 end) as successful_orders,
count(order_id) as total_orders
from runner_orders
group by runner_id)
select runner_id, 
       concat(round((successful_orders/total_orders)*100),"%") as `Success %`
from successful_deliveries
order by runner_id
````
