# 📌 MODULE 4: ENTITY FRAMEWORK CORE 8.0 & DAPPER

---

## 4.1 What is ORM?

**What is it?**
ORM (Object-Relational Mapping) is a technique that lets you work with a relational database using **C# objects** instead of writing raw SQL. The ORM automatically translates your C# code into SQL queries.

**The problem it solves:**
Without ORM, you write raw SQL strings, manually open/close connections, read `SqlDataReader` row by row, and manually map each column to a C# property. This is tedious, error-prone, and hard to maintain. ORM automates all of that.

**The trade-off:**
ORM generates SQL for you, which is convenient, but sometimes the generated SQL is not as optimal as hand-written SQL. For read-heavy, performance-critical queries, a micro-ORM like Dapper (which lets you write the SQL yourself) is often better.

> **ORM (Object-Relational Mapping)** maps **database tables to C# classes** and **rows to objects**. You write C# code instead of SQL.

**Interview Answer:** "ORM is a technique that lets you interact with a database using C# objects instead of writing raw SQL. EF Core is Microsoft's full ORM — it generates SQL from LINQ queries. The advantage is productivity; the trade-off is that generated SQL may not always be as efficient as hand-crafted queries."

---

## 4.2 EF Core vs EF Framework

| Feature            | EF Core              | EF Framework (EF6)    |
| ------------------ | -------------------- | --------------------- |
| Platform           | Cross-platform       | Windows only          |
| Performance        | Faster               | Slower                |
| .NET Support       | .NET 6/7/8           | .NET Framework only   |
| Approach           | Code-First preferred | Database-First common |
| Active Development | ✅ Yes               | ❌ Maintenance mode   |

**Interview Answer:** _"EF Core is the modern, cross-platform rewrite of Entity Framework. It supports .NET 6/7/8, is actively developed, and is significantly faster. EF6 only runs on Windows/.NET Framework and is in maintenance mode — no new features."_

---

## 4.2a Code-First vs Database-First

These are the two main **workflows** (approaches) for setting up EF Core in a project.

| Aspect                   | Code-First                                                   | Database-First                                           |
| ------------------------ | ------------------------------------------------------------ | -------------------------------------------------------- |
| **Definition**           | You write C# classes first; EF generates the database schema | Database already exists; EF generates C# classes from it |
| **Who controls schema?** | Developer (via C# code + migrations)                         | DBA / existing DB                                        |
| **How DB is created**    | `dotnet ef migrations add` → `dotnet ef database update`     | `dotnet ef dbcontext scaffold`                           |
| **Best for**             | New projects, greenfield development                         | Legacy DBs, large existing schemas                       |
| **Migration support**    | ✅ Full migrations history                                   | ❌ No migrations (DB is the source of truth)             |
| **Common in**            | Modern .NET projects                                         | Enterprise projects with existing DBs                    |

### Code-First Workflow

```csharp
// Step 1: Write your entity class
public class Product
{
    public int ProductId { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// Step 2: Add to DbContext
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
}

// Step 3: Create and apply migration
// dotnet ef migrations add CreateProductTable
// dotnet ef database update
// → EF creates the Products table in the DB automatically
```

### Database-First Workflow

```bash
# Scaffold (reverse engineer) C# classes from an existing database
dotnet ef dbcontext scaffold "Server=.;Database=ShopDB;Trusted_Connection=True;" \
    Microsoft.EntityFrameworkCore.SqlServer \
    --output-dir Models \
    --context ShopDbContext
# → Generates entity classes and DbContext from existing tables
```

**Interview Answer:** _"Code-First means I write C# entity classes and EF generates the database using migrations — I have full control over the schema through code. Database-First is the reverse: an existing database is scaffolded into C# classes. I prefer Code-First for new projects because migrations give a versioned history of schema changes."_

---

## 4.2b DbContext and DbSet — In Detail

### DbContext

`DbContext` is the **central class** in EF Core. It is the bridge between your C# application and the database.

**What it does:**

- Manages the database connection
- Tracks changes to entities (Change Tracker)
- Executes queries and saves changes
- Holds `DbSet<T>` properties for each entity

```csharp
public class AppDbContext : DbContext
{
    // Constructor: receives options (connection string, provider) via DI
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    // DbSet = represents a table in the database
    public DbSet<Employee> Employees { get; set; }
    public DbSet<Department> Departments { get; set; }

    // Optional: configure entity mappings (Fluent API)
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // ... configuration here
    }
}
```

### DbSet\<T\>

`DbSet<T>` represents a **table** in the database. It is used to query and save instances of `T`.

| Operation          | Method                                        |
| ------------------ | --------------------------------------------- |
| Add single record  | `_context.Employees.Add(emp)`                 |
| Add async          | `await _context.Employees.AddAsync(emp)`      |
| Find by PK         | `await _context.Employees.FindAsync(id)`      |
| Query (LINQ)       | `_context.Employees.Where(...).ToListAsync()` |
| Remove             | `_context.Employees.Remove(emp)`              |
| Commit all changes | `await _context.SaveChangesAsync()`           |

> **Key point:** Changes are only saved to the database when you call `SaveChangesAsync()`. Before that, EF is just tracking them in memory (Change Tracker).

**Interview Answer:** _"DbContext is the main class that coordinates EF Core — it holds the connection, tracks entity changes, and exposes DbSet properties. Each DbSet maps to a table. You query through DbSet using LINQ, and call SaveChangesAsync() to commit all tracked changes as a single unit of work."_

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

**How EF Core CRUD works:**
EF Core uses a **Unit of Work** pattern via the Change Tracker. When you fetch entities, EF Core starts tracking them. When you modify them, EF Core detects the changes. When you call `SaveChangesAsync()`, EF Core generates the appropriate SQL (INSERT/UPDATE/DELETE) and executes everything in a single database round trip.

**Key point:** Nothing hits the database until you call `SaveChangesAsync()`. This means you can make multiple changes, add multiple records, and they all go to the database in one batch.

**`FindAsync` vs `FirstOrDefaultAsync`:**
- `FindAsync(id)` — checks the Change Tracker's in-memory cache first, then queries DB. Best when fetching by primary key.
- `FirstOrDefaultAsync(e => e.Id == id)` — always queries the database. Use when filtering by non-PK columns.

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

**How it works:**
When you write LINQ against a `DbSet<T>`, EF Core does NOT execute the query immediately. It builds an **expression tree** — a tree-like representation of the query in memory. When you call `.ToListAsync()` or another terminal operator, EF Core's query provider **translates the expression tree into SQL** and sends it to the database.

This is why EF Core can take a C# lambda like `.Where(e => e.Department.Name == "IT")` and turn it into `WHERE Department = 'IT'` in SQL — the lambda is not a regular C# function; it's an expression tree that the compiler captures for EF Core to inspect and translate.

**IQueryable vs IEnumerable with EF Core:**
- `DbSet<T>` implements `IQueryable<T>` — chaining `.Where()`, `.Select()`, `.OrderBy()` on it keeps building the SQL query
- The moment you call `.ToList()` or `foreach`, the SQL is sent to the database
- If you accidentally cast to `IEnumerable<T>` first, all filtering happens in C# memory on the full table — very inefficient for large tables

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

## 4.8 EF Core Migrations

**What are migrations?**
Migrations are the mechanism EF Core uses to **keep your database schema in sync with your C# entity classes**. Every time you change an entity (add a property, rename a column, add a new entity), you create a migration. The migration is a C# class with two methods:
- `Up()` — applies the schema change (run with `database update`)
- `Down()` — reverses the change (used for rollback)

**Why migrations over manual SQL scripts?**
- **Version controlled** — migration files live in your git repo alongside code
- **Team-friendly** — each developer runs `database update` to apply everyone's migrations
- **Repeatable** — you can recreate the database from scratch at any point by running all migrations
- **Rollback support** — `database update PreviousMigrationName` rolls back to any point

**The workflow:**
1. Change your entity class
2. `dotnet ef migrations add DescriptiveName` — generates the migration C# file
3. Review the generated `Up()` and `Down()` methods
4. `dotnet ef database update` — applies it to the database

**Interview Answer:** "Migrations are version-controlled schema changes for your database. Each migration has `Up()` to apply a change and `Down()` to reverse it. They're committed to git alongside code, so the team always knows what schema changes were made and can apply them in order. This replaces manually written SQL DDL scripts."

```bash
# Add a migration (generates code to update DB schema)
dotnet ef migrations add AddSalaryColumn

# Apply to database
dotnet ef database update

# Seed data in migration
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.InsertData(
        table: "Departments",
        columns: new[] { "Name" },
        values: new object[] { "IT" });
}
```

---

## 4.9 Data Loading Strategies

**The problem:**
When you load an entity (e.g., an `Employee`), EF Core doesn't automatically load related entities (e.g., the `Department` they belong to). You have three strategies to control when and how related data is loaded.

**Why does this matter?**
Choosing the wrong strategy is a very common source of performance problems — especially the **N+1 problem** with lazy loading.

**The N+1 problem explained:**
If you load 100 employees with lazy loading and then access `employee.Department.Name` in a loop — EF Core fires 1 query to get employees, then 1 separate query per employee to load their department = **101 queries**. With eager loading (`.Include()`), it's just 1 query using a SQL JOIN.

| Strategy | How It Works | SQL Generated | Best For |
|---|---|---|---|
| **Eager Loading** | `.Include()` — loads related data **in the same query** | JOIN | When you know you'll need related data — avoids N+1 |
| **Lazy Loading** | Related data loaded **automatically on first access** (via proxy) | Separate query per access | Simple cases — dangerous in loops (N+1 problem) |
| **Explicit Loading** | You **manually trigger** loading with `.Load()` when ready | Separate query, your choice | When you sometimes need related data, sometimes don't |

**Interview Answer:** "Eager loading uses `.Include()` to load related data in a single SQL JOIN query — best when you know you'll need the related data. Lazy loading loads related data automatically the first time you access the navigation property, but this causes the N+1 problem in loops. Explicit loading is a manual call to load related data exactly when you need it."

```csharp
// Eager Loading
var emps = await _context.Employees
    .Include(e => e.Department)
    .ToListAsync();

// ThenInclude — for nested relationships
var emps = await _context.Employees
    .Include(e => e.Department)
        .ThenInclude(d => d.Manager)
    .ToListAsync();

// Explicit Loading
var emp = await _context.Employees.FindAsync(1);
await _context.Entry(emp).Reference(e => e.Department).LoadAsync();

// Lazy Loading — needs virtual navigation properties + proxy package
```

---

## 4.10 Relationships Configuration

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

## 4.11 Performance Best Practices

```csharp
// 1. AsNoTracking — for read-only queries (faster, no change tracking overhead)
var emps = await _context.Employees.AsNoTracking().ToListAsync();

// 2. Compiled Queries — pre-compile for repeated execution
private static readonly Func<AppDbContext, int, Task<Employee>> GetById =
    EF.CompileAsyncQuery((AppDbContext ctx, int id) =>
        ctx.Employees.FirstOrDefault(e => e.EmployeeId == id));

// Usage
var emp = await GetById(_context, 1);

// 3. RowVersion for Concurrency (Optimistic Concurrency)
public class Employee
{
    [Timestamp]
    public byte[] RowVersion { get; set; }
    // EF will throw DbUpdateConcurrencyException if another user
    // updated this record since you last read it
}

// 4. Select only what you need (avoid SELECT *)
var names = await _context.Employees
    .Select(e => e.Name)
    .ToListAsync();
```

---

## 4.12 Dapper — Micro ORM

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

### Setting Up Dapper

```bash
dotnet add package Dapper
dotnet add package Microsoft.Data.SqlClient
```

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

// QuerySingle — throws if 0 or more than 1 row
var emp = await connection.QuerySingleAsync<Employee>(
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

// Output Parameter
var p = new DynamicParameters();
p.Add("@DeptName", "IT");
p.Add("@Count", dbType: DbType.Int32, direction: ParameterDirection.Output);
await connection.ExecuteAsync("usp_GetEmpCount", p, commandType: CommandType.StoredProcedure);
int count = p.Get<int>("@Count");
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

## 4.13 IEnumerable vs IQueryable vs ICollection

> A very common interview question — these three interfaces look similar but behave very differently, especially with EF Core.

| Interface       | Where filtering happens   | Loads data when?               | Use case                                |
| --------------- | ------------------------- | ------------------------------ | --------------------------------------- |
| **IQueryable**  | In the **database** (SQL) | On `.ToList()` / terminal call | EF Core queries — always prefer this    |
| **IEnumerable** | In **C# memory**          | Immediately when assigned      | In-memory collections, LINQ to Objects  |
| **ICollection** | In **C# memory**          | Already loaded                 | Navigation properties, Add/Remove/Count |

```csharp
// ─── IQueryable<T> — deferred, runs in DB ─────────────────────────────────
IQueryable<Employee> query = _context.Employees
    .Where(e => e.Salary > 50000);              // ← No SQL executed yet!

var result = query.ToList();                    // ← SQL runs NOW
// Generated SQL: SELECT * FROM Employees WHERE Salary > 50000

// Chain more filters — still builds SQL
var filtered = _context.Employees
    .Where(e => e.DepartmentId == 1)
    .OrderBy(e => e.Name)
    .Take(10)
    .ToList();                                  // One efficient SQL query ✅


// ─── IEnumerable<T> — in-memory, loads everything first ──────────────────
IEnumerable<Employee> allEmps = _context.Employees.ToList();  // Loads ALL rows!
var highEarners = allEmps.Where(e => e.Salary > 50000);       // Filters in C# ❌ slow

// ⚠️ Performance trap with a 1 million row table:
// IQueryable → fetches ~10 matching rows from DB ✅
// IEnumerable → fetches 1,000,000 rows into memory, then filters in C# ❌


// ─── ICollection<T> — for navigation properties (already in memory) ───────
public class Department
{
    public int DepartmentId { get; set; }
    public string Name { get; set; }
    public ICollection<Employee> Employees { get; set; }  // Navigation prop ✅
}

// ICollection supports Add, Remove, Count, Contains
department.Employees.Add(newEmployee);
int count = department.Employees.Count;
```

**When to use which:**

| Scenario                               | Use                |
| -------------------------------------- | ------------------ |
| Querying database (filtering, paging)  | `IQueryable<T>` ✅ |
| Already loaded in memory, looping      | `IEnumerable<T>`   |
| Navigation property in EF Core entity  | `ICollection<T>`   |
| Need Add/Remove on a loaded collection | `ICollection<T>`   |

**Interview Answer:** _"IQueryable builds the SQL query and runs it in the database — only when you call ToList() or similar. IEnumerable loads everything into memory first and then filters in C#, which is very inefficient for large tables. ICollection is for in-memory collections that are already loaded — it supports Add/Remove/Count and is typically used for navigation properties in EF Core."_

---

## 4.14a Data Annotations in EF Core

> **Data Annotations** are attributes you place directly on your entity class properties to configure the EF Core mapping and also perform model validation (used by ASP.NET Core's model binding).

They live in two namespaces:

- `System.ComponentModel.DataAnnotations` — validation + naming
- `System.ComponentModel.DataAnnotations.Schema` — schema mapping

### Full Annotation Reference

```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

[Table("tbl_Employees", Schema = "hr")]   // Map to specific table/schema
public class Employee
{
    // ── Primary Key ──────────────────────────────────────────────────────────
    [Key]                                  // Marks as Primary Key
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]  // Auto-increment
    public int EmployeeId { get; set; }

    // ── Column Configuration ─────────────────────────────────────────────────
    [Column("EmployeeName")]               // Override column name in DB
    [Required]                             // NOT NULL in DB + validation
    [MaxLength(100)]                       // VARCHAR(100) in DB + validation
    [MinLength(2)]                         // Validation only (no DB constraint)
    public string Name { get; set; }

    [StringLength(50, MinimumLength = 2)]  // MaxLength + MinLength in one attribute
    public string Username { get; set; }

    [Column(TypeName = "decimal(18,2)")]   // Set exact SQL type
    [Range(0, 999999.99)]                  // Validation: value must be in range
    public decimal Salary { get; set; }

    [EmailAddress]                         // Validation: must be valid email format
    public string Email { get; set; }

    [Phone]                                // Validation: must be valid phone format
    public string Phone { get; set; }

    [Url]                                  // Validation: must be valid URL
    public string ProfileUrl { get; set; }

    [DataType(DataType.Date)]              // Hint for display/input formatting only
    public DateTime DateOfBirth { get; set; }

    [NotMapped]                            // NOT stored in DB (calculated/display only)
    public string FullDisplayName => $"{Name} ({Email})";

    // ── Foreign Key ──────────────────────────────────────────────────────────
    [ForeignKey("Department")]             // Explicit FK declaration
    public int DepartmentId { get; set; }
    public Department Department { get; set; }

    // ── Concurrency ──────────────────────────────────────────────────────────
    [Timestamp]                            // RowVersion for optimistic concurrency
    public byte[] RowVersion { get; set; }

    // ── Composite Key (NOT possible with Data Annotations — use Fluent API) ──
}
```

### Annotations Quick Reference Table

| Annotation                 | Namespace              | DB Effect         | Validation Effect |
| -------------------------- | ---------------------- | ----------------- | ----------------- |
| `[Key]`                    | DataAnnotations.Schema | Primary key       | —                 |
| `[Required]`               | DataAnnotations        | NOT NULL          | Field is required |
| `[MaxLength(n)]`           | DataAnnotations        | VARCHAR(n)        | Max length check  |
| `[MinLength(n)]`           | DataAnnotations        | None              | Min length check  |
| `[StringLength(max)]`      | DataAnnotations        | VARCHAR(max)      | Max length check  |
| `[Column("name")]`         | DataAnnotations.Schema | Renames column    | —                 |
| `[Table("name")]`          | DataAnnotations.Schema | Renames table     | —                 |
| `[NotMapped]`              | DataAnnotations.Schema | Not stored in DB  | —                 |
| `[ForeignKey("nav")]`      | DataAnnotations.Schema | FK column         | —                 |
| `[Range(min, max)]`        | DataAnnotations        | None              | Value range check |
| `[EmailAddress]`           | DataAnnotations        | None              | Email format      |
| `[Url]`                    | DataAnnotations        | None              | URL format        |
| `[Phone]`                  | DataAnnotations        | None              | Phone format      |
| `[Timestamp]`              | DataAnnotations        | rowversion type   | —                 |
| `[DatabaseGenerated(...)]` | DataAnnotations.Schema | Identity/Computed | —                 |
| `[ConcurrencyCheck]`       | DataAnnotations        | Optimistic lock   | —                 |

### Data Annotations vs Fluent API

| Aspect              | Data Annotations                   | Fluent API                               |
| ------------------- | ---------------------------------- | ---------------------------------------- |
| Location            | On the entity class (attributes)   | In `OnModelCreating` in DbContext        |
| Entity cleanliness  | ❌ Entity has persistence concerns | ✅ Entity is a clean POCO                |
| Power / flexibility | Limited                            | Full control                             |
| Composite keys      | ❌ Not supported                   | ✅ Supported                             |
| Preferred when      | Simple projects, quick setup       | Production apps, DDD, clean architecture |

**Interview Answer:** _"Data Annotations are attributes placed on entity class properties to configure both the EF Core schema mapping (column name, type, required) and ASP.NET Core validation (Range, EmailAddress, Required). They're quick and convenient but limited — for example, you can't define composite keys with them. For more complex configuration I use Fluent API in OnModelCreating, which keeps entity classes clean."_

---

## 4.14 Fluent API Configuration in EF Core

> **Fluent API** configures entity-to-table mappings in code — in the `OnModelCreating` override of your `DbContext`. It is more powerful than Data Annotations and keeps entity classes clean.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Employee>(entity =>
    {
        // Table and schema name
        entity.ToTable("tbl_Employees", schema: "hr");

        // Column configuration
        entity.Property(e => e.Name)
              .IsRequired()
              .HasMaxLength(100)
              .HasColumnName("EmployeeName");

        entity.Property(e => e.Salary)
              .HasColumnType("decimal(18,2)")
              .HasDefaultValue(0);

        entity.Property(e => e.CreatedAt)
              .HasDefaultValueSql("GETUTCDATE()");  // DB-side default

        // Unique index
        entity.HasIndex(e => e.Email)
              .IsUnique();

        // Composite index
        entity.HasIndex(e => new { e.DepartmentId, e.Name });

        // Relationship: Many employees → One department
        entity.HasOne(e => e.Department)
              .WithMany(d => d.Employees)
              .HasForeignKey(e => e.DepartmentId)
              .OnDelete(DeleteBehavior.Restrict);   // Prevent cascade delete
    });

    // Composite Primary Key (can only be done with Fluent API, not Data Annotations)
    modelBuilder.Entity<OrderItem>()
        .HasKey(oi => new { oi.OrderId, oi.ProductId });
}
```

**Fluent API vs Data Annotations:**

| Aspect              | Data Annotations                   | Fluent API                               |
| ------------------- | ---------------------------------- | ---------------------------------------- |
| Location            | On the entity class (attributes)   | In `OnModelCreating` in DbContext        |
| Entity cleanliness  | ❌ Entity has persistence concerns | ✅ Entity is a clean POCO                |
| Power / flexibility | Limited                            | Full control                             |
| Composite keys      | ❌ Not supported                   | ✅ Supported                             |
| Preferred when      | Simple projects, quick setup       | Production apps, DDD, clean architecture |

**Use Fluent API when:**

- You cannot or don't want to modify the entity class
- You need composite keys
- You need complex relationship configuration
- You want to keep entities as clean POCOs

---

## 📝 EF Core & Dapper Quick Recap

| Topic                | Key Point                                    |
| -------------------- | -------------------------------------------- |
| ORM                  | Maps DB tables ↔ C# classes                  |
| DbContext            | Bridge between code and DB                   |
| `AddAsync`           | Insert new record                            |
| `FindAsync`          | Find by primary key                          |
| `SaveChangesAsync`   | Commit all pending changes                   |
| `ExecuteDeleteAsync` | Bulk delete (EF Core 8)                      |
| `ExecuteUpdateAsync` | Bulk update (EF Core 8)                      |
| Eager Loading        | `.Include()` — load related data immediately |
| Lazy Loading         | Load on first access — watch for N+1!        |
| `AsNoTracking`       | Read-only query, faster performance          |
| Dapper `Query<T>`    | Map rows to C# objects                       |
| Dapper `Execute`     | INSERT / UPDATE / DELETE                     |
| Dapper `SplitOn`     | Map JOINs to related objects                 |
| SQL Injection        | Use `@params`, never string concatenation    |

---

_[← Back to README](./README.md)_
