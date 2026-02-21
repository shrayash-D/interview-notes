# 📘 Interview Preparation Notes — .NET Full Stack Developer

---

# 📌 MODULE 1: .NET 8 AND C# 12

---

## 1.1 Introduction to .NET 8

### What is .NET?

> **.NET** is a **free, open-source developer platform** by Microsoft for building many types of applications — web, desktop, mobile, cloud, gaming, IoT, etc.

**Interview Answer:** ".NET is a cross-platform, open-source framework by Microsoft that provides tools and libraries to build all kinds of applications like web APIs, desktop apps, and cloud services."

---

### Evolution: .NET Framework → .NET 6 → .NET 8

| Version                   | Key Point                                                  |
| ------------------------- | ---------------------------------------------------------- |
| **.NET Framework** (2002) | Windows-only, runs on CLR (Common Language Runtime)        |
| **.NET Core** (2016)      | Cross-platform rewrite, open-source, faster                |
| **.NET 5** (2020)         | Unified .NET Core + .NET Framework into one platform       |
| **.NET 6** (2021)         | LTS (Long Term Support), Minimal APIs introduced           |
| **.NET 7** (2022)         | Performance improvements, short-term support               |
| **.NET 8** (2023)         | **LTS**, best performance yet, Native AOT, improved Blazor |

**Interview Answer:** "Microsoft evolved from the Windows-only .NET Framework to the cross-platform .NET Core, then unified everything into a single .NET platform starting from .NET 5. .NET 8 is the latest LTS version with major performance improvements."

---

### Key Features and Improvements in .NET 8

| Feature                                    | Simple Explanation                                                                |
| ------------------------------------------ | --------------------------------------------------------------------------------- |
| **Native AOT (Ahead-of-Time Compilation)** | App is compiled to machine code at build time → faster startup, smaller size      |
| **Performance Improvements**               | Up to 20% faster than .NET 7 in many benchmarks                                   |
| **Improved Blazor**                        | Unified full-stack web UI with server + client rendering                          |
| **Minimal APIs Enhancements**              | Easier to build lightweight APIs with less code                                   |
| **Garbage Collector Improvements**         | Better memory management                                                          |
| **JSON Improvements**                      | Faster System.Text.Json serialization                                             |
| **Time Abstraction**                       | New `TimeProvider` class makes testing time-based code easier                     |
| **Frozen Collections**                     | `FrozenDictionary`, `FrozenSet` — read-only collections optimized for fast lookup |

**Example — Frozen Collection:**

```csharp
// Frozen collections are optimized for read-heavy scenarios
FrozenDictionary<string, int> ages = new Dictionary<string, int>
{
    { "Alice", 30 },
    { "Bob", 25 }
}.ToFrozenDictionary();

int age = ages["Alice"]; // Super fast lookup!
```

---

## 1.2 C# 12 Features

### New Features in C# 12

#### 1️⃣ Primary Constructors (for Classes & Structs)

> You can now add constructor parameters directly in the class declaration — less boilerplate code.

```csharp
// ❌ Old way (C# 11 and before)
public class Person
{
    private string _name;
    private int _age;

    public Person(string name, int age)
    {
        _name = name;
        _age = age;
    }
}

// ✅ New way (C# 12)
public class Person(string name, int age)
{
    public string Name => name;
    public int Age => age;
}
```

**Interview Answer:** "Primary constructors let you define constructor parameters directly on the class/struct declaration, reducing boilerplate code. The parameters are available throughout the class body."

---

#### 2️⃣ Collection Expressions

> A simpler, unified syntax to create collections using `[ ]`.

```csharp
// ❌ Old way
int[] numbers = new int[] { 1, 2, 3 };
List<string> names = new List<string> { "Alice", "Bob" };

// ✅ New way (C# 12)
int[] numbers = [1, 2, 3];
List<string> names = ["Alice", "Bob"];

// Spread operator (..) to combine collections
int[] first = [1, 2, 3];
int[] second = [4, 5, 6];
int[] combined = [..first, ..second]; // [1, 2, 3, 4, 5, 6]
```

---

#### 3️⃣ Default Lambda Parameters

> Lambda expressions can now have default parameter values.

```csharp
// ✅ C# 12
var greet = (string name = "World") => $"Hello, {name}!";

Console.WriteLine(greet());        // "Hello, World!"
Console.WriteLine(greet("Alice")); // "Hello, Alice!"
```

---

#### 4️⃣ Alias Any Type

> You can now use `using` alias for ANY type — tuples, arrays, generics, etc.

```csharp
// ✅ C# 12
using Point = (int X, int Y);
using NumberList = System.Collections.Generic.List<int>;

Point p = (10, 20);
NumberList nums = [1, 2, 3];
```

---

#### 5️⃣ Inline Arrays

> Fixed-size arrays allocated on the stack for high-performance scenarios.

```csharp
[System.Runtime.CompilerServices.InlineArray(10)]
public struct Buffer
{
    private int _element;
}
```

---

#### 6️⃣ Interceptors (Experimental)

> Allows rerouting method calls to different implementations at compile time. Used mainly by source generators.

**Interview Answer:** "C# 12 introduces primary constructors for classes, collection expressions with a simpler `[]` syntax, default lambda parameters, and the ability to alias any type. These features reduce boilerplate and make code cleaner."

---

---

# 📌 MODULE 2: C# BEST PRACTICES — SOLID PRINCIPLES

---

## What are SOLID Principles?

> **SOLID** is a set of **5 design principles** that help you write **clean, maintainable, and scalable** object-oriented code.

**Why SOLID?**

- Makes code **easy to understand** and **modify**
- Reduces **bugs** when adding new features
- Makes code **testable**
- Promotes **loose coupling**

---

## S — Single Responsibility Principle (SRP)

> **"A class should have only ONE reason to change."**

Each class should do **one thing only**.

```csharp
// ❌ BAD — Class does too many things
public class Employee
{
    public void CalculateSalary() { /* salary logic */ }
    public void SaveToDatabase() { /* DB logic */ }
    public void SendEmail() { /* email logic */ }
}

// ✅ GOOD — Each class has one job
public class SalaryCalculator
{
    public decimal Calculate(Employee emp) { /* salary logic */ }
}

public class EmployeeRepository
{
    public void Save(Employee emp) { /* DB logic */ }
}

public class EmailService
{
    public void SendEmail(string to, string body) { /* email logic */ }
}
```

**Interview Answer:** "SRP means a class should have only one responsibility or one reason to change. For example, separate your business logic, data access, and email sending into different classes."

---

## O — Open/Closed Principle (OCP)

> **"Classes should be OPEN for extension, but CLOSED for modification."**

You should be able to add new behavior **without changing** existing code.

```csharp
// ❌ BAD — Must modify class for each new shape
public class AreaCalculator
{
    public double Calculate(string shape, double r)
    {
        if (shape == "Circle") return 3.14 * r * r;
        if (shape == "Square") return r * r;
        // Adding new shape = modifying this class ❌
    }
}

// ✅ GOOD — Extend by adding new classes
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; set; }
    public double Area() => 3.14 * Radius * Radius;
}

public class Square : IShape
{
    public double Side { get; set; }
    public double Area() => Side * Side;
}

// Adding Triangle? Just create a new class — no existing code changes!
public class Triangle : IShape
{
    public double Base { get; set; }
    public double Height { get; set; }
    public double Area() => 0.5 * Base * Height;
}
```

**Interview Answer:** "OCP means you should extend behavior by adding new code (new classes), not by modifying existing code. Use interfaces and inheritance to achieve this."

---

## L — Liskov Substitution Principle (LSP)

> **"A child class should be able to replace its parent class without breaking anything."**

If class B inherits from class A, you should be able to use B wherever A is expected.

```csharp
// ❌ BAD — Penguin breaks the contract
public class Bird
{
    public virtual void Fly() { Console.WriteLine("Flying!"); }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new Exception("Penguins can't fly!"); // ❌ Breaks LSP
    }
}

// ✅ GOOD — Separate the concerns
public interface IBird { void Eat(); }
public interface IFlyingBird : IBird { void Fly(); }

public class Sparrow : IFlyingBird
{
    public void Eat() { /* eat */ }
    public void Fly() { /* fly */ }
}

public class Penguin : IBird
{
    public void Eat() { /* eat */ }
    // No Fly() — Penguins don't fly, no contract broken ✅
}
```

**Interview Answer:** "LSP means if you substitute a parent class with a child class, the program should still work correctly. If a subclass can't fulfill the parent's contract, you should redesign the hierarchy."

---

## I — Interface Segregation Principle (ISP)

> **"Don't force a class to implement methods it doesn't use."**

Split big interfaces into smaller, focused ones.

```csharp
// ❌ BAD — Too many methods in one interface
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
}

public class Robot : IWorker
{
    public void Work() { /* works */ }
    public void Eat() { /* Robots don't eat! */ } // ❌ Forced to implement
    public void Sleep() { /* Robots don't sleep! */ } // ❌ Forced to implement
}

// ✅ GOOD — Split interfaces
public interface IWorkable { void Work(); }
public interface IEatable { void Eat(); }
public interface ISleepable { void Sleep(); }

public class Human : IWorkable, IEatable, ISleepable
{
    public void Work() { }
    public void Eat() { }
    public void Sleep() { }
}

public class Robot : IWorkable
{
    public void Work() { } // Only implements what it needs ✅
}
```

**Interview Answer:** "ISP means interfaces should be small and specific. Instead of one big interface, create multiple smaller ones so classes only implement what they actually need."

---

## D — Dependency Inversion Principle (DIP)

> **"High-level modules should NOT depend on low-level modules. Both should depend on ABSTRACTIONS (interfaces)."**

```csharp
// ❌ BAD — Tightly coupled to SqlDatabase
public class OrderService
{
    private SqlDatabase _db = new SqlDatabase(); // ❌ Direct dependency

    public void SaveOrder(Order order)
    {
        _db.Save(order);
    }
}

// ✅ GOOD — Depends on abstraction
public interface IDatabase
{
    void Save(Order order);
}

public class SqlDatabase : IDatabase
{
    public void Save(Order order) { /* SQL save */ }
}

public class MongoDatabase : IDatabase
{
    public void Save(Order order) { /* Mongo save */ }
}

public class OrderService
{
    private readonly IDatabase _db;

    public OrderService(IDatabase db) // ✅ Injected via constructor
    {
        _db = db;
    }

    public void SaveOrder(Order order)
    {
        _db.Save(order);
    }
}
```

**Interview Answer:** "DIP means high-level classes should depend on interfaces, not on concrete implementations. This allows you to easily swap implementations (e.g., switch from SQL Server to MongoDB) without changing business logic. It's the foundation of Dependency Injection."

---

## SOLID Quick Reference Table

| Principle                     | One-liner                                    | Key Idea                              |
| ----------------------------- | -------------------------------------------- | ------------------------------------- |
| **S** - Single Responsibility | One class = one job                          | Don't mix concerns                    |
| **O** - Open/Closed           | Add features without changing existing code  | Use interfaces/inheritance            |
| **L** - Liskov Substitution   | Child replaces parent without breaking       | Subclass must honor parent's contract |
| **I** - Interface Segregation | Small, focused interfaces                    | Don't force unused methods            |
| **D** - Dependency Inversion  | Depend on abstractions, not concrete classes | Use interfaces + DI                   |

---

---

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

---

# 📌 MODULE 4: ENTITY FRAMEWORK CORE 8.0 & DAPPER

---

## 4.1 What is ORM?

> **ORM (Object-Relational Mapping)** maps **database tables to C# classes** and **rows to objects**. You write C# code instead of SQL.

**Interview Answer:** "ORM is a technique that lets you interact with a database using C# objects instead of writing raw SQL. EF Core is Microsoft's ORM for .NET."

---

## 4.2 EF Core vs EF Framework

| Feature            | EF Core              | EF Framework (EF6)    |
| ------------------ | -------------------- | --------------------- |
| Platform           | Cross-platform       | Windows only          |
| Performance        | Faster               | Slower                |
| .NET Support       | .NET 6/7/8           | .NET Framework only   |
| Approach           | Code-First preferred | Database-First common |
| Active Development | ✅ Yes               | ❌ Maintenance mode   |

---

## 4.3 New Features in EF Core 8

| Feature                        | Description                                                   |
| ------------------------------ | ------------------------------------------------------------- |
| **Complex Types**              | Value objects without identity (like Address)                 |
| **Raw SQL for Unmapped Types** | `SqlQuery<T>` for arbitrary queries                           |
| **Primitive Collections**      | Store `List<int>`, `List<string>` as JSON columns             |
| **Sentinel Values**            | Better handling of default values                             |
| **Lazy Loading Improvements**  | More efficient lazy loading                                   |
| **Bulk Updates & Deletes**     | `ExecuteUpdate()`, `ExecuteDelete()` without loading entities |

---

## 4.4 Setting Up EF Core in .NET 8

```bash
# Install NuGet packages
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

```csharp
// DbContext — the "bridge" between your code and database
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }
}

// In Program.cs (.NET 8)
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### EF Core CLI Commands

```bash
dotnet ef migrations add InitialCreate    # Create migration
dotnet ef database update                  # Apply migration to DB
dotnet ef migrations remove               # Remove last migration
dotnet ef database drop                    # Drop the database
```

---

## 4.5 Defining Entities and Relationships

```csharp
// Entity = C# class mapped to a database table
public class Employee
{
    public int EmployeeId { get; set; }        // Primary Key (by convention)
    public string Name { get; set; }
    public decimal Salary { get; set; }

    public int DepartmentId { get; set; }       // Foreign Key
    public Department Department { get; set; }  // Navigation Property
}

public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }

    public ICollection<Employee> Employees { get; set; }  // Navigation (One-to-Many)
}
```

---

## 4.6 CRUD Operations with EF Core

```csharp
// CREATE
var emp = new Employee { Name = "Alice", Salary = 75000 };
await _context.Employees.AddAsync(emp);
await _context.SaveChangesAsync();

// READ
var employee = await _context.Employees.FindAsync(1);
var firstEmp = await _context.Employees.FirstOrDefaultAsync(e => e.Name == "Alice");
var allEmps = await _context.Employees.ToListAsync();

// UPDATE
var emp = await _context.Employees.FindAsync(1);
emp.Salary = 80000;         // EF tracks this change automatically
await _context.SaveChangesAsync();

// DELETE
var emp = await _context.Employees.FindAsync(1);
_context.Employees.Remove(emp);
await _context.SaveChangesAsync();

// Bulk Delete (EF Core 8 — no loading needed!)
await _context.Employees.Where(e => e.Salary < 30000).ExecuteDeleteAsync();

// Bulk Update (EF Core 8)
await _context.Employees
    .Where(e => e.Department == "IT")
    .ExecuteUpdateAsync(e => e.SetProperty(x => x.Salary, x => x.Salary * 1.1m));
```

---

## 4.7 LINQ Queries in EF Core

```csharp
// WHERE + OrderBy
var itEmployees = await _context.Employees
    .Where(e => e.Department.Name == "IT")
    .OrderBy(e => e.Name)
    .ToListAsync();

// Projection to DTO
var dtos = await _context.Employees
    .Select(e => new EmployeeDto
    {
        FullName = e.Name,
        DeptName = e.Department.Name
    })
    .ToListAsync();

// Aggregation
var avgSalary = await _context.Employees.AverageAsync(e => e.Salary);
var totalCount = await _context.Employees.CountAsync();
```

---

## 4.8 Data Loading Strategies

| Strategy             | How It Works                                     | When to Use                       |
| -------------------- | ------------------------------------------------ | --------------------------------- |
| **Eager Loading**    | Loads related data immediately with `.Include()` | You know you'll need related data |
| **Lazy Loading**     | Loads related data on first access (auto)        | Careful — can cause N+1 problem   |
| **Explicit Loading** | Manually load related data with `.Load()`        | You decide when to load           |

```csharp
// Eager Loading
var emps = await _context.Employees
    .Include(e => e.Department)
    .ToListAsync();

// Explicit Loading
var emp = await _context.Employees.FindAsync(1);
await _context.Entry(emp).Reference(e => e.Department).LoadAsync();

// Lazy Loading — needs virtual navigation properties + proxy package
```

---

## 4.9 Relationships Configuration

```csharp
// One-to-One
modelBuilder.Entity<Employee>()
    .HasOne(e => e.Address)
    .WithOne(a => a.Employee)
    .HasForeignKey<Address>(a => a.EmployeeId);

// One-to-Many
modelBuilder.Entity<Department>()
    .HasMany(d => d.Employees)
    .WithOne(e => e.Department)
    .HasForeignKey(e => e.DepartmentId);

// Many-to-Many (EF Core 5+ auto-creates join table)
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students);
```

---

## 4.10 Performance Best Practices

```csharp
// 1. AsNoTracking — for read-only queries (faster)
var emps = await _context.Employees.AsNoTracking().ToListAsync();

// 2. Compiled Queries — pre-compile for repeated execution
private static readonly Func<AppDbContext, int, Task<Employee>> GetById =
    EF.CompileAsyncQuery((AppDbContext ctx, int id) =>
        ctx.Employees.FirstOrDefault(e => e.EmployeeId == id));

// 3. RowVersion for Concurrency
public class Employee
{
    [Timestamp]
    public byte[] RowVersion { get; set; }  // Optimistic concurrency
}
```

---

## 4.11 Dapper — Micro ORM

### What is Dapper?

> **Dapper** is a lightweight, high-performance **micro-ORM** for .NET. It extends `IDbConnection` with methods to map SQL results to C# objects.

**Interview Answer:** "Dapper is a micro-ORM that sits on top of ADO.NET. It's much faster than EF Core because it does minimal overhead — just maps SQL results to objects. Created by the Stack Overflow team."

### Dapper vs Entity Framework

| Feature         | Dapper                           | EF Core                       |
| --------------- | -------------------------------- | ----------------------------- |
| Type            | Micro-ORM                        | Full ORM                      |
| Speed           | ⚡ Fastest                       | Slower                        |
| SQL Control     | Full control (raw SQL)           | Generates SQL (LINQ)          |
| Change Tracking | ❌ No                            | ✅ Yes                        |
| Migrations      | ❌ No                            | ✅ Yes                        |
| Learning Curve  | Easy                             | Moderate                      |
| Best For        | Performance-critical, read-heavy | CRUD-heavy, rapid development |

---

### Querying Data with Dapper

```csharp
using Dapper;
using Microsoft.Data.SqlClient;

using var connection = new SqlConnection(connectionString);

// Query Multiple Rows
var employees = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Department = @Dept",
    new { Dept = "IT" });

// Query Single Row
var emp = await connection.QueryFirstOrDefaultAsync<Employee>(
    "SELECT * FROM Employees WHERE Id = @Id", new { Id = 1 });

// Query Scalar Value
var count = await connection.ExecuteScalarAsync<int>(
    "SELECT COUNT(*) FROM Employees");

// Query Multiple Result Sets
using var multi = await connection.QueryMultipleAsync(
    "SELECT * FROM Employees; SELECT * FROM Departments;");
var employees = await multi.ReadAsync<Employee>();
var departments = await multi.ReadAsync<Department>();
```

---

### Insert, Update, Delete with Dapper

```csharp
// INSERT
await connection.ExecuteAsync(
    "INSERT INTO Employees (Name, Salary) VALUES (@Name, @Salary)",
    new { Name = "Alice", Salary = 75000 });

// UPDATE
await connection.ExecuteAsync(
    "UPDATE Employees SET Salary = @Salary WHERE Id = @Id",
    new { Id = 1, Salary = 80000 });

// DELETE
await connection.ExecuteAsync(
    "DELETE FROM Employees WHERE Id = @Id", new { Id = 1 });

// Execute Stored Procedure
var result = await connection.QueryAsync<Employee>(
    "usp_GetEmployeesByDept",
    new { DeptName = "IT" },
    commandType: CommandType.StoredProcedure);
```

---

### Parameters & SQL Injection Prevention

```csharp
// ✅ SAFE — Parameterized (Dapper auto-parameterizes)
var emp = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Name = @Name", new { Name = userInput });

// ❌ UNSAFE — SQL Injection vulnerability
var emp = await connection.QueryAsync<Employee>(
    $"SELECT * FROM Employees WHERE Name = '{userInput}'");  // NEVER DO THIS!

// Dynamic Parameters
var parameters = new DynamicParameters();
parameters.Add("@Name", "Alice");
parameters.Add("@MinSalary", 50000);
var result = await connection.QueryAsync<Employee>(sql, parameters);

// WHERE IN
var ids = new[] { 1, 2, 3 };
var emps = await connection.QueryAsync<Employee>(
    "SELECT * FROM Employees WHERE Id IN @Ids", new { Ids = ids });
```

---

### Relationships with Dapper (SplitOn)

```csharp
// One-to-One / Many-to-One with SplitOn
var sql = @"SELECT e.*, d.* FROM Employees e
            INNER JOIN Departments d ON e.DepartmentId = d.DepartmentId";

var employees = await connection.QueryAsync<Employee, Department, Employee>(
    sql,
    (employee, department) =>
    {
        employee.Department = department;
        return employee;
    },
    splitOn: "DepartmentId"  // Where to split columns between objects
);

// One-to-Many
var sql = @"SELECT d.*, e.* FROM Departments d
            LEFT JOIN Employees e ON d.DepartmentId = e.DepartmentId";

var deptDict = new Dictionary<int, Department>();
var departments = await connection.QueryAsync<Department, Employee, Department>(
    sql,
    (dept, emp) =>
    {
        if (!deptDict.TryGetValue(dept.DepartmentId, out var existingDept))
        {
            existingDept = dept;
            existingDept.Employees = new List<Employee>();
            deptDict.Add(existingDept.DepartmentId, existingDept);
        }
        if (emp != null) existingDept.Employees.Add(emp);
        return existingDept;
    },
    splitOn: "EmployeeId"
);
```

---

### Dapper Plus — Bulk Operations

```csharp
// Bulk Insert
connection.BulkInsert(employeesList);

// Bulk Update
connection.BulkUpdate(employeesList);

// Bulk Delete
connection.BulkDelete(employeesList);

// Bulk Merge (Insert or Update)
connection.BulkMerge(employeesList);
```

---

---

# 📌 MODULE 5: ASP.NET CORE 8.0

---

## 5.1 Introduction to ASP.NET Core 8

### What is ASP.NET Core?

> **ASP.NET Core** is a cross-platform, high-performance framework for building modern web applications, APIs, and real-time apps.

### ASP.NET Core vs .NET Framework

| Feature              | ASP.NET Core         | ASP.NET (.NET Framework) |
| -------------------- | -------------------- | ------------------------ |
| Platform             | Cross-platform       | Windows only             |
| Performance          | Very fast            | Slower                   |
| Hosting              | Kestrel, IIS, Docker | IIS only                 |
| Open Source          | ✅ Yes               | ❌ No                    |
| Dependency Injection | Built-in             | Third-party needed       |

---

## 5.2 Project Templates

| Template        | Use Case                             |
| --------------- | ------------------------------------ |
| **MVC**         | Full web app with Views (HTML pages) |
| **Web API**     | RESTful APIs returning JSON          |
| **Minimal API** | Lightweight APIs with minimal code   |
| **Razor Pages** | Page-focused web apps                |
| **Blazor**      | SPA with C# instead of JavaScript    |

### Project Structure

```
MyApp/
├── Controllers/        # Handle HTTP requests
├── Models/             # Data classes
├── Views/              # Razor HTML templates
├── wwwroot/            # Static files (CSS, JS, images)
├── appsettings.json    # Configuration
├── Program.cs          # App entry point
```

---

## 5.3 MVC Architecture

> **MVC = Model + View + Controller**

| Component      | Responsibility                      | Example                 |
| -------------- | ----------------------------------- | ----------------------- |
| **Model**      | Data + business logic               | `Employee.cs`           |
| **View**       | UI (HTML/Razor)                     | `Index.cshtml`          |
| **Controller** | Handles requests, coordinates M & V | `EmployeeController.cs` |

### Request Lifecycle

```
User → URL → Routing → Controller → Action Method → Model (get data) → View (render HTML) → Response to User
```

---

## 5.4 Models

```csharp
// Model with Data Annotations for Validation
public class Employee
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; }

    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Range(18, 65)]
    public int Age { get; set; }

    [DataType(DataType.Currency)]
    public decimal Salary { get; set; }
}

// ViewModel — shapes data for a specific view
public class EmployeeListViewModel
{
    public List<Employee> Employees { get; set; }
    public string SearchTerm { get; set; }
    public int TotalCount { get; set; }
}
```

---

## 5.5 Controllers

```csharp
public class EmployeeController : Controller
{
    private readonly AppDbContext _context;

    // Constructor Injection
    public EmployeeController(AppDbContext context)
    {
        _context = context;
    }

    // GET: /Employee
    public async Task<IActionResult> Index()
    {
        var employees = await _context.Employees.ToListAsync();
        return View(employees);  // Passes data to View
    }

    // GET: /Employee/Details/5
    public async Task<IActionResult> Details(int id)
    {
        var emp = await _context.Employees.FindAsync(id);
        if (emp == null) return NotFound();
        return View(emp);
    }

    // POST: /Employee/Create
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Create(Employee employee)
    {
        if (ModelState.IsValid)
        {
            await _context.Employees.AddAsync(employee);
            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }
        return View(employee);  // Return with validation errors
    }
}
```

---

## 5.6 Razor Views

```html
@model List<Employee>
  <h2>Employee List</h2>

  <table class="table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Email</th>
        <th>Salary</th>
      </tr>
    </thead>
    <tbody>
      @foreach (var emp in Model) {
      <tr>
        <td>@emp.Name</td>
        <td>@emp.Email</td>
        <td>@emp.Salary.ToString("C")</td>
      </tr>
      }
    </tbody>
  </table>

  <!-- Layout View: _Layout.cshtml -->
  <!-- Partial View: _EmployeeCard.cshtml -->
  @await Html.PartialAsync("_EmployeeCard", employee)</Employee
>
```

---

## 5.7 Routing

### Convention-based Routing

```csharp
// In Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
// URL: /Employee/Details/5 → EmployeeController.Details(5)
```

### Attribute Routing

```csharp
[Route("api/[controller]")]
public class EmployeeController : Controller
{
    [HttpGet("{id}")]        // GET api/employee/5
    public IActionResult Get(int id) { ... }

    [HttpPost]               // POST api/employee
    public IActionResult Create(Employee emp) { ... }

    [Route("search/{name}")] // GET api/employee/search/Alice
    public IActionResult Search(string name) { ... }
}
```

### Route Constraints

```csharp
[HttpGet("{id:int}")]           // Only matches integers
[HttpGet("{name:alpha}")]       // Only matches alphabetic
[HttpGet("{id:min(1)}")]        // Minimum value of 1
```

### Tag Helpers (Generate URLs)

```html
<a asp-controller="Employee" asp-action="Details" asp-route-id="5"
  >View Details</a
>
<!-- Generates: <a href="/Employee/Details/5">View Details</a> -->
```

---

## 5.8 Forms and User Input

```html
@model Employee

<form asp-action="Create" method="post">
  <div class="form-group">
    <label asp-for="Name"></label>
    <input asp-for="Name" class="form-control" />
    <span asp-validation-for="Name" class="text-danger"></span>
  </div>

  <div class="form-group">
    <label asp-for="Email"></label>
    <input asp-for="Email" class="form-control" />
    <span asp-validation-for="Email" class="text-danger"></span>
  </div>

  <button type="submit" class="btn btn-primary">Save</button>
</form>
```

### Model Binding

> ASP.NET Core **automatically maps** form data, query strings, and route data to action method parameters or model properties.

```csharp
// Model Binding happens automatically
[HttpPost]
public IActionResult Create(Employee employee)  // Auto-filled from form data
{
    if (ModelState.IsValid) { /* save */ }
    return View(employee);
}
```

---

## 5.9 State Management

| Method       | Scope                    | Use Case                          |
| ------------ | ------------------------ | --------------------------------- |
| **ViewData** | Current request only     | Pass data from Controller to View |
| **ViewBag**  | Current request only     | Dynamic version of ViewData       |
| **TempData** | Current + next request   | Pass data between redirects       |
| **Session**  | Entire user session      | Shopping cart, user preferences   |
| **Cookies**  | Persistent (client-side) | "Remember me", preferences        |

```csharp
// ViewData
ViewData["Message"] = "Hello!";
// In View: @ViewData["Message"]

// ViewBag
ViewBag.Title = "Employee List";
// In View: @ViewBag.Title

// TempData (survives one redirect)
TempData["Success"] = "Employee created!";
return RedirectToAction("Index");
// In next View: @TempData["Success"]

// Session
HttpContext.Session.SetString("Username", "Alice");
var username = HttpContext.Session.GetString("Username");

// Configure Session in Program.cs
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(30);
});
app.UseSession();
```

---

## 5.10 Integrating EF Core with ASP.NET Core

```csharp
// Program.cs — Configure DbContext
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

builder.Services.AddControllersWithViews();
var app = builder.Build();
```

```json
// appsettings.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=MyAppDb;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

---

## 5.11 Dependency Injection (DI)

### What is DI?

> **Dependency Injection** is a design pattern where a class receives its dependencies from outside rather than creating them itself. ASP.NET Core has a **built-in DI container**.

### Service Lifetimes

| Lifetime      | Behavior                             | Use Case                        |
| ------------- | ------------------------------------ | ------------------------------- |
| **Transient** | New instance every time              | Lightweight, stateless services |
| **Scoped**    | One instance per HTTP request        | DbContext, per-request services |
| **Singleton** | One instance for entire app lifetime | Caching, logging, configuration |

```csharp
// Register Services in Program.cs
builder.Services.AddTransient<IEmailService, EmailService>();    // New each time
builder.Services.AddScoped<IEmployeeRepository, EmployeeRepository>();  // Per request
builder.Services.AddSingleton<ICacheService, CacheService>();   // One for all

// Inject into Controller
public class EmployeeController : Controller
{
    private readonly IEmployeeRepository _repo;
    private readonly IEmailService _email;

    public EmployeeController(IEmployeeRepository repo, IEmailService email)
    {
        _repo = repo;     // Injected by DI container
        _email = email;   // Injected by DI container
    }
}
```

**Interview Answer:** "DI is a pattern where dependencies are injected from outside instead of being created inside a class. ASP.NET Core has a built-in IoC container. There are three lifetimes: Transient (new every time), Scoped (once per request), and Singleton (once for the app). This follows the Dependency Inversion Principle from SOLID."

---

## 5.12 Publishing and Deployment

### appsettings for Different Environments

```
appsettings.json                  # Shared settings
appsettings.Development.json      # Dev-specific
appsettings.Production.json       # Prod-specific
```

```csharp
// Program.cs automatically reads based on ASPNETCORE_ENVIRONMENT
var environment = builder.Environment.EnvironmentName; // "Development" or "Production"
```

### Publishing Options

| Method                  | Command / Steps                                                      |
| ----------------------- | -------------------------------------------------------------------- |
| **Self-Contained**      | `dotnet publish -c Release --self-contained` (includes .NET runtime) |
| **Framework-Dependent** | `dotnet publish -c Release` (requires .NET on server)                |
| **IIS**                 | Publish → Copy to IIS folder → Configure Application Pool            |
| **Azure App Service**   | Publish from VS or `az webapp deploy`                                |
| **Docker**              | Create `Dockerfile` → `docker build` → `docker run`                  |

### Dockerfile Example

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

---

# 📌 QUICK INTERVIEW CHEAT SHEET

## Top 20 Questions to Expect

| #   | Question                             | Key Points to Mention                                                                            |
| --- | ------------------------------------ | ------------------------------------------------------------------------------------------------ |
| 1   | What's new in .NET 8?                | LTS, Native AOT, performance, Frozen Collections, Blazor improvements                            |
| 2   | What's new in C# 12?                 | Primary constructors, collection expressions, default lambda params                              |
| 3   | Explain SOLID principles             | S-Single job, O-Open for extension, L-Substitution, I-Small interfaces, D-Depend on abstractions |
| 4   | Clustered vs Non-Clustered Index?    | Clustered = sorts data, 1 per table. Non-clustered = separate structure, many allowed            |
| 5   | What is a CTE?                       | Temporary named result set, exists only during one query, improves readability                   |
| 6   | SP vs Function?                      | SP can do DML, function must return value, function can be used in SELECT                        |
| 7   | What are ACID properties?            | Atomicity, Consistency, Isolation, Durability                                                    |
| 8   | What is EF Core?                     | Microsoft's ORM, maps C# objects to DB tables, supports LINQ, migrations                         |
| 9   | EF Core vs Dapper?                   | EF=full ORM with tracking; Dapper=micro ORM, faster, raw SQL                                     |
| 10  | What are EF Core loading strategies? | Eager (Include), Lazy (auto), Explicit (manual Load)                                             |
| 11  | What is MVC?                         | Model=data, View=UI, Controller=handles requests                                                 |
| 12  | What is Dependency Injection?        | Classes receive dependencies from outside. Transient/Scoped/Singleton                            |
| 13  | What is Middleware?                  | Pipeline components that handle requests/responses                                               |
| 14  | Explain Routing in ASP.NET Core      | Convention-based vs Attribute routing, route constraints                                         |
| 15  | ViewData vs ViewBag vs TempData?     | ViewData=dict, ViewBag=dynamic, TempData=survives redirect                                       |
| 16  | What is AsNoTracking?                | Disables change tracking for read-only queries — improves performance                            |
| 17  | What is SQL Injection?               | Attacker injects malicious SQL. Prevent with parameterized queries                               |
| 18  | What are Triggers?                   | Auto-execute SQL on INSERT/UPDATE/DELETE events                                                  |
| 19  | What is a Deadlock?                  | Two transactions waiting for each other. SQL Server kills one.                                   |
| 20  | How do you deploy ASP.NET Core?      | IIS, Azure App Service, Docker, self-contained vs framework-dependent                            |

---

## Common Abbreviations

| Abbreviation | Full Form                         |
| ------------ | --------------------------------- |
| ORM          | Object-Relational Mapping         |
| DI           | Dependency Injection              |
| IoC          | Inversion of Control              |
| MVC          | Model-View-Controller             |
| API          | Application Programming Interface |
| CRUD         | Create, Read, Update, Delete      |
| DTO          | Data Transfer Object              |
| CTE          | Common Table Expression           |
| SP           | Stored Procedure                  |
| UDF          | User-Defined Function             |
| AOT          | Ahead-of-Time (Compilation)       |
| LTS          | Long Term Support                 |
| LINQ         | Language Integrated Query         |
| CLR          | Common Language Runtime           |
| DML          | Data Manipulation Language        |
| DDL          | Data Definition Language          |

---

> 📝 **Tip:** Always give **real-world examples** in interviews. E.g., "In my project, we used Dapper for reporting queries because performance was critical, and EF Core for CRUD operations."

> 🎯 **Tip:** When asked "What is X?", structure your answer as: **Definition → Why it's useful → Quick example.**

---

_Last Updated: February 21, 2026_
