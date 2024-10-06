
# Case Study #4 - Data Bank
The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need. This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!




## 📚 Table of contents

 - [Task]()
 - [Entity relationship diagram]()
 - [Dataset]()
 - [Questions & solutions]()


## 🏷 Task

Data Bank runs just like any other digital bank - but it isn’t only for banking activities, they also have the world’s most secure distributed data storage platform!

Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help!

The management team at Data Bank want to increase their total customer base - but also need some help tracking just how much data storage their customers will need.

This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!


All the information about this case study [Case Study #4 - Data Bank](https://8weeksqlchallenge.com/case-study-4/)

## 🏷 Entity relationship diagram
[ER diagram](https://dbdiagram.io/d/Pizza-Runner-5f3e085ccf48a141ff558487?utm_source=dbdiagram_embed&utm_medium=bottom_open)

## 🏷 Dataset
Danny has shared with you 3 key datasets for this case study:

- Regions
- Customer Nodes
- Customer Transactions

all the tables are avilable over [here](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3)


##  🏷 Questions & solutions

### 📝  A. Customer Nodes Exploration:

1. How many unique nodes are there on the Data Bank system?
```bash 
select count(distinct(node_id)) from customer_nodes;
```

2. What is the number of nodes per region?
```bash 
select region_id, count((node_id)) from customer_nodes 
group by region_id;
```

3. How many customers are allocated to each region?
```bash 
select region_id, count(distinct customer_id) from customer_nodes 
group by region_id;
```

4. How many days on average are customers reallocated to a different node?
```bash 
with cte as
(select customer_id, node_id, end_date - start_date as diff 
from customer_nodes  WHERE end_date != '9999-12-31' group by node_id, customer_id,start_date, end_date),
cte2 as(
select customer_id, node_id, sum(diff) as days
from cte group by customer_id,node_id
)

select round(avg(days),0) from cte2;
```


5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
```bash 
with cte as(
select end_date - start_date as diff 
from customer_nodes  WHERE end_date != '9999-12-31' group by node_id, customer_id,start_date, end_date
)

select percentile_cont(0.95) within group(order by diff)
as median from cte ;

select percentile_cont(0.8) within group(order by diff)
as median from cte ;

select percentile_cont(0.5) within group(order by diff)
as median from cte ;
```



### 📝  B. Customer Transactions:

1. What is the unique count and total amount for each transaction type?
```bash
select txn_type,count(*), sum(txn_amount) from customer_transactions
group by txn_type
order by txn_type
```

2. What is the average total historical deposit counts and amounts for all customers?
```bash
with cte as(
select count(customer_id) as counts, avg(txn_amount) as amount from customer_transactions
where txn_type= 'deposit'
group by customer_id)

select round(avg(counts)),
round(avg(amount))
from cte;
```

3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```bash
select *, extract(month from txn_date) as month from customer_transactions

with cte as(select customer_id,extract(month from txn_date) as month,
sum(case when txn_type='deposit' then 1 else 0 end) as deposit,
sum(case when txn_type='purchase' then 1 else 0 end) as purchase,
sum(case when txn_type='withdrawal' then 1 else 0 end) as withdrawal
from customer_transactions
group by customer_id, month)

select month, count(distinct customer_id) from cte where
(deposit>1) and (purchase>=1 or withdrawal>=1)
group by month order by month;
```

4. What is the closing balance for each customer at the end of the month?
```bash
SELECT 
    customer_id, 
    (DATE_TRUNC('month', txn_date) + INTERVAL '1 MONTH - 1 DAY') AS closing_month, 
    SUM(CASE 
      WHEN txn_type = 'withdrawal' OR txn_type = 'purchase' THEN -txn_amount
      ELSE txn_amount END) AS transaction_balance
  FROM customer_transactions
  GROUP BY customer_id, txn_date 
  order by customer_id;
```