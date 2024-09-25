
# Case Study #1 - Danny's Diner
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 




## Table of contents

 - [Task]()
 - [Entity relationship diagram]()
 - [Dataset]()
 - [Questions & solutions]()


## Task

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite.

All the information about this case study [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

## Entity relationship diagram

![er1](https://github.com/user-attachments/assets/9c964ef2-7a96-4bf3-81cd-5ec0da575302)

## Dataset
Danny has shared with you 3 key datasets for this case study:

- sales
- menu
- members

All three tables are avilable over [here](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138)
## Questions & solutions
1. What is the total amount each customer spent at the restaurant?

```bash
select s.customer_id, sum(m.price) from sales as s join
menu as m on s.product_id=m.product_id group by s.customer_id order by s.customer_id;
```
2. How many days has each customer visited the restaurant?

```bash
select customer_id, count(distinct(order_date)) from sales group by customer_id;
```
3.What was the first item from the menu purchased by each customer?
```bash
with cte as(select s.customer_id,s.product_id, row_number() over(partition by s.customer_id order by s.customer_id) 
as r from sales s )

select c.customer_id, m.product_name from cte as c join
menu as m on c.product_id=m.product_id where r=1;
```
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```bash
with cte as(select product_id as id, count(product_id) as n from sales group by product_id order by n desc limit 1)

select m.product_name, c.n from menu as m inner join
cte as c on m.product_id=c.id;
```
5. Which item was the most popular for each customer?
```bash
with cte as(select customer_id, product_id as id, count(product_id) as n from sales 
group by customer_id, product_id order by customer_id, n desc),
cte1 as(select customer_id,id, n, rank() over(partition by customer_id order by n desc) as r from cte)

select m.product_name, c.customer_id from menu as m inner join
cte1 as c on m.product_id=c.id and r=1 order by customer_id;
```
6. Which item was purchased first by the customer after they became a member?
```bash
with cte as(select m1.customer_id,s.product_id, s.order_date-m1.join_date as days from members as m1 inner join
sales as s on m1.customer_id=s.customer_id where s.order_date-m1.join_date>=0),
cte1 as(select customer_id,product_id,days, 
rank() over (partition by customer_id order by days) as r from cte)

select c.customer_id, m.product_name as afterjoined from menu as m join
cte1 as c on m.product_id=c.product_id where r=1 order by c.customer_id;

```
7. Which item was purchased just before the customer became a member?
```bash
with cte as(select m1.customer_id,s.product_id, s.order_date-m1.join_date as days from members as m1 inner join
sales as s on m1.customer_id=s.customer_id where s.order_date-m1.join_date<0)
,cte1 as(select customer_id,product_id,days, 
rank() over (partition by customer_id order by days desc) as r from cte)

select distinct(c.customer_id), m.product_name as afterjoined from menu as m join
cte1 as c on m.product_id=c.product_id where r=1 order by c.customer_id;

```
8. What is the total items and amount spent for each member before they became a member?
```bash
with cte as(select m1.customer_id,s.product_id, s.order_date-m1.join_date as days from members as m1 inner join
sales as s on m1.customer_id=s.customer_id where s.order_date-m1.join_date<0)

select c.customer_id, count((c.product_id)) as total_item, sum(m.price) as total_price
from cte as c join menu as m
on c.product_id=m.product_id
group by c.customer_id order by c.customer_id;

```
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```bash
with cte as(select product_id, product_name,
case when product_name='sushi' then price*20
else price*10
end as points 
from menu)
select s.customer_id, sum(c.points) as total_points from sales as s
inner join cte as c on s.product_id=c.product_id group by s.customer_id order by s.customer_id;

```
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```bash
with cte as(select m1.customer_id,s.product_id, s.order_date-m1.join_date as days from members 
as m1 inner join
sales as s on m1.customer_id=s.customer_id where s.order_date-m1.join_date between 0 and 7)
, cte1 as(select product_id, product_name,
price*20 as points 
from menu)

select c.customer_id, sum(c1.points) as total_points from cte as c
inner join cte1 as c1 on c.product_id=c1.product_id group by c.customer_id order by c.customer_id;
```
## Bonus Questions

1. Join All The Things:

```bash
select s.customer_id, s.order_date, m.product_name, m.price,
case when m1.join_date>s.order_date then 'N'
when m1.join_date<=s.order_date then 'Y'
else 'N' end as membership
from sales as s left join
members as m1 on s.customer_id=m1.customer_id
inner join menu as m on s.product_id=m.product_id
order by s.customer_id;
```

2. Rank All The Things:
Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
```bash
with cte as(
select s.customer_id, s.order_date, m.product_name, m.price,
case when m1.join_date>s.order_date then 'N'
when m1.join_date<=s.order_date then 'Y'
else 'N' end as membership
from sales as s left join
members as m1 on s.customer_id=m1.customer_id
inner join menu as m on s.product_id=m.product_id
order by s.customer_id
)

select *, case when
membership='N' then null
else rank() over (partition by customer_id,membership order by order_date) 
end as rank
from cte;
```
