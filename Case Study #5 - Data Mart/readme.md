#🛒Case Study #5: Data Mart

![5](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/28edc88e-90bb-4290-bb4a-ba74d0e73367)

## 📚Contents
- [Problem Statement](#problem-statement)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Question and Solution](#question-and-solution)

***
## Problem Statement
Data Mart is Danny’s latest venture and after running international operations for his online supermarket that specialises in fresh produce - Danny is asking for your support to analyse his sales performance.

In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.
Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.


***

 ## Entity Relationship Diagram
 ![image](https://github.com/puliraghavi/8-week-SQL-Challenge/assets/119037510/f4015e32-a86a-429e-859a-77da3b6266f1)

 ***

## Datasets
Weekly sales: Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a week_date value which represents the start of the sales week.

***
# Case Study Questions and Solutions
This case study had the most number of records amongst the all (17110 rows). I have conducted data cleaning, formatting, variance, before and after analysis in this case using MySQL.

##  A.Data cleaning steps
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:

1.Convert the week_date to a DATE format
2.Add a week_number as the second column for each week_date value
3.Add a month_number with the calendar month for each week_date value as the 3rd column
4.Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
5.Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value.

## Solution:
````sql
drop table if exists clean_weekly_sales;
create table clean_weekly_sales as (
select STR_TO_DATE(week_date,"%d/%m/%y") as week_date,
       week(STR_TO_DATE(week_date,"%d/%m/%y")) as week_number,
       month(STR_TO_DATE(week_date,"%d/%m/%y")) as month_number,
       year(STR_TO_DATE(week_date,"%d/%m/%y")) as calendar_year,
       region,
       platform,
       segment,
       case when right(segment,1) = "1" then "Young Adults"
            when right(segment,1) = "2" then "Middle Aged"
            when right(segment,1) in  ("3","4") then "Retirees" 
            else "unknown" end as age_band,
	   case when left(segment,1) = "C" then "Couples"
            when left(segment,1) = "F" then "Families" 
            else "unknown" end as demographic,
	   customer_type,
	   transactions,
       round((sales/transactions),2) as avg_transaction,
       sales
from weekly_sales);
````
***

## B. Data Exploration
**1.What day of the week is used for each week_date value?**
````sql
select distinct(dayname(week_date)) as week_day
from clean_weekly_sales;
````

**2.What range of week numbers are missing from the dataset?**
````sql
select distinct week_number from clean_weekly_sales;
````

**3.How many total transactions were there for each year in the dataset?**
````sql
select calendar_year, 
       concat(round(sum(transactions)/1000000,0),"M")
              as total_txns
from clean_weekly_sales
group by calendar_year
order by calendar_year;
````

**4.What is the total sales for each region for each month?**
````sql
select month_number,
	     monthname(week_date) as mnth,
       region,
       concat(round(sum(sales)/ 1000000,1),"M") as total_sales
from clean_weekly_sales
group by month_number,mnth,region
order by month_number,mnth,region;
````

**5.What is the total count of transactions for each platform**
````sql
select platform, 
       concat(round(sum(transactions)/1000000,1),"M")
              as total_txns
from clean_weekly_sales
group by platform;
````

**6.What is the percentage of sales for Retail vs Shopify for each month?**
````sql
with sales_cte as(
select calendar_year,
	   month_number,
       monthname(week_date) as mnth,
       sum(case when platform = "Retail" then sales
		   end) as retail_sales,
	   sum(case when platform = "Retail" then sales
		   end) as shopify_sales,
	   sum(sales) as total_sales
from clean_weekly_sales
group by calendar_year, month_number,mnth
order by calendar_year, month_number,mnth)
select calendar_year, 
       month_number,
       mnth,
       round(retail_sales/total_sales*100,2) as Retail_Percentage,
       round(shopify_sales/total_sales*100,2) as Shopify_Percentage
from sales_cte
group by calendar_year, month_number,mnth
order by calendar_year, month_number,mnth;
````

**7.What is the percentage of sales by demographic for each year in 
the dataset?**
````sql
with sales_contribution_cte as(
select calendar_year,
       demographic,
       sum(sales) as sales_contribution
from clean_weekly_sales
group by 1,2
order by 1),
total_Sales_cte as 
(select *,sum(sales_contribution) 
		  over(partition by calendar_year) as total_sales
from sales_contribution_cte)
select calendar_year,
       demographic,
       round(100*sales_contribution/total_sales,2) as `Percenatge breakdown`
from total_sales_cte
group by 1,2
order by 1,2;
````

**8.Which age_band and demographic values contribute the most to Retail sales?**
````sql
select age_band, 
       demographic,
       round(100*sum(sales)/
                 (select sum(sales) 
				  from clean_weekly_sales
				  where platform ="retail"),2) as total_sales
from clean_weekly_sales
where platform = "retail"
group by 1,2
order by 3 desc;
````

**9.Can we use the avg_transaction column to find the average 
transaction size for each year for Retail vs Shopify? 
If not - how would you calculate it instead?**
````sql
select calendar_year,
       platform,
       round(sum(sales)/sum(transactions),1) as avg_txn,
       round(avg(avg_transaction),1) as avg_of_avg
from clean_weekly_sales
group by 1,2
order by 1,2;
````
***

## C. Before & After Analysis

**1.What is the total sales for the 4 weeks before and after 
2020-06-15? What is the growth or reduction rate in 
actual values and percentage of sales?**

First we extract the week number value for the week "2020-06-15" and use it in our queries.

````sql
set @weeknum = (
SELECT DISTINCT week_number
  FROM clean_weekly_sales
  WHERE week_date = '2020-06-15'
  and calendar_year = "2020");
````

**Change in sales (4 weeks) before and after**  
````sql
with sales_change_4weeks as(
select 
  sum(case when week_number between @weeknum-4 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+3
      then sales end) as after_pct_sales
from clean_weekly_sales
where calendar_year ="2020"
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from sales_change_4weeks;
````

**2.What about the entire 12 weeks before and after?**
````sql
with sales_change_12weeks as(
select 
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
where calendar_year ="2020"
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from sales_change_12weeks;
````

**3.How do the sale metrics for these 2 periods before and 
after compare with the previous years in 2018 and 2019?**
````sql
with yoy_sales_change as (
  select
    calendar_year,
    sum(case when week_number between @weekNum-3 and @weekNum-1 
        then sales end) as before_sales,
    sum(case when week_number between @weekNum and @weekNum+3 
        then sales end) as after_sales
  from clean_weekly_sales
  group by calendar_year
)
select *,
  cast(100.0 * (after_sales-before_sales)/before_sales as decimal(5,2)) as 
       change_in_sales
from yoy_sales_change
order by calendar_year;
````
***

## D.Bonus Question
**Which areas of the business have the highest negative impact in 
sales metrics performance in 2020 for the 12 week before 
and after period?**

**1.Region**
````sql
with regional_sales as(
select region,
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
group by region
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from regional_sales;
````

**2.Platform**
````sql
with platform_sales as(
select platform,
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
group by platform
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from platform_sales;
````

**3.Age_band**
````sql
with age_band_sales as(
select age_band,
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
group by age_band
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from age_band_sales;
````

**4.Demographic**
````sql
with demographic_sales as(
select demographic,
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
group by demographic
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from demographic_sales;
````

**5.Customer_type**
````sql
with customer_type_sales as(
select customer_type,
  sum(case when week_number between @weeknum-12 and @weeknum-1
      then sales end) as before_pct_sales,
  sum(case when week_number between @weeknum and @weeknum+11
      then sales end) as after_pct_sales
from clean_weekly_sales
group by customer_type
)
select *, 
       concat(round(100*(after_pct_sales-before_pct_sales)
                  /before_pct_sales,2),"%") as `change in total sales`
from customer_type_sales;
````



 


