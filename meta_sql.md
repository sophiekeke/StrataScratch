**https://www.youtube.com/watch?v=Juel4a9F0gw**

with base as (
select date
,count(distinct case when status = 'Fraud' and spend >0 then Account end) as fraud_account,
,count(distinct case when spend >0 then Account end) as total_acc
from table)
select *, case when total_acc=0 then null else  1.00 * nvl(fraud_account,0)/total_acc end as fraud_ratio 
from base;


with f_lst as (
select Account from table where date < current_date and status ='Fraud')
select Account
from table 
where status ='Fraud' and date = current_date
and Account not in (select * from f_lst)



Product Sense:

Case: If FB wants to launch Messenger, what is the business value of Messenger and how to measure success?


**Product Question**: A question from an interview.

**Background**: After 2010, most users had already registered for FB. Users who continued to register afterwards were "late adopters." At this time, most old users had already completed the majority of their friend searches, adding friends, and building their social networks. New users need to actively add users. Therefore, old users are less likely to actively search for and add new users, which is not beneficial for engagement.

**PM Proposal**: Add a friend recommendation feature in the second position on a user's personal homepage news feed.

**Question**: How would you measure the success of this feature?







# StrataScratch SQL

**StrataScratch**  

https://platform.stratascratch.com/coding/10064-highest-energy-consumption?code_type=1 Highest Energy Consumption

Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.

```sql
WITH union_all AS (
    SELECT * 
    FROM fb_eu_energy 
    UNION ALL 
    SELECT * 
    FROM fb_asia_energy 
    UNION ALL 
    SELECT * 
    FROM fb_na_energy
),
agg AS (
    SELECT 
        date, 
        SUM(consumption) AS tot_consumption 
    FROM union_all 
    GROUP BY 1
),
rank_rs AS (
    SELECT 
        DENSE_RANK() OVER (ORDER BY tot_consumption DESC) AS rank_num,
        * 
    FROM agg
)
SELECT 
    date,
    tot_consumption 
FROM rank_rs 
WHERE rank_num = 1 
ORDER BY date;
```

https://platform.stratascratch.com/coding/2005-share-of-active-users?code_type=1 Share of Active Users

Output share of US users that are active. Active users are the ones with an "open" status in the table.

```sql
with base as ( 
    select * from fb_active_users 
    where country = 'USA'
) 
select 1.00 * count(distinct case when status='open' then user_id end) / count(distinct user_id ) as ratio 
from base ;
```


https://platform.stratascratch.com/coding/2007-rank-variance-per-country?code_type=1
Rank Variance Per Country
Which countries have risen in the rankings based on the number of comments between Dec 2019 vs Jan 2020? Hint: Avoid gaps between ranks when ranking countries.

```sql
with base as (
select --to_chart(c.created_at,'YYYYMM') as ym
 u.country as country
, sum(number_of_comments) as tot_number_of_comments
, sum(case when to_char(c.created_at,'YYYY-MM') = '2019-12' then number_of_comments else 0 end) as before_comment
, sum(case when to_char(c.created_at,'YYYY-MM') = '2020-01' then number_of_comments else 0 end) as after_comment
from fb_comments_count c 
inner join fb_active_users u on c.user_id = u.user_id
where to_char(c.created_at,'YYYY-MM') in ('2019-12','2020-01')
group by 1
),
--select * from base order by after_comment desc;
comp as
(select 
country, after_comment
,dense_rank() over (order by after_comment desc ) as rank_num_after
,before_comment
,dense_rank() over (order by before_comment desc ) as rank_num_before
from base)
--select * from comp;
select country ,rank_num_after,rank_num_before
from comp where (rank_num_after<rank_num_before) or before_comment=0
order by rank_num_after
;
```
Solution with step by step: https://www.stratascratch.com/blog/facebook-data-scientist-interview-questions/
Solution youtube video: https://www.youtube.com/watch?v=ek3OiqCkEEM
The tricky part is the Dec comment could be 0, so even the rank is the same, but should consider no record in that month, so needs to surface it.

```sql
WITH dec_summary AS (
    SELECT
        country,
        SUM(number_of_comments) AS number_of_comments_dec,
        DENSE_RANK() OVER (ORDER BY SUM(number_of_comments) DESC) AS country_rank
    FROM fb_active_users AS a
    LEFT JOIN fb_comments_count AS b
    ON a.user_id = b.user_id
    WHERE created_at <= '2019-12-31' AND created_at >= '2019-12-01'
        AND country IS NOT NULL
    GROUP BY country
),
jan_summary AS (
    SELECT
        country,
        SUM(number_of_comments) AS number_of_comments_jan,
        DENSE_RANK() OVER (ORDER BY SUM(number_of_comments) DESC) AS country_rank
    FROM fb_active_users AS a
    LEFT JOIN fb_comments_count AS b
    ON a.user_id = b.user_id
    WHERE created_at <= '2020-01-31' AND created_at >= '2020-01-01'
        AND country IS NOT NULL
    GROUP BY country
)
SELECT j.country
FROM jan_summary j
LEFT JOIN dec_summary d ON d.country = j.country
WHERE (j.country_rank < d.country_rank) OR d.country IS NULL;
```


More questions: https://www.stratascratch.com/blog/facebook-data-scientist-interview-questions/
