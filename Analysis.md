# Credit Card Transaction
# Q&A
**Author**: Kamal Nadh Lukka <br />
**Email**: kamallukka219@gmail.com <br />
**LinkedIn**: https://www.linkedin.com/in/kamallukka/ <br />
**download credit card transactions dataset from below link** : https://www.kaggle.com/datasets/thedevastator/analyzing-credit-card-spending-habits-in-india <br />

:exclamation: If you find this repository helpful, please consider giving it a :star:. Thanks! :exclamation:



** Exploring and Understanding  Dataset ** 

````sql
--changing table name as CCT

exec sp_rename '[dbo].[Credit_card_transactions$]', 'CCT';
select * from CCT;


--Exploring data 
select max(DATE) as maximumdate ,min(DATE)  as minimum_date  from cct;
select count(Gender) as gender_count ,Gender 
from cct 
group by gender;
````
** spliting names and country of city **

````sql
--spliting cities 
SELECT SUBSTRING(CITY,0,CHARINDEX(',',CITY)) as City_Name , city 
from cct
````

** write a query to print top 5 cities with highest spends   ** 
````sql
-- write a query to print top 5 cities with highest spends 
select  top 5 CITY,sum(Amount)  as amount_spent 
from CCT 
group by CITY
order by amount_spent desc
````

** and their percentage contribution of total credit card spends    ** 
````sql
--and their percentage contribution of total credit card spends 
with cities as 
(
select  top 5 CITY,sum(Amount)  as amount_spent 
from CCT 
group by CITY
order by amount_spent desc
)

select *,amount_spent /(select Sum(Amount) from cct)*100 as percentage1
from cities 
order by percentage1 desc


````
** write a query to print highest spend month and amount spent in that month for each card type  **
````sql
--write a query to print highest spend month and amount spent in that
--month for each card type
select * from CCT;
with mon as
(
select MONTH(date) as month , sum(Amount) as amount
from CCT
group by MONTH(date)
),
type1 as (
  select MONTH(date) as  mont , CARD_TYPE ,sum(Amount) as Amount
  from CCT
  group by MONTH(date),CARD_TYPE
)

select top 4 mon.month,type1.card_type,type1.Amount
from mon inner join type1 on mon.month=type1.mont
order by mon.month, type1.card_type, type1.Amount desc;
````

** write a query to print the transaction details(all columns from the table) for each card type when it reaches a cumulative of 1000000 total spends (We should have 4 rows in the o/p one for each card type)  **
````sql
-- write a query to print the transaction details(all columns from the table)
--for each card type when it reaches a cumulative of 1000000 total spends
--(We should have 4 rows in the o/p one for each card type)
EXEC sp_rename 'CCT.[INDEX]', 'T_ID', 'COLUMN';
with cum as (select *,sum(Amount) over(partition by card_type order by DATE,T_ID  ) cumulative_sum
from cct)

select * from (select *,rank() over (partition by card_type order by cumulative_sum )as rn  from cum where cumulative_sum>=1000000 ) as ranked where rn=1

````

** write a query to print highest spend month and amount spent in that
--month for each card typ  **

````sql

--write a query to print highest spend month and amount spent in that
--month for each card type
select * from CCT;
with mon as
(
select MONTH(date) as month , sum(Amount) as amount
from CCT
group by MONTH(date)
),
type1 as (
  select MONTH(date) as  mont , CARD_TYPE ,sum(Amount) as Amount
  from CCT
  group by MONTH(date),CARD_TYPE
)

select top 4 mon.month,type1.card_type,type1.Amount
from mon inner join type1 on mon.month=type1.mont
order by mon.month, type1.card_type, type1.Amount desc;
````

** write a query to find city which had lowest percentage spend for gold card type**
````sql
select top 1 city,sum(amount)/(select sum(Amount) as ss from CCT where CARD_TYPE='Gold' )*1.0 as GOLD_PERCENTAGE
from cct 
where CARD_TYPE='GOLD'  
group by CITY 
order by GOLD_PERCENTAGE
````
** -- write a query to print 3 columns: city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel) **

````sql
with cte as (select city,Type,sum(Amount) as expense from CCT  group by CITY,TYPE)
,cte2 as (select *,rank() over(partition by city order by expense desc) as high,
               rank() over(partition by city order by expense ) as low from cte )

select city,min(case when high=1 then type end )as high_expense , max(case when low=1 then type end) as low_expense from cte2 group by CITY 
````

**write a query to find percentage contribution of spends by females for each expense type**
````sql

select * from CCT
select CARD_TYPE , sum(Amount)/(select sum(Amount) from CCT) as percentage  
from CCT
where Gender='F'
group by CARD_TYPE
````

** which card and expense type combination saw highest month over month growth in Jan-2014 **
````sql
select  top 1 CARD_TYPE,TYPE ,sum(Amount) as spend
 from CCT
 where datepart(year,DATE)=2014 and datepart(MM,DATE)=01
 group by CARD_TYPE , TYPE
order by spend desc
````

** during weekends which city has highest total spend to total no of transcations ratio  **
````sql
select top 1 city , sum(amount)*1.0/count(1) as ratio
from CCT
where datepart(weekday,DATE) in (1,7)
group by city
order by ratio desc;

````

** which city took least number of days to reach its 500th transaction after the first transaction in that city **
````sql

with cte as (select *,rank() over (partition by city order by DATE ,T_ID) as trans from CCT)

select top 1 city ,datediff(day,min(date),max(date)) as numdays from cte where trans=1 or trans=500 group by CITY  having count(*)=2 order by numdays
````
