# 📌 MODULE 3: SQL for Interviews

> **Goal:** Complete SQL coverage from basics to advanced — theory-first, interview-ready definitions, and common scenarios. Every topic you need to crack SQL interview rounds.

---

## 1. Basics of SQL

### ❓ What is SQL?

**SQL (Structured Query Language)** is the standard language for managing and querying **relational databases**. It lets you create, read, update, and delete data, as well as define database structure.

**Interview Answer:** _"SQL is the standard language for interacting with relational databases. It's used to query data, define schema, control access, and manage transactions."_

---

### SQL vs NoSQL

| Aspect             | SQL (Relational)                         | NoSQL (Non-Relational)                    |
| ------------------ | ---------------------------------------- | ----------------------------------------- |
| **Schema**         | Fixed schema, predefined tables/columns  | Flexible schema, documents/key-value      |
| **Data Model**     | Tables with rows and columns             | Documents, key-value, graph, columnar     |
| **Scalability**    | Vertical (add more CPU/RAM)              | Horizontal (add more servers)             |
| **Transactions**   | ACID guaranteed                          | Eventual consistency (BASE model)         |
| **Query Language** | SQL (standardized)                       | Varies (MongoDB queries, Cassandra CQL)   |
| **Use Case**       | Structured data, transactions, reporting | Unstructured data, high-volume, real-time |
| **Examples**       | MySQL, PostgreSQL, SQL Server, Oracle    | MongoDB, Cassandra, Redis, DynamoDB       |

**Interview Answer:** _"SQL databases are relational with fixed schemas and ACID transactions — great for structured data. NoSQL databases have flexible schemas and scale horizontally — better for unstructured data and massive scale."_

---

### Types of SQL Commands

| Type    | Full Form                    | Purpose                          | Commands                              |
| ------- | ---------------------------- | -------------------------------- | ------------------------------------- |
| **DDL** | Data Definition Language     | Define/modify database structure | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| **DML** | Data Manipulation Language   | Modify data in tables            | `INSERT`, `UPDATE`, `DELETE`          |
| **DQL** | Data Query Language          | Retrieve data                    | `SELECT`                              |
| **DCL** | Data Control Language        | Control access permissions       | `GRANT`, `REVOKE`                     |
| **TCL** | Transaction Control Language | Manage transactions              | `COMMIT`, `ROLLBACK`, `SAVEPOINT`     |

```sql
-- DDL
CREATE TABLE Employees (Id INT PRIMARY KEY, Name VARCHAR(100));
ALTER TABLE Employees ADD Email VARCHAR(100);
DROP TABLE Employees;
TRUNCATE TABLE Employees;  -- Deletes all rows, can't rollback

-- DML
INSERT INTO Employees VALUES (1, 'John', 'john@example.com');
UPDATE Employees SET Email = 'newemail@example.com' WHERE Id = 1;
DELETE FROM Employees WHERE Id = 1;

-- DQL
SELECT * FROM Employees;

-- DCL
GRANT SELECT ON Employees TO User1;
REVOKE SELECT ON Employees FROM User1;

-- TCL
BEGIN TRANSACTION;
  UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
  SAVEPOINT sp1;
  UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;
ROLLBACK TO sp1;  -- Undo only second update
COMMIT;
```

**Interview Tip:** _"DELETE is DML (can rollback), TRUNCATE is DDL (faster, can't rollback)."_

---

## 2. SQL Queries

### SELECT Statement

```sql
SELECT * FROM Employees;                        -- All columns
SELECT Name, Salary FROM Employees;             -- Specific columns
SELECT Name AS EmployeeName FROM Employees;     -- Column alias
SELECT 'Salary:', Salary FROM Employees;        -- With literal
```

---

### WHERE Clause

```sql
SELECT * FROM Employees WHERE Salary > 50000;
SELECT * FROM Employees WHERE Department = 'IT' AND Salary > 60000;
SELECT * FROM Employees WHERE Department = 'IT' OR Department = 'HR';
```

---

### DISTINCT

> Removes duplicate rows from the result.

```sql
SELECT DISTINCT Department FROM Employees;
```

---

### ORDER BY

```sql
SELECT * FROM Employees ORDER BY Salary DESC;           -- Descending
SELECT * FROM Employees ORDER BY Department, Salary;    -- Multi-column
```

---

### LIMIT / TOP

```sql
-- SQL Server
SELECT TOP 10 * FROM Employees ORDER BY Salary DESC;

-- MySQL / PostgreSQL
SELECT * FROM Employees ORDER BY Salary DESC LIMIT 10;

-- SQL Server (modern syntax)
SELECT * FROM Employees ORDER BY Salary DESC OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;
```

---

### BETWEEN / IN / LIKE

```sql
-- BETWEEN
SELECT * FROM Employees WHERE Salary BETWEEN 50000 AND 80000;

-- IN
SELECT * FROM Employees WHERE Department IN ('IT', 'HR', 'Sales');

-- LIKE (wildcards: % = any characters, _ = single character)
SELECT * FROM Employees WHERE Name LIKE 'J%';        -- Starts with J
SELECT * FROM Employees WHERE Email LIKE '%@gmail.com';  -- Ends with @gmail.com
SELECT * FROM Employees WHERE Name LIKE '_o%';       -- Second character is 'o'
```

---

### NULL Handling

> **NULL** means "unknown" or "missing value" — it is **NOT** the same as empty string `''` or zero `0`.

```sql
-- Check for NULL
SELECT * FROM Employees WHERE ManagerId IS NULL;
SELECT * FROM Employees WHERE ManagerId IS NOT NULL;

-- NULL in comparisons
SELECT * FROM Employees WHERE Salary = NULL;   -- ❌ Wrong — always false
SELECT * FROM Employees WHERE Salary IS NULL;  -- ✅ Correct

-- COALESCE (return first non-NULL value)
SELECT Name, COALESCE(ManagerId, 0) AS ManagerId FROM Employees;

-- ISNULL / IFNULL (SQL Server / MySQL)
SELECT Name, ISNULL(ManagerId, 0) AS ManagerId FROM Employees;
```

**Interview Answer:** _"NULL represents a missing or unknown value. Use `IS NULL` / `IS NOT NULL` to check for NULLs — regular comparison operators like `=` don't work with NULL."_

---

## 3. Joins

> A **JOIN** combines rows from two or more tables based on a related column.

### JOIN Types

| JOIN Type           | What it returns                                                |
| ------------------- | -------------------------------------------------------------- |
| **INNER JOIN**      | Only matching rows from both tables                            |
| **LEFT JOIN**       | All rows from left table + matching rows from right            |
| **RIGHT JOIN**      | All rows from right table + matching rows from left            |
| **FULL OUTER JOIN** | All rows from both tables, with NULLs where no match           |
| **CROSS JOIN**      | Cartesian product — every row from left × every row from right |
| **SELF JOIN**       | Join table to itself (e.g., employee-manager hierarchy)        |

---

### INNER JOIN

> Returns only rows where there is a match in both tables.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentId = d.DepartmentId;
```

**Interview Scenario:** _"Get all employees with their department names — exclude employees without a department."_

---

### LEFT JOIN (LEFT OUTER JOIN)

> Returns all rows from the **left** table, and matching rows from the right. If no match, NULL for right table columns.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
LEFT JOIN Departments d ON e.DepartmentId = d.DepartmentId;
```

**Interview Scenario:** _"Get all employees, including those not assigned to any department (show NULL for department)."_

---

### RIGHT JOIN (RIGHT OUTER JOIN)

> Returns all rows from the **right** table, and matching rows from the left. If no match, NULL for left table columns.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
RIGHT JOIN Departments d ON e.DepartmentId = d.DepartmentId;
```

**Interview Scenario:** _"Get all departments, including those with no employees."_

---

### FULL OUTER JOIN

> Returns all rows from both tables. If no match, NULL for the missing side.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
FULL OUTER JOIN Departments d ON e.DepartmentId = d.DepartmentId;
```

**Interview Scenario:** _"Get all employees and all departments — show employees without departments AND departments without employees."_

---

### CROSS JOIN

> Cartesian product — every row from left table paired with every row from right table.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
CROSS JOIN Departments d;
-- If Employees has 10 rows, Departments has 5 → result has 50 rows
```

**Interview Scenario:** _"Generate all possible combinations (e.g., every product in every store)."_

---

### SELF JOIN

> Join a table to itself — useful for hierarchical data.

```sql
-- Get each employee and their manager's name
SELECT e.Name AS Employee, m.Name AS Manager
FROM Employees e
LEFT JOIN Employees m ON e.ManagerId = m.EmployeeId;
```

**Interview Scenario:** _"Show employee hierarchy: who reports to whom?"_

---

### JOIN Interview Scenarios

| Scenario                                                 | JOIN Type       |
| -------------------------------------------------------- | --------------- |
| All customers who placed orders                          | INNER JOIN      |
| All customers, including those who never ordered         | LEFT JOIN       |
| All orders, including those without customer info (rare) | RIGHT JOIN      |
| All employees and all departments (even orphans)         | FULL OUTER JOIN |
| Employee-Manager pairs (hierarchy)                       | SELF JOIN       |
| Every product × every store combination                  | CROSS JOIN      |

---

## 4. Aggregate Functions

> Aggregate functions **compute a single result** from multiple rows.

| Function        | What it does                         |
| --------------- | ------------------------------------ |
| `COUNT(*)`      | Count total rows                     |
| `COUNT(column)` | Count non-NULL values in that column |
| `SUM(column)`   | Sum of numeric values                |
| `AVG(column)`   | Average of numeric values            |
| `MAX(column)`   | Maximum value                        |
| `MIN(column)`   | Minimum value                        |

```sql
SELECT COUNT(*) FROM Employees;                       -- Total employees
SELECT COUNT(ManagerId) FROM Employees;               -- Employees with a manager
SELECT SUM(Salary) FROM Employees;                    -- Total salary bill
SELECT AVG(Salary) FROM Employees;                    -- Average salary
SELECT MAX(Salary), MIN(Salary) FROM Employees;       -- Highest and lowest
```

---

### GROUP BY

> Groups rows that have the same value in specified column(s), then applies aggregate functions to each group.

```sql
-- Count employees per department
SELECT Department, COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY Department;

-- Average salary per department
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department;
```

**Rule:** Every column in `SELECT` must either be in `GROUP BY` or inside an aggregate function.

---

### HAVING

> Filters groups **after** aggregation (like `WHERE` but for groups).

```sql
-- Departments with more than 5 employees
SELECT Department, COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY Department
HAVING COUNT(*) > 5;

-- Departments where average salary > 70000
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department
HAVING AVG(Salary) > 70000;
```

**WHERE vs HAVING:**

- `WHERE` filters **rows before** grouping
- `HAVING` filters **groups after** aggregation

```sql
-- Correct order:
SELECT Department, COUNT(*)
FROM Employees
WHERE Salary > 50000        -- Filter rows first
GROUP BY Department
HAVING COUNT(*) > 3;        -- Filter groups after
```

---

### Real-World Interview Problems

**1. Count employees by department**

```sql
SELECT Department, COUNT(*) AS EmployeeCount
FROM Employees
GROUP BY Department;
```

**2. Department with highest total salary**

```sql
SELECT TOP 1 Department, SUM(Salary) AS TotalSalary
FROM Employees
GROUP BY Department
ORDER BY TotalSalary DESC;
```

**3. Average salary per department, only depts with avg > 60k**

```sql
SELECT Department, AVG(Salary) AS AvgSalary
FROM Employees
GROUP BY Department
HAVING AVG(Salary) > 60000;
```

**4. Top 3 highest-paid employees**

```sql
SELECT TOP 3 Name, Salary
FROM Employees
ORDER BY Salary DESC;
```

---

## 5. Subqueries

> A **subquery** is a query nested inside another query. It runs first, then its result is used by the outer query.

### Types by Return Value

| Type             | Returns          | Used with          |
| ---------------- | ---------------- | ------------------ |
| **Single-row**   | One value        | `=`, `>`, `<`      |
| **Multi-row**    | Multiple values  | `IN`, `ANY`, `ALL` |
| **Multi-column** | Multiple columns | `IN`, `EXISTS`     |

---

### Single-Row Subquery

```sql
-- Employee with the highest salary
SELECT Name, Salary
FROM Employees
WHERE Salary = (SELECT MAX(Salary) FROM Employees);
```

---

### Multi-Row Subquery with IN

```sql
-- Employees in departments that have 'IT' or 'HR'
SELECT Name
FROM Employees
WHERE DepartmentId IN (SELECT DepartmentId FROM Departments WHERE Name IN ('IT', 'HR'));
```

---

### ANY, ALL

```sql
-- Salary greater than ANY salary in IT dept (greater than the minimum in IT)
SELECT Name, Salary
FROM Employees
WHERE Salary > ANY (SELECT Salary FROM Employees WHERE Department = 'IT');

-- Salary greater than ALL salaries in IT dept (greater than the maximum in IT)
SELECT Name, Salary
FROM Employees
WHERE Salary > ALL (SELECT Salary FROM Employees WHERE Department = 'IT');
```

---

### Correlated Subquery

> A subquery that references columns from the outer query — runs once per row of the outer query.

```sql
-- Employees earning above their department's average
SELECT e1.Name, e1.Salary, e1.Department
FROM Employees e1
WHERE e1.Salary > (
    SELECT AVG(e2.Salary)
    FROM Employees e2
    WHERE e2.Department = e1.Department  -- ← Correlated: references outer query
);
```

---

### Subquery vs JOIN

| Aspect          | Subquery                                           | JOIN                                   |
| --------------- | -------------------------------------------------- | -------------------------------------- |
| **Readability** | Can be easier for simple logic                     | Better for multiple tables             |
| **Performance** | Can be slower (runs multiple times for correlated) | Usually faster                         |
| **Use case**    | Filtering based on aggregated/computed value       | Combining columns from multiple tables |

**When to use subqueries:** Filtering rows based on aggregate results (e.g., "salary > department average").

**When to use JOINs:** Combining data from multiple tables into a result set.

---

## 6. Constraints

> Constraints are **rules enforced on table columns** to maintain data integrity.

| Constraint      | Purpose                                                                       |
| --------------- | ----------------------------------------------------------------------------- |
| **PRIMARY KEY** | Uniquely identifies each row; auto-creates clustered index; NOT NULL enforced |
| **FOREIGN KEY** | Enforces referential integrity — links to a PK or Unique Key in another table |
| **UNIQUE**      | Ensures all values in column(s) are unique; allows one NULL                   |
| **NOT NULL**    | Column must have a value — cannot be NULL                                     |
| **CHECK**       | Enforces a condition (e.g., `Salary > 0`)                                     |
| **DEFAULT**     | Provides a default value if none is specified                                 |

```sql
CREATE TABLE Employees (
    EmployeeId   INT          PRIMARY KEY,              -- PK: unique, not null, clustered index
    Name         VARCHAR(100) NOT NULL,                 -- Must have a value
    Email        VARCHAR(100) UNIQUE,                   -- No duplicates (allows 1 NULL)
    Salary       DECIMAL(10,2) CHECK (Salary > 0),      -- Must be positive
    Department   VARCHAR(50)  DEFAULT 'Unassigned',     -- Default if not provided
    ManagerId    INT,
    FOREIGN KEY (ManagerId) REFERENCES Employees(EmployeeId)  -- FK: must exist in Employees.EmployeeId
);
```

**Interview Answer:** _"Constraints enforce data integrity rules. Primary Key uniquely identifies rows, Foreign Key enforces relationships, Unique prevents duplicates, NOT NULL prevents missing values, CHECK enforces custom conditions, and DEFAULT provides fallback values."_

---

## 7. Keys & Relationships

### Types of Keys

| Key Type             | Definition                                                                                                                   |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Primary Key (PK)** | A column (or set of columns) that uniquely identifies each row. NOT NULL enforced. Only one PK per table.                    |
| **Foreign Key (FK)** | A column that references a PK or Unique Key in another table — enforces referential integrity.                               |
| **Unique Key**       | Enforces uniqueness but allows one NULL. Can have multiple per table.                                                        |
| **Candidate Key**    | Any column (or set) that could serve as a PK — all are unique and not null. One is chosen as PK, rest become alternate keys. |
| **Composite Key**    | A PK made of **multiple columns** combined (e.g., `OrderId + ProductId`).                                                    |
| **Surrogate Key**    | An artificial key (often auto-increment integer) with no business meaning — just for uniqueness.                             |
| **Natural Key**      | A real-world identifier (e.g., SSN, Email) used as PK — has business meaning.                                                |

```sql
-- Composite Primary Key
CREATE TABLE OrderItems (
    OrderId    INT,
    ProductId  INT,
    Quantity   INT,
    PRIMARY KEY (OrderId, ProductId)  -- Composite PK
);

-- Surrogate Key (auto-increment)
CREATE TABLE Employees (
    EmployeeId INT IDENTITY(1,1) PRIMARY KEY,  -- Surrogate key
    Email      VARCHAR(100)
);

-- Foreign Key
CREATE TABLE Orders (
    OrderId     INT PRIMARY KEY,
    EmployeeId  INT,
    FOREIGN KEY (EmployeeId) REFERENCES Employees(EmployeeId)
);
```

---

### Referential Integrity

> Ensures that FKs always point to valid PKs — can't have orphan records.

**Actions on DELETE/UPDATE:**

| Action                   | What happens to child rows when parent is deleted/updated |
| ------------------------ | --------------------------------------------------------- |
| **CASCADE**              | Automatically delete/update child rows                    |
| **SET NULL**             | Set FK to NULL in child rows                              |
| **SET DEFAULT**          | Set FK to default value                                   |
| **NO ACTION / RESTRICT** | Prevent delete/update if children exist                   |

```sql
CREATE TABLE Orders (
    OrderId     INT PRIMARY KEY,
    EmployeeId  INT,
    FOREIGN KEY (EmployeeId) REFERENCES Employees(EmployeeId)
        ON DELETE CASCADE     -- Delete orders when employee is deleted
        ON UPDATE CASCADE     -- Update OrderId if EmployeeId changes
);
```

---

## 8. Normalization & Schema Design

### What is Normalization?

> **Normalization** is the process of organizing database schema to **reduce redundancy** and **improve data integrity**. Data is split into multiple related tables.

**Why normalize?**

- Eliminate duplicate data (saves storage)
- Prevent update anomalies (one change doesn't require updating 10 rows)
- Improve consistency

---

### Normal Forms

| Form     | Rule                                                                                                          | Problem it solves                                              |
| -------- | ------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| **1NF**  | Each column contains atomic (indivisible) values; no repeating groups                                         | No arrays or comma-separated lists in a column                 |
| **2NF**  | Must be in 1NF + all non-key columns depend on the **entire** PK (no partial dependency)                      | Composite PK: prevent columns depending on only part of the PK |
| **3NF**  | Must be in 2NF + no transitive dependencies (non-key columns depend only on PK, not on other non-key columns) | Prevent indirect dependencies between non-key columns          |
| **BCNF** | Stricter 3NF: every determinant must be a candidate key                                                       | Rare edge cases of 3NF that still have anomalies               |

---

### Example: Normalization Steps

**Unnormalized (0NF):**

```
Orders Table:
OrderId | CustomerName | CustomerEmail | Products                  | Quantities
1       | John Doe     | john@ex.com   | Laptop, Mouse, Keyboard   | 1, 2, 1
```

❌ Problems: `Products` is comma-separated (not atomic), redundant customer data.

---

**1NF (Atomic values):**

```
Orders:
OrderId | CustomerName | CustomerEmail | Product  | Quantity
1       | John Doe     | john@ex.com   | Laptop   | 1
1       | John Doe     | john@ex.com   | Mouse    | 2
1       | John Doe     | john@ex.com   | Keyboard | 1
```

✅ Each column has atomic values.
❌ Still redundant: `CustomerName` and `CustomerEmail` repeated for each product.

---

**2NF (No partial dependency):**

```
Orders:
OrderId | CustomerName | CustomerEmail

OrderItems:
OrderId | Product  | Quantity
```

✅ Non-key columns (`Product`, `Quantity`) depend on the full composite PK `(OrderId, Product)`.
❌ Still redundant: `CustomerName`, `CustomerEmail` depend on `CustomerName` (transitive).

---

**3NF (No transitive dependency):**

```
Customers:
CustomerId | CustomerName | CustomerEmail

Orders:
OrderId | CustomerId

OrderItems:
OrderId | ProductId | Quantity

Products:
ProductId | ProductName
```

✅ All non-key columns depend only on the PK. No redundancy.

---

### Denormalization

> **Denormalization** is intentionally adding redundancy to improve **read performance** at the cost of update complexity.

**When to denormalize:**

- Analytics/reporting databases (data warehouses)
- Read-heavy systems where writes are rare
- To avoid complex JOINs in frequently-run queries

**Example:** Store `CustomerName` directly in `Orders` table to avoid JOINing `Customers` table every time.

---

### Fact Tables & Dimension Tables (Data Warehousing)

| Type                | Purpose                                                 | Characteristics                                    |
| ------------------- | ------------------------------------------------------- | -------------------------------------------------- |
| **Fact Table**      | Stores measurable events (sales, transactions)          | Large, many rows, foreign keys to dimensions       |
| **Dimension Table** | Stores descriptive attributes (customer, product, date) | Smaller, denormalized, used for filtering/grouping |

**Star Schema:**

```
Fact_Sales (center)
  ├── FK: CustomerId → Dim_Customer
  ├── FK: ProductId → Dim_Product
  └── FK: DateId → Dim_Date
```

---

## 9. Indexes

### ❓ What is an Index?

An **index** is a separate data structure that SQL Server maintains to speed up data retrieval. Without an index, SQL does a **full table scan** (reads every row). With an index, it can jump directly to matching rows.

**Analogy:** Like an index in a book — instead of reading every page to find "recursion", you look in the index and go to page 245.

**Trade-off:**

- ✅ Faster `SELECT` (reads)
- ❌ Slower `INSERT`, `UPDATE`, `DELETE` (writes) — index must be updated too

**Interview Answer:** _"An index speeds up queries by allowing the database to locate rows without scanning the whole table. The trade-off is slower writes because indexes must be updated. Clustered indexes physically sort the table; non-clustered indexes are separate structures."_

---

### Clustered Index

> Physically **sorts and stores the table data** in index order. The table **IS** the clustered index.

- Only **1** per table (can only sort data one way)
- Primary Key automatically creates a clustered index

**Analogy:** A phone book sorted by last name — the data is physically organized.

```sql
CREATE CLUSTERED INDEX IX_Employee_Id ON Employees(EmployeeId);
```

---

### Non-Clustered Index

> A **separate structure** with index key + pointer to the actual row. Table data is not reordered.

- **Multiple** allowed per table
- Like an index in a textbook — doesn't change page order, just tells you where to look

```sql
CREATE NONCLUSTERED INDEX IX_Employee_Name ON Employees(Name);
```

---

### Covering Index

> A non-clustered index that **includes all columns** a query needs — the query can be fully satisfied from the index without going to the table.

```sql
CREATE NONCLUSTERED INDEX IX_Emp_Cover
ON Employees(Department)
INCLUDE (Name, Salary);  -- Includes extra columns

-- This query uses only the index:
SELECT Name, Salary FROM Employees WHERE Department = 'IT';
```

---

### Unique Index

> Enforces uniqueness on the indexed column(s).

```sql
CREATE UNIQUE INDEX IX_Employee_Email ON Employees(Email);
```

---

### Filtered Index

> An index on a **subset of rows** (with a `WHERE` clause) — smaller and more efficient.

```sql
CREATE NONCLUSTERED INDEX IX_Active_Employees
ON Employees(Name)
WHERE IsActive = 1;  -- Only indexes active employees
```

---

### Index Scan vs Index Seek

| Type           | What it does                      | Performance                 |
| -------------- | --------------------------------- | --------------------------- |
| **Index Scan** | Reads the entire index (or table) | Slow (like full table scan) |
| **Index Seek** | Jumps directly to matching rows   | Fast                        |

**Interview Tip:** You want **Index Seek** in your execution plan, not **Index Scan**.

---

### When NOT to Use Indexes

| Scenario                                                             | Why                                           |
| -------------------------------------------------------------------- | --------------------------------------------- |
| Small tables (< 1000 rows)                                           | Full scan is already fast                     |
| Columns with very few unique values (e.g., `IsActive` with only 0/1) | Index doesn't help — scans most rows anyway   |
| Heavy INSERT/UPDATE/DELETE workload                                  | Index maintenance overhead outweighs benefits |
| Columns rarely used in WHERE, JOIN, ORDER BY                         | No query benefit                              |

---

## 10. Views

### ❓ What is a View?

A **view** is a **virtual table** based on a `SELECT` query. It doesn't store data itself — it dynamically pulls data from underlying tables when queried.

**Why use views?**

- **Security** — Hide sensitive columns, expose only needed data
- **Simplify** — Encapsulate complex JOINs, users query a simple view
- **Abstraction** — Change underlying tables without breaking application queries

```sql
-- Create view
CREATE VIEW vw_ActiveEmployees AS
SELECT EmployeeId, Name, Department
FROM Employees
WHERE IsActive = 1;

-- Query it like a table
SELECT * FROM vw_ActiveEmployees;

-- Modify view
ALTER VIEW vw_ActiveEmployees AS
SELECT EmployeeId, Name, Department, Salary
FROM Employees WHERE IsActive = 1;

-- Drop view
DROP VIEW vw_ActiveEmployees;
```

---

### Types of Views

| Type                            | Description                                                          |
| ------------------------------- | -------------------------------------------------------------------- |
| **Simple View**                 | Based on one table, no aggregates/GROUP BY                           |
| **Complex View**                | Joins, GROUP BY, aggregates                                          |
| **Indexed (Materialized) View** | Physically stores result — faster reads, must manually refresh       |
| **Updatable View**              | Can `INSERT`/`UPDATE`/`DELETE` through the view (restrictions apply) |

---

### Materialized Views

> A view that **physically stores** the result. Must be manually refreshed when underlying data changes.

```sql
-- SQL Server: Indexed View
CREATE VIEW vw_DeptSalary WITH SCHEMABINDING AS
SELECT Department, SUM(Salary) AS TotalSalary, COUNT_BIG(*) AS EmployeeCount
FROM dbo.Employees
GROUP BY Department;

CREATE UNIQUE CLUSTERED INDEX IX_DeptSalary ON vw_DeptSalary(Department);
```

---

### Updatable vs Non-Updatable Views

**Updatable View** (can `INSERT`/`UPDATE`/`DELETE`):

- Single table
- No `DISTINCT`, `GROUP BY`, `HAVING`, `UNION`
- All `NOT NULL` columns included

**Non-Updatable View:**

- Contains `JOIN`, `GROUP BY`, calculated columns, subqueries
- Read-only

---

## 11. Stored Procedures

### ❓ What is a Stored Procedure?

A **stored procedure (SP)** is a **pre-compiled block of SQL code** saved in the database that you execute by name. It's compiled and cached on first run — subsequent calls reuse the cached execution plan.

**Why use SPs?**

- **Performance** — Pre-compiled, execution plan cached
- **Security** — Grant `EXECUTE` permission without exposing tables
- **Reusability** — Write once, call from multiple apps
- **Reduced network traffic** — One call vs multiple SQL statements
- **Encapsulation** — Centralize business logic

**Interview Answer:** _"A stored procedure is pre-compiled SQL code stored in the database. Benefits: cached execution plan (faster), security (users can execute SP without table access), reusability, and less network traffic."_

---

### Creating and Executing SPs

```sql
-- Create SP
CREATE PROCEDURE usp_GetEmployeesByDept
    @DeptName VARCHAR(50),
    @MinSalary DECIMAL(10,2) = 0  -- Default parameter
AS
BEGIN
    SELECT Name, Salary, Department
    FROM Employees
    WHERE Department = @DeptName AND Salary >= @MinSalary
    ORDER BY Salary DESC;
END;

-- Execute
EXEC usp_GetEmployeesByDept @DeptName = 'IT', @MinSalary = 50000;
EXEC usp_GetEmployeesByDept 'IT', 50000;  -- Positional

-- Modify
ALTER PROCEDURE usp_GetEmployeesByDept ...

-- Delete
DROP PROCEDURE usp_GetEmployeesByDept;
```

---

### Input and Output Parameters

```sql
-- SP with OUTPUT parameter
CREATE PROCEDURE usp_GetEmpCount
    @DeptName VARCHAR(50),
    @EmpCount INT OUTPUT          -- Output parameter
AS
BEGIN
    SELECT @EmpCount = COUNT(*) FROM Employees WHERE Department = @DeptName;
END;

-- Call with OUTPUT
DECLARE @Count INT;
EXEC usp_GetEmpCount @DeptName = 'IT', @EmpCount = @Count OUTPUT;
PRINT @Count;
```

---

### SP Use Cases

| Use Case                   | Example                                          |
| -------------------------- | ------------------------------------------------ |
| **Complex business logic** | Multi-step calculations, conditional logic       |
| **Logging**                | Insert audit records on every critical operation |
| **Reports**                | Pre-defined queries for dashboards               |
| **Batch processing**       | Nightly ETL jobs                                 |
| **Transaction management** | BEGIN/COMMIT/ROLLBACK in one atomic SP           |

---

## 12. Functions

### ❓ What is a User-Defined Function?

A **function** always **returns a value** and can be used directly in SQL expressions like `SELECT` and `WHERE`. Unlike SPs, functions **cannot** perform `INSERT`/`UPDATE`/`DELETE`.

---

### Types of Functions

| Type                                     | Returns                           | Use Case                                      |
| ---------------------------------------- | --------------------------------- | --------------------------------------------- |
| **Scalar Function**                      | Single value (one number, string) | Used in `SELECT`, `WHERE`                     |
| **Inline Table-Valued (ITVF)**           | A table (single `SELECT`)         | Used in `FROM` like a table, best performance |
| **Multi-Statement Table-Valued (MSTVF)** | A table (multiple statements)     | Complex logic, slower                         |

---

### Scalar Function

```sql
CREATE FUNCTION fn_GetFullName(@FirstName VARCHAR(50), @LastName VARCHAR(50))
RETURNS VARCHAR(100)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;

-- Use it
SELECT dbo.fn_GetFullName('John', 'Doe');  -- "John Doe"
SELECT Name, dbo.fn_GetFullName(FirstName, LastName) AS FullName FROM Employees;
```

---

### Inline Table-Valued Function (ITVF)

```sql
CREATE FUNCTION fn_GetEmployeesByDept(@Dept VARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT Name, Salary FROM Employees WHERE Department = @Dept
);

-- Use it like a table
SELECT * FROM fn_GetEmployeesByDept('IT');
```

---

### Multi-Statement Table-Valued Function (MSTVF)

```sql
CREATE FUNCTION fn_GetTopEarners(@TopN INT)
RETURNS @Result TABLE (Name VARCHAR(100), Salary DECIMAL(18,2))
AS
BEGIN
    INSERT INTO @Result
    SELECT TOP (@TopN) Name, Salary
    FROM Employees
    ORDER BY Salary DESC;

    -- Can add more logic here
    RETURN;
END;

SELECT * FROM fn_GetTopEarners(5);
```

---

### Deterministic vs Non-Deterministic Functions

| Type                  | Meaning                         | Examples                         |
| --------------------- | ------------------------------- | -------------------------------- |
| **Deterministic**     | Same input → always same output | `ABS()`, `LEN()`, `SUBSTRING()`  |
| **Non-Deterministic** | Output can vary                 | `GETDATE()`, `RAND()`, `NEWID()` |

**Interview Tip:** Indexed views and computed columns can only use deterministic functions.

---

### SP vs Function

| Feature                    | Stored Procedure    | Function            |
| -------------------------- | ------------------- | ------------------- |
| Returns                    | Zero or many values | Must return a value |
| Use in SELECT              | ❌ Cannot           | ✅ Can              |
| DML (INSERT/UPDATE/DELETE) | ✅ Allowed          | ❌ Not allowed      |
| Transactions               | ✅ Can use          | ❌ Cannot           |
| Call another SP            | ✅ Can              | ❌ Cannot           |

---

## 13. Window Functions (Very Important!)

### ❓ What are Window Functions?

Window functions perform calculations **across a set of rows** related to the current row — but unlike `GROUP BY`, they **do not collapse rows**. Every original row stays in the result with an extra calculated column.

**Key concept:** `OVER()` defines the "window" (set of rows) for the calculation.

---

### OVER()

```sql
-- Each employee's salary + average salary of their department
SELECT Name, Department, Salary,
       AVG(Salary) OVER(PARTITION BY Department) AS DeptAvgSalary
FROM Employees;

-- Running total of salaries
SELECT Name, Salary,
       SUM(Salary) OVER(ORDER BY EmployeeId) AS RunningTotal
FROM Employees;
```

---

### PARTITION BY

> Divides rows into groups for the window calculation — like `GROUP BY` but rows are not collapsed.

```sql
SELECT Name, Department, Salary,
       RANK() OVER(PARTITION BY Department ORDER BY Salary DESC) AS RankInDept
FROM Employees;
-- Ranks employees within each department
```

---

### ORDER BY (in window functions)

> Defines the ordering for ranking and running aggregates.

```sql
SELECT Name, Salary,
       ROW_NUMBER() OVER(ORDER BY Salary DESC) AS RowNum
FROM Employees;
```

---

### ROW_NUMBER, RANK, DENSE_RANK

| Function       | Behavior                                            | Example                  |
| -------------- | --------------------------------------------------- | ------------------------ |
| `ROW_NUMBER()` | Always unique — no ties                             | 1, 2, 3, 4               |
| `RANK()`       | Tied rows get same rank, next rank **skips**        | 1, 2, 2, **4** (skips 3) |
| `DENSE_RANK()` | Tied rows get same rank, next rank **doesn't skip** | 1, 2, 2, **3** (no skip) |

```sql
SELECT Name, Salary,
    ROW_NUMBER() OVER(ORDER BY Salary DESC) AS RowNum,
    RANK()       OVER(ORDER BY Salary DESC) AS RankNum,
    DENSE_RANK() OVER(ORDER BY Salary DESC) AS DenseRankNum
FROM Employees;

-- Result:
-- Name     Salary  RowNum  RankNum  DenseRankNum
-- Alice    90000   1       1        1
-- Bob      80000   2       2        2
-- Charlie  80000   3       2        2
-- Dave     70000   4       4        3   ← Notice the difference!
```

**Interview Tip:** The difference between `RANK` and `DENSE_RANK` is the most common follow-up question.

---

### LAG and LEAD

> Access values from **previous** (LAG) or **next** (LEAD) rows.

```sql
-- Compare each employee's salary to the previous one
SELECT Name, Salary,
       LAG(Salary, 1, 0) OVER(ORDER BY Salary DESC) AS PrevSalary,
       LEAD(Salary, 1, 0) OVER(ORDER BY Salary DESC) AS NextSalary
FROM Employees;

-- Calculate salary difference from previous employee
SELECT Name, Salary,
       Salary - LAG(Salary, 1, 0) OVER(ORDER BY Salary DESC) AS SalaryDiff
FROM Employees;
```

---

### Running Total

```sql
SELECT Name, Salary,
       SUM(Salary) OVER(ORDER BY EmployeeId ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS RunningTotal
FROM Employees;

-- Shortcut (same result):
SELECT Name, Salary,
       SUM(Salary) OVER(ORDER BY EmployeeId) AS RunningTotal
FROM Employees;
```

---

### Moving Averages

```sql
-- 3-row moving average
SELECT Name, Salary,
       AVG(Salary) OVER(ORDER BY EmployeeId ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS MovingAvg
FROM Employees;
```

---

## 14. Transactions

### ❓ What is a Transaction?

A **transaction** groups multiple SQL operations into a **single atomic unit** — either **all succeed and commit**, or **all fail and rollback**. No partial commits.

**Classic Example:** Bank transfer — debit Account A, credit Account B. Both must happen or neither.

---

### ACID Properties

| Property        | Meaning                                  | Bank Example                                       |
| --------------- | ---------------------------------------- | -------------------------------------------------- |
| **Atomicity**   | All or nothing — no partial commits      | Transfer: both debit AND credit happen, or neither |
| **Consistency** | DB moves from one valid state to another | Balance can't go negative (if rule enforced)       |
| **Isolation**   | Concurrent transactions don't interfere  | Two users booking last seat — only one wins        |
| **Durability**  | Once committed, data survives crashes    | Even server crash after COMMIT doesn't lose data   |

**Interview Answer:** _"ACID guarantees transactions are atomic (all or nothing), consistent (data stays valid), isolated (concurrent transactions don't interfere), and durable (committed data survives crashes)."_

---

### COMMIT, ROLLBACK, SAVEPOINT

```sql
BEGIN TRANSACTION;

BEGIN TRY
    UPDATE Accounts SET Balance = Balance - 1000 WHERE AccountId = 1;
    UPDATE Accounts SET Balance = Balance + 1000 WHERE AccountId = 2;

    COMMIT TRANSACTION;  -- ✅ Both succeed
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION;  -- ❌ Undo everything
    THROW;
END CATCH;
```

---

### SAVEPOINT

> A **checkpoint** within a transaction — rollback to a savepoint without undoing everything.

```sql
BEGIN TRANSACTION;
    INSERT INTO Orders VALUES(1, 'Order1');

    SAVE TRANSACTION SavePoint1;

    INSERT INTO Orders VALUES(2, 'Order2');

    ROLLBACK TRANSACTION SavePoint1;  -- Only undoes Order2

COMMIT TRANSACTION;  -- Order1 is saved
```

---

### Transaction Isolation Levels

| Level                        | Dirty Read | Non-Repeatable Read | Phantom Read | Performance            |
| ---------------------------- | ---------- | ------------------- | ------------ | ---------------------- |
| **READ UNCOMMITTED**         | ✅ Yes     | ✅ Yes              | ✅ Yes       | Fastest (no locks)     |
| **READ COMMITTED** (Default) | ❌ No      | ✅ Yes              | ✅ Yes       | Good                   |
| **REPEATABLE READ**          | ❌ No      | ❌ No               | ✅ Yes       | Slower                 |
| **SERIALIZABLE**             | ❌ No      | ❌ No               | ❌ No        | Slowest (full locks)   |
| **SNAPSHOT**                 | ❌ No      | ❌ No               | ❌ No        | Good (uses versioning) |

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN TRANSACTION;
    SELECT * FROM Employees WHERE Salary > 50000;
COMMIT;
```

---

### Deadlocks

> A **deadlock** occurs when two transactions are **waiting for each other** to release locks — neither can proceed. SQL Server kills one (the "victim").

**Prevention:**

- Access tables in the same order in all transactions
- Keep transactions short
- Use proper indexing to reduce lock duration

---

## 15. Performance Optimization

### EXPLAIN / EXECUTION PLAN

> Shows **how** SQL Server executes a query — reveals table scans, index usage, join types.

```sql
-- SQL Server
SET SHOWPLAN_TEXT ON;
GO
SELECT * FROM Employees WHERE Department = 'IT';
GO
SET SHOWPLAN_TEXT OFF;
GO

-- Or use SSMS "Display Estimated Execution Plan" (Ctrl+L)
```

**Look for:**

- ❌ **Table Scan** / **Index Scan** → BAD (reads all rows)
- ✅ **Index Seek** → GOOD (jumps to matching rows)

---

### Index Optimization

| Problem                               | Solution                                                    |
| ------------------------------------- | ----------------------------------------------------------- |
| Query does full table scan            | Add index on `WHERE`/`JOIN` columns                         |
| Query uses index scan instead of seek | Index exists but isn't selective — check WHERE clause       |
| Too many indexes                      | Remove unused indexes (check `sys.dm_db_index_usage_stats`) |
| Index fragmentation > 30%             | REBUILD index                                               |

---

### Avoiding SELECT \*

> `SELECT *` retrieves all columns — wastes network bandwidth and memory.

```sql
-- ❌ Bad
SELECT * FROM Employees;

-- ✅ Good
SELECT EmployeeId, Name, Salary FROM Employees;
```

---

### Avoiding Subquery Misuse

| Problem                                            | Solution                              |
| -------------------------------------------------- | ------------------------------------- |
| Correlated subquery in `WHERE` (runs once per row) | Rewrite as `JOIN` or `EXISTS`         |
| Subquery in `SELECT` list (runs once per row)      | Use `JOIN` or window function instead |

```sql
-- ❌ Slow (correlated subquery)
SELECT Name,
       (SELECT AVG(Salary) FROM Employees e2 WHERE e2.Department = e1.Department) AS AvgSalary
FROM Employees e1;

-- ✅ Fast (window function)
SELECT Name, AVG(Salary) OVER(PARTITION BY Department) AS AvgSalary
FROM Employees;
```

---

### Sharding & Partitioning

| Technique        | What it is                                                                                                             |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Partitioning** | Split a large table into smaller pieces (partitions) based on a key (e.g., date range). All partitions in the same DB. |
| **Sharding**     | Distribute data across multiple databases/servers. Each shard is independent.                                          |

**Example:** Partition `Orders` table by year — `Orders_2023`, `Orders_2024`, `Orders_2025`. Queries on 2024 data only touch one partition.

---

## 16. Triggers

### ❓ What is a Trigger?

A **trigger** is SQL code that **automatically executes** when a specific event (`INSERT`, `UPDATE`, `DELETE`) occurs on a table. You don't call it manually — the database fires it automatically.

**Special tables inside triggers:**

- `INSERTED` — Contains new rows (available in `INSERT` and `UPDATE`)
- `DELETED` — Contains old rows (available in `DELETE` and `UPDATE`)
- In `UPDATE`, `DELETED` has old values, `INSERTED` has new values

---

### Types of Triggers

| Type                    | When it fires                     |
| ----------------------- | --------------------------------- |
| **AFTER Trigger** (FOR) | After the DML operation completes |
| **INSTEAD OF Trigger**  | Replaces the DML operation        |

---

### Trigger Examples

```sql
-- AFTER INSERT Trigger — Audit log
CREATE TRIGGER trg_AfterInsertEmployee
ON Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO AuditLog(Action, EmployeeName, ActionDate)
    SELECT 'INSERT', Name, GETDATE()
    FROM INSERTED;  -- INSERTED is a special table with new rows
END;

-- INSTEAD OF DELETE — Soft delete
CREATE TRIGGER trg_InsteadOfDelete
ON Employees
INSTEAD OF DELETE
AS
BEGIN
    UPDATE Employees SET IsActive = 0
    WHERE EmployeeId IN (SELECT EmployeeId FROM DELETED);
END;

-- AFTER UPDATE — Track salary changes
CREATE TRIGGER trg_AfterUpdateSalary
ON Employees
AFTER UPDATE
AS
BEGIN
    IF UPDATE(Salary)
    BEGIN
        INSERT INTO SalaryHistory(EmployeeId, OldSalary, NewSalary, ChangeDate)
        SELECT i.EmployeeId, d.Salary, i.Salary, GETDATE()
        FROM INSERTED i
        INNER JOIN DELETED d ON i.EmployeeId = d.EmployeeId;
    END
END;
```

---

### Trigger Use Cases

| Use Case                     | Example                                               |
| ---------------------------- | ----------------------------------------------------- |
| **Audit logs**               | Auto-log every INSERT/UPDATE/DELETE                   |
| **Enforcing business rules** | Prevent salary decrease, ensure referential integrity |
| **Cascading updates**        | Update related tables automatically                   |
| **Soft deletes**             | Set `IsActive = 0` instead of deleting                |

---

### When to Be Careful with Triggers

| Risk                   | Why                                                        |
| ---------------------- | ---------------------------------------------------------- |
| **Hidden logic**       | Developers don't know trigger is running — hard to debug   |
| **Performance**        | Triggers fire on every matching DML — adds overhead        |
| **Cascading triggers** | A trigger can fire another trigger, causing complex chains |
| **Difficult testing**  | Triggers don't show in application code                    |

---

## 17. Advanced SQL

### Common Table Expressions (CTE)

**What is it?**
A **CTE** is a temporary named result set defined with `WITH` that exists only for the duration of one query. It improves readability and is the **only way** to write recursive queries in SQL Server.

```sql
-- Basic CTE
WITH HighPaidEmployees AS (
    SELECT Name, Salary, Department
    FROM Employees
    WHERE Salary > 70000
)
SELECT * FROM HighPaidEmployees WHERE Department = 'IT';
```

---

### Recursive CTE

**What is it?**
A CTE that **calls itself** — used for hierarchical data (employee-manager, category trees).

**Structure:**

1. **Anchor member** — starting row(s), no recursion
2. **UNION ALL**
3. **Recursive member** — references the CTE itself

```sql
-- Employee hierarchy (employee → manager → CEO)
WITH OrgChart AS (
    -- Anchor: Top-level manager (no manager above)
    SELECT EmployeeId, Name, ManagerId, 0 AS Level
    FROM Employees WHERE ManagerId IS NULL

    UNION ALL

    -- Recursive: Get employees under each manager
    SELECT e.EmployeeId, e.Name, e.ManagerId, oc.Level + 1
    FROM Employees e
    INNER JOIN OrgChart oc ON e.ManagerId = oc.EmployeeId
)
SELECT * FROM OrgChart;
```

---

### PIVOT / UNPIVOT

> **PIVOT** converts rows to columns. **UNPIVOT** converts columns to rows.

```sql
-- PIVOT: Convert rows to columns
SELECT * FROM (
    SELECT Department, Year, Sales
    FROM SalesData
) AS SourceTable
PIVOT (
    SUM(Sales) FOR Year IN ([2023], [2024], [2025])
) AS PivotTable;

-- Result:
-- Department | 2023  | 2024  | 2025
-- IT         | 50000 | 60000 | 70000

-- UNPIVOT: Convert columns to rows
SELECT Department, Year, Sales
FROM PivotTable
UNPIVOT (
    Sales FOR Year IN ([2023], [2024], [2025])
) AS UnpivotTable;
```

---

### Ranking Queries (Recap)

```sql
-- Top 3 salaries per department
SELECT * FROM (
    SELECT Name, Department, Salary,
           ROW_NUMBER() OVER(PARTITION BY Department ORDER BY Salary DESC) AS Rank
    FROM Employees
) AS Ranked
WHERE Rank <= 3;
```

---

### MERGE Statement (UPSERT)

> Performs **INSERT, UPDATE, and DELETE** in a single statement based on matching conditions.

```sql
MERGE INTO TargetTable AS T
USING SourceTable AS S
ON T.Id = S.Id
WHEN MATCHED THEN
    UPDATE SET T.Name = S.Name, T.Salary = S.Salary
WHEN NOT MATCHED BY TARGET THEN
    INSERT (Id, Name, Salary) VALUES (S.Id, S.Name, S.Salary)
WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

---

### Full-Text Search

> Search large text columns efficiently (articles, descriptions).

```sql
-- Enable full-text search on a column
CREATE FULLTEXT INDEX ON Articles(Content)
    KEY INDEX PK_Article;

-- Search
SELECT * FROM Articles
WHERE CONTAINS(Content, 'SQL OR database');
```

---

## 🎯 Top 50 SQL Interview Questions

### Basics & Queries

1. What is SQL? What are its types (DDL, DML, DQL, DCL, TCL)?
2. What is the difference between `DELETE`, `TRUNCATE`, and `DROP`?
3. What is `NULL`? How do you check for `NULL`?
4. Difference between `WHERE` and `HAVING`?
5. What is `DISTINCT`? When would you use it?
6. What is `LIMIT` / `TOP`?
7. Explain `BETWEEN`, `IN`, `LIKE` with examples.

### Joins

8. What are the types of JOINs? Explain each.
9. Difference between `INNER JOIN` and `LEFT JOIN`?
10. What is a `SELF JOIN`? Give an example.
11. What is a `CROSS JOIN`?
12. When would you use `FULL OUTER JOIN`?

### Aggregates & Grouping

13. What are aggregate functions? Name 5.
14. What is `GROUP BY`? How does it work?
15. Difference between `WHERE` and `HAVING`?
16. Write a query: Count employees per department, show only depts with > 5 employees.

### Subqueries

17. What is a subquery? Types?
18. Difference between correlated and non-correlated subquery?
19. What is `EXISTS`? How is it different from `IN`?
20. When to use subquery vs `JOIN`?

### Constraints & Keys

21. What are constraints? Name all types.
22. Difference between `PRIMARY KEY` and `UNIQUE KEY`?
23. What is a `FOREIGN KEY`? What is referential integrity?
24. What is a composite key? Give an example.
25. Surrogate key vs Natural key?

### Normalization

26. What is normalization? Why is it important?
27. Explain 1NF, 2NF, 3NF with examples.
28. What is denormalization? When would you denormalize?

### Indexes

29. What is an index? Why use it?
30. Difference between clustered and non-clustered index?
31. What is a covering index?
32. When should you NOT use an index?
33. What is index scan vs index seek?

### Views, SPs, Functions

34. What is a view? Types?
35. What is a materialized view?
36. What is a stored procedure? Benefits?
37. Difference between stored procedure and function?
38. What are input/output parameters in SP?

### Window Functions

39. What are window functions? Difference from `GROUP BY`?
40. Explain `ROW_NUMBER`, `RANK`, `DENSE_RANK` with example.
41. What is `PARTITION BY`?
42. What are `LAG` and `LEAD`?
43. How do you calculate a running total?

### Transactions

44. What is a transaction? Explain ACID properties.
45. What are transaction isolation levels?
46. What is a deadlock? How to prevent it?
47. Difference between `COMMIT` and `ROLLBACK`?

### Performance & Advanced

48. How do you optimize a slow query?
49. What is an execution plan?
50. What is a trigger? When to use it?

---

_[← Back to README](./README.md) | [Next: Module 4 →](./Module4_EFCore_Dapper.md)_
