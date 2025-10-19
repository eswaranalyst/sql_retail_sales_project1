# Retail Sales Analysis SQL Project

## Project Overview

**Project Title**: Retail Sales Analysis  
**Database**: `retail_sales`

## Objectives

1. **Set up a retail sales database**
    . Created and populated a retail sales database with the provided sales data.
2. **Data Cleaning**:
    . Identified and removed  with missing or null values to ensure data accuracy
3. **Business Analysis**:
    . Answered key business questions such as top-selling products, revenue trends, and customer segments.

## Project Structure

### 1. Database Setup

- **Database Creation**: A new database named `retail_sales` was created to store and manage all retail transaction data.
- **Table Creation**: A table named `Retail_sales` is created to store the sales data. The table structure includes columns for transaction_id, transaction_date, transaction_time,	customer_id, product_id, product_category, quantity, price, total_amount, payment_method,	store_location.

```sql
Create Table
    Retail_sales
           (transaction_id	INT,
            transaction_date DATE,	
            transaction_time TIME,
            customer_id	INT,
            product_id	INT,
            product_category VARCHAR(25),
            quantity INT,
            price	FLOAT,
            total_amount FLOAT,
            payment_method	VARCHAR(25),
            store_location  VARCHAR(25)
            );
            
Select*
from retail_sales;

select count(*)
   from retail_sales;
            
```

### 2. Data Exploration & Cleaning
```sql
select * from retail_sales
where
    transaction_id is null
    or
    transaction_date is null
    or
	transaction_time is null
    or
    customer_id is null
    or
    product_id is null
    or
    product_category is null
    or
    quantity is null
    or
    price is null
    or
    total_amount is null
    or
    payment_method is null
    or
    store_location is null;
```

### 3. Data Analysis & Findings

A series of SQL queries were written to analyze the retail sales data and answer critical business questions. These queries focus on identifying sales trends, top-performing products, customer behavior, and revenue insights to support data-driven decision-making.

1. **Calculate a rolling total of sales (total_amount) per product_category ordered by transaction_date.**:
```sql
select 
     transaction_Id,
     transaction_date,
     product_Category,
     total_amount,
     sum(total_amount)over(partition by product_Category order by transaction_date,transaction_id 
     ) as rolling_total
   from retail_sales
   order by product_category,transaction_date;
```

2. **Find the top 3 transactions by total_amount for each store_location.**:
```sql
select
     transaction_id,
     store_location,
     customer_id,
     product_category,
     total_amount
from (
	 SELECT
	 transaction_id,
     store_location,
     customer_id,
     product_category,
     total_amount,
     row_number()OVER(
     partition by store_location
     order by total_amount desc
     ) as rn
 from retail_sales
     ) as ranked_sales
 where rn <=3
  order by store_location,rn;
```

3. **List all transactions where the total_amount is greater than the average total_amount of the dataset.**:
```sql
select*
from retail_sales
where total_amount > (select
                      avg(total_amount)
                      from retail_sales);
```

4. **Show product categories with total sales > 10,000.**:
```sql
select
	product_category,
    sum(total_amount) as total_sales
from retail_sales
group by product_category
having sum(total_amount) > 10000;
```

5. **Create a CTE that calculates monthly sales and then select the month with the highest sales.**:
```sql
with monthly_sales as (
	select
    date_format(transaction_date,'%Y-%m')as month,
    sum(total_amount) as total_sales
from retail_sales
	group by date_format(transaction_date,'%Y-%m')
    )
select
      month,total_sales
from monthly_Sales
order by total_sales desc
limit 1;
```

6. **Add a Column sales_type in query - Keep'HIGH' If total amount > 500.'medium'if total amount is between 200 and 500,'low' if total amount < 500.**:
```sql
select 
	case 
		when total_amount > 500 then 'HIGH'
        When total_amount between 200 and 500 then 'MEDIUM'
        else 'LOW'
	END AS sales_type,
count(*) as transaction_count
from retail_sales
group by 
    case 
        when total_amount > 500 then 'HIGH'
        When total_amount between 200 and 500 then 'MEDIUM'
        else 'LOW'
     end;
```

7. **Categorize transactions by time: 'Morning' for 06:00:00–11:59:59,'Afternoon' for 12:00:00–17:59:59,'Evening' for 18:00:00–21:59:59,'Night' for 22:00:00–05:59:59.**:
```sql
select 
	transaction_id,
    transaction_time,
	case
		when time(transaction_time) between '06:00:00' and'11:59:59' then 'MORNING'
        When time(transaction_time) between '12:00:00' and'17:59:59' then 'AFTERNOON'
        WHEN Time(transaction_time) between '18:00:00' and'21:59:59' then 'EVENING'
        else 'NIGHT'
    End AS Time_Of_The_Day
from retail_sales;
```

8. **For each transaction, display transaction_id and the average total_amount of all transactions in the same store_location.**:
```sql
SELECT 
    transaction_id,
    store_location,
    total_amount,
    AVG(total_amount) OVER (PARTITION BY store_location) AS avg_store_sales
FROM retail_sales;
```

9. **Show the store with the highest average transaction amount.**:
```sql
select
	store_location,avg_transaction
from(
     select
		store_location,
        avg(total_amount) as avg_transaction
	 from retail_sales
     group by store_location
     )t
order by avg_transaction desc
limit 1;

```

10. **Using a CTE for monthly totals, join it with store_locations to see which store contributes most each month.**:
```sql
With monthly_Store_sales as (
	select 
		date_format(transaction_date,'%Y-%m') as Month,
        store_location,
        sum(total_amount)as total_sales
	from retail_sales
    group by date_format(transaction_date,'%Y-%m'),store_location
    )
select m.month,
	   m.store_location,
       m.total_sales
from(
	select
		month,
        store_location,
        total_sales,
        rank()over(partition by store_location order by total_sales desc) as sales_rank
	from monthly_Store_sales
    )m
where m.sales_rank =1;
```
11. **Show each store's sales as a percentage of total sales.**
```sql
select
	store_location,
    sum(total_amount) as store_sales,
    round(sum(total_amount)/(select sum(total_amount) from retail_sales) *100,2) as sales_percentage 
from retail_sales
group by store_location
order by sales_percentage desc;
```
12. **Retrieve transactions where quantity > average quantity for that product category.**
```sql
select *
 from (
	select*,
    avg(quantity)over(partition by product_category)as avg_quantity
    from retail_sales
    )t
    where quantity > avg_quantity;
```
13. **Find peak hour of transactions (hour with most transactions) across all days.**
```sql
select 
	hour(transaction_time) as transaction_hour,
    count(*) as transaction_count
from retail_sales
group by hour(transaction_time)
order by transaction_count desc
limit 1;
```


## Findings

- **Customer Behavior: Most transactions fall under the “LOW” range <$200 .
- **Peak Hours: Sales peak between 12 PM and 5 PM, suggesting strong afternoon traffic.
- **Store With Highest Transaction**: The analysis revealed that the Los Angeles store location recorded the highest total transaction amount across all branches. This suggests that Los Angeles is a key revenue hub, driven by high customer traffic, larger transaction       values, and consistent sales performance compared to other locations.
- **Best Product Category**: The analysis reveals that the electronics category achieved the highest sales.

## Reports

- **Sales Summary**: The retail sales dataset records over 2,000 transactions, generating significant revenue across all stores. Electronics, Clothing  are the top-selling categories, while the Los Angeles store contributes the highest total sales.Peak transaction         activity occurs between 12 PM and 5 PM, with high-value transactions driving a large portion of overall revenue.
- **Trend Analysis**: Electronics and Clothing consistently dominate revenue, while other categories maintain steady sales. Furthermore, a small number of top-performing stores account for the majority of total sales, illustrating a strong concentration of revenue in       key locations.
- **Store Sales By Percentage**: The top 3 store locations that is Los Angeles with 15.24%, New York with 13.14%, Houston with 12.96%.

## Conclusion

The retail sales analysis demonstrates key insights into store performance, product categories, and customer behavior. Electronics, Clothing  are the top-selling categories, with high-value transactions contributing disproportionately to revenue. Morning and Afternoon periods, especially 11-5 PM, are peak transaction times, while a few top-performing stores, like Los Angeles, drive the majority of total sales. These findings highlight opportunities for data-driven decisions in marketing, staffing, and inventory management.

