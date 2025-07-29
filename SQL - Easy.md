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
