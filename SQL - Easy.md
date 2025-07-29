 # DataLemur SQL - Easy

 ### 1. **Histogram of Tweets**

 Assume you're given a table Twitter tweet data, 
write a query to obtain a histogram of tweets posted per user in 2022. 
Output the tweet count per user as the bucket and the number of Twitter users who fall into that bucket.

In other words, group the users by the number of tweets they posted in 2022 and count the number of users in each group.

```sql
select 
	tweet_count as tweet_bucket,
	count(*) as users_num
from
	(select 
		count(tweet_id) tweet_count,
		user_id
	from 
		tweets
	where 
		tweet_date between '2022-01-01' and '2022-12-31'
	group by 
		user_id)
table1
group by 
	tweet_count

```

### 2. **DataScience Skills**

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. You want to find candidates who are proficient in Python, Tableau, and PostgreSQL.

Write a query to list the candidates who possess all of the required skills for the job. 
Sort the output by candidate ID in ascending order.

```sql
select 
	candidate_id
from 
	candidates
where 
	skill in ('Python', 'Tableau', 'PostgreSQL')
group by 
	candidate_id
having 
	count(distinct skill) >= 3
order by 
	candidate_id
```

**OR**

```sql
select 
	candidate_id
from 
	candidates
group by 
	candidate_id
having 
	sum(
		(case when skill = 'Python' then 1 else 0 end) +
		(case when skill = 'Tableau' then 1 else 0 end) +
		(case when skill = 'PostgreSQL' then 1 else 0 end) 
		) = 3
order by 
	candidate_id
```

### 3. **Pages With No Likes**

Assume you're given two tables containing data about Facebook Pages and their respective likes (as in "Like a Facebook Page").

Write a query to return the IDs of the Facebook pages that have zero likes. 
The output should be sorted in ascending order based on the page IDs.


```sql
select 
	p.page_id 
from 
	pages p
left join 
	page_likes pl on pl.page_id = p.page_id
where 
	pl.page_id is null
```


### 4. **Unfinished Parts**

Tesla is investigating production bottlenecks and they need your help to extract the relevant data. 
Write a query to determine which parts have begun the assembly process but are not yet finished.

Assumptions:
parts_assembly table contains all parts currently in production, each at varying stages of the assembly process.
An unfinished part is one that lacks a finish_date.
This question is straightforward, so let's approach it with simplicity in both thinking and solution.

```sql
select 
	part,
	assembly_step
from 
	parts_assembly
where 
	finish_date is null
```


### 5. **Laptop Vs Mobile Viewership**

Assume you're given the table on user viewership categorised by device type where the three types are laptop, tablet, and phone.

Write a query that calculates the total viewership for laptops and mobile devices where mobile is defined as the sum of tablet and phone viewership. 
Output the total viewership for laptops as laptop_reviews and the total viewership for mobile devices as mobile_views.

```sql
with views as(
	select 
		count(*) num_views,
		device_type
	from 
		viewership
	group by 
		device_type
)

select 
	sum
		(case when device_type = 'laptop' then num_views else 0 end) as laptop_views,
	sum
		(case when device_type in ('phone','tablet') then num_views else 0 end) mobile_views
from
	views
```


### 6. **Average Post Hiatus**

Given a table of Facebook posts, for each user who posted at least twice in 2021, write a query to find the number of days between each user’s first post of the year and last post of the year in the year 2021. Output the user and number of the days between each user's first and last post.

```sql
select 
	user_id,
	days_between
from
	(
	select 
		user_id,
		max(post_date::date) - min(post_date::date)  as days_between
	from 
		posts
	where 
		date_part('year',post_date) = 2021
	group by 
		user_id
	having 
		count(post_id)>1
)table1
```


### 7. **Teams Power Users** 

Write a query to identify the top 2 Power Users who sent the highest number of messages on Microsoft Teams in August 2022. Display the IDs of these 2 users along with the total number of messages they sent. Output the results in descending order based on the count of the messages.

Assumption:
No two users have sent the same number of messages in August 2022.


```sql
select 
	sender_id,
	count(message_id) num_msgs
from 
	messages
where 
	(sent_date::date) between '2022-08-01' and '2022-08-31'
group by 
	sender_id
order by 
	count(message_id) desc
limit 2 
```


## 8. **Well Paid Employees**

Companies often perform salary analyses to ensure fair compensation practices. 
One useful analysis is to check if there are any employees earning more than their direct managers.

As a HR Analyst, you're asked to identify all employees who earn more than their direct managers. The result should include the employee's ID and name.

```sql
select 
	e.employee_id,
	e.name
from 
	employee e
join 	
	employee m on m.employee_id = e.manager_id
where 
	e.salary > m.salary
```


### 9. **Final Account Balance**

Given a table containing information about bank deposits and withdrawals made using Paypal, write a query to retrieve the final account balance for each account, taking into account all the transactions recorded in the table with the assumption that there are no missing transactions.

```sql
select 
	account_id,
	sum (
		 case 
		 	when transaction_type = 'Deposit' then amount 
			when transaction_type = 'Withdrawal' then amount * (-1) 
		 	else 0
		 end 
		)as total_balance
from 
	transactions
group by
	account_id
```


### 10. **Quickbooks Vs TurboTax**


Intuit provides a range of tax filing products, including TurboTax and QuickBooks, available in various versions.

Write a query to determine the total number of tax filings made using TurboTax and QuickBooks.
Each user can file taxes once a year using only one product.

```sql
select	
	count(
		case when product like 'TurboTax%' then 1 end) as turbotax_total,
	count(
		case when product like 'QuickBooks%' then 1 end) as quickbooks_total
	
from
	filed_taxes
```

### 11. **App Click-Thru Rate**

Assume you have an events table on Facebook app analytics. Write a query to calculate the click-through rate (CTR) for the app in 2022 and round the results to 2 decimal places.

Definition and note:
Percentage of click-through rate (CTR) = 100.0 * Number of clicks / Number of impressions
To avoid integer division, multiply the CTR by 100.0, not 100.

```sql
with table1 as
	(
	select 
		app_id,
		sum(
			case when event_type = 'click' then 1 
			else 0
			end) 
		as num_clicks,
		sum(
			case when event_type = 'impression' then 1
			else 0
			end)
		as num_impressions
	from 
		events
	where 
		date_part('year',timestamp) = 2022
	group by 
		app_id
)

select 
	app_id,
	round(100.00 * num_clicks / nullif (num_impressions,0),2) as ctr_percentage
from 
	table1
```


### 12. **IBM DB2 Product Analysis**

IBM is analyzing how their employees are utilizing the Db2 database by tracking the SQL queries executed by their employees. The objective is to generate data to populate a histogram that shows the number of unique queries run by employees during the third quarter of 2023 (July to September). Additionally, it should count the number of employees who did not run any queries during this period.

Display the number of unique queries as histogram categories, along with the count of employees who executed that number of unique queries.

```sql
with table1 as
	(
	select
		employee_id,
		count(distinct query_id) unique_query_count
	from
		queries
	where 	
		date(query_starttime) between '2023-07-01' and '2023-09-30'
	group by 
		employee_id
),
table2 as
	(
	select 
		unique_query_count,
		count(*) employee_count
	from 
		table1
	group by unique_query_count
),
table3 as
	(
	select 
		count(*) employee_count
	from
		employees
	where 
		employee_id not in (select employee_id from table1) 
)

select * from table2
union all 
select 0 as unique_query_count, employee_count from table3
order by unique_query_count
```


### 13. **Cards Issued Difference**

Your team at JPMorgan Chase is preparing to launch a new credit card, and to gain some insights, you're analyzing how many credit cards were issued each month.

Write a query that outputs the name of each credit card and the difference in the number of issued cards between the month with the highest issuance cards 
and the lowest issuance. Arrange the results based on the largest disparity.

```sql
select 
	card_name,
	max(issued_amount) - min(issued_amount) difference
from
	monthly_cards_issued
group by 
	card_name
order by 
	difference desc
```


### 14. **Compressed Mean**

You're trying to find the mean number of items per order on Alibaba, rounded to 1 decimal place using tables which includes information on the count of items in each order (item_count table) and the corresponding number of orders for each item count (order_occurrences table).

```sql
select 
	round(sum(item_count*order_occurrences)::numeric/sum(order_occurrences),1)
from 
	items_per_order
```

**OR**

```sql
select 
	round(sum(item_count*order_occurrences)*1.0/sum(order_occurrences),1)
from 
	items_per_order
```


### 15. **Pharmacy Analytics I**


CVS Health is trying to better understand its pharmacy sales, and how well different products are selling. Each drug can only be produced by one manufacturer.
Write a query to find the top 3 most profitable drugs sold, and how much profit they made. Assume that there are no ties in the profits. Display the result from the highest to the lowest total profit.

Definition:
cogs stands for Cost of Goods Sold which is the direct cost associated with producing the drug.
total Profit = Total Sales - Cost of Goods Sold

```sql
select 
	drug,
	sum(total_sales) - sum(cogs) as total_profit
from 
	pharmacy_sales
group by
	drug
order by 
	total_profit desc
limit 3
```


### 16. ***Pharmacy Analytics II**

CVS Health is analyzing its pharmacy sales data, and how well different products are selling in the market. Each drug is exclusively manufactured by a single manufacturer.

Write a query to identify the manufacturers associated with the drugs that resulted in losses for CVS Health and calculate the total amount of losses incurred.

Output the manufacturer's name, the number of drugs associated with losses, and the total losses in absolute value. Display the results sorted in descending order with the highest losses displayed at the top.

```sql
select 
	manufacturer,
	count(*) drug_count,
	abs(sum(cogs) - sum(total_sales)) as losses
from 	
	pharmacy_sales
where 
  cogs > total_sales
group by 
	manufacturer
order by 
  losses desc
```


### 17. **Pharmacy Analytics III**


CVS Health wants to gain a clearer understanding of its pharmacy sales and the performance of various products.

Write a query to calculate the total drug sales for each manufacturer. Round the answer to the nearest million and report your results in descending order of total sales. In case of any duplicates, sort them alphabetically by the manufacturer name.

Since this data will be displayed on a dashboard viewed by business stakeholders, please format your results as follows: "$36 million"

```sql
select 
	manufacturer,
	concat('$', round(sum(total_sales)/1000000),' million') sale
from 
	pharmacy_sales_2
group by 
	manufacturer
order by 
	sale desc, manufacturer asc
```


### 18. **Patient Support Analysis I***

UnitedHealth Group (UHG) has a program called Advocate4Me, which allows policy holders (or, members) to call an advocate and receive support for their health care needs – whether that's claims and benefits support, drug coverage, pre- and post-authorisation, medical records,
emergency assistance, or member portal services.

Write a query to find how many UHG policy holders made three, or more calls, assuming each call is identified by the case_id column.


```sql
select 
	count(policy_holder_id) policy_holder_count
from 
	(	
	select 
		policy_holder_id,
		count(case_id) num_calls
	from 
		callers
	group by 
		policy_holder_id
	having 
		count(case_id) >= 3
)a
```


### 19. **Most Expensive Purchase**

Amazon is trying to identify their high-end customers. To do so, they first need your help to write a query that obtains the most expensive purchase made by each customer. Order the results by the most expensive purchase first.

```sql
select 
	customer_id,
	max(purchase_amount) purchase_amount
from
	transactions_1
group by 
	customer_id
order by 
	max(purchase_amount) desc
```



### 20. **Apple Pay Volume**

Visa is analysing its partnership with ApplyPay. Calculate the total transaction volume for each merchant where the transaction was performed via ApplePay.

Output the merchant ID and the total transactions. For merchants with no ApplePay transactions, output their total transaction volume as 0. Display the result in descending order of the transaction volume.

```sql
select 
	merchant_id,
	sum(
		case when payment_method like 'Apple%' or payment_method like 'apple%'  then transaction_amount
		else 0
		end) as total_transaction
from 
	transactions_2
group by 
	merchant_id
order by 
	2 desc
```
