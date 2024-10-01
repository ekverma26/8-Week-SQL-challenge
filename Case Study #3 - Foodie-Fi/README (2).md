
# Case Study #3 - Foodie-Fi
Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.




## 📚 Table of contents

 - [Task]()
 - [Entity relationship diagram]()
 - [Dataset]()
 - [Questions & solutions]()


## 📌 Task

Danny has a startup 'Foodie-Fi' and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.. 


All the information about this case study [Case Study #3 - Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)

## 📌 Entity relationship diagram
[ER diagram](https://dbdiagram.io/d/Pizza-Runner-5f3e085ccf48a141ff558487?utm_source=dbdiagram_embed&utm_medium=bottom_open)

## 📌 Dataset
Danny has shared with you 2 key datasets for this case study:

- plans
- subscriptions

Both the tables are avilable over [here](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16)


##  📌 Questions & solutions

### 🖍  A. Customer Journey:

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

```bash
SELECT
  sub.customer_id,
  plans.plan_id, 
  plans.plan_name,  
  sub.start_date
FROM foodie_fi.plans
JOIN foodie_fi.subscriptions AS sub
  ON plans.plan_id = sub.plan_id
WHERE sub.customer_id IN (1,2,11,13,15,16,18,19);
```

Explanation:

Based on the results above, I have selected three customers to focus on and will now share their onboarding journey.

Customer 1: This customer initiated their journey by starting the free trial on 1 Aug 2020. After the trial period ended, on 8 Aug 2020, they subscribed to the basic monthly plan.

Customer 13: The onboarding journey for this customer began with a free trial on 15 Dec 2020. Following the trial period, on 22 Dec 2020, they subscribed to the basic monthly plan. After three months, on 29 Mar 2021, they upgraded to the pro monthly plan.

Customer 15: Initially, this customer commenced their onboarding journey with a free trial on 17 Mar 2020. Once the trial ended, on 24 Mar 2020, they upgraded to the pro monthly plan. However, the following month, on 29 Apr 2020, the customer decided to terminate their subscription and subsequently churned until the paid subscription ends.

### 🖍  B. Data Analysis Questions:

1. How many customers has Foodie-Fi ever had?
```bash 
select count(distinct(customer_id)) from subscriptions;
```
  

2. What is the monthly distribution of trial plan start_date values for our dataset use the start of the month as the group by value?
```bash 
select extract(month from s.start_date) as month,
count(s.customer_id) as customers from subscriptions as s
join plans as p on p.plan_id=s.plan_id where
s.plan_id=0
group by month order by month;
```

3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name.
```bash 
with s as(select customer_id, plan_id, start_date from subscriptions where
extract(year from start_date) > 2020)

select p.plan_id, p.plan_name, count(s.customer_id) from 
plans as p join s on
p.plan_id=s.plan_id 
group by p.plan_id,p.plan_name
order by p.plan_id;
```
4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

```bash 
select p.plan_name, count(distinct(s.customer_id)) ,
ROUND(100.0 * COUNT(s.customer_id)/ (SELECT COUNT(DISTINCT customer_id) from subscriptions), 1) as percentages 
from subscriptions as s join plans as p on p.plan_id=s.plan_id where p.plan_name='churn'
group by p.plan_name;
```


5. How many customers have churned straight after their initial free trial what percentage is this rounded to the nearest whole number?
```bash 
with cte as(
select s.customer_id, p.plan_id, 
row_number() over(partition by s.customer_id order by s.start_date) as row_n
from subscriptions as s
join plans as p on p.plan_id=s.plan_id
)

select 
count(case when row_n=2 and plan_id=4 then 1 else 0 end) as count_customers,
ROUND(100.0 * COUNT(case when row_n=2 and plan_id=4  then 1 else 0 end)/ (SELECT COUNT(DISTINCT customer_id) 
from subscriptions)) as percentages 
from cte 
where plan_id=4 and row_n=2;
```

6.  What is the number and percentage of customer plans after their initial free trial?
```bash 
with cte as(
select customer_id,plan_id, 
lead(plan_id) over(partition by customer_id order by plan_id) as next_plan
from subscriptions 
)

select next_plan,
count(customer_id) as c_customrs,
round(100* count(customer_id)/(select count(distinct(customer_id)) from subscriptions), 1) as percentage
from cte
where next_plan is not null and plan_id=0
group by next_plan
order by next_plan;
```

7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```bash 
with cte as(
select s.customer_id, p.plan_id, p.plan_name, s.start_date,
lead(s.start_date) over (partition by customer_id  order by start_date) as next_date
from subscriptions as s
join plans as p on p.plan_id=s.plan_id where s.start_date<='2020-12-31'
)

select plan_name, count(distinct customer_id),
round(100.0* count(distinct customer_id)/(select count(distinct(customer_id)) from subscriptions), 1) as percentage
from cte where next_date is null
group by plan_name ;
```


8.  How many customers have upgraded to an annual plan in 2020?
```bash 
select count(distinct customer_id) from subscriptions where plan_id=3 and 
start_date<='2020-12-31';
```

9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
```bash 
with cte1 as(
select customer_id, plan_id, start_date  from
subscriptions where plan_id=0
), 
cte2 as(
select customer_id, plan_id, start_date  from
subscriptions where plan_id=3
)

select round(avg(c2.start_date - c1.start_date),0) from cte1 as c1
join cte2 as c2 on c1.customer_id=c2.customer_id;
```


10.  Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)?
```bash 
with cte1 as(
select customer_id, plan_id, start_date  from
subscriptions where plan_id=0
), 
cte2 as(
select customer_id, plan_id, start_date  from
subscriptions where plan_id=3
),
bin as (
select width_bucket(c2.start_date - c1.start_date, 0,365,12) as days
from cte1 as c1 join cte2 as c2 
on c1.customer_id=c2.customer_id
)

select 
((days-1)*30 || '-' || (days*30) || ' days') as day,
count(*) as customers from bin group by days
order by days;
```

11.  How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```bash 
with cte as(
select s.customer_id, p.plan_id, s.start_date,
lead(p.plan_id) over (partition by s.customer_id  order by s.start_date) as next_date
from subscriptions as s
join plans as p on p.plan_id=s.plan_id where extract(year from s.start_date)=2020
)

select count(customer_id) from cte
where plan_id=2 and next_date=1;
```

