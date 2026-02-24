# 📗 Module 10: Cloud & DevOps Basics using Azure

> **Goal:** Simple, interview-ready definitions for every topic. Theory first, tables for quick recall, common interview Q&A at the end.

---

## 10.1 Introduction to the Cloud

### ❓ What is the Cloud?
The **Cloud** is the delivery of computing services (servers, storage, databases, networking, software) over the **internet** — instead of owning and managing physical hardware.

> _"Instead of buying a server, you rent computing power from Microsoft/Amazon/Google and pay only for what you use."_

---

### ⭐ 5 Characteristics of Cloud (must know!)

| Characteristic | Simple Meaning |
|---|---|
| **On-demand self-service** | Spin up resources anytime, no manual approval needed |
| **Broad network access** | Access from anywhere via internet |
| **Resource pooling** | Resources are shared across many customers (multi-tenant) |
| **Rapid elasticity** | Scale up/down instantly based on demand |
| **Measured service** | Pay only for what you use (like electricity) |

---

### CapEx vs OpEx

| | CapEx | OpEx |
|---|---|---|
| **Full form** | Capital Expenditure | Operational Expenditure |
| **Meaning** | Upfront investment in physical assets | Ongoing pay-as-you-go costs |
| **Example** | Buy a server for $50,000 | Pay $500/month Azure bill |
| **Cloud?** | ❌ Traditional IT | ✅ Cloud model |

> **Interview tip:** Cloud converts CapEx to OpEx — no upfront hardware cost, just monthly bills.

---

### IaaS, PaaS, SaaS

| Model | Full Form | You Manage | Provider Manages | Azure Example |
|---|---|---|---|---|
| **IaaS** | Infrastructure as a Service | OS, App, Data | Hardware, network, virtualization | Azure VMs |
| **PaaS** | Platform as a Service | App & Data only | OS, runtime, hardware | Azure App Service, Azure SQL |
| **SaaS** | Software as a Service | Nothing — just use it | Everything | Office 365, Azure DevOps |

**Pizza Analogy:**
```
IaaS = You rent kitchen + oven. You cook from scratch.
PaaS = Dough + sauce provided. You add toppings.
SaaS = Pizza delivered. You just eat it.
```

**Interview Answer:** _"IaaS gives you a VM — you manage everything above hardware. PaaS (App Service, Azure SQL) lets you deploy code/data while Azure manages OS and runtime. SaaS (Office 365) is fully managed — you just use the software."_

---

### Types of Cloud

| Type | Description | Use Case |
|---|---|---|
| **Public Cloud** | Shared infra managed by provider | Most businesses (Azure, AWS, GCP) |
| **Private Cloud** | Dedicated infra for one org | Banks, government |
| **Hybrid Cloud** | Mix of public + private | Sensitive data on-prem, rest in cloud |
| **Multi-cloud** | Use multiple providers | Avoid vendor lock-in |

---

### Main Cloud Providers

| Provider | Cloud Name |
|---|---|
| Microsoft | **Azure** |
| Amazon | **AWS** (Amazon Web Services) |
| Google | **GCP** (Google Cloud Platform) |

---

## 10.2 Introduction to Azure

### What is Azure?
> **Microsoft Azure** is Microsoft's cloud platform with 200+ services — compute, storage, networking, databases, AI, DevOps, and more.

---

### Regions and Availability Zones

| Term | Definition |
|---|---|
| **Region** | A geographical area with one or more Azure data centers (e.g., East US, West Europe) |
| **Availability Zone (AZ)** | Physically separate data centers **within** a region — for high availability (99.99% SLA) |
| **Region Pair** | Two regions paired for disaster recovery (e.g., East US ↔ West US) |

> **Rule:** Choose the region **closest to your users** for low latency. Use Availability Zones for fault tolerance.

---

### Azure Portal
> Web-based UI at **portal.azure.com** — create, manage, and monitor all Azure resources.

---

### Azure CLI & PowerShell
> Command-line tools to manage Azure resources via scripts — used for automation.
- **Azure CLI** (`az`) — cross-platform, works on Linux/Mac/Windows
- **Azure PowerShell** (`Az` module) — Windows-friendly, integrates with PowerShell scripts

---

### Account → Subscription → Resource Group

```
Azure Account
  └── Subscription (billing unit — e.g., "Dev" or "Production")
        └── Resource Group (logical container — group related resources)
              ├── VM
              ├── Azure SQL Database
              └── Storage Account
```

---

### Finding / Creating / Removing a Resource
| Action | How |
|---|---|
| **Create** | Portal → "Create a resource" → search → fill form → Create |
| **Find** | Portal → search bar at top, or browse by service |
| **Remove** | Open resource → Delete, OR delete the whole Resource Group |

> **Tip:** Deleting a **Resource Group** deletes all resources inside it — useful for cleanup.

---

## 10.3 Azure Basic Concepts

### Resource Groups
> A **logical container** that groups related Azure resources together.
> Resources in a group share the same **lifecycle** — deploy, update, and delete together.

**Interview Answer:** _"A resource group is a container for related Azure resources. If I have a web app, its database, and storage account all for the same project, I put them in one resource group so I can manage, monitor, and delete them together."_

---

### Storage Accounts

| Type | Use Case |
|---|---|
| **Blob Storage** | Files, images, videos, backups (any unstructured data) |
| **Table Storage** | NoSQL key-value data |
| **Queue Storage** | Message queues between services |
| **File Storage** | Shared network files (like a network drive) |

---

### SLA (Service Level Agreement)
> Azure's **guarantee of uptime** for each service.

| SLA | Downtime per month |
|---|---|
| 99% | ~7.2 hours |
| 99.9% | ~43 minutes |
| 99.95% | ~21 minutes |
| 99.99% | ~4 minutes |

---

### Cost in Azure
- Pay for compute by the **second/minute**
- Storage by **GB/month**
- Network by **data transferred out**
- Use **Azure Cost Calculator** to estimate before creating resources
- Set **budgets and alerts** to avoid surprise bills

---

## 10.4 Azure Compute

### Virtual Machines (VMs)
> An **IaaS** service — gives you a full virtual computer in the cloud. You choose the OS, install software, and manage everything above the hardware.

**When to use:** Legacy apps, full OS control needed, custom software that can't run on PaaS.

---

### ARM Template (Infrastructure as Code)
> A **JSON file** that defines Azure resources so you can deploy the same infrastructure consistently every time.
> Think of it as a "recipe" for your Azure environment.

**Interview Answer:** _"ARM templates are JSON files that describe your Azure infrastructure as code — you can version-control them, deploy the same environment repeatedly, and avoid manual mistakes."_

---

### App Service (PaaS Web Hosting)
> Fully managed **PaaS** hosting for web apps — you just deploy your code, Azure manages the OS, runtime, patching, and scaling.

**When to use:** Web apps, REST APIs (.NET, Node.js, Python, Java, etc.)

#### App Service Tiers

| Tier | Use Case | Key Feature |
|---|---|---|
| **Free (F1)** | Dev/test | No custom domain |
| **Basic (B1-B3)** | Low traffic | Custom domain, manual scale |
| **Standard (S1-S3)** | Production | Auto-scale, deployment slots |
| **Premium (P1-P3)** | High traffic | More power, VNet integration |

---

### Auto Scaling
> Automatically **increase or decrease** the number of app instances based on CPU, memory, or request count.
- **Scale out** = add more instances (horizontal)
- **Scale in** = remove instances
- Rules: _"If CPU > 70% for 5 minutes → add 1 instance"_

---

### Deployment Slots
> **Staging environments** within the same App Service.
> Deploy to "staging", test it, then **swap** staging ↔ production with **zero downtime**.

```
Deploy to staging → Test → Swap → Production updated instantly
```

**Interview Answer:** _"Deployment slots let you deploy to a staging environment first, test it, then swap to production with zero downtime. If something goes wrong, you can swap back immediately."_

---

### Deployment Types

| Type | Meaning |
|---|---|
| **Zip Deploy** | Upload a `.zip` of your published app |
| **Git Deploy** | Push directly from a git repo |
| **Azure DevOps Pipeline** | Automated CI/CD deployment |
| **Container Deploy** | Deploy a Docker container image |

---

### Azure Functions (Serverless)
> Run **small pieces of code** without managing any servers. You pay only when the function actually executes.

**Triggers (what starts a function):**

| Trigger | When it fires |
|---|---|
| **HttpTrigger** | HTTP request hits a URL |
| **TimerTrigger** | Scheduled (like a cron job) |
| **BlobTrigger** | File uploaded to Blob Storage |
| **QueueTrigger** | Message arrives in a queue |
| **EventGridTrigger** | An Azure event is published |

**Interview Answer:** _"Azure Functions are serverless — you write a function, configure a trigger, and Azure runs it when the trigger fires. No servers to manage, and you pay per execution (very cheap for infrequent tasks)."_

---

### AKS (Azure Kubernetes Service)
> **Managed Kubernetes** in Azure — Azure manages the Kubernetes control plane; you manage your containerized workloads.

**Interview Answer:** _"AKS is managed Kubernetes. Instead of setting up and maintaining Kubernetes yourself, Azure handles the control plane. You just deploy your container workloads."_

---

### Containers & Docker (quick recap)
> A **container** packages your app and all its dependencies into a portable unit that runs consistently anywhere.
- **Docker** = tool to build and run containers
- **Azure Container Registry (ACR)** = Azure's private registry to store your Docker images
- **AKS** = run containers at scale with Kubernetes

---

### How to Choose Compute Type?

| Scenario | Use |
|---|---|
| Full OS control, custom software | **Azure VM (IaaS)** |
| Web app / REST API, just deploy code | **App Service (PaaS)** |
| Small functions, event-driven, infrequent | **Azure Functions (Serverless)** |
| Containerized microservices at scale | **AKS** |

---

## 10.5 Azure Networking

### Virtual Networks (VNets)
> A **private network** in Azure — your resources communicate securely inside it, isolated from the public internet.

```
VNet (10.0.0.0/16)
  ├── Subnet 1: Web tier   → App Service / VMs (public-facing)
  ├── Subnet 2: App tier   → APIs (internal)
  └── Subnet 3: Data tier  → SQL, Redis (no internet access)
```

---

### Subnets
> Subdivisions of a VNet. Separate resources into **tiers** for security and organization.

---

### Secure VM Access
| Method | Description |
|---|---|
| **Bastion** | Secure RDP/SSH in browser — no public IP needed on VM |
| **Just-in-Time (JIT) Access** | Open port only when needed, auto-close after |
| **NSG (Network Security Group)** | Firewall rules to allow/deny traffic |

---

### Service Endpoint vs Private Link

| | Service Endpoint | Private Link |
|---|---|---|
| **Traffic** | Stays in Azure backbone | Stays in your VNet |
| **Access** | Any resource of that type | Specific resource only |
| **Public IP** | Service still has public IP | Service gets a private IP |
| **Cost** | Free | Has a cost |

---

### Load Balancer vs Application Gateway

| Service | OSI Layer | Use Case |
|---|---|---|
| **Load Balancer** | Layer 4 (TCP/UDP) | Distribute traffic to VMs |
| **Application Gateway** | Layer 7 (HTTP/HTTPS) | Route by URL, SSL termination, WAF |

**Interview tip:** _Load Balancer = network traffic. Application Gateway = web traffic with smarter rules._

---

## 10.6 Data in Azure

### Azure SQL Database
> Fully managed **SQL Server** in the cloud — no patching, backups, or HA setup needed. Just create and query.

**When to use:** Relational data, transactions, structured schema.

---

### Cosmos DB
> Fully managed **NoSQL** database — globally distributed, supports multiple APIs (SQL, MongoDB, Cassandra).

---

### SQL vs NoSQL (Azure SQL vs Cosmos DB)

| | Azure SQL | Cosmos DB |
|---|---|---|
| **Type** | Relational | NoSQL (document, key-value) |
| **Schema** | Fixed | Flexible / schemaless |
| **Scalability** | Vertical | Horizontal (unlimited) |
| **Query** | T-SQL | SQL API, MongoDB API, etc. |
| **SLA** | 99.99% | 99.999% |
| **Use case** | Transactions, structured data | Global scale, flexible data |

**Interview Answer:** _"Azure SQL is a managed SQL Server — use it for relational data with transactions. Cosmos DB is NoSQL — use it when you need global distribution, flexible schema, or massive scale."_

---

### Azure Storage (Blob)
> Store any file (images, PDFs, backups) in the cloud at low cost.
- **SAS URL** = a temporary, time-limited link to a blob — share without giving your account key

---

### CDN (Content Delivery Network)
> Caches static content (images, CSS, JS) on **edge servers** close to users worldwide — faster load times.

---

## 10.7 Messaging in Azure

### Why Messaging?
> Messaging **decouples** services — Service A puts a message in a queue; Service B processes it later. Neither needs to be running at the same time.

---

### Messaging Comparison

| Service | Type | Best For |
|---|---|---|
| **Storage Queue** | Simple queue | Basic decoupling, cheap, low volume |
| **Service Bus Queue** | Enterprise queue | Ordered delivery, retry, dead-letter |
| **Service Bus Topic** | Pub/Sub | Multiple subscribers to one message |
| **Event Grid** | Event routing | React to Azure resource events (blob created, VM stopped) |
| **Event Hubs** | Event streaming | High-throughput (millions/sec), IoT, analytics |

---

### Storage Queue
> Simplest queue. FIFO, up to 64KB per message. Cheap and easy.

---

### Service Bus
> **Enterprise-grade** message broker.
- **Queue** = one sender, one receiver (point-to-point)
- **Topic + Subscription** = one sender, **multiple receivers** (publish-subscribe)
- Features: guaranteed delivery, ordering, dead-letter queue, retry policy

---

### Event Grid
> Publish-Subscribe for **events** (not messages). Reacts to things that happen in Azure.
```
Blob file uploaded → Event Grid → Azure Function → process the file
VM stopped → Event Grid → Logic App → send alert email
```

---

### Event Hubs
> Designed for **high-throughput event streaming** — millions of events per second.
- Use case: IoT telemetry, clickstream data, log aggregation

---

## 10.8 Azure Active Directory (AAD / Entra ID)

### Key Concepts

| Term | Definition |
|---|---|
| **Tenant** | Your organization's instance of AAD (e.g., `cognizant.onmicrosoft.com`) |
| **User** | A person with an identity in the tenant |
| **Group** | Collection of users (e.g., Developers, Admins) |
| **App Registration** | Registers your app with AAD for OAuth/OIDC authentication |
| **Service Principal** | Identity for an application or automated service |
| **MFA** | Multi-Factor Authentication — password + second factor (phone/authenticator app) |
| **RBAC** | Role-Based Access Control — assign permissions via roles, not individually |

---

### Azure Roles (RBAC)

| Role | What you can do |
|---|---|
| **Owner** | Full control, including assigning access to others |
| **Contributor** | Create and manage resources, but can't manage access |
| **Reader** | View resources only, no changes |
| **Custom Role** | Define exactly which permissions are needed |

**Interview Answer:** _"RBAC in Azure lets you assign roles (Owner, Contributor, Reader) to users or groups at different scopes (subscription, resource group, or individual resource). This follows the principle of least privilege."_

---

### OAuth 2.0 + JWT Flow
```
1. User logs in → Azure AD authenticates
2. Azure AD issues JWT access token
3. App sends token in every API call: Authorization: Bearer <token>
4. API validates token with Azure AD
```

---

## 10.9 Azure DevOps Overview

### What is Azure DevOps?
> A set of **cloud-based developer tools** that cover the entire software delivery lifecycle — from planning to deployment.

---

### Key Components (5 services)

| Service | Purpose |
|---|---|
| **Azure Boards** | Agile project management — User Stories, Tasks, Bugs, Sprints |
| **Azure Repos** | Git source control — host and manage code |
| **Azure Pipelines** | CI/CD — automate build, test, and deploy |
| **Azure Test Plans** | Manual and automated testing |
| **Azure Artifacts** | Package management — NuGet, npm, Maven feeds |

**Interview Answer:** _"Azure DevOps has 5 services: Boards for planning, Repos for source control, Pipelines for CI/CD, Test Plans for testing, and Artifacts for package management."_

---

## 10.10 Azure Repos (Git)

### What is Azure Repos?
> Azure's **Git hosting** service — like GitHub but inside Azure DevOps. Supports pull requests, branch policies, and code reviews.

---

### Git Basics (must know commands)

| Command | What it does |
|---|---|
| `git init` | Create a new local repo |
| `git clone <url>` | Copy a remote repo locally |
| `git add .` | Stage all changed files |
| `git commit -m "msg"` | Save snapshot of staged changes |
| `git push origin main` | Upload commits to remote |
| `git pull` | Fetch and merge latest from remote |
| `git checkout -b feature/x` | Create and switch to new branch |
| `git merge feature/x` | Merge a branch into current branch |
| `git log --oneline` | Short history of commits |

---

### Branching Strategies

| Strategy | Description |
|---|---|
| **Feature Branch** | Each feature in its own branch → merge via PR |
| **GitFlow** | main + develop + feature/release/hotfix branches |
| **Trunk-Based Development** | Everyone commits to main, use feature flags |

**Common flow:**
```
main ← production
  └── develop ← integration
        ├── feature/login
        └── bugfix/null-check
```

---

### Pull Request (PR)
> A request to **merge** your branch into another. Team reviews the code before it merges.

**PR Best Practices:**
- Keep PRs small (< 400 lines)
- Write a clear description
- All tests must pass
- At least 1 reviewer approval
- Use **branch policies** to enforce these rules automatically

---

### Merging Strategies

| Strategy | Description |
|---|---|
| **Merge Commit** | Keeps full branch history |
| **Squash Merge** | Combines all commits into one clean commit |
| **Rebase** | Replays commits on top of target branch (linear history) |

---

## 10.11 Azure Boards

### What is Azure Boards?
> Agile project management tool — track work items (stories, tasks, bugs) with boards, backlogs, and sprints.

---

### Work Item Hierarchy

```
Epic (large goal)
  └── Feature (capability)
        └── User Story ("As a user, I want to...")
              └── Task (specific technical work)
              └── Bug (defect to fix)
```

| Type | Example |
|---|---|
| **Epic** | "User Management Module" |
| **Feature** | "User Login" |
| **User Story** | "As a user, I want to log in so I can access my data" |
| **Task** | "Create JWT authentication endpoint" |
| **Bug** | "Login fails when email has uppercase letters" |

---

### Agile & Scrum

| Term | Definition |
|---|---|
| **Sprint** | Time-boxed work period, usually 2 weeks |
| **Product Backlog** | All upcoming work, prioritized by product owner |
| **Sprint Backlog** | Work committed for this sprint |
| **Daily Standup** | 15-min daily: What did I do? What will I do? Blockers? |
| **Sprint Review** | Demo completed work to stakeholders at end of sprint |
| **Sprint Retrospective** | Team reflects: what went well, what to improve |
| **Velocity** | How many story points a team completes per sprint |
| **Story Point** | Relative effort estimate (1, 2, 3, 5, 8, 13...) |

---

### Kanban Board
> Visual board showing work item states:
```
To Do → In Progress → In Review → Done
```

---

## 10.12 Azure Pipelines — CI/CD

### What is CI/CD?

| Term | Full Form | Meaning |
|---|---|---|
| **CI** | Continuous Integration | Every commit auto-triggers: build → test. Catch broken code immediately. |
| **CD** | Continuous Delivery | After CI, auto-deploy to staging. Human approves production. |
| **CD** | Continuous Deployment | Fully automated including production — no manual step. |

**Why CI/CD?**
> Without it, deployments are manual, error-prone, and infrequent (big, scary releases).
> With CI/CD, every commit is automatically built, tested, and deployed — small, safe, frequent releases.

**Interview Answer:** _"CI/CD automates the entire build-test-deploy cycle. Every time I push code, the pipeline builds it and runs all tests automatically. If everything passes, it deploys to staging. Production deployment can require a manual approval gate. This replaces slow, risky manual deployments."_

---

### Building Blocks of a Pipeline

| Concept | Definition |
|---|---|
| **Pipeline** | The entire automated workflow |
| **Stage** | A phase of the pipeline (Build, Test, Deploy) |
| **Job** | A unit of work that runs on an agent |
| **Step** | An individual task or script in a job |
| **Agent** | The machine that runs the pipeline (hosted by Azure or self-hosted) |
| **Artifact** | The output of the build stage (e.g., compiled zip file) |
| **Trigger** | What starts the pipeline (push, PR, schedule) |

---

### YAML vs Visual Designer

| | YAML | Visual Designer |
|---|---|---|
| **Format** | Code (`.yml` file in repo) | Drag-and-drop UI |
| **Version control** | ✅ Yes — tracked in git | ❌ No |
| **Preferred** | ✅ Industry standard | Legacy / beginners |

> **Always prefer YAML** — it's version-controlled, reviewable in PRs, and portable.

---

### Triggers

| Trigger Type | When it runs |
|---|---|
| **Push trigger** | When code is pushed to a branch |
| **PR trigger** | When a pull request is created/updated |
| **Scheduled trigger** | On a cron schedule (e.g., every night at 2 AM) |
| **Manual trigger** | Someone clicks "Run pipeline" in the UI |

---

### Agent Pools

| Type | Description |
|---|---|
| **Microsoft-hosted** | Azure provides the VM (Ubuntu, Windows, macOS) — clean each run |
| **Self-hosted** | You manage the VM — use when you need custom software or faster builds |

---

### YAML Pipeline Structure (simplified)

```yaml
trigger:
  branches:
    include: [main]       # Run when pushed to main

pool:
  vmImage: ubuntu-latest  # Agent machine

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - script: dotnet restore
          - script: dotnet build
          - script: dotnet test

  - stage: DeployStaging
    dependsOn: Build
    jobs:
      - deployment: DeployToStaging
        environment: Staging    # Can have approval gates
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: "MyApp-Staging"

  - stage: DeployProd
    dependsOn: DeployStaging
    jobs:
      - deployment: DeployToProduction
        environment: Production  # Requires manual approval
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    appName: "MyApp-Production"
```

---

### Pipeline Variables

| Type | Description |
|---|---|
| **Static variable** | Defined in YAML, visible to all |
| **Built-in variable** | Provided by Azure (e.g., `$(Build.BuildId)`, `$(Build.SourceBranchName)`) |
| **Secret variable** | Set in UI, never shown in logs — used for passwords, API keys |

---

## 10.13 Advanced Azure Pipelines

### Multi-Stage Pipelines
> Split the pipeline into **stages**: Build → Test → Deploy Staging → Deploy Production.
> Each stage can depend on the previous one passing.

---

### Environments and Approvals
> An **Environment** in Azure DevOps represents a deployment target (Staging, Production).
> You can add **approval gates** — a specific person must approve before the stage runs.

```
Build passes → Auto-deploy to Staging → Manual approval → Deploy to Production
```

---

### Deployment Strategies

| Strategy | Description | Use Case |
|---|---|---|
| **runOnce** | Deploy once to all targets | Simple deployments |
| **Rolling** | Replace old instances one at a time | Minimize downtime |
| **Blue-Green** | Two identical environments, swap traffic | Zero downtime, instant rollback |
| **Canary** | Send small % of traffic to new version first | Test in production safely |

---

### Pipeline Templates
> Reusable YAML snippets — define steps once, reference from multiple pipelines.
> Avoids copy-pasting the same build steps across 10 pipelines.

---

### Dependency Management
> Use `dependsOn` in YAML to control stage/job order:
```yaml
- stage: DeployProd
  dependsOn: DeployStaging     # Only runs if DeployStaging succeeded
  condition: succeeded()
```

---

## 10.14 Azure Release Pipelines

### What is a Release Pipeline?
> The **classic** (non-YAML) way to define deployments in Azure DevOps. Still used in many projects.
> You pick an artifact (build output) and define deployment stages with gates.

---

### Deployment Environments
> Each stage in a release pipeline targets an environment: Dev → QA → Staging → Production.

---

### Deployment Gates and Approvals
| Feature | Description |
|---|---|
| **Pre-deployment approval** | A person must approve before stage starts |
| **Post-deployment approval** | A person confirms deployment was successful |
| **Gates** | Automated checks (e.g., no active incidents, all tests passing) before proceeding |

---

### Variables and Configuration
> Store environment-specific values (connection strings, API URLs) as pipeline variables — different value per stage, secrets stored securely.

---

## 10.15 Azure Test Plans

### What is Azure Test Plans?
> Azure DevOps service for **manual and automated testing** — organize test cases, track results, integrate with CI pipelines.

---

### Test Plans Structure

```
Test Plan (e.g., "Sprint 5 Regression")
  └── Test Suite (e.g., "Login Feature")
        ├── Test Case: Valid login → redirects to dashboard
        ├── Test Case: Wrong password → shows error message
        └── Test Case: Locked account → shows locked message
```

---

### Types of Testing

| Type | Description |
|---|---|
| **Unit Test** | Test a single method/class in isolation |
| **Integration Test** | Test how multiple components work together |
| **Manual Test** | Tester follows test cases step by step |
| **Automated Test** | Code runs tests — integrated into pipeline |
| **Load Test** | Simulate many concurrent users (Azure Load Testing) |

---

### Automated Testing in Pipeline
> Run unit tests automatically on every commit:
```yaml
- script: dotnet test --logger trx
- task: PublishTestResults@2      # Shows test results in pipeline UI
- task: PublishCodeCoverageResults@1  # Shows code coverage %
```

**Interview Answer:** _"In Azure Test Plans, you create test plans with test cases for manual testing. For automated testing, you integrate `dotnet test` into the pipeline and publish results — so every build shows pass/fail stats and code coverage."_

---

## 🎯 Top Interview Questions — Module 10

### Cloud Basics
| # | Question | Key Answer |
|---|---|---|
| 1 | What is the cloud? | Delivery of computing services over the internet, pay-as-you-go |
| 2 | What are the 5 characteristics of cloud? | On-demand, broad access, resource pooling, elasticity, measured service |
| 3 | What is CapEx vs OpEx? | CapEx = buy hardware upfront; OpEx = monthly cloud bill. Cloud = OpEx |
| 4 | Difference between IaaS, PaaS, SaaS? | IaaS = VM (you manage OS); PaaS = App Service (you manage code); SaaS = Office 365 (just use it) |
| 5 | Types of clouds? | Public, Private, Hybrid, Multi-cloud |

### Azure Services
| # | Question | Key Answer |
|---|---|---|
| 6 | What is a Region and Availability Zone? | Region = geo area with data centers; AZ = separate DCs within a region for HA |
| 7 | What is a Resource Group? | Logical container for related resources that share same lifecycle |
| 8 | VM vs App Service? | VM = IaaS (full control); App Service = PaaS (just deploy code) |
| 9 | What is a Deployment Slot? | Staging env; swap to prod with zero downtime |
| 10 | What is Azure Functions? | Serverless; runs on triggers (HTTP, Timer, Queue, Blob) |
| 11 | What is AKS? | Managed Kubernetes — you run containers, Azure manages control plane |
| 12 | Azure SQL vs Cosmos DB? | SQL = relational/transactions; Cosmos DB = NoSQL/global scale/flexible schema |
| 13 | Service Bus vs Event Grid? | Service Bus = messaging (queues/topics, guaranteed delivery); Event Grid = event routing (react to Azure events) |
| 14 | What is RBAC in Azure? | Role-based access — assign Owner/Contributor/Reader roles to users/groups |

### Azure DevOps
| # | Question | Key Answer |
|---|---|---|
| 15 | What are the 5 Azure DevOps services? | Boards, Repos, Pipelines, Test Plans, Artifacts |
| 16 | What is CI/CD? | CI = auto build+test on commit; CD = auto deploy to staging, manual approval for prod |
| 17 | YAML vs Visual Designer? | YAML is preferred — version-controlled, in git, reviewable in PRs |
| 18 | What is an Environment with approval? | Deployment target with a gate — person must approve before stage runs |
| 19 | What is a Deployment Slot / Blue-Green? | Two identical environments, switch traffic with zero downtime |
| 20 | Difference between User Story, Task, Bug? | Story = user feature request; Task = technical work; Bug = defect |
| 21 | What is a Sprint? | Time-boxed work period (2 weeks); team commits to sprint backlog |
| 22 | What is a PR? Best practices? | Merge request; keep small, require reviews, pass tests, use branch policies |

---

_[← Module 9](./Module9_Docker.md) | [Back to README](./README.md)_
