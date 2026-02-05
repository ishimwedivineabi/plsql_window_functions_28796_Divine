# Business problem:
Kigali Supermarket needs to understand customer buying patterns and sales performance. The current system shows basic sales data but cannot answer important business questions. 



# Database Schema
1. Customers Table
Stores information about individuals making purchases.

SQL
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    location VARCHAR(100),
    phone_number VARCHAR(20),
    join_date DATE
);
2. Transaction Type Table
A lookup table to define the nature of a transaction (e.g., "Sale," "Return," "Exchange").

SQL
CREATE TABLE Transaction_type (
    type_id INT PRIMARY KEY,
    type_name VARCHAR(50) NOT NULL
);
3. Products Table
Contains details regarding the items available for sale.

SQL
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    prod_name VARCHAR(100) NOT NULL,
    prod_category VARCHAR(50),
    unit_cost DECIMAL(10, 2),
    selling_price DECIMAL(10, 2)
);
4. Transactions Table
The central fact table that links customers, products, and transaction types.

SQL
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    type_id INT,
    trans_date DATE,
    quantity INT,
    total_amount DECIMAL(12, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id),
    FOREIGN KEY (type_id) REFERENCES Transaction_type(type_id)
);
#  ER Diagram

<img width="979" height="490" alt="image" src="https://github.com/user-attachments/assets/b83b4094-6614-4b2f-a1a1-c672da7a9e2d" />
# JOIN queries

SQL
SELECT 
    t.transaction_id,
    c.customer_name,
    p.prod_name,
    tt.type_name AS transaction_category,
    t.trans_date,
    t.quantity,
    t.total_amount
FROM Transactions t
JOIN customers c ON t.customer_id = c.customer_id
JOIN products p ON t.product_id = p.product_id
JOIN Transaction_type tt ON t.type_id = tt.type_id;
## 2. Sales Summary by Customer

SQL
SELECT 
    c.customer_name,
    COUNT(t.transaction_id) AS total_orders,
    SUM(t.total_amount) AS lifetime_value
FROM customers c
LEFT JOIN Transactions t ON c.customer_id = t.customer_id
GROUP BY c.customer_name
ORDER BY lifetime_value DESC;
## 3. Product Performance 

SQL
SELECT 
    p.prod_name,
    SUM(t.quantity) AS units_sold,
    SUM(t.total_amount) AS total_revenue,
    SUM(t.total_amount - (p.unit_cost * t.quantity)) AS estimated_profit
FROM products p
JOIN Transactions t ON p.product_id = t.product_id
GROUP BY p.prod_name;
# windows Function  queries
## Ranking the  customers according to the total spent amount

SELECT 
    customer_name,
    SUM(total_amount) as total_spent,
    ROW_NUMBER() OVER (ORDER BY SUM(total_amount) DESC) as row_num,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) as revenue_rank,
    DENSE_RANK() OVER (ORDER BY SUM(total_amount) DESC) as dense_rank
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY customer_name;

<img width="979" height="537" alt="image" src="https://github.com/user-attachments/assets/b8455032-444a-4334-86b2-92818a4dc91e" />

## this query ranks products within each category
SELECT 
    product_name,
    prod_category,
    SUM(quantity) as units_sold,
    RANK() OVER (PARTITION BY prod_category ORDER BY SUM(quantity) DESC) as category_rank,
    PERCENT_RANK() OVER (PARTITION BY prod_category ORDER BY SUM(quantity) DESC) as percentile
FROM products p
JOIN transactions t ON p.product_id = t.product_id
GROUP BY product_name, prod_category;

<img width="979" height="520" alt="image" src="https://github.com/user-attachments/assets/3ad8dce4-f107-4568-9961-bb1c4654a010" />

## this query ranks month and total amount monthly sales made


SELECT 
    DATE_FORMAT(trans_date, '%Y-%m') as month,
    SUM(total_amount) as monthly_sales,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) as sales_rank,
    DENSE_RANK() OVER (ORDER BY SUM(total_amount) DESC) as dense_sales_rank
FROM transactions
GROUP BY DATE_FORMAT(trans_date, '%Y-%m');


 ## Rank payment methods by usage

SELECT 
    type_name as payment_method,
    COUNT(transaction_id) as usage_count,
    ROW_NUMBER() OVER (ORDER BY COUNT(transaction_id) DESC) as popularity_rank,
    PERCENT_RANK() OVER (ORDER BY COUNT(transaction_id) DESC) as usage_percentile
FROM transactions t
JOIN transaction_type tt ON t.type_id = tt.type_id
GROUP BY type_name;


 ## this query shows the customers who visits our supermarket many times
 
SELECT 
    customer_name,
    COUNT(transaction_id) as visit_count,
    RANK() OVER (ORDER BY COUNT(transaction_id) DESC) as frequency_rank,
    DENSE_RANK() OVER (ORDER BY COUNT(transaction_id) DESC) as dense_freq_rank
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY customer_name;


## this query shows the total amount that is spent every day

SELECT 
    trans_date,
    SUM(total_amount) as daily_sales,
    SUM(SUM(total_amount)) OVER (ORDER BY trans_date ROWS UNBOUNDED PRECEDING) as running_total
FROM transactions
GROUP BY trans_date
ORDER BY trans_date;


 ## this query Calculates 3-day moving average of sales
 
SELECT 
    trans_date,
    SUM(total_amount) as daily_sales,
    AVG(SUM(total_amount)) OVER (ORDER BY trans_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3d
FROM transactions
GROUP BY trans_date
ORDER BY trans_date;

## Find min and max sales values per product

SELECT 
    product_name,
    MIN(total_amount) as min_sale,
    MAX(total_amount) as max_sale,
    AVG(total_amount) as avg_sale
FROM transactions t
JOIN products p ON t.product_id = p.product_id
GROUP BY product_name;



 ## Compare ROWS vs RANGE framing


SELECT 
    DATE_FORMAT(trans_date, '%Y-%m') as month,
    SUM(total_amount) as monthly_sales,
    AVG(SUM(total_amount)) OVER (ORDER BY DATE_FORMAT(trans_date, '%Y-%m') ROWS BETWEEN 1 PRECEDING AND CURRENT ROW) as rows_avg,
    AVG(SUM(total_amount)) OVER (ORDER BY DATE_FORMAT(trans_date, '%Y-%m') RANGE BETWEEN 1 PRECEDING AND CURRENT ROW) as range_avg
FROM transactions
GROUP BY DATE_FORMAT(trans_date, '%Y-%m');


  ## this query calculates the Cumulative average spending per customer

SELECT 
    customer_name,
    trans_date,
    total_amount,
    AVG(total_amount) OVER (PARTITION BY c.customer_id ORDER BY trans_date ROWS UNBOUNDED PRECEDING) as cumulative_avg
FROM transactions t
JOIN customers c ON t.customer_id = c.customer_id
ORDER BY customer_name, trans_date;



 ## Calculate monthly growth percentage


WITH monthly_sales AS (
    SELECT 
        DATE_FORMAT(trans_date, '%Y-%m') as month,
        SUM(total_amount) as sales
    FROM transactions
    GROUP BY DATE_FORMAT(trans_date, '%Y-%m')
)
SELECT 
    month,
    sales,
    LAG(sales, 1) OVER (ORDER BY month) as prev_month,
    ROUND(((sales - LAG(sales, 1) OVER (ORDER BY month)) / LAG(sales, 1) OVER (ORDER BY month)) * 100, 2) as growth_pct
FROM monthly_sales;


 ## in this query are Finding  next purchase date for each customer

SELECT 
    customer_name,
    trans_date,
    total_amount,
    LEAD(trans_date, 1) OVER (PARTITION BY c.customer_id ORDER BY trans_date) as next_purchase_date
FROM transactions t
JOIN customers c ON t.customer_id = c.customer_id
ORDER BY customer_name, trans_date;


## this query we are finding each day's sales with previous day


SELECT 
    trans_date,
    SUM(total_amount) as daily_sales,
    LAG(SUM(total_amount), 1) OVER (ORDER BY trans_date) as prev_day_sales,
    SUM(total_amount) - LAG(SUM(total_amount), 1) OVER (ORDER BY trans_date) as daily_change
FROM transactions
GROUP BY trans_date
ORDER BY trans_date;

 ## Analyze product sales trend with lead/lag


SELECT 
    product_name,
    trans_date,
    quantity,
    LAG(quantity, 1) OVER (PARTITION BY p.product_id ORDER BY trans_date) as prev_quantity,
    LEAD(quantity, 1) OVER (PARTITION BY p.product_id ORDER BY trans_date) as next_quantity
FROM transactions t
JOIN products p ON t.product_id = p.product_id
ORDER BY product_name, trans_date;


 ## Analyze customer spending pattern over time


SELECT 
    customer_name,
    trans_date,
    total_amount,
    LAG(total_amount, 1) OVER (PARTITION BY c.customer_id ORDER BY trans_date) as prev_amount,
    LEAD(total_amount, 1) OVER (PARTITION BY c.customer_id ORDER BY trans_date) as next_amount
FROM transactions t
JOIN customers c ON t.customer_id = c.customer_id
ORDER BY customer_name, trans_date;


## this are Segmenting customers into 4 quartiles using NTILE function


SELECT 
    customer_name,
    SUM(total_amount) as total_spent,
    NTILE(4) OVER (ORDER BY SUM(total_amount) DESC) as spending_quartile
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY customer_name;


 ## Analyze product performance using percentiles


SELECT 
    product_name,
    SUM(total_amount) as total_revenue,
    CUME_DIST() OVER (ORDER BY SUM(total_amount)) as cumulative_distribution,
    ROUND(CUME_DIST() OVER (ORDER BY SUM(total_amount)) * 100, 2) as percentile
FROM products p
JOIN transactions t ON p.product_id = t.product_id
GROUP BY product_name;


 ## Segment customers by visit frequency

SELECT 
    customer_name,
    COUNT(transaction_id) as visit_count,
    NTILE(4) OVER (ORDER BY COUNT(transaction_id) DESC) as frequency_segment
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY customer_name;



## Analyze payment method distribution

SELECT 
    type_name as payment_method,
    COUNT(transaction_id) as usage_count,
    CUME_DIST() OVER (ORDER BY COUNT(transaction_id)) as cumulative_dist
FROM transactions t
JOIN transaction_type tt ON t.type_id = tt.type_id
GROUP BY type_name;

 ## Segment products by price range


SELECT 
    product_name,
    selling_price,
    NTILE(4) OVER (ORDER BY selling_price) as price_quartile,
    CUME_DIST() OVER (ORDER BY selling_price) as price_percentile
FROM products;








