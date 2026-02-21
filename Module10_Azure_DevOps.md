# 📗 Module 10: Cloud & DevOps Basics using Azure

> **Focus:** Understand cloud fundamentals, key Azure services (Compute, Networking, Data, Messaging, Identity), and Azure DevOps for CI/CD pipelines.

---

## 10.1 What is the Cloud?

> The **Cloud** is the delivery of computing services (servers, storage, databases, networking, software) over the internet instead of owning and managing physical hardware.

### Characteristics of the Cloud
| Characteristic | Meaning |
|---|---|
| **On-demand self-service** | Provision resources instantly without human help |
| **Broad network access** | Access from anywhere via internet |
| **Resource pooling** | Share resources across many customers |
| **Rapid elasticity** | Scale up/down quickly based on demand |
| **Measured service** | Pay only for what you use |

### CapEx vs OpEx
| Type | Meaning | Example | Cloud? |
|---|---|---|---|
| **CapEx** (Capital Expenditure) | Upfront investment in physical assets | Buy servers, data center | ❌ No — traditional IT |
| **OpEx** (Operational Expenditure) | Ongoing costs for services | Monthly cloud bill | ✅ Yes — cloud model |

**Cloud benefit:** Convert CapEx to OpEx — no upfront cost, pay as you go.

### Cloud Service Models
| Model | You Manage | Provider Manages | Example |
|---|---|---|---|
| **IaaS** (Infrastructure) | OS, App, Data | Hardware, Networking, Virtualization | Azure VMs |
| **PaaS** (Platform) | App, Data | OS, Runtime, Hardware | Azure App Service |
| **SaaS** (Software) | Nothing | Everything | Office 365, Gmail |

### Types of Clouds
| Type | Description | Use Case |
|---|---|---|
| **Public Cloud** | Shared infrastructure managed by provider | Most businesses (Azure, AWS, GCP) |
| **Private Cloud** | Dedicated infrastructure for one org | Banks, government (security-sensitive) |
| **Hybrid Cloud** | Mix of public + private | Sensitive data on-premises, rest in cloud |
| **Multi-cloud** | Use multiple cloud providers | Avoid vendor lock-in |

---

## 10.2 Introduction to Azure

### What is Azure?
> **Microsoft Azure** is Microsoft's cloud platform with 200+ services across compute, storage, networking, AI, databases, DevOps, and more.

### Azure Regions and Zones
| Term | Description |
|---|---|
| **Region** | A geographical area with one or more data centers (e.g., East US, West Europe) |
| **Availability Zone** | Physically separate data centers within a region — for high availability |
| **Region Pair** | Two regions paired for disaster recovery (East US ↔ West US) |

```
Choose region closest to users for low latency.
Use Availability Zones for 99.99% SLA on VMs.
```

### Azure Portal
> Web UI at portal.azure.com — create, manage, monitor all resources.

### Azure CLI
```bash
# Install Azure CLI, then login
az login

# List subscriptions
az account list --output table

# Create a Resource Group
az group create --name MyResourceGroup --location eastus

# List all resources in a group
az resource list --resource-group MyResourceGroup --output table

# Delete a resource group (deletes everything in it!)
az group delete --name MyResourceGroup --yes
```

### Azure Subscriptions and Resource Groups
```
Account
  └── Subscription (billing unit — e.g., "Dev" or "Production")
        └── Resource Group (logical container for related resources)
              ├── VM
              ├── SQL Database
              └── Storage Account
```

---

## 10.3 Azure Basic Concepts

### Resource Groups
> Logical containers that group related resources together. A resource group should contain resources that share the same lifecycle (deploy, update, delete together).

```bash
az group create --name MyApp-RG --location "East US"
```

### Storage Accounts
> Provides cloud storage for blobs, files, queues, and tables.

| Storage Type | Use Case |
|---|---|
| **Blob Storage** | Files, images, videos, backups |
| **Table Storage** | NoSQL key-value data |
| **Queue Storage** | Message queues between services |
| **File Storage** | Shared network files (like a network drive) |

### SLA (Service Level Agreement)
> Azure guarantees a certain uptime percentage per service.

| SLA | Downtime per month |
|---|---|
| 99% | ~7.2 hours |
| 99.9% | ~43 minutes |
| 99.95% | ~21 minutes |
| 99.99% | ~4 minutes |

---

## 10.4 Azure Compute

### Virtual Machines (VMs)
> Full control over OS and software. Use when you need IaaS.

```bash
# Create VM
az vm create \
  --resource-group MyRG \
  --name MyVM \
  --image Win2022Datacenter \
  --admin-username azureuser \
  --admin-password YourPassword123!

# Start/Stop VM
az vm start --resource-group MyRG --name MyVM
az vm stop  --resource-group MyRG --name MyVM

# Delete VM
az vm delete --resource-group MyRG --name MyVM --yes
```

### ARM Templates (Infrastructure as Code)
> **ARM (Azure Resource Manager) Templates** are JSON files that define Azure resources — deploy consistently every time.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": { "type": "string" }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2022-09-01",
      "name": "[parameters('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2"
    }
  ]
}
```

```bash
az deployment group create --resource-group MyRG --template-file template.json
```

### Azure App Service
> **PaaS** hosting for web apps — no OS management needed. Just deploy your code.

```bash
# Create App Service Plan (compute resources)
az appservice plan create --name MyPlan --resource-group MyRG --sku B1 --is-linux

# Create Web App
az webapp create --name MyWebApp --resource-group MyRG --plan MyPlan --runtime "DOTNET:8.0"

# Deploy from local folder
az webapp deploy --resource-group MyRG --name MyWebApp --src-path ./publish
```

### App Service Tiers
| Tier | Use Case | Features |
|---|---|---|
| **Free (F1)** | Dev/test | 1 GB, no custom domain |
| **Basic (B1-B3)** | Low traffic | Custom domain, manual scale |
| **Standard (S1-S3)** | Production | Auto-scale, deployment slots |
| **Premium (P1-P3)** | High traffic | More CPU/RAM, VNet integration |

### Deployment Slots
> Staging environments — deploy to "staging" slot, test, then **swap** to production with zero downtime.

```bash
az webapp deployment slot create --name MyWebApp --resource-group MyRG --slot staging
az webapp deployment slot swap   --name MyWebApp --resource-group MyRG --slot staging
```

### Azure Functions (Serverless)
> Run code without managing servers — pay only when the function executes.

```csharp
// HTTP triggered function
public static class EmployeeFunction
{
    [FunctionName("GetEmployee")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "employees/{id}")] HttpRequest req,
        int id,
        ILogger log)
    {
        log.LogInformation($"Getting employee {id}");
        var employee = await _service.GetByIdAsync(id);
        return new OkObjectResult(employee);
    }
}

// Triggers:
// HttpTrigger    — HTTP request
// TimerTrigger   — scheduled (like cron)
// BlobTrigger    — when file uploaded to blob
// QueueTrigger   — when message arrives in queue
// EventGridTrigger — when event published
```

### AKS (Azure Kubernetes Service)
> Managed Kubernetes in Azure — Azure handles the control plane; you manage worker nodes and workloads.

```bash
# Create AKS cluster
az aks create --resource-group MyRG --name MyAKS --node-count 3 --generate-ssh-keys

# Get credentials (connect kubectl to AKS)
az aks get-credentials --resource-group MyRG --name MyAKS

# Deploy to AKS
kubectl apply -f deployment.yml
```

---

## 10.5 Azure Networking

### Virtual Networks (VNets)
> **VNet** is a private network in Azure — resources inside can communicate securely. Like your own data center network in the cloud.

```
VNet: 10.0.0.0/16
  ├── Subnet 1: 10.0.1.0/24  ← Web tier (VMs, App Service)
  ├── Subnet 2: 10.0.2.0/24  ← App tier (APIs)
  └── Subnet 3: 10.0.3.0/24  ← Data tier (SQL, Redis)
```

```bash
# Create VNet
az network vnet create --name MyVNet --resource-group MyRG --address-prefix 10.0.0.0/16

# Create subnet
az network vnet subnet create --name WebSubnet --vnet-name MyVNet \
  --resource-group MyRG --address-prefix 10.0.1.0/24
```

### Load Balancer vs Application Gateway
| Service | OSI Layer | Use Case |
|---|---|---|
| **Load Balancer** | Layer 4 (TCP/UDP) | Distribute traffic to VMs |
| **Application Gateway** | Layer 7 (HTTP/HTTPS) | Route by URL, SSL termination, WAF |

### Service Endpoint vs Private Link
| Feature | Service Endpoint | Private Link |
|---|---|---|
| **Traffic** | Stays in Azure backbone | Stays in your VNet |
| **Access** | Any Azure service of that type | Specific resource only |
| **Public IP** | Service still has public IP | Service gets a private IP |
| **Cost** | Free | Has a cost |

---

## 10.6 Data in Azure

### Azure SQL Database
> Fully managed SQL Server in the cloud — no patching, backups handled by Azure.

```bash
# Create SQL Server
az sql server create --name mysqlserver --resource-group MyRG \
  --admin-user sqladmin --admin-password YourPass123!

# Create database
az sql db create --resource-group MyRG --server mysqlserver \
  --name EmployeeDB --service-objective S0

# Connection string
Server=tcp:mysqlserver.database.windows.net,1433;Database=EmployeeDB;
User=sqladmin@mysqlserver;Password=YourPass123!;
```

### Cosmos DB
> Fully managed **NoSQL** database — globally distributed, multiple APIs (SQL, MongoDB, Cassandra, etc.)

| Feature | Azure SQL | Cosmos DB |
|---|---|---|
| **Type** | Relational (SQL) | NoSQL (document, key-value) |
| **Schema** | Fixed schema | Flexible (schemaless) |
| **Scalability** | Vertical | Horizontal (unlimited) |
| **Query** | T-SQL | SQL API, MongoDB API, etc. |
| **Use case** | Structured data, transactions | Unstructured data, global scale |
| **SLA** | 99.99% | 99.999% |

```csharp
// Use Cosmos DB with .NET SDK
var client = new CosmosClient("connection-string");
var database = client.GetDatabase("EmployeeDB");
var container = database.GetContainer("Employees");

// Add item
var employee = new { id = "1", name = "John", department = "IT" };
await container.CreateItemAsync(employee, new PartitionKey("IT"));

// Query items
var query = new QueryDefinition("SELECT * FROM c WHERE c.department = @dept")
    .WithParameter("@dept", "IT");
var results = container.GetItemQueryIterator<Employee>(query);
```

### Azure Storage
```csharp
// Upload to Blob Storage
var blobServiceClient = new BlobServiceClient("connection-string");
var containerClient = blobServiceClient.GetBlobContainerClient("documents");
await containerClient.CreateIfNotExistsAsync();

var blobClient = containerClient.GetBlobClient("report.pdf");
await blobClient.UploadAsync(fileStream);

// Generate SAS URL (temporary access link)
var sasUri = blobClient.GenerateSasUri(BlobSasPermissions.Read, DateTimeOffset.UtcNow.AddHours(1));
```

---

## 10.7 Messaging in Azure

### Storage Queue
> Simple FIFO message queue — basic, cheap, up to 64KB per message.

```csharp
var queueClient = new QueueClient("connection-string", "my-queue");
await queueClient.CreateIfNotExistsAsync();

// Send message
await queueClient.SendMessageAsync("Hello from Employee Service!");

// Receive and process message
var message = await queueClient.ReceiveMessageAsync();
Console.WriteLine(message.Value.MessageText);
await queueClient.DeleteMessageAsync(message.Value.MessageId, message.Value.PopReceipt);
```

### Event Grid
> Publish-Subscribe service for **event-driven** architectures. Route events from Azure resources to handlers.

```
Blob uploaded → Event Grid → Azure Function → Resize image
VM deallocated → Event Grid → Logic App → Send email alert
```

### Service Bus
> Enterprise-grade message broker — supports **queues** and **topics/subscriptions**, guaranteed delivery, dead-letter queue.

```csharp
// Send message
var client = new ServiceBusClient("connection-string");
var sender = client.CreateSender("employee-events");
await sender.SendMessageAsync(new ServiceBusMessage("Employee created: John Doe"));

// Receive message
var receiver = client.CreateReceiver("employee-events");
var message = await receiver.ReceiveMessageAsync();
Console.WriteLine(message.Body.ToString());
await receiver.CompleteMessageAsync(message);
```

### Messaging Comparison
| Service | Type | Use Case |
|---|---|---|
| **Storage Queue** | Simple queue | Basic decoupling, cheap |
| **Service Bus Queue** | Enterprise queue | Ordered, guaranteed, retry logic |
| **Service Bus Topic** | Pub/Sub | Multiple subscribers to one message |
| **Event Grid** | Event routing | React to Azure resource events |
| **Event Hubs** | Event streaming | High-throughput (millions/sec), IoT, analytics |

---

## 10.8 Azure Active Directory (AAD / Entra ID)

### Key Concepts
| Term | Description |
|---|---|
| **Tenant** | An organization's instance of AAD (e.g., cognizant.onmicrosoft.com) |
| **User** | A person with an account in the tenant |
| **Group** | Collection of users (e.g., Developers, Admins) |
| **App Registration** | Registers your app with AAD for OAuth/OIDC |
| **Service Principal** | Identity for applications/services |
| **MFA** | Multi-Factor Authentication (password + phone/app) |
| **RBAC** | Role-Based Access Control — assign roles to users/groups |

### Azure Roles (RBAC)
| Role | Permission |
|---|---|
| **Owner** | Full control including access management |
| **Contributor** | Create/manage resources but can't manage access |
| **Reader** | View resources only |
| **Custom Role** | Define exactly what permissions are needed |

```bash
# Assign role to user
az role assignment create \
  --assignee user@domain.com \
  --role Contributor \
  --scope /subscriptions/{sub-id}/resourceGroups/MyRG
```

### OAuth 2.0 + JWT in Azure
```
1. User logs in via Azure AD (Microsoft login)
2. Azure AD issues JWT access token
3. App includes token in API calls: Authorization: Bearer <token>
4. API validates token against Azure AD

// Protect API with Azure AD
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(builder.Configuration.GetSection("AzureAd"));
```

---

## 10.9 Azure DevOps Overview

> **Azure DevOps** is a set of developer tools for planning, developing, testing, and deploying software.

### Key Components
| Service | Purpose |
|---|---|
| **Azure Boards** | Agile project management (User Stories, Tasks, Bugs) |
| **Azure Repos** | Git source control |
| **Azure Pipelines** | CI/CD build and release automation |
| **Azure Test Plans** | Manual and automated testing |
| **Azure Artifacts** | Package management (NuGet, npm, Maven) |

---

## 10.10 Azure Repos (Git)

### Git Basics
```bash
git init                          # Initialize a new repo
git clone <url>                   # Clone a repo
git add .                         # Stage all changes
git commit -m "message"           # Commit changes
git push origin main              # Push to remote
git pull                          # Pull latest changes
git status                        # Check what changed
git log --oneline                 # Short commit history
```

### Branching Strategy
```
main          ← Production code only
  └── develop ← Integration branch
        ├── feature/login     ← New feature
        ├── feature/reports   ← Another feature
        └── bugfix/null-check ← Bug fix
```

### Git Workflow
```bash
# Feature branch workflow
git checkout -b feature/add-employee    # Create and switch to branch
# ... make changes ...
git add .
git commit -m "Add employee endpoint"
git push origin feature/add-employee   # Push branch
# Create Pull Request in Azure DevOps
# Team reviews → Approve → Merge to develop
```

### Pull Request (PR) Best Practices
- Small PRs (< 400 lines) — easier to review
- Clear description of what changed and why
- All tests must pass before merge
- At least 1 reviewer approval required
- Use **branch policies** to enforce these rules

---

## 10.11 Azure Boards

### Work Item Types
| Type | Description |
|---|---|
| **Epic** | Large feature spanning multiple sprints (e.g., "User Management Module") |
| **Feature** | A specific capability (e.g., "User Login") |
| **User Story** | A feature from user perspective: "As a user, I want to log in so I can access my data" |
| **Task** | A specific technical task within a story (e.g., "Create JWT endpoint") |
| **Bug** | A defect that needs fixing |

### Scrum Concepts
| Term | Description |
|---|---|
| **Sprint** | Time-boxed work period (usually 2 weeks) |
| **Sprint Backlog** | Stories/tasks committed for this sprint |
| **Product Backlog** | All upcoming work, prioritized |
| **Daily Standup** | 15-min daily meeting: what did I do, what will I do, blockers? |
| **Sprint Review** | Demo completed work to stakeholders |
| **Sprint Retrospective** | Team discusses what went well and what to improve |
| **Velocity** | How many story points a team completes per sprint |

### Kanban Board States
```
To Do → In Progress → In Review → Done
```

---

## 10.12 Azure Pipelines — CI/CD

### What is CI/CD?
| Term | Full Form | Meaning |
|---|---|---|
| **CI** | Continuous Integration | Auto-build and test on every commit |
| **CD** | Continuous Delivery | Auto-deploy to staging on every merge |
| **CD** | Continuous Deployment | Auto-deploy to production (no manual step) |

### YAML Pipeline (azure-pipelines.yml)
```yaml
trigger:
  branches:
    include:
      - main         # Run when code pushed to main

pool:
  vmImage: ubuntu-latest   # Build agent (Ubuntu, Windows, macOS)

variables:
  buildConfiguration: 'Release'

stages:

# Stage 1: Build and Test
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - task: UseDotNet@2
      inputs:
        version: '8.0.x'

    - script: dotnet restore
      displayName: 'Restore packages'

    - script: dotnet build --configuration $(buildConfiguration)
      displayName: 'Build'

    - script: dotnet test --configuration $(buildConfiguration) --logger trx
      displayName: 'Run tests'

    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: '**/*.trx'

    - task: DotNetCoreCLI@2
      displayName: 'Publish'
      inputs:
        command: publish
        publishWebProjects: true
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'

    - task: PublishBuildArtifacts@1
      inputs:
        pathToPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: 'drop'

# Stage 2: Deploy to Staging
- stage: DeployStaging
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployToStaging
    environment: Staging
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyServiceConnection'
              appName: 'MyWebApp-Staging'
              package: '$(Pipeline.Workspace)/drop/*.zip'

# Stage 3: Deploy to Production (with approval)
- stage: DeployProd
  dependsOn: DeployStaging
  jobs:
  - deployment: DeployToProduction
    environment: Production   # Has approval gate configured
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'MyServiceConnection'
              appName: 'MyWebApp-Production'
              package: '$(Pipeline.Workspace)/drop/*.zip'
```

### Pipeline Triggers
```yaml
# Push trigger
trigger:
  branches: [main, develop]
  paths:
    include: ['src/**']    # Only trigger for changes in src/

# Pull Request trigger
pr:
  branches: [main]

# Scheduled trigger
schedules:
- cron: "0 2 * * *"       # Every night at 2 AM
  displayName: Nightly Build
  branches:
    include: [main]
```

### Pipeline Variables
```yaml
variables:
  myVar: 'hello'                          # Static variable
  $(Build.BuildId)                        # Built-in: build number
  $(Build.SourceBranchName)               # Built-in: branch name
  $(System.DefaultWorkingDirectory)       # Built-in: root folder

# Secret variables (set in Azure DevOps UI, not in YAML)
# Reference them: $(mySecretVar) — value is masked in logs
```

---

## 10.13 Azure Test Plans

### Testing Types
| Type | Description |
|---|---|
| **Unit Test** | Test a single method/class in isolation |
| **Integration Test** | Test how multiple components work together |
| **Manual Test** | Human tester follows test cases step by step |
| **Load Test** | Simulate many concurrent users (Azure Load Testing) |

### Test Plans in Azure DevOps
```
Test Plan (e.g., "Sprint 5 Regression")
  └── Test Suite (e.g., "Login Feature")
        ├── Test Case 1: Valid login shows dashboard
        ├── Test Case 2: Invalid password shows error
        └── Test Case 3: Locked account shows message
```

### Running Tests in Pipeline
```yaml
- script: dotnet test MyProject.Tests --logger "trx;LogFileName=results.trx" --collect:"XPlat Code Coverage"
  displayName: 'Run unit tests'

- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'VSTest'
    testResultsFiles: '**/*.trx'

- task: PublishCodeCoverageResults@1
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

---

## 🎯 Interview Questions for Module 10

| # | Question |
|---|---|
| 1 | What is the cloud? What are the 5 characteristics of cloud computing? |
| 2 | What is the difference between IaaS, PaaS, and SaaS? Give Azure examples. |
| 3 | What is CapEx vs OpEx? How does cloud help with this? |
| 4 | What is an Azure Region, Availability Zone, and Region Pair? |
| 5 | What is a Resource Group in Azure? |
| 6 | What is the difference between Azure VM and Azure App Service? |
| 7 | What is a Deployment Slot? How do you do a zero-downtime deployment? |
| 8 | What is an Azure Function? What triggers are available? |
| 9 | What is AKS? How is it different from running Docker on a VM? |
| 10 | What is the difference between Azure Service Bus and Event Grid? |
| 11 | What is Azure Active Directory? What is a Tenant? What is RBAC? |
| 12 | What is CI/CD? What is the difference between CI and CD? |
| 13 | What are the 5 Azure DevOps services? |
| 14 | What is a YAML pipeline in Azure DevOps? What is a trigger? |
| 15 | What is the difference between a User Story, Task, and Bug in Azure Boards? |
| 16 | What is a Sprint? Explain the Scrum process. |
| 17 | What is Cosmos DB? When would you choose it over Azure SQL? |
| 18 | What is an ARM Template? What is it used for? |

---

*[← Module 9](./Module9_Docker.md) | [Back to README](./README.md)*
