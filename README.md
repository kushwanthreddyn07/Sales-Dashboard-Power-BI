# SQL - Power BI Sales Dashboard  


## Introduction   


Atliq Hardware is a computer hardware manufacturing company that has customers globally. In this project, I have created a dynamic sales dashboard designed to provide you with a comprehensive view of your sales performance through Key Performance Indicators (KPIs), while also giving you the power to delve deeper into your product and customer data.  


## Problem Statement   


All sales data is in  7 different fact tables and 2 dimension tables dim_product and dim_customer.

![tables](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/2912f4c8-b3b1-450f-9f22-1f8c601de92e)

First, need to extract all the useful data into the Power BI platform.

![datamodel23](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/fc97727b-13a2-42df-8cc3-a4e62d11296f)


There are 7 fact tables, and information from each table is needed. The current data model we have is very complex and hard to understand. There are so many relations between all tables and many more relations will be needed to connect the unconnected tables. So, it's better to create a data model with a star schema. Star schemas are straightforward and easy to understand. They have a central fact table connected to dimension tables, making it simple to navigate and query the data. And also star schemas boost performances.  


## Skills Demonstrated  


The following features were used:-
1. SQL:- Joins, Cte
2. Power BI:- Power Query, Modelling, Calculated Columns, Bookmarks, DAX, Buttons, Filters, Measures, Page Navigations, Tab Navigations.
3. 

## Data Transformation   


1. As we have seen in the above picture, we have a messy data model, to get a clean and understandable data model data transformation must be done.
   
2. In the data model, every table has a relationship with at least one other table except the fact_freight_cost table. If we observe carefully fact_freight_cost table has a market column and the dim_customer table also has a market column, so these two tables can be connected, but freight cost will also be changed based on fiscal year. We need to connect the fact_freight_cost table with market and fiscal_year, then we will get the correct data.
   
3. We can take fact_sales_monthly tables as the base table because it has all the sales transaction data, and join all the other tables such that we will only have one fact table with all the required data.

**SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)** 

*-- Joining fact_sales_monthly and fact_act_est tables tables on fiscal_year,product_code,customer_code to fetch forecast quantity of products. Performing left join to fetch all transactions from the fact_sales_monthly table*

3. Using CTEs, as we have to join multiple tables continuously.

**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code))  
SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)**

*--Joining cte1(previously joined table) and fact_pre_invoice_deductions tables to fetch pre_invoice_discount_percent of products. Performing left join to fetch all transactions from the left table*



**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year))  
SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code)**

*--Joining cte2(previously joined table) and fact_post_invoice_deductions tables to fetch discount_pct and other_deductions_pct of products. Performing left join to fetch all transactions from the left table*



**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)),  
cte3 as (SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code))  
SELECT * FROM cte3 LEFT JOIN fact_gross_price gp USING(product_code,fiscal_year)**

*--Joining cte3(previously joined table) and fact_gross_price tables using product_code and fiscal_year to fetch the products gross_price.Performing left join to fetch all transactions from the left table*



**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)),  
cte3 as (SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code)),  
cte4 as (SELECT * FROM cte3 LEFT JOIN fact_gross_price gp USING(product_code,fiscal_year))  
SELECT c4.*,c.market  FROM cte4 c4 LEFT JOIN dim_customer c USING(customer_code)**

*--Joining cte4(previously joined table) and dim_customer tables to fetch the market of customers. We already have the fiscal year column in the cte4 table, if we bring the market column into this table using the customer code column, then I can join the fact_freight_cost table easily. c4* means all columns from the cte4 table and c.market means only market column from dim_customer table. Performing left join to fetch all transactions from the left table*



**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)),  
cte3 as (SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code)),  
cte4 as (SELECT * FROM cte3 LEFT JOIN fact_gross_price gp USING(product_code,fiscal_year)),  
cte5 as (SELECT c4.*,c.market  FROM cte4 c4 LEFT JOIN dim_customer c USING(customer_code))  
SELECT * FROM cte5 c5 LEFT JOIN fact_freight_cost fc USING(market,fiscal_year)**

*--Joining cte5(previously joined table) and fact_freight_cost tables using market and fiscal_year to fetch freight_pct and other_cost_pct of products.  Performing left join to fetch all transactions from the left table*



**with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)),  
cte3 as (SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code)),  
cte4 as (SELECT * FROM cte3 LEFT JOIN fact_gross_price gp USING(product_code,fiscal_year)),  
cte5 as (SELECT c4.*,c.market  FROM cte4 c4 LEFT JOIN dim_customer c USING(customer_code)),  
cte6 as (SELECT * FROM cte5 c5 LEFT JOIN fact_freight_cost fc USING(market,fiscal_year))  
SELECT * FROM cte6 c6 LEFT JOIN fact_manufacturing_cost mc USING(product_code,fiscal_year)**

*--Joining cte6(previously joined table) and fact_manufacturing_cost tables using product_code and fiscal_year to fetch manufacturing_cost of products.  Performing left join to fetch all transactions from the left table*

**CREATE VIEW fact_transactions as (with cte1 as (SELECT s.date,s.fiscal_year,s.product_code,s.customer_code,s.sold_quantity,e.forecast_quantity FROM fact_sales_monthly s LEFT JOIN fact_act_est e USING(fiscal_year,product_code,customer_code)),  
cte2 as (SELECT * FROM cte1 c1 LEFT JOIN fact_pre_invoice_deductions pr USING(customer_code,fiscal_year)),  
cte3 as (SELECT * FROM cte2 c2 LEFT JOIN fact_post_invoice_deductions pd USING(date,product_code,customer_code)),  
cte4 as (SELECT * FROM cte3 LEFT JOIN fact_gross_price gp USING(product_code,fiscal_year)),  
cte5 as (SELECT c4.*,c.market  FROM cte4 c4 LEFT JOIN dim_customer c USING(customer_code)),  
cte6 as (SELECT * FROM cte5 c5 LEFT JOIN fact_freight_cost fc USING(market,fiscal_year)),  
cte7 as (SELECT * FROM cte6 c6 LEFT JOIN fact_manufacturing_cost mc USING(product_code,fiscal_year)  
SELECT * FROM cte7 )**

*--Creating view fact_transactionss for storing all the required data that is joined into a single table. We can store it as a table but this data set has around 600k rows, it would be easier to store a view than to store a table with so many columns and rows*


![fact_transactions](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/17991b69-af39-4a12-aa96-56332a2f32bc)

4. Now we have all the required columns from 7 fact tables stored in a single table fact_transactions. We can easily create a star schema data model with fact_transactions as the fact table and dim_product, and dim_customer tables as dimension tables. 

5. In the fact_transaction table, we have transaction_date and fiscal_year columns. For Atliq Hardware Fiscal year starts from September 1st to August 31st. For example, if we take the transaction date 2019-09-01, for this date fiscal_year is showing as 2020. The fiscal year 2020 starts on September 1st 2019, and ends on August 31st 2020. Similarly fiscal year 2022 starts on September 1st, 2021, and ends on August 31st, 2022. If we add 4 months to the transaction date we can get the fiscal_date of that transaction. We will create a date table so that it will be easy for us to create any sort of time intelligence function.

6. First we should load data into Power BI then we can create any required transformations and can create a date dimension table. 

![data into power bi](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/bdee17ab-27eb-4ea5-a088-c745ab4c07bc)

7. Fetching fact table fact_transactions and dimension tables dim_product and dim_customer from My SQL database to Powe BI. We can directly transform data and can perform required transformations in power query then we can load data.

8. In the fact_transaction table, changed the type of fiscal_year column and customer_code column to text and we will not perform any aggregations on these columns. There is no need to perform any major data transformations as we already have clean data. We only need to create an extra date table.
 
9. To create a date table we need all the dates that are in the fact_transaction table. We can get the minimum date and maximum date from the fact_transaction table and pass it to create a date table with all the dates that are in the fact_transaction.
   
![date table creation](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/263a5b6b-2fc1-43a4-9ee6-aa8afce7de61)

11. All the dates in the fact_transactions table are the first day of their respective month. So we will convert all dates in the dates columns to the first day of their respective month and remove duplicates so that we will have dates as the first day of their respective month for each year. All the dates are in the calendar_date column.
    

**fiscal_date = Date.AddMonths([calendar_date],4)**  *--Creating fiscal_date column by adding 4 months to the calendar_date*

**fiscal_year = Date.Year([fiscal_date])**  *--Extracting fiscal_year from the fiscal_date column*

**fiscal_month_number = Date.Month([fiscal_date])**  *--Extracting month number from the fiscal_date column, so that we can sort calendar_date_month in this order*

![date table](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/760dd9c6-d4ac-4da5-92a1-a7571c900b81)

 12. Close and load all these tables. In the dim_date table, we need to create another column to represent the month, as the fiscal year starts in September and ends in August we need September as the first month and August as the last month.
     
**calendar_date_month = FORMAT(dim_date[calendar_date],"MMM")**  *--Extracting months from calendar date in this format Jan*

*--We need to sort the calendar_date_month column with fiscal_month_number in order to get September as the first month and August as the last month*

![caluculated date column](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/9b2000be-0e69-47c9-b050-853957990a60)    



## Data Modelling   

1. All our tables are ready, now we can establish the relationship between the fact table and dimension tables. Our data model will be Star Schema as we have transformed all fact tables into only one fact table, we can easily use the Star Schema data model.

![star schema](https://github.com/kushwanthreddyn07/Sales-Dashboard-Power-BI/assets/144375008/ef0b89a9-16b6-4961-baca-8a0fc5ad1ec3)


Here fact_transactions table is in the middle and dimension tables are surrounded, this schema or this formation is known as Star Schema. Now we can easily understand our model and the connections between all the tables. All dimension tables are connected with the fact table with the one-many relationship. Now it will be very easy for us to fetch the related data across different tables. 















