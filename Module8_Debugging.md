# 📗 Module 8: Application Debugging — Backend

> **Focus:** Find and fix bugs fast using Visual Studio tools — breakpoints, watch windows, exception settings, Postman + debugging together, and profiling.

---

## 8.1 Why Debugging Matters

> **Debugging** is the process of identifying, isolating, and fixing bugs in your code. Good debugging skills save hours of frustration.

### Common Challenges in Backend Debugging
| Challenge | Example |
|---|---|
| Silent failures | API returns 200 but data is wrong |
| Async bugs | Race conditions in async/await code |
| Null references | `NullReferenceException` deep in the call stack |
| Data layer issues | EF Core generates unexpected SQL |
| Environment-specific bugs | Works locally, fails in production |
| Performance issues | API works but is too slow |

---

## 8.2 Setting Breakpoints in Visual Studio

> A **breakpoint** pauses execution at a specific line so you can inspect the state of the program.

### How to Add Breakpoints
| Method | How |
|---|---|
| Click the gutter | Click in the left margin next to a line number |
| `F9` | Toggle breakpoint on/off on current line |
| Right-click → Breakpoint | More options |

### Breakpoint Types

| Type | Description | How to Set |
|---|---|---|
| **Line Breakpoint** | Pause at specific line | Click in gutter / F9 |
| **Conditional Breakpoint** | Pause only when condition is true | Right-click breakpoint → Conditions |
| **Hit Count Breakpoint** | Pause after hit N times | Right-click → Conditions → Hit Count |
| **Function Breakpoint** | Pause when function is called | Debug → New Breakpoint → Function |
| **Data Breakpoint** | Pause when variable value changes | Right-click variable → Break When Value Changes |
| **Tracepoint** | Log message WITHOUT pausing | Right-click → Actions |

```csharp
// Conditional breakpoint example — only pause when id == 5
public async Task<Employee> GetByIdAsync(int id)
{
    // Set conditional breakpoint here: id == 5
    var emp = await _context.Employees.FindAsync(id);
    return emp;
}

// Useful for loops — only pause when i == 100
for (int i = 0; i < 1000; i++)
{
    // Conditional breakpoint: i == 100
    ProcessEmployee(employees[i]);
}
```

### Managing Breakpoints
```
Breakpoints Window:  Debug → Windows → Breakpoints (Ctrl+Alt+B)
  - Enable/disable all
  - Delete all
  - Export/import breakpoint lists
  - See all breakpoints and their conditions
```

---

## 8.3 Inspecting Variables

### Watch Window
> **Watch Window** lets you monitor specific variables and expressions while debugging.

```
Open: Debug → Windows → Watch → Watch 1  (or Ctrl+D+W)

Add expressions to watch:
  - Variable names: emp, id, dto
  - Expressions: emp.Name.Length, employees.Count()
  - Method calls: employees.Any(e => e.IsActive)
  - LINQ: employees.Where(e => e.Salary > 50000).ToList()
```

### Locals Window
> **Locals Window** automatically shows ALL variables in the current scope — no need to type them.

```
Open: Debug → Windows → Locals  (Ctrl+Alt+V, L)

Shows:
  - Method parameters
  - Local variables
  - 'this' object (current class instance)
```

### Autos Window
> Shows variables used in the **current line and 3 lines around it** — focused view.

```
Open: Debug → Windows → Autos  (Ctrl+Alt+V, A)
```

### Immediate Window
> **Execute any C# expression** while paused at a breakpoint — extremely powerful.

```
Open: Debug → Windows → Immediate  (Ctrl+Alt+I)

Examples:
  emp.Name                  ← View value
  emp.Name = "Test"         ← Modify value!
  _context.Employees.Count() ← Execute LINQ
  id * 2                    ← Math expressions
  employees.Select(e => e.Name).ToList()  ← Full LINQ
```

### Quick Watch
> Right-click any variable → **Quick Watch** — instant popup showing value + expression tree.

---

## 8.4 Stepping Through Code

| Action | Key | Description |
|---|---|---|
| **Step Over** | `F10` | Execute current line, go to next line (don't enter method calls) |
| **Step Into** | `F11` | Enter the method being called on this line |
| **Step Out** | `Shift+F11` | Finish current method, return to caller |
| **Continue** | `F5` | Run until next breakpoint |
| **Run to Cursor** | `Ctrl+F10` | Run until cursor position |
| **Set Next Statement** | `Ctrl+Shift+F10` | Jump execution to a different line ⚠️ |

```csharp
public async Task<IActionResult> GetEmployee(int id)
{
    var emp = await _service.GetByIdAsync(id);   // F11 → enters GetByIdAsync
                                                   // F10 → executes it, moves to next line
    if (emp == null)
        return NotFound();                         // F10 → skips into if body if false
    
    var dto = _mapper.Map<EmployeeDto>(emp);       // F11 → enters Map() in AutoMapper
    return Ok(dto);                                // Shift+F11 → returns from method
}
```

### Call Stack Window
> Shows the **chain of method calls** that led to the current line — trace back how you got here.

```
Open: Debug → Windows → Call Stack  (Ctrl+Alt+C)

Example:
  ► EmployeeService.GetByIdAsync()      ← You are here
    EmployeesController.GetById()
    Microsoft.AspNetCore.Mvc...
    Microsoft.AspNetCore.Http...
```

---

## 8.5 Handling Exceptions in Visual Studio

### Exception Settings Window
```
Open: Debug → Windows → Exception Settings  (Ctrl+Alt+E)
```

| Setting | Behavior |
|---|---|
| **Break when thrown** (checked) | Pause AT the throw point, even if caught |
| **Break when thrown** (unchecked) | Only pause on UNHANDLED exceptions |

```csharp
// With "Break when thrown" for NullReferenceException:
// Execution pauses RIGHT HERE even though exception is caught below
try
{
    var name = employee.Department.Name;   // ← Pauses here if Department is null
}
catch (NullReferenceException ex)
{
    // Without the setting, only pause here
}
```

### Common Exception Debugging Workflow
```
1. Run into exception
2. Exception Helper popup shows: exception type + message
3. Click "View Details" for full stack trace
4. Click "Copy" to copy exception details
5. Check Inner Exception for root cause
6. Use "Enable Edit and Continue" to fix and continue (Ctrl+Shift+F10)
```

### First Chance vs Second Chance Exceptions
| Type | When | Action |
|---|---|---|
| **First Chance** | When exception is first thrown | VS notifies you even if caught |
| **Second Chance** | When exception is NOT caught | VS always pauses |

---

## 8.6 Using Postman for API Testing

### Setting Up Postman
```
1. Install Postman (postman.com)
2. Create Workspace → Create Collection (e.g., "Employee API")
3. Add Environment (e.g., "Local Dev") with variables:
   - baseUrl = http://localhost:5001
   - token = (set by login script)
```

### Import API Definition
```
1. Export Swagger JSON: http://localhost:5001/swagger/v1/swagger.json
2. Postman → Import → paste URL or upload file
3. All endpoints auto-created!
```

### Making Requests
```
GET     {{baseUrl}}/api/employees             ← Get all
GET     {{baseUrl}}/api/employees/1           ← Get by ID
POST    {{baseUrl}}/api/employees             ← Create
PUT     {{baseUrl}}/api/employees/1           ← Update
DELETE  {{baseUrl}}/api/employees/1           ← Delete

Headers:
  Content-Type: application/json
  Authorization: Bearer {{token}}

Body (POST/PUT) — raw JSON:
{
  "name": "John Doe",
  "email": "john@example.com",
  "salary": 55000
}
```

### Pre-request Script — Auto Login
```javascript
// Runs BEFORE each request — auto-fetches fresh token
pm.sendRequest({
    url: pm.environment.get("baseUrl") + "/api/auth/login",
    method: "POST",
    header: { "Content-Type": "application/json" },
    body: {
        mode: "raw",
        raw: JSON.stringify({ username: "admin", password: "pass123" })
    }
}, (err, response) => {
    pm.environment.set("token", response.json().token);
});
```

### Test Scripts
```javascript
// Assertions after response
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Response time < 500ms", () => pm.expect(pm.response.responseTime).to.be.below(500));
pm.test("Has employees array", () => {
    const body = pm.response.json();
    pm.expect(body).to.be.an("array");
    pm.expect(body.length).to.be.greaterThan(0);
});
```

---

## 8.7 Debugging API Calls with Postman + Visual Studio

> Use Postman to **trigger requests** and Visual Studio **breakpoints** to inspect what happens.

### Workflow
```
1. Set breakpoint in your Controller/Service
2. Start the API in Debug mode (F5 in Visual Studio)
3. Send request from Postman
4. Visual Studio pauses at breakpoint
5. Inspect variables, step through code
6. Continue (F5) → Postman receives response
```

### Inspecting Request Data
```csharp
[HttpPost]
public async Task<IActionResult> Create([FromBody] CreateEmployeeDto dto)
{
    // Set breakpoint here — inspect:
    // dto.Name, dto.Email, dto.Salary
    // Check if ModelState is valid
    if (!ModelState.IsValid)
        return BadRequest(ModelState);  // Add breakpoint here to see validation errors

    // ... rest of action
}
```

### Inspecting Response Data
```
In Postman:
  - Body tab: See JSON response
  - Headers tab: Check Content-Type, X-Correlation-Id
  - Status: 200/201/400/404/500
  - Time: How long the request took

If error:
  - Check the VS Output window for detailed exception
  - Check Response body for error message
  - Check VS Immediate Window for variable state
```

---

## 8.8 Advanced Debugging Techniques

### Edit and Continue
> Modify code **while paused at a breakpoint** and continue without restarting. No need to stop, rebuild, and restart for small fixes!

```
1. Pause at breakpoint
2. Edit the code
3. Press F5 or F10 to continue — changes apply immediately!

Limitations:
  - Can't add/remove class members
  - Can't change method signatures
  - Disabled for some project types
```

### Hot Reload (.NET 8)
> Apply code changes **without stopping** the app — even faster than Edit and Continue.

```
Toolbar: Click 🔥 Hot Reload button
Or: Alt+F10

Supported changes:
  ✅ Method body changes
  ✅ Adding new methods/fields
  ✅ Changing property values
  ❌ Changing method signatures
  ❌ Interface changes
```

### Remote Debugging

> Debug an application running on a **remote server** (dev server, Azure VM) from your local Visual Studio.

```
On Remote Server:
  1. Install Visual Studio Remote Debugger (msvsmon.exe)
  2. Run msvsmon.exe as administrator
  3. Configure firewall to allow port 4024

In Visual Studio (Local):
  1. Debug → Attach to Process
  2. Connection type: Remote (no authentication)
  3. Connection target: server-ip:4024
  4. Select the process: dotnet.exe
  5. Attach

Now breakpoints in your local VS code pause execution on the remote server!
```

### Attach to Process
> Connect debugger to an **already running** application (without restarting it).

```
Debug → Attach to Process  (Ctrl+Alt+P)
Select: dotnet.exe (or w3wp.exe for IIS)
Click Attach

Use for:
  - IIS-hosted applications
  - Docker containers
  - Azure App Service (with remote debugger)
```

### Debugging Docker Containers
```
1. Add Docker support to project (right-click → Add → Docker Support)
2. Set docker-compose as startup project
3. Press F5 — VS automatically attaches debugger to container
4. Breakpoints work as normal!
```

---

## 8.9 Profiling and Performance Analysis

### Visual Studio Profiler
```
Open: Debug → Performance Profiler  (Alt+F2)

Tools:
  CPU Usage      ← Which methods use most CPU?
  Memory Usage   ← Where are memory leaks?
  .NET Async     ← Find async deadlocks and bottlenecks
  Database       ← Which EF Core queries are slow?
```

### Reading CPU Usage Report
```
1. Start profiling → Use the app → Stop profiling
2. Report shows hot paths (red = hot = slow)
3. Expand call tree to find the slow method
4. Flame graph shows time spent in each method

Example finding: GetAllEmployees() takes 80% of CPU time
→ Drill down → EF Core query runs N+1 queries
→ Fix: Add .Include() for eager loading
```

### Identifying Performance Bottlenecks
```csharp
// ❌ N+1 Query Problem (common perf issue)
var employees = await _context.Employees.ToListAsync();
foreach (var emp in employees)
{
    // This fires a separate SQL query for EACH employee!
    var deptName = emp.Department.Name;  // N+1 ← detected by profiler
}

// ✅ Fix: Eager load
var employees = await _context.Employees
    .Include(e => e.Department)  // One query instead of N+1
    .ToListAsync();

// ❌ Loading too much data
var employees = await _context.Employees.ToListAsync();  // Loads ALL columns
var names = employees.Select(e => e.Name);

// ✅ Fix: Project in SQL
var names = await _context.Employees.Select(e => e.Name).ToListAsync();
```

### Useful Diagnostic Tools
| Tool | Purpose |
|---|---|
| **Visual Studio Profiler** | CPU and memory profiling |
| **dotTrace / dotMemory** | JetBrains profiler (more advanced) |
| **Application Insights** | Production monitoring in Azure |
| **MiniProfiler** | Lightweight profiler for EF Core/SQL queries |
| **BenchmarkDotNet** | Benchmark specific methods |

```bash
# MiniProfiler — shows query time in Swagger/browser
dotnet add package MiniProfiler.AspNetCore.Mvc
dotnet add package MiniProfiler.EntityFrameworkCore
```

---

## 🎯 Interview Questions for Module 8

| # | Question |
|---|---|
| 1 | What is a breakpoint? What types of breakpoints exist in Visual Studio? |
| 2 | What is the difference between Step Into, Step Over, and Step Out? |
| 3 | What is the difference between the Watch, Locals, and Autos windows? |
| 4 | What is the Immediate Window? What can you do with it? |
| 5 | What is the Call Stack? How do you use it during debugging? |
| 6 | What is the Exception Settings window? What is First Chance exception? |
| 7 | What is Edit and Continue? What are its limitations? |
| 8 | How do you debug an ASP.NET Core API using Postman? |
| 9 | How do you attach the Visual Studio debugger to a running process? |
| 10 | What is Remote Debugging? How do you set it up? |
| 11 | What is the N+1 query problem? How do you detect and fix it? |
| 12 | What tools do you use to profile and find performance bottlenecks? |

---

*[← Module 7](./Module7_Microservices.md) | [Back to README](./README.md) | [Next Module →](./Module9_Docker.md)*
