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

## 3.2 Advanced SQL Concepts

### OVER() Clause

> Performs calculations **across a set of rows** related to the current row, without collapsing them into groups (unlike GROUP BY).

```sql
-- Get each employee's salary AND the average salary of their department
SELECT Name, Department, Salary,
       AVG(Salary) OVER(PARTITION BY Department) AS DeptAvgSalary
FROM Employees;
```

### PARTITION BY

> Divides the result set into **partitions** (groups) for window functions. Like GROUP BY, but keeps all rows.

```sql
SELECT Name, Department, Salary,
       SUM(Salary) OVER(PARTITION BY Department) AS DeptTotal
FROM Employees;
-- Each row shows individual salary + total department salary
```

### ROW_NUMBER, RANK, DENSE_RANK

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

> A **temporary named result set** that exists only during one query. Makes complex queries readable.

```sql
-- Basic CTE
WITH HighPaidEmployees AS (
    SELECT Name, Salary, Department
    FROM Employees
    WHERE Salary > 70000
)
SELECT * FROM HighPaidEmployees WHERE Department = 'IT';
```

### Recursive CTE

> A CTE that **calls itself** — useful for hierarchies (org charts, parent-child data).

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

> An **Index** is like a **book's index** — it helps SQL Server find data quickly without scanning every row.

**Interview Answer:** "An index is a data structure that improves the speed of data retrieval on a table. Without an index, SQL Server does a full table scan. With an index, it can jump directly to the relevant rows."

### Types of Indexes

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

> A **Stored Procedure (SP)** is a **pre-compiled collection of SQL statements** saved in the database that can be executed repeatedly.

**Benefits:** Faster execution (pre-compiled), reusability, security, reduced network traffic.

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

| Type                             | Returns                     | Usage                    |
| -------------------------------- | --------------------------- | ------------------------ |
| **Scalar Function**              | Single value                | Can use in SELECT, WHERE |
| **Inline Table-Valued**          | Table (single SELECT)       | Use like a table         |
| **Multi-Statement Table-Valued** | Table (multiple statements) | More complex logic       |

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

> A **Trigger** is a special stored procedure that **automatically runs** when a specific event (INSERT, UPDATE, DELETE) happens on a table.

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

> A **Transaction** is a group of SQL operations that are treated as a **single unit** — either **ALL succeed** or **ALL fail**.

### ACID Properties

| Property            | Meaning                                      | Example                                                 |
| ------------------- | -------------------------------------------- | ------------------------------------------------------- |
| **A - Atomicity**   | All or nothing                               | Transfer money: debit AND credit both happen or neither |
| **C - Consistency** | DB goes from one valid state to another      | Balance can't go negative if rule says so               |
| **I - Isolation**   | Transactions don't interfere with each other | Two people booking last seat — only one succeeds        |
| **D - Durability**  | Once committed, data survives crashes        | Even if server crashes after COMMIT, data is safe       |

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
