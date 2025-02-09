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

https://platform.stratascratch.com/coding/10064-highest-energy-consumption?code_type=1 
- **Highest Energy Consumption**

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

https://platform.stratascratch.com/coding/2005-share-of-active-users?code_type=1 
- **Share of Active Users**

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
- **Rank Variance Per Country**
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

- **The tricky part is the Dec comment could be 0, so even the rank is the same, but should consider no record in that month, so needs to surface it.**

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

https://platform.stratascratch.com/coding/10288-clicked-vs-non-clicked-search-results?code_type=1
- **Clicked Vs Non-Clicked Search Results**
The 'position' column represents the position of the search results, and 'has_clicked' column represents whether the user has clicked on this result. Calculate the percentage of clicked search results that were in the top 3 positions. Also, calculate the percentage of non-clicked search results that were in the top 3 positions. Both percentages should be with respect to the total number of search records (all positions and both clicked and non-clicked searches). Output both percentages in the same row as two columns.

```sql
with base as (
select 
 count(case when search_results_position<=3 and clicked=1 then search_id end) as search_clicked_cnt
,count(case when search_results_position<=3 and clicked=0 then search_id  end) as search_nonclicked_cnt
,count( search_id ) as dem_search_cnt
from fb_search_events
)
select 1.00 * coalesce(search_clicked_cnt,0)/dem_search_cnt as clicked_perc
,1.00 * coalesce(search_nonclicked_cnt,0)/dem_search_cnt as nonclicked_perc
from base
 ;
 ```
- **The official solution**
```sql
SELECT (AVG(CASE
                WHEN search_results_position <=3
                     AND clicked = 1 THEN 1.0
                ELSE 0
            END)::float)*100 AS top_3_clicked,
       (AVG(CASE
                WHEN search_results_position <=3
                     AND clicked = 0 THEN 1.0
                ELSE 0
            END)::float)*100 AS top_3_notclicked
FROM fb_search_events
 ```

https://platform.stratascratch.com/coding/10061-popularity-of-hack?code_type=1
- **Popularity of Hack**
Meta/Facebook has developed a new programing language called Hack.To measure the popularity of Hack they ran a survey with their employees. The survey included data on previous programing familiarity as well as the number of years of experience, age, gender and most importantly satisfaction with Hack. Due to an error location data was not collected, but your supervisor demands a report showing average popularity of Hack by office location. Luckily the user IDs of employees completing the surveys were stored.
Based on the above, find the average popularity of the Hack per office location.
Output the location along with the average popularity.

```sql
select e.location
,avg(s.popularity) as avg_pop from facebook_hack_survey s
inner join facebook_employees e on s.employee_id = e.id
group by 1
;
```

