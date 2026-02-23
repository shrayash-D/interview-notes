# 📗 Module 7: Microservices Architecture

> **Focus:** Break a monolith into small, independent services — learn communication patterns, data management, security, monitoring, and deployment.

---

## 7.1 What is Microservices Architecture?

> **Microservices** is an architectural style where an application is built as a collection of **small, independent services**, each responsible for a **single business capability**, running in its own process and communicating via APIs.

### Microservices vs Monolithic Architecture

| Feature           | Monolithic                       | Microservices                          |
| ----------------- | -------------------------------- | -------------------------------------- |
| **Structure**     | One large application            | Many small services                    |
| **Deployment**    | Deploy entire app for any change | Deploy only the changed service        |
| **Scaling**       | Scale entire app                 | Scale only the bottleneck service      |
| **Tech stack**    | Single technology                | Each service can use different tech    |
| **Development**   | One team works on everything     | Small teams own individual services    |
| **Failure**       | One bug can crash everything     | One service fails, others keep running |
| **Complexity**    | Simple to start                  | Complex (networking, data, ops)        |
| **Communication** | In-process method calls          | HTTP, gRPC, messaging                  |
| **Data**          | Shared database                  | Each service has its own database      |

**Interview Answer:** "Microservices splits a large application into small, independently deployable services, each owning its own data and business logic. It gives better scalability, resilience, and team autonomy, but adds complexity in networking, distributed data, and operations."

---

## 7.2 Advantages and Challenges

### ✅ Advantages

- **Independent deployment** — update one service without touching others
- **Scalability** — scale only the high-traffic service (e.g., scale Order service 10x, keep User service at 1x)
- **Fault isolation** — one service crash doesn't take down the whole app
- **Technology flexibility** — use Python for ML service, .NET for API, Node.js for notifications
- **Small teams** — each team owns one or two services end-to-end
- **Faster development** — smaller codebase = easier to understand and change

### ❌ Challenges

- **Network latency** — service calls go over the network (slow vs in-process calls)
- **Distributed transactions** — a transaction spanning 2 services is hard to keep consistent
- **Service discovery** — how does Service A find Service B's URL?
- **More moving parts** — need load balancers, API gateways, service registries
- **Data consistency** — no single database means eventual consistency challenges
- **Testing** — integration testing is harder across multiple services

---

## 7.3 Setting Up a Microservices Project

```
Solution/
├── Services/
│   ├── EmployeeService/        ← ASP.NET Core Web API
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Data/               ← Its OWN DbContext
│   │   └── Program.cs
│   │
│   ├── DepartmentService/      ← Separate API, separate DB
│   │   ├── Controllers/
│   │   └── Program.cs
│   │
│   └── NotificationService/    ← Handles emails/SMS
│
├── ApiGateway/                 ← Single entry point (Ocelot/YARP)
├── docker-compose.yml          ← Run all services together
└── .github/workflows/          ← CI/CD pipelines
```

---

## 7.4 Inter-Service Communication

**The fundamental question:** How does Service A talk to Service B?

**Two communication styles:**

- **Synchronous** — Service A sends a request and **waits** for Service B's response before continuing. If Service B is slow or down, Service A is blocked.
- **Asynchronous** — Service A publishes a message to a **message broker** and continues immediately. Service B reads and processes the message whenever it's ready. No waiting, no blocking.

**When to use which:**

- **Synchronous (HTTP/gRPC)** — When Service A needs the response to continue its own work (e.g., get user details to include in an order)
- **Asynchronous (Message broker)** — When Service A doesn't need an immediate response (e.g., send a notification email after order placed)

**Interview Answer:** "Microservices communicate either synchronously (HTTP/REST or gRPC — caller waits for response) or asynchronously (message broker like RabbitMQ/Azure Service Bus — fire and forget). Sync is simpler but creates tight coupling and propagates failures. Async is more resilient but adds complexity."

### Option 1: HTTP/REST (Synchronous)

> Service A calls Service B directly over HTTP. Simple but tightly coupled — if B is down, A fails too.

```csharp
// Register HttpClient for DI
builder.Services.AddHttpClient<IDepartmentClient, DepartmentClient>(client =>
{
    client.BaseAddress = new Uri("http://department-service/");
});

// Typed HTTP client
public class DepartmentClient : IDepartmentClient
{
    private readonly HttpClient _http;

    public DepartmentClient(HttpClient http) { _http = http; }

    public async Task<DepartmentDto> GetDepartmentAsync(int id)
    {
        var response = await _http.GetAsync($"api/departments/{id}");
        response.EnsureSuccessStatusCode();
        return await response.Content.ReadFromJsonAsync<DepartmentDto>();
    }
}

// Use in EmployeeService
public class EmployeeService
{
    private readonly IDepartmentClient _deptClient;

    public async Task<EmployeeDetailDto> GetEmployeeAsync(int id)
    {
        var emp = await _empRepo.GetByIdAsync(id);
        var dept = await _deptClient.GetDepartmentAsync(emp.DepartmentId); // HTTP call
        return new EmployeeDetailDto { Name = emp.Name, DeptName = dept.Name };
    }
}
```

### Option 2: gRPC (Synchronous, High Performance)

> **gRPC** uses binary Protocol Buffers instead of JSON — much faster than REST, ideal for internal service-to-service communication.

```bash
dotnet add package Grpc.AspNetCore
```

```protobuf
// employee.proto — define service contract
syntax = "proto3";

service EmployeeService {
  rpc GetEmployee (GetEmployeeRequest) returns (EmployeeResponse);
  rpc GetAllEmployees (Empty) returns (EmployeeListResponse);
}

message GetEmployeeRequest { int32 id = 1; }
message EmployeeResponse {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

```csharp
// Server — implement gRPC service
public class EmployeeGrpcService : EmployeeServiceBase
{
    public override async Task<EmployeeResponse> GetEmployee(
        GetEmployeeRequest request, ServerCallContext context)
    {
        var emp = await _repo.GetByIdAsync(request.Id);
        return new EmployeeResponse { Id = emp.Id, Name = emp.Name, Email = emp.Email };
    }
}

// Client — call the gRPC service
var channel = GrpcChannel.ForAddress("https://employee-service");
var client = new EmployeeService.EmployeeServiceClient(channel);
var reply = await client.GetEmployeeAsync(new GetEmployeeRequest { Id = 1 });
```

### Option 3: Message Broker (Asynchronous)

> Services communicate via messages on a **queue/topic**. Producer sends a message; consumer processes it later. Decouples services completely.

```csharp
// Azure Service Bus / RabbitMQ
// Publisher (EmployeeService sends event when employee is created)
public class EmployeeCreatedEvent
{
    public int EmployeeId { get; set; }
    public string Email { get; set; }
    public DateTime CreatedAt { get; set; }
}

// After creating employee, publish event
await _serviceBusClient.PublishAsync(new EmployeeCreatedEvent
{
    EmployeeId = employee.Id,
    Email = employee.Email,
    CreatedAt = DateTime.UtcNow
});

// NotificationService subscribes and sends welcome email
public class EmployeeCreatedHandler
{
    public async Task HandleAsync(EmployeeCreatedEvent evt)
    {
        await _emailService.SendWelcomeEmailAsync(evt.Email);
    }
}
```

### Communication Patterns Comparison

| Pattern             | Type         | Use When                                      |
| ------------------- | ------------ | --------------------------------------------- |
| **HTTP/REST**       | Synchronous  | Need immediate response; simple calls         |
| **gRPC**            | Synchronous  | High performance, internal services           |
| **Message Broker**  | Asynchronous | No immediate response needed; fire-and-forget |
| **Event Streaming** | Asynchronous | Audit logs, real-time data pipelines          |

---

## 7.5 Service Discovery

**What is it?**
In a microservices system, services run in containers with dynamic IPs that change on restart, scaling, or redeployment. You can't hardcode `http://192.168.1.5:5001` — that IP might be different tomorrow. Service discovery solves this: each service **registers itself** with a central registry, and other services look up the current address by name.

**Two models:**

- **Client-side discovery** — the calling service queries the registry and picks an address itself (Netflix Ribbon pattern)
- **Server-side discovery** — the caller sends to a load balancer/gateway, which does the lookup (Kubernetes does this)

**API Gateway Pattern:**
Rather than having each client know about every microservice's address, an **API Gateway** is a single entry point. Clients talk to one URL; the gateway routes to the right service. It also handles cross-cutting concerns: authentication, rate limiting, SSL termination, logging.

> **Service Discovery** is how services find each other's addresses dynamically instead of using hardcoded IPs.

```
Without Discovery:  Service A calls http://192.168.1.5:5001 (hardcoded IP — breaks if server moves)
With Discovery:     Service A asks registry "where is EmployeeService?" → gets current URL
```

### Tools

- **Consul** — standalone service registry
- **Kubernetes DNS** — K8s auto-registers services (e.g., `http://employee-service`)
- **Azure API Management** — managed discovery + gateway
- **Eureka** — Netflix OSS (Java world, also available for .NET)

### API Gateway Pattern

> A single entry point that routes requests to the correct microservice. Clients talk to ONE endpoint.

```
Client → API Gateway (port 80)
           ├── /api/employees  → EmployeeService (port 5001)
           ├── /api/depts      → DepartmentService (port 5002)
           └── /api/notify     → NotificationService (port 5003)

// Ocelot — Popular .NET API Gateway
dotnet add package Ocelot

// ocelot.json
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/api/employees/{everything}",
      "DownstreamScheme": "http",
      "DownstreamHostAndPorts": [{ "Host": "employee-service", "Port": 80 }],
      "UpstreamPathTemplate": "/api/employees/{everything}",
      "UpstreamHttpMethod": ["GET", "POST", "PUT", "DELETE"]
    }
  ]
}
```

---

## 7.6 Data Management in Microservices

**The hardest part of microservices is data.**
In a monolith, you can do a database JOIN across tables and wrap everything in one transaction. In microservices, each service owns its own database — you can't do cross-service JOINs, and distributed transactions are extremely hard.

**Database per Service — why it's the rule:**
If all services share one database, changing a table schema in one service can break other services. The database becomes a coupling point — defeating the purpose of microservices. Database-per-service means each service is truly independent.

**Eventual consistency:**
Because services have separate databases, data can be temporarily inconsistent. For example, after an order is placed, the inventory service may not have processed the stock reduction yet. The system will eventually become consistent — this is called **eventual consistency** and is a design trade-off you accept in distributed systems.

### Database per Service Pattern ✅ (Recommended)

> Each microservice has its **own private database**. No other service can access it directly — they must go through the service's API.

```
EmployeeService     → EmployeeDB     (SQL Server)
DepartmentService   → DepartmentDB   (PostgreSQL)
NotificationService → NotificationDB (MongoDB)
OrderService        → OrderDB        (SQL Server)
```

**Why?** Loose coupling — EmployeeService can change its DB schema without breaking other services.

### Shared Database Pattern ❌ (Anti-pattern)

> All services share one database — creates tight coupling, one service's schema change can break others.

### Data Consistency Challenges

```
Problem: Place an Order requires:
  1. Create order in OrderService DB ✓
  2. Reduce inventory in InventoryService DB ✗ (network fails!)
  Result: Order exists but inventory not reduced ← Inconsistent!
```

### SAGA Pattern — Distributed Transactions

> A **SAGA** is a sequence of local transactions with compensating transactions for rollback.

```
Choreography SAGA (event-driven):
  1. OrderService creates order → publishes OrderCreated event
  2. InventoryService receives event → reduces stock → publishes StockReduced event
  3. PaymentService receives event → charges card → publishes PaymentProcessed event
  If step 3 fails → PaymentService publishes PaymentFailed → InventoryService restores stock
```

---

## 7.7 EF Core in Microservices

```csharp
// Each service has its OWN DbContext — no sharing!
// EmployeeService/Data/EmployeeDbContext.cs
public class EmployeeDbContext : DbContext
{
    public EmployeeDbContext(DbContextOptions<EmployeeDbContext> options) : base(options) { }

    public DbSet<Employee> Employees { get; set; }
    // No Departments, Orders here! That belongs to their own services
}

// Program.cs
builder.Services.AddDbContext<EmployeeDbContext>(opt =>
    opt.UseSqlServer(builder.Configuration.GetConnectionString("EmployeeDb")));

// Migrations — run per service
dotnet ef migrations add InitialCreate --project EmployeeService
dotnet ef database update --project EmployeeService
```

---

## 7.8 JWT Authentication in Microservices

> In microservices, the **API Gateway** validates the JWT token once. Then it passes the token (or user info) to downstream services via request headers.

```
Client → API Gateway (validates JWT) → EmployeeService (trusts gateway)

// API Gateway validates token, adds claims to header
context.Request.Headers.Add("X-User-Id", userId);
context.Request.Headers.Add("X-User-Role", role);

// EmployeeService reads from header (no re-validation needed)
var userId = context.Request.Headers["X-User-Id"].ToString();

// OR: Each service validates the JWT independently (more secure)
// All services share the same JWT signing key
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(/* same config as in the gateway */);
```

---

## 7.9 Health Checks

> **Health checks** tell orchestrators (Kubernetes, load balancers) whether a service is healthy and ready to receive traffic.

```bash
dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks
dotnet add package AspNetCore.HealthChecks.SqlServer
```

```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("Default"),  // DB check
        name: "database",
        failureStatus: HealthStatus.Unhealthy)
    .AddCheck("self", () => HealthCheckResult.Healthy());    // Basic liveness check

app.MapHealthChecks("/health", new HealthCheckOptions
{
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});

// GET /health
// Returns: { "status": "Healthy", "checks": [ { "name": "database", "status": "Healthy" } ] }
```

### Health Check Types

| Type          | URL               | Purpose                                    |
| ------------- | ----------------- | ------------------------------------------ |
| **Liveness**  | `/health/live`    | Is the service running? (restart if fails) |
| **Readiness** | `/health/ready`   | Is the service ready for traffic?          |
| **Startup**   | `/health/startup` | Did the service finish starting?           |

---

## 7.10 Monitoring and Logging Strategy

```csharp
// Correlation ID — trace a request across multiple services
public class CorrelationIdMiddleware
{
    public async Task InvokeAsync(HttpContext context, RequestDelegate next)
    {
        var correlationId = context.Request.Headers["X-Correlation-Id"].FirstOrDefault()
                            ?? Guid.NewGuid().ToString();

        context.Response.Headers.Add("X-Correlation-Id", correlationId);

        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            await next(context);  // All logs in this request will include CorrelationId
        }
    }
}
```

### Distributed Tracing Tools

| Tool                             | Purpose                                             |
| -------------------------------- | --------------------------------------------------- |
| **Serilog**                      | Structured logging                                  |
| **Seq**                          | Log aggregation and search UI                       |
| **Jaeger / Zipkin**              | Distributed tracing (trace request across services) |
| **OpenTelemetry**                | Standard for collecting traces, metrics, logs       |
| **Azure Monitor / App Insights** | Azure-native monitoring                             |

---

## 7.11 CI/CD with GitHub Actions

```yaml
# .github/workflows/employee-service.yml
name: Build and Deploy Employee Service

on:
  push:
    branches: [main]
    paths:
      - "Services/EmployeeService/**" # Only trigger when this service changes

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup .NET 8
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: "8.0.x"

      - name: Build
        run: dotnet build Services/EmployeeService --configuration Release

      - name: Test
        run: dotnet test Services/EmployeeService.Tests

      - name: Build Docker image
        run: docker build -t employee-service:${{ github.sha }} ./Services/EmployeeService

      - name: Push to Azure Container Registry
        run: |
          docker tag employee-service:${{ github.sha }} myregistry.azurecr.io/employee-service:latest
          docker push myregistry.azurecr.io/employee-service:latest

      - name: Deploy to AKS
        run: kubectl set image deployment/employee-service employee-service=myregistry.azurecr.io/employee-service:latest
```

---

## 7.12 Deployment Strategies

### Rolling Update (Default in Kubernetes)

```
Old pods replaced one at a time → Zero downtime
v1 v1 v1 v1  →  v2 v1 v1 v1  →  v2 v2 v1 v1  →  v2 v2 v2 v2
```

### Blue-Green Deployment

```
Blue  = current production (v1) — live traffic
Green = new version (v2)      — no traffic yet

1. Deploy v2 to Green environment
2. Run tests on Green
3. Switch load balancer from Blue to Green — instant cutover!
4. Keep Blue as rollback for 24h, then decommission
```

### Canary Deployment

```
Route 5% of traffic to new version, 95% to old version.
Monitor for errors → if OK, gradually increase to 100%
If problems → instantly switch back to 0% on new version
```

| Strategy       | Downtime | Rollback       | Cost         |
| -------------- | -------- | -------------- | ------------ |
| Rolling Update | Zero     | Slow (re-roll) | Low          |
| Blue-Green     | Zero     | Instant        | 2x resources |
| Canary         | Zero     | Instant        | Low          |

---

## 7.13 Docker in Microservices

```dockerfile
# Dockerfile for EmployeeService
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["EmployeeService.csproj", "."]
RUN dotnet restore
COPY . .
RUN dotnet build -c Release -o /app/build

FROM build AS publish
RUN dotnet publish -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "EmployeeService.dll"]
```

```yaml
# docker-compose.yml — Run all services together
version: "3.8"
services:
  employee-service:
    build: ./Services/EmployeeService
    ports: ["5001:80"]
    environment:
      - ConnectionStrings__Default=Server=sqlserver;Database=EmployeeDB;...
    depends_on:
      - sqlserver

  department-service:
    build: ./Services/DepartmentService
    ports: ["5002:80"]

  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      SA_PASSWORD: "YourPassword123!"
      ACCEPT_EULA: "Y"
    ports: ["1433:1433"]
```

---

## 🎯 Interview Questions for Module 7

| #   | Question                                                                       |
| --- | ------------------------------------------------------------------------------ |
| 1   | What is microservices architecture? How is it different from monolithic?       |
| 2   | What are the advantages of microservices? What are the challenges?             |
| 3   | What is synchronous vs asynchronous communication in microservices?            |
| 4   | What is gRPC? When would you use it over REST?                                 |
| 5   | What is an API Gateway? Why is it needed in microservices?                     |
| 6   | What is the Database per Service pattern? Why is shared DB an anti-pattern?    |
| 7   | What is the SAGA pattern? What problem does it solve?                          |
| 8   | How do you handle JWT authentication across multiple microservices?            |
| 9   | What is a health check? What is the difference between liveness and readiness? |
| 10  | What is a Correlation ID? Why is it important in microservices?                |
| 11  | What is Blue-Green deployment? What is Canary deployment?                      |
| 12  | What is service discovery? Name some tools.                                    |
| 13  | How does CI/CD work with GitHub Actions for a microservices project?           |

---

_[← Module 6](./Module6_WebAPI.md) | [Back to README](./README.md) | [Next Module →](./Module8_Debugging.md)_
