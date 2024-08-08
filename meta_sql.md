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
