````markdown
# 🧼 SQL Sales Data Cleaning & Analysis Project

This project focuses on cleaning and analyzing a raw sales dataset using MySQL. The data originally exists in a table called `complex`, which we clean, transform, and analyze using SQL queries.

---

## 📥 1. Duplicate Raw Table
Create a clean copy of the original raw table.

```sql
select * from complex;

create table sales like complex;

insert sales select * from complex;
````

---

## 🧹 2. Remove Duplicates

Eliminate duplicate rows using `distinct`.

```sql
create table temp_sales as select distinct * from sales;

drop table sales;

rename table temp_sales to sales;
```

---

## 🔠 3. Standardize Text Format

* Lowercase `category`
* Uppercase `region`
* Capitalize `payment_method`

```sql
update sales 
set category = lower(category), 
    region = upper(region), 
    payment_method = concat(upper(left(payment_method, 1)), lower(substr(payment_method, 2)));
```

---

## 📆 4. Format Date

Ensure `order_date` is stored as a `DATE` datatype.

```sql
alter table sales modify order_date date;
```

---

## 🚫 5. Replace Empty Strings with NULL

Handle blank strings in key text columns.

```sql
update sales 
set customer_name = nullif(trim(customer_name), ''), 
    city = nullif(trim(city), ''), 
    phone_number = nullif(trim(phone_number), '');
```

---

## ❌ 6. Remove Incomplete Rows

Delete rows with `NULL` in important columns.

```sql
delete from sales 
where product is null or quantity is null or price is null or order_date is null;
```

---

## 💰 7. Add Total Sale Column

Add a calculated field for revenue after discounts.

```sql
alter table sales add column total_sale decimal(10,2);

update sales 
set total_sale = quantity * price * (1 - ifnull(discount_applied, 0)/100);
```

---

## 💸 8. Top Customers by Spending

Find who spent the most.

```sql
select customer_name, 
       sum(total_sale) as total_spent, 
       rank() over (order by sum(total_sale) desc) as spending_rank 
from sales 
where customer_name is not null 
group by customer_name;
```

---

## 🔁 9. Frequent Returners

List customers who returned more than once.

```sql
select customer_name, count(*) as return_count 
from sales 
where return_requested = 'Yes' 
group by customer_name 
having count(*) > 1;
```

---

## 🛍️ 10. Most Sold Product per Region

Using window function to find the best-seller by region.

```sql
with product_counts as (
    select region, product, count(*) as cnt,
           rank() over (partition by region order by count(*) desc) as rnk
    from sales 
    group by region, product
)
select region, product 
from product_counts 
where rnk = 1;
```

---

## 📅 11. Monthly Revenue

Revenue by month.

```sql
select date_format(order_date, '%Y-%m') as month, 
       sum(total_sale) as revenue 
from sales 
group by month 
order by month;
```

---

## ⏳ 12. First and Last Orders per Customer

Track customer lifecycle.

```sql
select a.customer_name, 
       min(a.order_date) as first_order, 
       max(b.order_date) as last_order 
from sales a 
join sales b on a.customer_name = b.customer_name 
group by a.customer_name;
```

---

## 🌍 13. First Order Percentage by Region

Analyze new customer activity.

```sql
select region, 
       round(sum(is_first_order = 'Yes')/count(*)*100, 2) as first_order_percent 
from sales 
group by region;
```

---

## 📦 14. Fill Missing Categories from Product Mapping

Use existing product–category mapping.

```sql
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
```

---

## 👶 15. Fill Missing Age with "Unknown"

Standardize unknown ages.

```sql
update sales
set customer_age = 'Unknown'
where customer_age is null or trim(customer_age) = '';
```

---

## 🧮 16. Remove Records with Zero Quantity

Zero quantity means no sale.

```sql
delete from sales where quantity = 0;
```

---

## 🧾 Final Check

Preview the cleaned table.

```sql
select * from sales;
```

---

## ✅ Summary

We:

* Removed duplicates and nulls
* Cleaned and standardized fields
* Derived metrics like `total_sale`
* Analyzed spending behavior, returns, and trends
* Used CTEs, window functions, joins, and subqueries
