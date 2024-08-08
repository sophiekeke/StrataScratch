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




