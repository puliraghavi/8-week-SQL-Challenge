🏦💵 Case Study #4: Data Bank

![4](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/1c9871ae-21a7-400e-a7a8-6800b699c05c)
Link to the case study - https://8weeksqlchallenge.com/case-study-4/

## 📚Contents
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***
## Problem Statement
Danny was amused by the concept of Neo-banks and thought that there should be some sort of intersection between these new age banks, cryptocurrency and the data world…so he decides to launch a new initiative - Data Bank!
Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

***

 ## Entity Relationship Diagram
 ![image](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/68feb05e-b1bb-4b6e-b2b8-94e95557c241)

 ***

## Datasets
- Regions Table
- Customer Nodes: Data Bank is also run off a network of nodes where both money and data is stored across the globe. In a traditional banking sense - you can think of these nodes as bank branches or stores that exist around the world.
- Customer Transactions : Stores all customer deposits, withdrawals and purchases

***

# Case Study Questions and Solutions
I have studied the case and solved the following questions using MySQL.This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments

## A. Customer Nodes Exploration
**1.How many unique nodes are there on the Data Bank system?**
````sql
select count(distinct node_id) as unique_nodes
from customer_nodes;
````

**2.What is the number of nodes per region?**
````sql
select r.region_name,
       count(distinct c.node_id) as total_nodes
from regions r 
left join customer_nodes c
using(region_id)
group by r.region_name;
````

**3.How many customers are allocated to each region?**
````sql
select r.region_name,
       count(distinct c.customer_id) as total_customers
from regions r 
left join customer_nodes c
using(region_id)
group by r.region_name;
````

**4.How many days on average are customers reallocated to a 
different node?**
````sql
with initial_node as (
select customer_id, 
       node_id, 
	     min(start_date) as first_node_date
from customer_nodes
group by customer_id,node_id),
next_node as (
select *, 
	     datediff(first_node_date, 
       lead(first_node_date) over(partition by customer_id
				                     order by first_node_date)) as days_diff
from initial_node)
select concat(abs(round(avg(days_diff)))," days") 
       as `avg days of reallocation`
from next_node;
````

**5.What is the median, 80th and 95th percentile for this same 
reallocation days metric for each region?**

## B. Customer Transactions

**1.What is the unique count and total amount for each transaction type?**
````sql
select txn_type, 
       count(*) as unique_count, 
       concat(round((sum(txn_amount)/1000000),2),"M") as total_amount
from customer_transactions
group by txn_type;
````

**2.What is the average total historical deposit counts and 
amounts for all customers?**
````sql
with deposits as (
select customer_id,
       count(customer_id)as txn_count,
       sum(txn_amount) as total_amt
from customer_transactions
where txn_type = "deposit"
group by customer_id)
select round(avg(txn_count)) as avg_deposit_count,
	   round(avg(total_amt)) as avg_deposit_amt
from deposits;
````

**3.For each month - how many Data Bank customers make more than 1
 deposit and either 1 purchase or 1 withdrawal in a single month?**
 ````sql
with monthly_txns as (
select customer_id,
       monthname(txn_date) as mnth,
       sum(case when txn_type = "deposit" then 1 else 0 end)
           as deposit_count,
       sum(case when txn_type = "purchase" then 1 else 0 end)
           as purchase_count,
       sum(case when txn_type = "withdrawal" then 1 else 0 end)
           as withdrawal_count
from customer_transactions
group by customer_id,
         mnth)
select mnth,
       count(distinct customer_id)
from monthly_txns
where (deposit_count>1) and 
      ((purchase_count=1) or (withdrawal_count=1))
group by mnth
order by mnth;
````

**4.What is the closing balance for each customer at the end of 
the month?**

**5.What is the percentage of customers who increase their closing
 balance by more than 5%?**




