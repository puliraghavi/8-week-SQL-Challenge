# 🥑 Case Study #3: Foodie-Fi

![3](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/d1c5fe5e-2777-4039-aacb-174b24547422)

## 📚Contents
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***
## Problem Statement
This time Danny wanted to create a new streaming service that only had food related content - something like Netflix but with only cooking shows!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

***

 ## Entity Relationship Diagram
 ![Screenshot 2024-03-29 020229](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/cfd5d306-7874-4c9b-9cb5-2bacabc04524)

 ***
 ## Datasets
 - plans: At the moment, Foodie-Fi has 4 plans -trial,basic monthly, pro monthly, pro annual. And also a plan_id for churned customers
 - subscriptions: shows the exact date where their specific plan_id starts. Also includes upgrades, customer churn records

***

# Case Study Questions

## A. Customer Journey
Based off the 8 sample customers provided below in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

![image](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/48a4bf02-0a49-4f19-af26-1f0809b0e929)

**Solution:**

````sql
select s.*,
	   p.plan_name,
	   p.price
from subscriptions s
join plans p 
using(plan_id)
where customer_id in (1, 2, 11, 13, 15, 16, 18, 19)
order by customer_id,start_date;
````

## B. Data Analysis Questions
**1.How many customers has Foodie-Fi ever had?**
````sql
select count(distinct customer_id) as total_customers
from subscriptions;
````

**2.What is the monthly distribution of trial plan start_date values 
for our dataset - use the start of the month as the group by value**
````sql
select monthname(start_date) as month_,
       count(s.customer_id) as trial_pack_subscriptions
from subscriptions s
join plans p
using (plan_id)
where plan_name = "trial"
group by month_
order by field(month_, 'January', 'February', 'March', 
'April', 'May', 'June', 'July', 'August', 'September', 
'October', 'November', 'December');
````

**3.What plan start_date values occur after the year 2020 for our 
dataset? Show the breakdown by count of events for each plan_name**
````sql
select p.plan_id,
       p.plan_name, 
       count(s.customer_id) as `count of events`
from plans p
join subscriptions s
using (plan_id)
where s.start_date >= '2021-01-01'
group by p.plan_id,
         p.plan_name
order by p.plan_id;
````

**4.What is the customer count and percentage of customers 
who have churned rounded to 1 decimal place?**
````sql
select count(s.customer_id) as churn_count,
       concat(round(100*count(distinct s.customer_id)/ 
                    (select count(distinct customer_id)
                     from subscriptions),1),"%") as `Churn_%`
from subscriptions s
join plans p 
using (plan_id)
where plan_id = 4;
````
       
**5.How many customers have churned straight after their initial 
free trial - what percentage is this rounded to the nearest whole number?**
````sql
with after_trial_churn as(
select s.customer_id,
       p.plan_name,
       lead(p.plan_name) over(partition by s.customer_id
                              order by s.start_date) as next_plan
from subscriptions s
join plans p
using (plan_id))
select count(customer_id) as churn_count,
       concat(round(100*count(distinct customer_id)/
	           (select count(distinct customer_id)
              from subscriptions)),"%") as `churn %`
from after_trial_churn
where plan_name = "trial" and
      next_plan = "churn";
````

**6.What is the number and percentage of customer plans after their initial free trial?**
````sql
with after_trial as(
select s.customer_id,
       p.plan_name,
       lead(p.plan_name) over(partition by s.customer_id
                         order by s.start_date) as next_plan
from subscriptions s
join plans p
using (plan_id))
select next_plan,
       count(customer_id) as churn_count,
       concat(round(100*count(distinct customer_id)/
	           (select count(distinct customer_id)
              from subscriptions)),"%") as `percentage breakdown`
from after_trial
where plan_name = "trial" and
      next_plan is not null
group by next_plan
order by field(next_plan,"basic monthly","pro monthly","pro annual","churn");
````

**7.What is the customer count and percentage breakdown 
of all 5 plan_name values at 2020-12-31?**
````sql
with cte1 as (
select p.plan_name,
       customer_id,
       start_date,
       lead(s.start_date) over(partition by s.customer_id
                          order by s.start_date) as next_date
from plans p 
join subscriptions s
using (plan_id)
where s.start_date <= '2020-12-31')
select plan_name,
       count(distinct customer_id) as customer_count,
       concat(round(100*count(distinct customer_id)/
	           (select count(distinct customer_id)
              from subscriptions),1),"%") as `percentage breakdown `
from cte1
where next_date is null
group by plan_name
order by field(plan_name,"trial","basic monthly","pro monthly","pro annual","churn");
````

**8.How many customers have upgraded to an annual plan in 2020?**
````sql
select count(distinct customer_id)
from subscriptions s 
where plan_id = 3 and year(start_date) =2020;
````

**9.How many days on average does it take for a customer to 
upgrade to an annual plan from the day they join Foodie-Fi**
````sql
with initial_plan as (
select s.customer_id,
	     s.start_date as trial_date
from subscriptions s 
join plans p 
using (plan_id)
where p.plan_name  ="trial"
),
annual_plan as (
select s.customer_id,
	     s.start_date as annual_date
from subscriptions s 
join plans p 
using (plan_id)
where p.plan_name  ="pro annual")
select 
   concat(round(avg(datediff(annual_date,trial_date)))," days") as avgg
from initial_plan 
join annual_plan 
using (customer_id);
````

**10.Can you further breakdown this average value into 30 day 
periods (i.e. 0-30 days, 31-60 days etc)**

**11.How many customers downgraded from a pro monthly to a 
basic monthly plan in 2020?**
````sql
with cte1 as (
select s.customer_id,
       p.plan_id,
       p.plan_name,
       lead(p.plan_id) over(partition by s.customer_id
			                 order by s.start_date) as next_plan_id
from subscriptions s
join plans p
using(plan_id)
where year(start_date) = "2020"
)
select count(customer_id) as churn_count
from cte1
where plan_id = 2 and
      next_plan_id = 1;
````   
***
