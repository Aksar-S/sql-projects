-- 📦 Import and Structure Setup
select * from complex;

create table sales like complex;

select * from sales;

insert into sales
select * from complex;

-- 🧹 Remove Duplicates
create table temp_sales as
select distinct * from sales;

drop table sales;

rename table temp_sales to sales;

select * from sales;

-- 🧼 Standardize Text Formatting
update sales 
set category = lower(category), 
    region = upper(region), 
    payment_method = concat(upper(left(payment_method, 1)), lower(substr(payment_method, 2)));

select * from sales;

-- 📅 Fix Date Format
alter table sales modify order_date date;

-- 🔍 Nullify Blanks in Key Columns
update sales 
set customer_name = nullif(trim(customer_name), ''), 
    city = nullif(trim(city), ''), 
    phone_number = nullif(trim(phone_number), '');

-- 🗑️ Remove Rows with Missing Essential Data
delete from sales 
where product is null 
   or quantity is null 
   or price is null 
   or order_date is null;

select * from sales;

-- 💰 Add Calculated Field: Total Sale
alter table sales add column total_sale decimal(10,2);

update sales 
set total_sale = quantity * price * (1 - ifnull(discount_applied, 0)/100);

-- 🥇 Top Customers by Spending
select customer_name, 
       sum(total_sale) as total_spent, 
       rank() over (order by sum(total_sale) desc) as spending_rank 
from sales 
where customer_name is not null 
group by customer_name;

-- 🔁 Customers with Multiple Returns
select customer_name, 
       count(*) as return_count 
from sales 
where return_requested = 'Yes' 
group by customer_name 
having count(*) > 1;

-- 📦 Most Sold Product by Region
with product_counts as (
    select region, product, count(*) as cnt,
           rank() over (partition by region order by count(*) desc) as rnk
    from sales 
    group by region, product
)
select region, product 
from product_counts 
where rnk = 1;

-- 📆 Revenue by Month
select date_format(order_date, '%Y-%m') as month, 
       sum(total_sale) as revenue 
from sales 
group by month 
order by month;

-- 📅 First and Last Order by Customer
select a.customer_name, 
       min(a.order_date) as first_order, 
       max(b.order_date) as last_order 
from sales a 
join sales b 
  on a.customer_name = b.customer_name 
group by a.customer_name;

-- 📊 First Order Percentage by Region
select region, 
       round(sum(is_first_order = 'Yes') / count(*) * 100, 2) as first_order_percent 
from sales 
group by region;

-- 🧠 Auto-Fill Missing Category by Product
update sales s1
join (
    select product, category
    from sales
    where category is not null and trim(category) != ''
    group by product, category
) as reference
on s1.product = reference.product
set s1.category = reference.category
where s1.category is null or trim(s1.category) = '';

-- 🧑 Handle Missing Customer Age
update sales
set customer_age = 'Unknown'
where customer_age is null or trim(customer_age) = '';

-- 🗑️ Remove Rows with Quantity = 0
delete from sales 
where quantity = 0;

-- ✅ Final View
select * from sales;
