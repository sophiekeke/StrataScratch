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







***StrataScratch SQL***

**StrataScratch**  

https://platform.stratascratch.com/coding/10064-highest-energy-consumption?code_type=1 Highest Energy Consumption

Find the date with the highest total energy consumption from the Meta/Facebook data centers. Output the date along with the total energy consumption across all data centers.

with union_all as ( 
    select * from fb_eu_energy 
    union all 
    select * from fb_asia_energy 
    union all 
    select * from fb_na_energy
), 
agg as ( 
    select date, 
           sum(consumption) as tot_consumption 
    from union_all 
    group by 1
), 
rank_rs as ( 
    select dense_rank() over (order by tot_consumption desc ) as rank_num,
           * 
    from agg
) 
select date,
       tot_consumption 
from rank_rs 
where rank_num=1 
order by date;

https://platform.stratascratch.com/coding/2005-share-of-active-users?code_type=1 Share of Active Users

Output share of US users that are active. Active users are the ones with an "open" status in the table.

with base as ( 
    select * from fb_active_users 
    where country = 'USA'
) 
select 1.00 * count(distinct case when status='open' then user_id end) / count(distinct user_id ) as ratio 
from base ;




