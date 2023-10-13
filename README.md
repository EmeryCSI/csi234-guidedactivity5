# Renton Technical College CSI-234

<br />

<div align="center">  
    <img src="logo.jpg" alt="Logo">
    <h3 align="center">Guided Activity 5</h3>
</div>

# Guided Activity 5: Indexing and Query Performance

This repository is a part of CSI-234 at Renton Technical College.

Clone this repository to your local machine and complete the instructions below. You will be submitting screenshots as well as SQL code in this repository.

## Guided Activity 5: Clone the repository and make a screenshots folder

1. Clone the repository to your local machine using GitHub Desktop or another GitHub tool.
2. Make note of the folder where you cloned the repository.
3. Click "Show in Explorer" to open the repository folder.
4. Inside of this folder, create a "Screenshots" folder. This is where you will save your screenshots for this assignment.
5. Include your SQL Queries in a .SQL file inside of the repository.

## Guided Activity 5: Let's create some dummy data to work with.

### 1. Create a new Schema with data

```sql
GO
CREATE SCHEMA Activity5;
GO
```
### 2. Add some tables with data to the schema, Run the following Query. This will take some time. Be Patient. We are creating 3 tables with 10,000 rows of data.

```sql
-- Drop tables if they exist
IF OBJECT_ID('Activity5.OrderDetails', 'U') IS NOT NULL
    DROP TABLE Activity5.OrderDetails;

IF OBJECT_ID('Activity5.Orders', 'U') IS NOT NULL
    DROP TABLE Activity5.Orders;

IF OBJECT_ID('Activity5.Customers', 'U') IS NOT NULL
    DROP TABLE Activity5.Customers;

-- Create Customers table
CREATE TABLE Activity5.Customers (
    CustomerID INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Email NVARCHAR(100),
    Phone NVARCHAR(15),
    Zip NVARCHAR(10)
);

-- Fill Customers table with 10,000 records
DECLARE @Counter1 INT = 1;
WHILE @Counter1 <= 10000
BEGIN
    INSERT INTO Activity5.Customers (FirstName, LastName, Email, Phone, Zip)
    VALUES (
        'FirstName' + CAST(@Counter1 AS NVARCHAR),
        'LastName' + CAST(@Counter1 AS NVARCHAR),
        'email' + CAST(@Counter1 AS NVARCHAR) + '@example.com',
        CAST((RAND() * 10000000000) + 1000000000 AS NVARCHAR),
        CAST((RAND() * 100000) + 10000 AS NVARCHAR)
    );
    SET @Counter1 = @Counter1 + 1;
END;

-- Create Orders table
CREATE TABLE Activity5.Orders (
    OrderID INT PRIMARY KEY IDENTITY(1,1),
    CustomerID INT FOREIGN KEY REFERENCES Activity5.Customers(CustomerID),
    OrderDate DATE,
    OrderStatus NVARCHAR(50),
    ShippingAddress NVARCHAR(255)
);

-- Fill Orders table with 10,000 records
DECLARE @Counter2 INT = 1;
WHILE @Counter2 <= 10000
BEGIN
    INSERT INTO Activity5.Orders (CustomerID, OrderDate, OrderStatus, ShippingAddress)
    VALUES (
        (RAND() * 10000) + 1,
        DATEADD(DAY, (RAND() * 365), '2020-01-01'),
        CASE CAST((RAND() * 3) + 1 AS INT)
            WHEN 1 THEN 'Shipped'
            WHEN 2 THEN 'Pending'
            ELSE 'Cancelled'
        END,
        'Address' + CAST(@Counter2 AS NVARCHAR)
    );
    SET @Counter2 = @Counter2 + 1;
END;

-- Create OrderDetails table
CREATE TABLE Activity5.OrderDetails (
    OrderDetailID INT PRIMARY KEY IDENTITY(1,1),
    OrderID INT FOREIGN KEY REFERENCES Activity5.Orders(OrderID),
    ProductName NVARCHAR(100),
    Quantity INT,
    UnitPrice DECIMAL(10, 2),
    Discount DECIMAL(5, 2)
);

-- Fill OrderDetails table with 10,000 records
DECLARE @Counter3 INT = 1;
WHILE @Counter3 <= 10000
BEGIN
    INSERT INTO Activity5.OrderDetails (OrderID, ProductName, Quantity, UnitPrice, Discount)
    VALUES (
        (RAND() * 10000) + 1,
        'Product' + CAST((RAND() * 100) AS NVARCHAR),
        (RAND() * 10) + 1,
        (RAND() * 100) + 1,
        (RAND() * 0.5)
    );
    SET @Counter3 = @Counter3 + 1;
END;
```

### 3. Creating our first index

Run the following Queries with the Execution Plan enabled

```sql
SELECT * 
FROM Activity5.Customers 
WHERE CustomerID = 1;

SELECT * FROM 
Activity5.Customers 
WHERE LastName = 'LastName100';
```

Answer the following questions in your .sql file
What was the estimated CPU Cost of the first query?
What was the estimated CPU Cost of the second query?
What was the Number of Rows Read by the first query?
What was the Number of Rows Read by the first query?

### Before Indexing:

SQL Server will perform a full table scan, reading every row in the Customers table to find the rows that match the condition. This is inefficient and time-consuming, especially for large tables.
Index to Create: Non-clustered index on LastName.

```sql
CREATE INDEX idx_Customers_LastName ON Activity5.Customers(LastName);
```
How It Works:

A non-clustered index is like an index in a book. It creates a separate data structure that holds a pointer to the data in the table. This allows SQL Server to quickly locate the data without scanning the entire table.
After Indexing:

SQL Server will use the non-clustered index to quickly find the rows that match the condition, significantly reducing the number of rows it needs to read.

Run the query again and check the Query Plan
```sql
SELECT * FROM 
Activity5.Customers 
WHERE LastName = 'LastName100';
```

Take a screenshot of the new query plan. Notice the difference. There are now two operations performed a key lookup and an index seek. The actual number of rows read is 1 and the compute cost is significantly reduced.

### 4. Composite Indexes

A composite index is a type of database index that includes two or more columns from a table. While standard indexes are built using a single column, composite indexes are constructed using a combination of columns. This allows for faster search, retrieval, and sorting of database records based on multiple criteria.

Lets pull some data from the OrderDetails table. Here we filtered on Quantity and Discount

```sql
SELECT * 
FROM Activity5.OrderDetails
WHERE Quantity = 10 AND Discount = .3
```

Run this query and take note of the Execution Plan. Take a screenshot.

Let's create a composite index for these columns on the OrderDetails table.

```sql
CREATE INDEX idx_OrderDetails_Quantity_Discount 
ON Activity5.OrderDetails(Quantity, Discount);
```

Re-run the query and screenshot the execution plan. Notice that your new index is now being used. Screenshot the new execution plan. Notice the significant savings on CPU cost. Notice we have gone from performing and index scan (slow) to an index seek(fast).

### 5. Indexing and Joins

First lets find a customer in your database who has placed multiple orders. Remember this data was randomly generated so your results will be different than mine.


```sql
SELECT TOP 1 c.CustomerID, c.FirstName, c.LastName, COUNT(o.OrderID) AS NumberOfOrders
FROM Activity5.Customers c
JOIN Activity5.Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID, c.FirstName, c.LastName
ORDER BY NumberOfOrders DESC;
```
Run this query and take note of the Customer FirstName with the most orders. Mine was FirstName7502.

Lets get some order info for this customer
```sql
SELECT c.FirstName, c.LastName, o.OrderDate 
FROM Activity5.Customers c
JOIN Activity5.Orders o ON c.CustomerID = o.CustomerID
-- Make sure to change this to your customer with the most orders
WHERE c.FirstName = 'FirstName7502';
```

Take a screenshot of the Query Plan for this query.

This query is badly optimized. We need to create two indexes, one on the firstName column in the customers table and another on the Orders.CustomerId Column. CustomerId is a foreign key on the orders table and they do not have indexes by default. 

Lets create our indexes:
```sql
CREATE INDEX idx_Orders_CustomerID ON Activity5.Orders(CustomerID);
CREATE INDEX idx_Customers_FirstName ON Activity5.Customers(FirstName);
```

Now re-run the query and screenshot the Query Plan. Notice how the number of rows read have reduced dramatically.

### Commit your changes to GitHub

1. Create a new commit with the message "Completed guided activity 5"
2. Remember to include your .sql files where you wrote you answers.
3. Push your changes to GitHub

Feel free to message your instructor or the TA on Canvas if you have any questions.
