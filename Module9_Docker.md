# 📗 Module 9: Docker Basics

> **Focus:** Containerize applications with Docker — from basic commands to multi-container apps with Docker Compose, and an intro to container orchestration.

---

## 9.1 What is Docker?

> **Docker** is a platform that packages an application and all its dependencies into a **container** — a lightweight, portable, self-contained unit that runs consistently anywhere.

**The Problem Docker Solves:**
```
Dev says: "It works on my machine!"
Docker says: "Ship your machine."
```

```
Without Docker:
  Dev Machine:  .NET 8, SQL Server 2022, Redis 7 → Works ✅
  Production:   .NET 6, SQL Server 2019, Redis 5 → Crashes ❌

With Docker:
  Container includes: app + .NET 8 + correct config → Works everywhere ✅
```

---

## 9.2 Docker vs Virtual Machine

| Feature | Virtual Machine (VM) | Docker Container |
|---|---|---|
| **Virtualizes** | Entire OS | Only application and dependencies |
| **Size** | GBs (full OS image) | MBs (just app + libs) |
| **Boot time** | Minutes | Seconds |
| **Performance** | Slower (hypervisor overhead) | Faster (native OS) |
| **Isolation** | Complete OS isolation | Process-level isolation |
| **Use case** | Run different OS (Linux on Windows) | Package and run applications |

```
VM:           [App][App][App]
              [OS] [OS] [OS]
              [  Hypervisor  ]
              [   Hardware   ]

Docker:       [App][App][App]
              [Lib][Lib][Lib]
              [  Docker Engine]
              [  Host OS     ]
              [  Hardware    ]
```

**Interview Answer:** "A VM virtualizes the full OS — heavy and slow to start. A Docker container shares the host OS kernel — lightweight, starts in seconds. Docker is ideal for packaging applications; VMs are for running different operating systems."

---

## 9.3 Docker Editions

| Edition | Use Case | Price |
|---|---|---|
| **Docker Desktop** | Local development on Windows/Mac | Free for personal/small business |
| **Docker Engine** | Linux servers (no GUI) | Free, open source |
| **Docker Hub** | Public image registry | Free for public images |

---

## 9.4 Basic Docker Commands

```bash
# IMAGE COMMANDS
docker images                          # List all local images
docker pull nginx                      # Download image from Docker Hub
docker rmi nginx                       # Remove (delete) an image
docker rmi $(docker images -q)         # Remove ALL images

# CONTAINER COMMANDS
docker ps                              # List RUNNING containers
docker ps -a                           # List ALL containers (including stopped)
docker stop <container-id>             # Stop a running container
docker rm <container-id>               # Remove a stopped container
docker rm $(docker ps -aq)             # Remove ALL stopped containers
docker logs <container-id>             # See container logs
docker logs -f <container-id>          # Follow (tail) container logs

# EXECUTE INSIDE CONTAINER
docker exec -it <container-id> bash    # Open bash shell inside container
docker exec -it myapp sh               # Use sh if bash not available

# SYSTEM
docker system prune                    # Remove all unused resources
docker stats                           # Live CPU/memory usage of all containers
docker inspect <container-id>          # Detailed JSON info about container
```

---

## 9.5 Docker Run

```bash
# Basic run
docker run nginx                       # Run nginx (pulls if not local, runs in foreground)

# --name: give container a name
docker run --name mywebserver nginx

# -d: detached mode (background)
docker run -d nginx

# -p: publish ports  HOST:CONTAINER
docker run -d -p 8080:80 nginx         # Access via localhost:8080, maps to container port 80

# -e: environment variables
docker run -d -e ASPNETCORE_ENVIRONMENT=Production myapp

# --rm: auto-remove container when it stops
docker run --rm nginx

# -it: interactive terminal (for commands, not web apps)
docker run -it ubuntu bash

# -v: mount volume  HOST_PATH:CONTAINER_PATH
docker run -d -v C:/data:/app/data myapp

# Full example: run SQL Server
docker run -d \
  --name sqlserver \
  -e SA_PASSWORD=YourPassword123! \
  -e ACCEPT_EULA=Y \
  -p 1433:1433 \
  mcr.microsoft.com/mssql/server:2022-latest

# Append a command (override default CMD)
docker run ubuntu echo "Hello World"   # Just print and exit
docker run ubuntu ls /app              # List files and exit
```

---

## 9.6 Docker Images

### What is a Docker Image?
> A **Docker Image** is a read-only template (blueprint) for creating containers. It contains the app, runtime, libraries, and config.

### Image Layers
> Images are built in **layers** — each layer represents one instruction in the Dockerfile. Layers are **cached** and **shared** between images.

```
Layer 5:  COPY app files         ← app-specific (changes often)
Layer 4:  RUN dotnet restore     ← cached if .csproj unchanged
Layer 3:  COPY *.csproj          ← cached if .csproj unchanged
Layer 2:  FROM mcr.../sdk:8.0    ← base image (large, rarely changes)
Layer 1:  FROM mcr.../aspnet:8.0 ← runtime layer (shared)
```

### Dockerfile — Build Custom Image

```dockerfile
# syntax=docker/dockerfile:1

# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy and restore (cached if .csproj unchanged)
COPY ["MyApi.csproj", "."]
RUN dotnet restore

# Copy everything and build
COPY . .
RUN dotnet build -c Release -o /app/build

# Stage 2: Publish
FROM build AS publish
RUN dotnet publish -c Release -o /app/publish --no-restore

# Stage 3: Final image (small — no SDK, just runtime)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
EXPOSE 80
EXPOSE 443
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "MyApi.dll"]
```

### Build and Run the Image
```bash
# Build image (tag it as myapi:1.0)
docker build -t myapi:1.0 .

# Build with specific Dockerfile
docker build -t myapi:1.0 -f Dockerfile.prod .

# Run the image
docker run -d -p 5000:80 --name myapi-container myapi:1.0

# Check it works
curl http://localhost:5000/api/employees
```

### .dockerignore (like .gitignore)
```
bin/
obj/
*.user
.vs/
.git/
**/*.md
```

---

## 9.7 Docker Compose

### What is Docker Compose?
> **Docker Compose** lets you define and run **multi-container applications** with a single `docker-compose.yml` file. Perfect for local development.

**Without Compose:**
```bash
# Run 3 containers manually — tedious and error-prone
docker run -d --name sqlserver -e SA_PASSWORD=... -p 1433:1433 mssql/server
docker run -d --name redis -p 6379:6379 redis
docker run -d --name myapi -e Db=... --link sqlserver --link redis -p 5000:80 myapi:1.0
```

**With Compose:**
```bash
docker-compose up -d    # Start everything
docker-compose down     # Stop everything
```

### docker-compose.yml
```yaml
version: '3.8'

services:

  # Your API
  api:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__Default=Server=sqlserver;Database=EmployeeDB;User=sa;Password=YourPassword123!
      - ConnectionStrings__Redis=redis:6379
    depends_on:
      - sqlserver
      - redis
    volumes:
      - ./logs:/app/logs   # Mount logs folder

  # SQL Server
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      SA_PASSWORD: "YourPassword123!"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    volumes:
      - sqldata:/var/opt/mssql   # Persist data

  # Redis Cache
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  sqldata:   # Named volume — data survives container restart
```

### Docker Compose Commands
```bash
docker-compose up                  # Start (foreground)
docker-compose up -d               # Start (background/detached)
docker-compose up --build          # Rebuild images before starting
docker-compose down                # Stop and remove containers
docker-compose down -v             # Also remove volumes (wipes DB data)
docker-compose ps                  # List running services
docker-compose logs api            # Logs for specific service
docker-compose logs -f             # Follow all logs
docker-compose exec api bash       # Shell into running container
docker-compose restart api         # Restart one service
```

---

## 9.8 Docker Registry

### What is a Docker Registry?
> A **Docker Registry** stores and distributes Docker images. Think of it like GitHub but for Docker images.

| Registry | Description |
|---|---|
| **Docker Hub** | Default public registry (hub.docker.com) |
| **Azure Container Registry (ACR)** | Private registry in Azure |
| **Amazon ECR** | AWS private registry |
| **GitHub Container Registry** | GitHub's registry (ghcr.io) |
| **Self-hosted** | Run your own with `registry:2` image |

```bash
# Docker Hub
docker login                                     # Login to Docker Hub
docker tag myapi:1.0 myusername/myapi:1.0        # Tag for Docker Hub
docker push myusername/myapi:1.0                 # Push to Docker Hub
docker pull myusername/myapi:1.0                 # Pull from Docker Hub

# Azure Container Registry
az acr login --name myregistry
docker tag myapi:1.0 myregistry.azurecr.io/myapi:1.0
docker push myregistry.azurecr.io/myapi:1.0
```

---

## 9.9 Docker Engine

### Docker Engine Components
```
                    ┌─────────────────────────────┐
Your terminal       │         Docker Engine         │
docker commands ──► │                               │
                    │  ┌──────────┐  ┌───────────┐ │
                    │  │ Docker   │  │   REST    │ │
                    │  │  CLI     │─►│    API    │ │
                    │  └──────────┘  └─────┬─────┘ │
                    │                      │        │
                    │               ┌──────▼──────┐ │
                    │               │ Docker Daemon│ │
                    │               │  (dockerd)  │ │
                    │               └──────┬──────┘ │
                    └──────────────────────┼────────┘
                                           │
                    ┌──────────────────────▼────────┐
                    │    Containers & Images         │
                    └───────────────────────────────┘
```

| Component | Role |
|---|---|
| **Docker CLI** | Command-line tool you type commands into |
| **REST API** | Interface between CLI and daemon |
| **Docker Daemon (dockerd)** | Background service that manages containers, images, networks |
| **containerd** | Low-level container runtime (called by daemon) |

---

## 9.10 Docker Storage

### Storage Drivers
> Docker uses a storage driver to manage the container's read-write layer. The most common is `overlay2`.

### Data Persistence Options

| Type | Description | Use Case |
|---|---|---|
| **Volumes** | Managed by Docker in `/var/lib/docker/volumes` | Databases, persistent data |
| **Bind Mounts** | Mount a specific host folder into container | Dev — share source code |
| **tmpfs Mounts** | Stored in host memory (RAM) only | Temp/sensitive data |

```bash
# VOLUMES
docker volume create mydata              # Create named volume
docker volume ls                         # List volumes
docker volume inspect mydata            # Volume details
docker volume rm mydata                  # Remove volume

# Use volume in container
docker run -d -v mydata:/var/opt/mssql sqlserver

# BIND MOUNTS (use full path)
docker run -d -v C:/code/myapp:/app myapi   # Windows
docker run -d -v /home/user/app:/app myapi  # Linux

# Changing storage driver for container
docker run --storage-opt dm.basesize=20G myimage

# TMPFS (memory only)
docker run --tmpfs /app/temp myimage
```

---

## 9.11 Docker Networking

### Default Networks
```bash
docker network ls     # List all networks

NAME     DRIVER  SCOPE
bridge   bridge  local    ← Default for standalone containers
host     host    local    ← Container shares host network
none     null    local    ← No networking
```

| Network | Description |
|---|---|
| **bridge** | Default. Containers on same bridge can talk to each other. Need `-p` to expose to host. |
| **host** | Container uses the host's network directly. No port mapping needed. Linux only. |
| **none** | No network access at all. Maximum isolation. |
| **overlay** | Multi-host networking (for Docker Swarm / Kubernetes) |

```bash
# Create custom network
docker network create my-network

# Run containers on same network (can communicate by name)
docker run -d --name api --network my-network myapi
docker run -d --name db  --network my-network sqlserver

# api container can reach db container using hostname "db"
# Connection string: Server=db;Database=...

# Inspect a network
docker network inspect my-network

# Remove a network
docker network rm my-network
```

---

## 9.12 Container Orchestration

### What is Container Orchestration?
> **Container Orchestration** automates the deployment, scaling, management, and networking of containers across multiple hosts.

**Why is it needed?**
```
One container on one machine = Docker is enough ✅
10 containers across 5 servers = Need orchestration ✅
100 containers, auto-scaling, self-healing, rolling updates = Need Kubernetes ✅
```

### Benefits of Container Orchestration
- **Auto-scaling** — spin up more containers when traffic increases
- **Self-healing** — restart failed containers automatically
- **Load balancing** — distribute traffic across container instances
- **Rolling updates** — update containers with zero downtime
- **Service discovery** — containers find each other by name
- **Secret management** — manage passwords and keys centrally

### Docker vs Kubernetes

| Feature | Docker Swarm | Kubernetes (K8s) |
|---|---|---|
| **Complexity** | Simple | Complex but powerful |
| **Scaling** | Basic | Advanced (HPA, VPA) |
| **Ecosystem** | Small | Massive (CNCF) |
| **Learning curve** | Low | High |
| **Production use** | Small-medium | Enterprise standard |
| **Cloud support** | Limited | All major clouds (AKS, EKS, GKE) |

### Kubernetes Key Concepts

| Term | What it is |
|---|---|
| **Pod** | Smallest unit — one or more containers running together |
| **Deployment** | Manages replicas of pods (ensures N copies running) |
| **Service** | Network endpoint to access pods (stable IP/DNS name) |
| **Namespace** | Logical grouping of resources |
| **Node** | A server (VM or physical) that runs pods |
| **Cluster** | A set of nodes managed by Kubernetes |
| **kubectl** | CLI tool to manage Kubernetes |

```yaml
# deployment.yml — Deploy 3 replicas of the API
apiVersion: apps/v1
kind: Deployment
metadata:
  name: employee-api
spec:
  replicas: 3              # Run 3 instances
  selector:
    matchLabels:
      app: employee-api
  template:
    metadata:
      labels:
        app: employee-api
    spec:
      containers:
      - name: employee-api
        image: myregistry.azurecr.io/employee-api:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
---
apiVersion: v1
kind: Service
metadata:
  name: employee-api-service
spec:
  selector:
    app: employee-api
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer       # Expose to internet
```

```bash
# kubectl commands
kubectl apply -f deployment.yml         # Create/update resources
kubectl get pods                        # List pods
kubectl get deployments                 # List deployments
kubectl describe pod <pod-name>        # Details + events
kubectl logs <pod-name>                # Pod logs
kubectl scale deployment employee-api --replicas=5  # Scale up
kubectl rollout restart deployment/employee-api     # Rolling restart
kubectl delete -f deployment.yml        # Delete resources
```

---

## 🎯 Interview Questions for Module 9

| # | Question |
|---|---|
| 1 | What is Docker? What problem does it solve? |
| 2 | What is the difference between a Docker Image and a Container? |
| 3 | What is the difference between Docker and a Virtual Machine? |
| 4 | What is a Dockerfile? Explain multi-stage builds. |
| 5 | What is Docker Compose? When would you use it? |
| 6 | What are the three types of Docker storage? When do you use Volumes vs Bind Mounts? |
| 7 | What is the difference between bridge, host, and none networks in Docker? |
| 8 | What is a Docker Registry? What is Docker Hub? |
| 9 | What is container orchestration? Why is it needed? |
| 10 | What is Kubernetes? What is the difference between a Pod, Deployment, and Service? |
| 11 | What is Docker Engine made of? Explain the CLI, REST API, and Daemon. |
| 12 | What is the purpose of `.dockerignore`? |
| 13 | What does `-d` mean in `docker run -d`? What about `-p`? |

---

*[← Module 8](./Module8_Debugging.md) | [Back to README](./README.md) | [Next Module →](./Module10_Azure_DevOps.md)*
