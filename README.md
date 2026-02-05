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



