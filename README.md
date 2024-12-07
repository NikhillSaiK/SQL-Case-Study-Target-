# SQL-Case-Study-Target

1.Import the dataset and do usual exploratory analysis steps like checking the 
structure & characteristics of the dat 
aset: 
A.Data type of all columns in the "customers" table. 
Ans: select COLUMN_NAME,DATA_TYPE from target.INFORMATION_SCHEMA.COLUMNS 
where table_name='customers' 
Insights:  There are two types of datatypes and mostly are of string type. 


1B.Get the time range between which the orders were placed. 
Ans:  select min(order_purchase_timestamp) as first_day_of_orders, 
max(order_purchase_timestamp) as last_day_of_orders from `target.orders` 
Insights: The orders were placed from 4th september 2016 to 17th october 2018. 


1c.Count the Cities & States of customers who ordered during the given period. 
Ans: select count(distinct customer_city) as num_of_cities, 
count(distinct customer_state) as num_of_states  
from `target.customers` 
Insights: There are 4119 cities and 27 states. 


2.In-depth Exploration: 
A.Is there a growing trend in the no. of orders placed over the past years? 
Ans: select year_orders,count(order_id) as no_of_orders from  
(select *,extract(year from order_purchase_timestamp) as year_orders, 
from `target.orders`) 
group by year_orders 
order by year_orders 
Insights: As we see in the above table no.of.orders have been increased from 2016 to 
2018. There are only 329 orders in 2016 
and the orders have increased to 54011 in 2018. 


2B.Can we see some kind of monthly seasonality in terms of the no. of orders being 
placed? 
Ans:  select extract(month from order_purchase_timestamp) as month_orders 
,count(order_id) as order_count from `target.orders` 
group by month_orders 
order by month_orders  
Insights: We can see the monthly seasonality at the months of May, July and August. 


2C.During what time of the day, do the Brazilian customers mostly place their orders? 
(Dawn, Morning, Afternoon or Night) 
• 0-6 hrs : Dawn 
• 7-12 hrs : Mornings 
• 13-18 hrs : Afternoon 
• 19-23 hrs : Night 
Ans:select timeline,count(order_id) as no_of_orders from (select case when hours 
between 0 and 6 then 1 when hours between 7 and    
12 then 2 when hours between 13 
and 18 then 3 
when hours between 19 and 23 then 4 end as timeline,order_id from  
(select *,extract(hour from order_purchase_timestamp) as hours from 
`target.orders`)x)y 
group by timeline 
order by timeline 
Insights: The Brazilian customers do mostly place their orders at Afternoon(13-18 hrs)  


3.Evolution of E-commerce orders in the Brazil region 
A.Get the month on month no. of orders placed in each state. 
Ans: select customer_state,years,months,count(order_id) as no_of_orders from ( 
select customer_state,order_id,extract(year from order_purchase_timestamp) as years 
, 
extract(month from order_purchase_timestamp)  
as months from `target.customers` c join `target.orders` o  
on c.customer_id=o.customer_id)x  
group by x.customer_state,years,months 
order by years,months 
Insights: No of orders placed in each state during each month from 2016 to 2018. 


3B.How are the customers distributed across all the states? 
Ans:select customer_state,count(distinct customer_id) no_of_customers from 
`target.customers` 
group by customer_state 
Insights: This is the table of unique number of customers in each state and SP state has 
most unique customers. 


4.Impact on Economy: Analyze the money movement by e-commerce by looking at 
order prices, freight and others. 
A.Get the % increase in the cost of orders from year 2017 to 2018 (include months 
between Jan to Aug only). 
You can use the "payment_value" column in the payments table to get the cost 
of orders. 
Ans:with x as ( 
SELECT Round(Sum(pay.payment_value),2) as sum2017 FROM `target.orders` as ord 
JOIN `target.payments` as pay using (order_id) 
Where Extract(Year from ord.order_purchase_timestamp) = 2017 
and Extract(Month from ord.order_purchase_timestamp) BETWEEN 1 and 8 
), 
y as ( 
SELECT Round(Sum(pay.payment_value),2) as sum2018 FROM `target.orders` as ord 
JOIN `target.payments` as pay using (order_id) 
Where Extract(Year from ord.order_purchase_timestamp) = 2018 
and Extract(Month from ord.order_purchase_timestamp) BETWEEN 1 and 8 
) 
Select sum2018 as Sumof2018,sum2017 as Sumof2017, Round((sum2018- 
sum2017)/sum2017*100,2) as increased_percentage from x,y 
Insights: The % increase in the cost of orders from year 2017 to 2018 is 136.98. 


4B.Calculate the Total & Average value of order price for each state. 
Ans:select customer_state,avg(price) as average_price,sum(price) as Total_price 
from `target.orders` o join `target.customers` c 
on o.customer_id=c.customer_id 
join `target.order_items` oi on o.order_id=oi.order_id 
group by customer_state 
order by customer_state 
Insights: These are the Total and Average value of order price for each state.  


4C.Calculate the Total & Average value of order freight for each state. 
Ans:select customer_state,avg(freight_value) as 
average_freight_value,sum(freight_value) as Total_freight_value from 
`target.orders` o join `target.customers` c on o.customer_id=c.customer_id 
join `target.order_items` oi on o.order_id=oi.order_id 
group by customer_state 
order by customer_state 
Insights: These are the Total and Average value of order freight for each state 


5.Analysis based on sales, freight and delivery time. 
A.Find the no. of days taken to deliver each order from the order’s purchase date as 
delivery time.Also, calculate the difference (in days) between the estimated & 
actual delivery date of an order.Do this in a single query. 
Ans:select order_id,customer_id, 
date_diff(order_delivered_customer_date,order_purchase_timestamp,day) as 
time_to_deliver, 
date_diff(order_estimated_delivery_date,order_delivered_customer_date,day) as 
diff_estimated_delivery, 
case when order_delivered_customer_date<order_estimated_delivery_date then 'Early' 
else 'Late' end as Delivery_status 
from `target.orders` 
Insights: The time to delivery is approx 30 to 35 days . In the diff_estimated_delivery 
column the negative sign represents late and the positive represents early delivery .  


5.B.Find out the top 5 states with the highest & lowest average freight value. 
Ans: select * from (select customer_state,avg(freight_value) as 
average_freight_value from `target.orders` o join `target.customers`  
c on o.customer_id=c.customer_id 
join `target.order_items` oi on o.order_id=oi.order_id 
group by customer_state 
order by average_freight_value desc  
limit 5) x 
union all 
select * from (select customer_state,avg(freight_value) as average_freight_value 
from `target.orders` o join `target.customers`  
c on o.customer_id=c.customer_id 
join `target.order_items` oi on o.order_id=oi.order_id 
group by customer_state 
order by average_freight_value   
limit 5)y 
Insights: The first five are the highest average freight values and the next five are the 
lowest average freight values. 


5.C.Find out the top 5 states with the highest & lowest average delivery time. 
Ans:select * from (select 
customer_state,avg(date_diff(date(order_delivered_customer_date),date(order_purchas
 e_timestamp),day)) as average 
from `target.orders` o join `target.customers` c  
on c.customer_id=o.customer_id 
group by customer_state 
order by average desc 
limit 5)y  
union all 
select * from (select 
customer_state,avg(date_diff(date(order_delivered_customer_date),date(order_purchas
 e_timestamp),day)) as 
average 
from `target.orders` o join `target.customers` c  
on c.customer_id=o.customer_id 
group by customer_state 
order by average  
limit 5) z 
Insights: Top 5 are the highest average delivery time states and remaining are the 
lowest average delivery time states.  


5.D.Find out the top 5 states where the order delivery is really fast as compared to the 
estimated date of delivery. 
You can use the difference between the averages of actual & estimated delivery date to 
figure out how fast the delivery was for each state. 
Ans:select 
customer_state,avg(date_diff(order_estimated_delivery_date,order_delivered_customer
 _date,day)) as average 
from `target.orders` o join `target.customers` c  
on c.customer_id=o.customer_id 
group by customer_state 
order by average desc 
limit 5 
Insights: These are the top 5 states where the order delivery is really fast as compared to 
the estimated date of delivery. 


6.Analysis based on the payments: 
A.Find the month on month no. of orders placed using different payment types. 
Ans: 
select extract(year from order_purchase_timestamp) years, extract(month from 
order_purchase_timestamp) as months,payment_type, 
count(p.order_id) as count_order from `target.orders` o join `target.payments` p on  
o.order_id=p.order_id 
group by years,months,payment_type 
order by years,months,payment_type 
Insights:The month on month no. of orders placed using different payment types like 
creditcard and upi . Most of them are using credit card. 


6B.Find the no. of orders placed on the basis of the payment installments that have 
been paid. 
Ans:select payment_installments,count(distinct order_id) as order_count from 
`target.payments` 
group by payment_installments 
order by payment_installments 
Insights: This is the no. of orders placed on the basis of the payment installments that 
have been paid.Most of them are paying in one time installment.
