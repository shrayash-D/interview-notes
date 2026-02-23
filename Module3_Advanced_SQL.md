# 📌 MODULE 3: ADVANCED SQL USING SQL SERVER

---

## 3.1 Getting Started

### What is SQL Server?

> **SQL Server** is a **Relational Database Management System (RDBMS)** by Microsoft used to store, retrieve, and manage data.

**Interview Answer:** "SQL Server is Microsoft's enterprise RDBMS that supports T-SQL for querying data. It provides tools for data storage, security, reporting, and integration services."

### SQL Server Editions

| Edition        | Use Case                                  |
| -------------- | ----------------------------------------- |
| **Enterprise** | Full features, large-scale production     |
| **Standard**   | Mid-level, limited features vs Enterprise |
| **Express**    | Free, lightweight, max 10GB database      |
| **Developer**  | Free, full features, for dev/testing only |

### Key Components

| Component                               | What It Does                                 |
| --------------------------------------- | -------------------------------------------- |
| **Database Engine**                     | Core service for storing and processing data |
| **SSMS (SQL Server Management Studio)** | GUI tool to manage SQL Server                |
| **SSIS (Integration Services)**         | ETL tool — Extract, Transform, Load data     |
| **SSRS (Reporting Services)**           | Build and manage reports                     |
| **SSAS (Analysis Services)**            | Data analysis and mining                     |

### SQL Server Instance

> An **instance** is an independent installation of SQL Server. One machine can have **multiple instances** (default + named instances).

---

## 3.1a Primary Key vs Unique Key

> Both enforce uniqueness on a column — but with important differences.

| Aspect              | **Primary Key**               | **Unique Key**                         |
| ------------------- | ----------------------------- | -------------------------------------- |
| NULL allowed?       | ❌ Never (NOT NULL enforced)  | ✅ One NULL allowed per column         |
| Per table           | Only **1** primary key        | **Multiple** unique keys allowed       |
| Index created       | Clustered index (by default)  | Non-clustered index (by default)       |
| FK reference target | ✅ FKs typically reference PK | ✅ FK can also reference a Unique Key  |
| Purpose             | Uniquely identifies every row | Enforces uniqueness for alternate keys |

```sql
CREATE TABLE Employees (
    EmployeeId  INT          PRIMARY KEY,           -- PK: not null, clustered, 1 per table
    Email       VARCHAR(100) UNIQUE,                -- UK: allows 1 null, non-clustered
    Phone       VARCHAR(20)  UNIQUE,                -- Multiple UKs allowed ✅
    Name        VARCHAR(100) NOT NULL
);

-- Composite Primary Key
CREATE TABLE OrderItems (
    OrderId    INT,
    ProductId  INT,
    Quantity   INT,
    PRIMARY KEY (OrderId, ProductId)                -- Composite PK
);

-- A Foreign Key can reference a Unique Key too
CREATE TABLE Invoices (
    InvoiceId  INT PRIMARY KEY,
    Email      VARCHAR(100) REFERENCES Employees(Email)  -- Referencing UK ✅
);
```

**Interview Answer:** _"A Primary Key enforces uniqueness AND NOT NULL. Only one PK per table; it creates a clustered index. A Unique Key also enforces uniqueness but allows one NULL per column, and you can have multiple unique keys per table. Both can be referenced by foreign keys."_

---

## 3.2 Advanced SQL Concepts

### OVER() Clause — Window Functions

**What is it?**
The `OVER()` clause turns an aggregate function (like `SUM`, `AVG`, `COUNT`) into a **window function**. The key difference from `GROUP BY` is: window functions calculate across a set of rows but **do not collapse those rows** — every original row stays in the result with an extra calculated column added.

**In plain English:** `GROUP BY` squishes many rows into one summary row per group. `OVER()` keeps all original rows but adds a calculation column alongside each one.

**Why use it?**
Imagine you want each employee's salary AND the average salary of their department in the same row. With `GROUP BY` you can't do this — you'd either get just the average, or you'd need a self-join. `OVER()` makes it trivial.

```sql
-- Get each employee's salary AND the average salary of their department
SELECT Name, Department, Salary,
       AVG(Salary) OVER(PARTITION BY Department) AS DeptAvgSalary
FROM Employees;
-- Each row keeps its individual data + gets the department average alongside it
```

### PARTITION BY

**What is it?**
`PARTITION BY` is used inside `OVER()` to divide rows into groups (partitions) for the window calculation. It works like `GROUP BY` for defining the scope of the window function, but again — rows are NOT collapsed.

```sql
SELECT Name, Department, Salary,
       SUM(Salary) OVER(PARTITION BY Department) AS DeptTotal
FROM Employees;
-- Each row shows individual salary + total department salary side-by-side
```

**Interview Answer:** "Window functions with `OVER()` perform calculations across a related set of rows without collapsing them. `PARTITION BY` defines the grouping scope. Unlike `GROUP BY`, you keep all original rows and get the aggregate alongside each — perfect for things like 'show each employee's salary vs their department average'."

### ROW_NUMBER, RANK, DENSE_RANK

**What are they?**
These are **ranking window functions** that assign a number to each row based on an ordering. The difference is how they handle tied values.

- **ROW_NUMBER** — Always unique. Even if two rows have the same value, they get different numbers. Order between ties is arbitrary.
- **RANK** — Tied rows get the same rank, then the next rank **skips** the gap (1, 2, 2, 4 — rank 3 is skipped).
- **DENSE_RANK** — Tied rows get the same rank, but the next rank does **not skip** (1, 2, 2, 3 — no gap).

**Interview tip:** The difference between `RANK` and `DENSE_RANK` is the most common follow-up question.

| Function       | What It Does                          | Handles Ties             |
| -------------- | ------------------------------------- | ------------------------ |
| `ROW_NUMBER()` | Unique sequential number for each row | No ties — always unique  |
| `RANK()`       | Ranking with gaps for ties            | 1, 2, 2, **4** (skips 3) |
| `DENSE_RANK()` | Ranking without gaps for ties         | 1, 2, 2, **3** (no skip) |

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
-- Dave     70000   4       4        3   ← See the difference!
```

### GROUPING SETS, CUBE, ROLLUP

| Feature         | What It Does                              |
| --------------- | ----------------------------------------- |
| `GROUPING SETS` | Group by specific combinations you define |
| `ROLLUP`        | Hierarchical subtotals (top-down)         |
| `CUBE`          | All possible combinations of groupings    |

```sql
-- GROUPING SETS — custom groupings
SELECT Department, Year, SUM(Sales)
FROM SalesData
GROUP BY GROUPING SETS ((Department), (Year), ());

-- ROLLUP — hierarchical subtotals
SELECT Department, Year, SUM(Sales)
FROM SalesData
GROUP BY ROLLUP(Department, Year);

-- CUBE — all combinations
SELECT Department, Year, SUM(Sales)
FROM SalesData
GROUP BY CUBE(Department, Year);
```

### Common Table Expression (CTE)

**What is it?**
A CTE is a **temporary named result set** that you define at the top of a query using the `WITH` keyword. It exists only for the duration of that one query — it is not stored, not cached, just a named reference.

**Why use it?**

- Makes complex queries readable by breaking them into named steps
- Replaces deeply nested subqueries that are hard to read
- Can be referenced multiple times within the same query
- The only way to write recursive queries in SQL Server

**CTE vs Subquery vs Temp Table:**

- **Subquery** — Nested inside the main query, often repeated if needed multiple times, hard to read
- **CTE** — Named, at the top, readable, but exists only once per query execution
- **Temp Table** — Physically stored in `tempdb`, survives across multiple queries in a session, good for large datasets or reuse across statements

```sql
-- Basic CTE — replace a messy nested subquery with a named, readable block
WITH HighPaidEmployees AS (
    SELECT Name, Salary, Department
    FROM Employees
    WHERE Salary > 70000
)
SELECT * FROM HighPaidEmployees WHERE Department = 'IT';
-- Much more readable than: SELECT * FROM (SELECT ... FROM Employees WHERE ...) sub WHERE ...
```

**Interview Answer:** "A CTE is a temporary named result set defined with `WITH` that exists only for one query. It improves readability, replaces nested subqueries, and is the only way to write recursive queries in SQL Server. Unlike a temp table, a CTE isn't stored — it's just a named reference inside a single query."

### Recursive CTE

**What is it?**
A recursive CTE is a CTE that **calls itself** — it has two parts joined with `UNION ALL`:

1. **Anchor member** — the starting row(s), no recursion
2. **Recursive member** — references the CTE itself to get the next level

**When to use it:**

- Organizational hierarchies (employee → manager → CEO)
- Category trees (subcategory → parent category)
- Bill of materials
- Any parent-child relationship stored in the same table

```sql
-- Get employee hierarchy
WITH OrgChart AS (
    -- Anchor: Top-level manager
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

### MERGE Statement

> Performs **INSERT, UPDATE, and DELETE** in a **single statement** based on matching conditions. Often called "UPSERT".

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

### PIVOT and UNPIVOT

> **PIVOT** = Rows → Columns | **UNPIVOT** = Columns → Rows

```sql
-- PIVOT: Convert rows to columns
SELECT * FROM (
    SELECT Department, Year, Sales
    FROM SalesData
) AS SourceTable
PIVOT (
    SUM(Sales) FOR Year IN ([2023], [2024], [2025])
) AS PivotTable;

-- UNPIVOT: Convert columns to rows
SELECT Department, Year, Sales
FROM PivotTable
UNPIVOT (
    Sales FOR Year IN ([2023], [2024], [2025])
) AS UnpivotTable;
```

---

## 3.3 Views and Indexes

### What is a View?

> A **View** is a **virtual table** based on a SELECT query. It doesn't store data itself — it pulls data from underlying tables.

```sql
-- Create a view
CREATE VIEW vw_ActiveEmployees AS
SELECT EmployeeId, Name, Department
FROM Employees WHERE IsActive = 1;

-- Use it like a table
SELECT * FROM vw_ActiveEmployees;

-- Modify a view
ALTER VIEW vw_ActiveEmployees AS
SELECT EmployeeId, Name, Department, Salary
FROM Employees WHERE IsActive = 1;

-- Drop a view
DROP VIEW vw_ActiveEmployees;
```

### Types of Views

| Type                            | Description                             |
| ------------------------------- | --------------------------------------- |
| **Simple View**                 | Based on one table, no functions/groups |
| **Complex View**                | Joins, functions, GROUP BY              |
| **Indexed (Materialized) View** | Physically stores data — faster reads   |
| **Partitioned View**            | Combines tables from multiple servers   |

### What is an Index?

**What is it?**
An index is a **separate data structure** that SQL Server maintains alongside your table to speed up data lookups. Without an index, SQL Server must scan every row (full table scan) to find matching data. With an index, it can jump directly to the relevant rows.

**Real-world analogy:** A book's index at the back. Instead of reading every page to find "recursion", you look it up in the index and go directly to page 245.

**The trade-off:** Indexes speed up reads (`SELECT`) but slow down writes (`INSERT`, `UPDATE`, `DELETE`) because SQL Server must also update the index when data changes. Don't index every column — index columns used frequently in `WHERE`, `JOIN`, and `ORDER BY`.

**Interview Answer:** "An index is a data structure that improves SELECT performance by allowing SQL Server to locate rows without scanning the entire table. The trade-off is slower writes because the index must be updated too. Clustered indexes physically sort the table data; non-clustered indexes are separate structures with pointers to the actual rows."

### Types of Indexes

| Type                    | Description                                                                                                       | Count per table                                     |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| **Clustered Index**     | Sorts and physically stores the actual table data in this order. The table IS the clustered index.                | Only **1** — because you can only sort data one way |
| **Non-Clustered Index** | A separate structure with index key + pointer back to the actual row.                                             | **Multiple** allowed                                |
| **Unique Index**        | Enforces uniqueness on the indexed column(s)                                                                      | Multiple                                            |
| **Covering Index**      | Non-clustered index that includes all columns the query needs (via INCLUDE) — avoids going back to the base table | Multiple                                            |
| **Filtered Index**      | Index on a subset of rows (with a WHERE clause) — smaller, more efficient for selective queries                   | Multiple                                            |

**Clustered vs Non-Clustered — in plain English:**

- **Clustered** = The rows ARE stored in index order. Like a phone book sorted by last name. There can only be one because you can only sort data one way. Primary Keys automatically get a clustered index.
- **Non-clustered** = A separate lookup structure. Like the index in a textbook — it doesn't rearrange the pages, it just tells you which page to go to.

| Type                    | Description                                                                             |
| ----------------------- | --------------------------------------------------------------------------------------- |
| **Clustered Index**     | Sorts and stores actual table data. **Only 1 per table.** Primary Key auto-creates one. |
| **Non-Clustered Index** | Separate structure with pointers to data. **Multiple allowed.**                         |
| **Unique Index**        | Ensures all values in the column are unique                                             |
| **Covering Index**      | Includes all columns needed by a query (avoids going back to table)                     |
| **Filtered Index**      | Index on a subset of rows (with WHERE)                                                  |

```sql
-- Create Clustered Index
CREATE CLUSTERED INDEX IX_Employee_Id ON Employees(EmployeeId);

-- Create Non-Clustered Index
CREATE NONCLUSTERED INDEX IX_Employee_Name ON Employees(Name);

-- Create Covering Index (INCLUDE extra columns)
CREATE NONCLUSTERED INDEX IX_Emp_Cover
ON Employees(Department)
INCLUDE (Name, Salary);

-- Drop Index
DROP INDEX IX_Employee_Name ON Employees;
```

### Index Fragmentation

> Over time, as data is inserted/updated/deleted, indexes get **fragmented** (data becomes scattered). This slows down queries.

```sql
-- Check fragmentation
SELECT * FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('Employees'), NULL, NULL, 'LIMITED');

-- Fix: Reorganize (< 30% fragmentation)
ALTER INDEX IX_Employee_Name ON Employees REORGANIZE;

-- Fix: Rebuild (> 30% fragmentation)
ALTER INDEX IX_Employee_Name ON Employees REBUILD;
```

---

## 3.4 Stored Procedures and User-Defined Functions

### What is a Stored Procedure?

**What is it?**
A stored procedure is a **named, saved block of SQL code** stored inside the database that you can execute by name. The first time it runs, SQL Server compiles it and caches the execution plan — subsequent calls reuse the cached plan, making it faster.

**Why use stored procedures?**

- **Performance:** Pre-compiled and cached execution plan
- **Security:** Grant EXECUTE permission without exposing the underlying tables
- **Reusability:** Write once, call from anywhere (app, reports, other SPs)
- **Reduced network traffic:** Send one `EXEC` call instead of a long SQL query
- **Encapsulation:** Business logic lives in the database, centralized

**Stored Procedure vs ad-hoc SQL:**
Ad-hoc SQL (SQL written directly in your app code) is compiled fresh every time unless the execution plan is cached by coincidence. Stored procedures guarantee the plan is cached and reused.

**Interview Answer:** "A stored procedure is a pre-compiled block of SQL stored in the database. Benefits are performance (cached execution plan), security (grant EXEC without table access), reusability, and reduced network traffic. The main downside is that business logic is split between the application and database, making maintenance harder."

### Types of Stored Procedures

| Type             | Description                                             |
| ---------------- | ------------------------------------------------------- |
| **User-defined** | Created by developers for custom logic                  |
| **System**       | Built-in (`sp_help`, `sp_rename`, etc.)                 |
| **Temporary**    | `#local` (current session) or `##global` (all sessions) |
| **Extended**     | Call external programs (legacy, use CLR now)            |

```sql
-- Create a Stored Procedure
CREATE PROCEDURE usp_GetEmployeesByDept
    @DeptName NVARCHAR(50),
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

-- Modify
ALTER PROCEDURE usp_GetEmployeesByDept ...

-- Delete
DROP PROCEDURE usp_GetEmployeesByDept;

-- With OUTPUT parameter
CREATE PROCEDURE usp_GetEmpCount
    @DeptName NVARCHAR(50),
    @EmpCount INT OUTPUT
AS
BEGIN
    SELECT @EmpCount = COUNT(*) FROM Employees WHERE Department = @DeptName;
END;

-- Call with OUTPUT
DECLARE @Count INT;
EXEC usp_GetEmpCount @DeptName = 'IT', @EmpCount = @Count OUTPUT;
PRINT @Count;
```

### User-Defined Functions (UDFs)

**What is it?**
A UDF is a reusable piece of SQL logic that **always returns a value** and can be used directly inside SQL statements like `SELECT`, `WHERE`, and `FROM`. Unlike stored procedures, functions cannot modify data (no `INSERT`, `UPDATE`, `DELETE`).

**Three types:**

- **Scalar** — Returns a single value (one number, one string). Used inline in SELECT/WHERE.
- **Inline Table-Valued (ITVF)** — Returns a table from a single SELECT. SQL Server can inline/optimize it. Preferred over multi-statement TVF for performance.
- **Multi-Statement Table-Valued (MSTVF)** — Returns a table but uses multiple statements with conditional logic. Less performant because SQL Server treats the result as opaque — it can't optimize inside it.

**Interview Answer:** "Functions always return a value and can be used in SQL expressions. Scalar functions return a single value; table-valued functions return a table and can be used in FROM clauses like a table. Unlike stored procedures, functions cannot perform DML operations and cannot use transactions."

```sql
-- Scalar Function
CREATE FUNCTION fn_GetFullName(@FirstName NVARCHAR(50), @LastName NVARCHAR(50))
RETURNS NVARCHAR(100)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;

SELECT dbo.fn_GetFullName('John', 'Doe'); -- "John Doe"

-- Inline Table-Valued Function
CREATE FUNCTION fn_GetEmployeesByDept(@Dept NVARCHAR(50))
RETURNS TABLE
AS
RETURN (
    SELECT Name, Salary FROM Employees WHERE Department = @Dept
);

SELECT * FROM fn_GetEmployeesByDept('IT');

-- Multi-Statement Table-Valued Function
-- Use when you need multiple statements / conditional logic before returning
CREATE FUNCTION dbo.GetTopEarners(@TopN INT)
RETURNS @Result TABLE (Name NVARCHAR(100), Salary DECIMAL(18,2))
AS
BEGIN
    INSERT INTO @Result
    SELECT TOP (@TopN) Name, Salary
    FROM Employees
    ORDER BY Salary DESC;

    -- Can do more logic here (IF, WHILE, multiple INSERTs, etc.)
    RETURN;
END;

SELECT * FROM dbo.GetTopEarners(5);
```

### Stored Procedure vs Function

| Feature                    | Stored Procedure    | Function            |
| -------------------------- | ------------------- | ------------------- |
| Returns                    | Zero or many values | Must return a value |
| Usage in SELECT            | ❌ Cannot           | ✅ Can              |
| DML (INSERT/UPDATE/DELETE) | ✅ Allowed          | ❌ Not allowed      |
| Transaction                | ✅ Can use          | ❌ Cannot           |
| Call another SP            | ✅ Can              | ❌ Cannot           |

---

## 3.5 Triggers and Cursors

### What is a Trigger?

**What is it?**
A trigger is SQL code that **automatically executes** when a specific event (`INSERT`, `UPDATE`, `DELETE`) occurs on a table. You don't call it manually — the database fires it automatically.

**Special tables inside triggers:**

- `INSERTED` — Contains the new rows (available in INSERT and UPDATE triggers)
- `DELETED` — Contains the old rows (available in DELETE and UPDATE triggers)
- In an UPDATE trigger, `DELETED` has the old values and `INSERTED` has the new values

**When to use triggers:**

- Auditing — auto-log every INSERT/UPDATE/DELETE to an audit table
- Enforcing business rules that can't be done with constraints
- Soft deletes — intercept DELETE and set `IsActive = 0` instead

**Why to be careful with triggers:**

- **Hidden logic** — A developer doing an INSERT has no idea a trigger is running behind the scenes
- **Hard to debug** — Trigger errors appear as part of the DML operation, confusing the caller
- **Performance** — Triggers fire on every matching DML event, adding overhead
- **Cascading triggers** — A trigger can fire another trigger, causing hard-to-trace chains

### Types of Triggers

| Type                    | When It Fires                      |
| ----------------------- | ---------------------------------- |
| **AFTER Trigger** (FOR) | After the DML operation completes  |
| **INSTEAD OF Trigger**  | Replaces the DML operation         |
| **Logon Trigger**       | When a user session is established |

```sql
-- AFTER INSERT Trigger — Log new employees
CREATE TRIGGER trg_AfterInsertEmployee
ON Employees
AFTER INSERT
AS
BEGIN
    INSERT INTO AuditLog(Action, EmployeeName, ActionDate)
    SELECT 'INSERT', Name, GETDATE()
    FROM INSERTED;  -- INSERTED is a special table with new rows
END;

-- INSTEAD OF DELETE — Soft delete instead of hard delete
CREATE TRIGGER trg_InsteadOfDelete
ON Employees
INSTEAD OF DELETE
AS
BEGIN
    UPDATE Employees SET IsActive = 0
    WHERE EmployeeId IN (SELECT EmployeeId FROM DELETED);
END;
```

**Advantages:** Automatic auditing, enforce business rules, maintain data integrity.
**Disadvantages:** Hidden logic (hard to debug), performance overhead, can cause chain reactions.

### What is a Cursor?

> A **Cursor** lets you process rows **one at a time** instead of as a set. Think of it as a `for` loop for SQL.

```sql
-- Cursor Example
DECLARE @Name NVARCHAR(50), @Salary DECIMAL(10,2);

DECLARE emp_cursor CURSOR FOR
    SELECT Name, Salary FROM Employees WHERE Department = 'IT';

OPEN emp_cursor;
FETCH NEXT FROM emp_cursor INTO @Name, @Salary;

WHILE @@FETCH_STATUS = 0
BEGIN
    PRINT @Name + ' earns ' + CAST(@Salary AS NVARCHAR);
    FETCH NEXT FROM emp_cursor INTO @Name, @Salary;
END;

CLOSE emp_cursor;
DEALLOCATE emp_cursor;
```

### Cursor Life Cycle

`DECLARE → OPEN → FETCH → (PROCESS) → CLOSE → DEALLOCATE`

### Types of Cursors

| Type             | Description                              |
| ---------------- | ---------------------------------------- |
| **Static**       | Snapshot of data at open time            |
| **Dynamic**      | Reflects real-time changes               |
| **Forward-Only** | Can only move forward (default, fastest) |
| **Keyset**       | Detects changes to existing rows         |

⚠️ **Cursor Alternatives (Preferred):** Use `SET-based operations`, `WHILE loops with temp tables`, `CTEs`, or `Window Functions` instead. Cursors are slow!

---

## 3.6 Exception Handling

### TRY...CATCH in SQL Server

```sql
BEGIN TRY
    -- Code that might fail
    INSERT INTO Employees(Name, Salary) VALUES('John', -100);
END TRY
BEGIN CATCH
    -- Handle the error
    SELECT
        ERROR_NUMBER()    AS ErrorNumber,
        ERROR_MESSAGE()   AS ErrorMessage,
        ERROR_SEVERITY()  AS ErrorSeverity,
        ERROR_STATE()     AS ErrorState,
        ERROR_LINE()      AS ErrorLine,
        ERROR_PROCEDURE() AS ErrorProcedure;
END CATCH;
```

### THROW vs RAISERROR

| Feature         | THROW                | RAISERROR       |
| --------------- | -------------------- | --------------- |
| Introduced      | SQL Server 2012      | Older           |
| Severity        | Always 16            | Customizable    |
| Re-throw        | `THROW;` (no params) | Cannot re-throw |
| **Recommended** | ✅ Yes (modern)      | Legacy use      |

```sql
-- THROW
THROW 50001, 'Custom error message', 1;

-- RAISERROR
RAISERROR('Error occurred: %s', 16, 1, 'Invalid data');

-- Re-throw in CATCH
BEGIN CATCH
    PRINT 'Logging error...';
    THROW; -- Re-throws original error
END CATCH;
```

---

## 3.7 Transactions

### What is a Transaction?

**What is it?**
A transaction is a group of SQL operations that are treated as a **single atomic unit** — either ALL operations succeed and commit, or if ANY operation fails, ALL are rolled back as if none of them happened.

**The classic example:** Bank transfer. You need to debit Account A AND credit Account B. If the debit succeeds but the credit fails (e.g., network error), you can't leave Account A deducted without Account B receiving the money. A transaction ensures both happen or neither happens.

**ACID — the four guarantees of transactions:**

| Property        | Meaning                                                 | Bank Example                                         |
| --------------- | ------------------------------------------------------- | ---------------------------------------------------- |
| **Atomicity**   | All or nothing — no partial commits                     | Transfer: both debit AND credit happen, or neither   |
| **Consistency** | DB goes from one valid state to another valid state     | Balance can't go below zero if business rule says so |
| **Isolation**   | Concurrent transactions don't interfere with each other | Two users booking the last seat — only one wins      |
| **Durability**  | Once committed, data survives crashes, restarts         | Even server crash after COMMIT doesn't lose data     |

**Interview Answer:** "A transaction groups SQL operations into an atomic unit — either all succeed or all fail, maintaining database integrity. ACID properties guarantee: Atomicity (all or nothing), Consistency (DB stays valid), Isolation (concurrent transactions don't interfere), Durability (committed data survives crashes)."

```sql
BEGIN TRANSACTION;

BEGIN TRY
    UPDATE Accounts SET Balance = Balance - 1000 WHERE AccountId = 1;
    UPDATE Accounts SET Balance = Balance + 1000 WHERE AccountId = 2;

    COMMIT TRANSACTION; -- ✅ Both succeed
END TRY
BEGIN CATCH
    ROLLBACK TRANSACTION; -- ❌ Undo everything
    THROW;
END CATCH;
```

### Savepoints

> A **savepoint** is a checkpoint within a transaction — you can rollback to a specific point without undoing everything.

```sql
BEGIN TRANSACTION;
    INSERT INTO Orders VALUES(1, 'Order1');

    SAVE TRANSACTION SavePoint1;

    INSERT INTO Orders VALUES(2, 'Order2');

    ROLLBACK TRANSACTION SavePoint1; -- Only undoes Order2
COMMIT TRANSACTION; -- Order1 is saved
```

### Transaction Isolation Levels

| Level                        | Dirty Read | Non-Repeatable Read | Phantom Read | Performance            |
| ---------------------------- | ---------- | ------------------- | ------------ | ---------------------- |
| **READ UNCOMMITTED**         | ✅ Yes     | ✅ Yes              | ✅ Yes       | Fastest                |
| **READ COMMITTED** (Default) | ❌ No      | ✅ Yes              | ✅ Yes       | Good                   |
| **REPEATABLE READ**          | ❌ No      | ❌ No               | ✅ Yes       | Slower                 |
| **SERIALIZABLE**             | ❌ No      | ❌ No               | ❌ No        | Slowest                |
| **SNAPSHOT**                 | ❌ No      | ❌ No               | ❌ No        | Good (uses versioning) |

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Deadlocks

> A **deadlock** occurs when two transactions are **waiting for each other** to release locks — neither can proceed. SQL Server automatically kills one (the "victim").

**Prevention:** Access tables in the same order, keep transactions short, use proper indexing.

---

## 📝 SQL Quick Recap

| Topic               | Key Point                                                 |
| ------------------- | --------------------------------------------------------- |
| `OVER()`            | Window function — works on a set of rows without GROUP BY |
| `PARTITION BY`      | Split rows into groups for window functions               |
| `ROW_NUMBER`        | Always unique — no ties                                   |
| `RANK`              | Gaps on ties (1,2,2,4)                                    |
| `DENSE_RANK`        | No gaps on ties (1,2,2,3)                                 |
| CTE                 | Temp named result, one query only                         |
| Recursive CTE       | Self-referencing, for hierarchies                         |
| MERGE               | INSERT + UPDATE + DELETE in one statement                 |
| PIVOT               | Rows → Columns                                            |
| UNPIVOT             | Columns → Rows                                            |
| Clustered Index     | Sorts table data, only 1 per table                        |
| Non-Clustered Index | Separate structure, many allowed                          |
| Trigger             | Auto-fires on DML events                                  |
| Cursor              | Row-by-row processing (avoid if possible!)                |
| ACID                | Atomicity, Consistency, Isolation, Durability             |

---

_[← Back to README](./README.md)_
