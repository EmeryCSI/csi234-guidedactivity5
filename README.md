# Renton Technical College CSI-234

<br />

<div align="center">  
    <img src="logo.jpg" alt="Logo">
    <h3 align="center">Guided Activity 4</h3>
</div>

# Guided Activity 4: Advanced SQL Concepts

This repository is a part of CSI-234 at Renton Technical College.

Clone this repository to your local machine and complete the instructions below. You will be submitting screenshots as well as SQL code in this repository.

## Guided Activity 4: Clone the repository and make a screenshots folder

1. Clone the repository to your local machine using GitHub Desktop or another GitHub tool.
2. Make note of the folder where you cloned the repository.
3. Click "Show in Explorer" to open the repository folder.
4. Inside of this folder, create a "Screenshots" folder. This is where you will save your screenshots for this assignment.
5. Iclude your SQL Queries in a .SQL file inside of the repository.

## Guided Activity 4: Advanced SQL Concepts

### 1. Views

A view is a virtual table based on the result-set of an SQL statement. It contains rows and columns, just like a real table. The fields in a view are fields from one or more real tables in the database.

```sql
CREATE VIEW CustomerView AS
SELECT FirstName, LastName, EmailAddress, Phone
FROM SalesLT.Customer;
```

1. Query the newly created View and take a screenshot of the results.
2. Modify the above view to include the CompanyName column. Take a screenshot of the modified view.

### 2. Functions

SQL functions are methods which return a value after processing on the input values.

```sql
CREATE FUNCTION GetProductsTotalValue(@ProductId INT)
RETURNS DECIMAL(18, 2) -- Adjust precision and scale as needed
AS
BEGIN
    RETURN (
        SELECT SUM(OrderQty * UnitPrice)
        FROM SalesLT.SalesOrderDetail 
        WHERE ProductId = @ProductId 
    )
END;
```

1. Call the function: SELECT dbo.GetProductsTotalValue(836) AS TotalValue;
2. Take a screenshot of the function output.
3. Exercise: Using this as a template create a new function which returns the total quantity of products sold for a given productID. Take a screnshot of the output

### 3. Stored Procedures

As previously demonstrated, stored procedures are reusable SQL code snippets that can accept parameters and return data. They help in encapsulating logic and improving performance. They are similar to functions except that stored prcedures return result sets (tables)

```sql
CREATE PROCEDURE GetProductDetails
@ProductID INT
AS
BEGIN
    SELECT * FROM SalesLT.Product WHERE ProductID = @ProductID;
END;
```

1. Call the stored procedure: EXEC GetProductDetails 863;
2. Take a screenshot of the sctored procedure output output.
3. Exercise: Using this as a template create a new function which includes the Name field from the productcategory table based on a given product id. Take a screenshot of the output.

### 4. Indexing

Indexes can dramatically improve the performance of data retrieval. Without an index, the database must perform a full table scan to retrieve data, which can be slow for large tables. With an index, the database can quickly locate the data without scanning the entire table.

However, it's worth noting that while indexes speed up data retrieval, they can slow down data modification operations like INSERT, UPDATE, and DELETE because the index also needs to be updated.

When you designate a primary key, a unique clustered index is automatically created on that field.

Lets make an index based on the ProductNumber field in the Product table.

```sql
CREATE INDEX idx_Product_ProductNumber
ON SalesLT.Product(ProductNumber);
```
1.Refresh your connection to the database.
2. Expand the SalesLT.Product table in SSMS or Azure Data Studio. There should be a folder called Indexes. Open that folder and take a screenshot of your new index.

![image](https://github.com/EmeryCSI/csi234-guidedactivity4/assets/102991550/8176ecbc-de15-4769-ab2e-48fc203ca384)


Lets make a composite index on the customer table based on the FirstName and LastName fields. This will speed up queries that look for customers based on their name.

```sql
CREATE INDEX idx_Customer_FirstName_LastName
ON SalesLT.Customer(FirstName, LastName);
```

1. Refresh your connection to the database.
2. Expand the SalesLT.Customer table in SSMS or Azure Data Studio. There should be a folder called Indexes. Open that folder and take a screenshot of your new index.

![image](https://github.com/EmeryCSI/csi234-guidedactivity4/assets/102991550/ad85d315-ece2-41c2-820c-7a597fdb3c0f)

### 5. Schema

A schema is a named container for database objects. It provides a namespace that allows multiple objects of the same name to co-exist within the same database. In relational databases, a schema is often used to logically group related tables, views, and other database objects together. This logical grouping aids in organization, management, and security of the database.

Schemas can be used to manage permissions. By granting or denying permissions on a schema, you can control access to all the objects within that schema. This makes it easier to manage security at a broader level rather than setting permissions on individual objects.

Schemas help in organizing database objects into logical groups. This can be especially useful in large databases where there are hundreds or thousands of tables and other objects. For example, all inventory-related tables can be grouped under an "Inventory" schema, while sales-related tables can be grouped under a "Sales" schema.

Tables can only belong to one Schema.

Lets Create a new Schema in our database for Human Resources. Run the following Code:
```sql
CREATE SCHEMA HR;
```
```sql
CREATE TABLE HR.JobTitle (
    JobTitleID INT PRIMARY KEY IDENTITY(1,1),
    Title NVARCHAR(100),
    Description NVARCHAR(255)
);
```

```sql
CREATE TABLE HR.Employee (
    EmployeeID INT PRIMARY KEY IDENTITY(1,1),
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Position NVARCHAR(100),
    HireDate DATE,
    JobTitleID INT FOREIGN KEY REFERENCES HR.JobTitle(JobTitleID)
);
```

```sql
CREATE TABLE HR.Department (
    DepartmentID INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName NVARCHAR(100)
);
```

```sql
CREATE TABLE HR.EmployeeDepartment (
    EmployeeDepartmentID INT PRIMARY KEY IDENTITY(1,1),
    EmployeeID INT FOREIGN KEY REFERENCES HR.Employee(EmployeeID),
    DepartmentID INT FOREIGN KEY REFERENCES HR.Department(DepartmentID)
);
```

```sql
INSERT INTO HR.JobTitle (Title, Description)
VALUES ('Manager', 'Oversees and manages a team or department'),
       ('Analyst', 'Analyzes data and provides insights'),
       ('Developer', 'Develops and maintains software applications'),
       ('HR Specialist', 'Manages human resources tasks and policies'),
       ('Sales Representative', 'Handles sales and customer relationships'),
       ('IT Support', 'Provides technical support and assistance'),
       ('Marketing Manager', 'Oversees marketing strategies and campaigns'),
       ('Product Manager', 'Manages product lifecycle and strategy');
```

```sql
INSERT INTO HR.Employee (FirstName, LastName, Position, HireDate, JobTitleID)
VALUES ('John', 'Doe', 'Manager', '2020-01-15', 1),
       ('Jane', 'Smith', 'Analyst', '2019-05-10', 2),
       ('Alice', 'Johnson', 'Developer', '2018-03-20', 3),
       ('Robert', 'Brown', 'HR Specialist', '2021-02-01', 4),
       ('Emily', 'Clark', 'Sales Representative', '2019-11-12', 5),
       ('Michael', 'White', 'IT Support', '2018-07-05', 6),
       ('Sophia', 'Green', 'Marketing Manager', '2020-04-22', 7),
       ('James', 'Black', 'Product Manager', '2017-09-30', 8);
```
```sql
INSERT INTO HR.Department (DepartmentName)
VALUES ('Finance'),
       ('IT'),
       ('Sales'),
       ('Marketing'),
       ('Product Management'),
       ('Customer Support');
```
```sql
INSERT INTO HR.EmployeeDepartment (EmployeeID, DepartmentID)
VALUES (1, 1),  -- John Doe belongs to Finance
       (2, 2),  -- Jane Smith belongs to IT
       (3, 2),  -- Alice Johnson also belongs to IT
       (4, 3),  -- Robert Brown belongs to Sales
       (5, 4),  -- Emily Clark belongs to Marketing
       (6, 2),  -- Michael White belongs to IT
       (7, 4),  -- Sophia Green belongs to Marketing
       (8, 5);  -- James Black belongs to Product Management
```

1.Refresh your connection to the database.
2.When completed you should have 4 new tables in your database that belong to the new HR Schema.

![image](https://github.com/EmeryCSI/csi234-guidedactivity4/assets/102991550/cf298bf2-2188-4cf3-b486-4dbfeabdb18c)

3. Take a screenshot of either your HR diagram OR your HR tables listed in Azure data Studio.
4. We will use these tables for Independent Activity 1.


### Commit your changes to GitHub

1. Create a new commit with the message "Completed guided activity 4"
2. Push your changes to GitHub

Feel free to message your instructor or the TA on Canvas if you have any questions.
