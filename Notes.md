# 🐳 Docker — Complete Reference Guide
> From beginner fundamentals to advanced multi-container deployments.

---

## 📚 Table of Contents

1. [What is Docker?](#-what-is-docker)
2. [Core Concepts](#-core-concepts)
3. [Installation & System Check](#-installation--system-check)
4. [Image Management](#-image-management)
5. [Running Containers](#-running-containers)
6. [Container Lifecycle](#-container-lifecycle)
7. [Logs & Debugging](#-logs--debugging)
8. [Dockerfile — Building Custom Images](#-dockerfile--building-custom-images)
9. [Docker Networking](#-docker-networking)
10. [Volumes & Data Persistence](#-volumes--data-persistence)
11. [Bind Mounts](#-bind-mounts)
12. [Volume vs Bind Mount](#-volume-vs-bind-mount)
13. [MySQL Container Setup](#-mysql-container-setup)
14. [Two-Tier App (Backend + MySQL)](#-two-tier-app-backend--mysql)
15. [Docker Compose](#-docker-compose)
16. [Image Distribution](#-image-distribution)
17. [Cleanup](#-cleanup)
18. [Common Mistakes](#-common-mistakes)

---

## 🔍 What is Docker?

Docker is a platform that packages applications and their dependencies into isolated units called **containers**, ensuring they run consistently across any environment.

### Why Use Docker?

| Problem (Without Docker) | Solution (With Docker) |
|---|---|
| "It works on my machine" | Containers behave identically everywhere |
| Dependency conflicts between projects | Each container has its own isolated environment |
| Slow, heavy Virtual Machines | Containers share the host OS kernel — much lighter |
| Complex deployment steps | Ship a single image, run anywhere |

---

## 🧱 Core Concepts

| Term | Analogy | Description |
|---|---|---|
| **Image** | Class (in OOP) | A read-only blueprint used to create containers |
| **Container** | Object (instance) | A running instance of an image |
| **Dockerfile** | Recipe | A script of instructions to build a custom image |
| **Volume** | External hard drive | Persistent storage that survives container deletion |
| **Network** | LAN / private subnet | Allows containers to communicate with each other |
| **Docker Hub** | App Store / npm registry | Public registry for storing and sharing images |

---

## ⚙️ Installation & System Check

```bash
# Verify Docker is installed and the daemon is running
docker version

# Check system-wide Docker info (containers, images, storage driver)
docker info
```

---

## 📦 Image Management

```bash
# List all locally available images
docker images

# Download an image from Docker Hub
docker pull nginx
docker pull mysql:8          # Pull a specific version (tag)

# Search for images on Docker Hub
docker search ubuntu

# Remove a local image
docker rmi nginx

# Remove all unused images
docker image prune -a
```

> 💡 **Tip:** Always pin a specific image tag (e.g., `mysql:8`) in production. Avoid `latest` as it can break builds silently.

---

## 🚀 Running Containers

```bash
# Quick test — runs, prints output, and exits
docker run hello-world

# Full featured run command
docker run -d -p 5000:5000 --name myapp nginx
```

### Common `docker run` Flags

| Flag | Example | Purpose |
|---|---|---|
| `-d` | `-d` | Detached mode — runs in background |
| `-p` | `-p 8080:80` | Maps `host_port:container_port` |
| `--name` | `--name myapp` | Assigns a readable name to the container |
| `-e` | `-e MY_VAR=value` | Sets an environment variable |
| `-v` | `-v vol:/data` | Mounts a volume |
| `--network` | `--network mynet` | Connects container to a specific network |
| `--rm` | `--rm` | Auto-removes the container when it exits |
| `-it` | `-it` | Interactive terminal (for shells) |

---

## 🔄 Container Lifecycle

```bash
# List running containers
docker ps

# List ALL containers (including stopped)
docker ps -a

# Stop a running container (graceful shutdown)
docker stop <container_id or name>

# Force-kill a container immediately
docker kill <container_id or name>

# Start a stopped container
docker start <container_id or name>

# Restart a container
docker restart <container_id or name>

# Remove a stopped container
docker rm <container_id or name>

# Force remove a running container
docker rm -f <container_id or name>
```

---

## 🐛 Logs & Debugging

```bash
# View container output / error logs
docker logs <container_id or name>

# Stream logs in real-time (like tail -f)
docker logs -f <container_id or name>

# Open an interactive bash shell inside a running container
docker exec -it <container_id or name> bash

# Run a single command inside a container
docker exec <container_id> ls /app

# Inspect full container metadata (IP, mounts, env vars, etc.)
docker inspect <container_id or name>

# View real-time resource usage (CPU, memory)
docker stats
```

> 💡 **Tip:** If `bash` is not available in a container (e.g., Alpine Linux), try `sh` instead.

---

## 📝 Dockerfile — Building Custom Images

A `Dockerfile` is a plain text file with step-by-step instructions to build a Docker image.

### Example — Python Flask App

```dockerfile
# 1. Base image — start from an official Python image
FROM python:3.9

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy all project files into the container
COPY . .

# 4. Install dependencies
RUN pip install flask

# 5. Command to run when the container starts
CMD ["python", "app.py"]
```

### Build & Run

```bash
# Build an image from Dockerfile in current directory
docker build -t myapp .

# Build with a specific Dockerfile path
docker build -f Dockerfile.prod -t myapp:prod .

# Run the built image
docker run -d -p 5000:5000 myapp
```

### Key Dockerfile Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Sets the base image |
| `WORKDIR` | Sets the working directory |
| `COPY` | Copies files from host to container |
| `ADD` | Like COPY, but also supports URLs and tar extraction |
| `RUN` | Executes a command during image build |
| `CMD` | Default command when container starts (overridable) |
| `ENTRYPOINT` | Fixed command that always runs (CMD appends to it) |
| `EXPOSE` | Documents which port the container listens on |
| `ENV` | Sets environment variables |
| `ARG` | Build-time variables (not available at runtime) |

---

## 🌐 Docker Networking

By default, containers are **isolated** and cannot reach each other. You must explicitly put them on the same network.

```bash
# Create a custom bridge network
docker network create my-network

# List all networks
docker network ls

# Connect a running container to a network
docker network connect my-network myapp

# Disconnect a container from a network
docker network disconnect my-network myapp

# Inspect a network (see connected containers, subnet, etc.)
docker network inspect my-network

# Remove a network
docker network rm my-network
```

### Network Types

| Type | Use Case |
|---|---|
| `bridge` | Default. Isolated network for containers on one host |
| `host` | Container shares the host's network directly |
| `none` | No networking at all |
| `overlay` | For multi-host communication (Docker Swarm) |

> 💡 **Key Insight:** Within the same network, containers can reach each other using their **container name** as the hostname. For example, a backend container can connect to MySQL using `MYSQL_HOST=mysql` if the MySQL container is named `mysql`.

---

## 💾 Volumes & Data Persistence

**Problem:** All data inside a container is lost when the container is deleted.  
**Solution:** Volumes store data outside the container lifecycle.

```bash
# Create a named volume
docker volume create mysql-data

# List all volumes
docker volume ls

# Inspect a volume (see its mount path on the host)
docker volume inspect mysql-data

# Use a volume when running a container
docker run -d \
  --name mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8

# Remove a volume (only if no container is using it)
docker volume rm mysql-data

# Remove all unused volumes
docker volume prune
```

### ✅ Persistence Test — Prove It Works

```bash
# Step 1: Create a container with a volume and insert data
docker run -d --name mysql1 -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:8
docker exec -it mysql1 mysql -u root -p
# → INSERT some data here

# Step 2: Delete the container
docker rm -f mysql1

# Step 3: Start a NEW container using the same volume
docker run -d --name mysql2 -v mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root mysql:8

# Step 4: Verify data is still there
docker exec -it mysql2 mysql -u root -p
# → Your data is intact ✅
```

---

## 📂 Bind Mounts

A bind mount maps a **specific folder on your host machine** directly into the container.

```bash
# Linux / macOS
docker run -d --name mysql \
  -v /home/user/mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8

# Windows
docker run -d --name mysql \
  -v D:\mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  mysql:8
```

> 💡 **Use Case:** Great for development — changes to your local code are reflected inside the container instantly without rebuilding the image.

---

## ⚖️ Volume vs Bind Mount

| Feature | Named Volume | Bind Mount |
|---|---|---|
| Managed by Docker | ✅ Yes | ❌ No |
| Portable across machines | ✅ Yes | ❌ No (host path may differ) |
| Direct access from host | ❌ Indirect | ✅ Yes |
| Best for | Production data | Local development |
| CLI syntax | `-v volume-name:/path` | `-v /host/path:/container/path` |

---

## 🗄️ MySQL Container Setup

```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=mydb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=apppass \
  -p 3307:3306 \
  -v mysql-data:/var/lib/mysql \
  mysql:8
```

### Environment Variables Explained

| Variable | Required | Purpose |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | ✅ Yes | Sets the root password |
| `MYSQL_DATABASE` | Optional | Auto-creates a database on startup |
| `MYSQL_USER` | Optional | Creates a non-root user |
| `MYSQL_PASSWORD` | Optional | Password for `MYSQL_USER` |

### Connect to MySQL Shell

```bash
# Enter the MySQL interactive shell
docker exec -it mysql-db mysql -u root -p

# Run a single SQL command
docker exec -it mysql-db mysql -u root -proot123 -e "SHOW DATABASES;"
```

> 💡 **Port Note:** We use `3307:3306` to avoid conflicts if MySQL is already running locally on port 3306.

---

## 🏗️ Two-Tier App (Backend + MySQL)

This is a classic pattern: a web backend talks to a MySQL database, both running as containers on the same Docker network.

```bash
# Step 1: Create a shared network
docker network create two-tier

# Step 2: Run MySQL on the network
docker run -d \
  --name mysql \
  --network two-tier \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=devops \
  -v mysql-data:/var/lib/mysql \
  mysql:8

# Step 3: Run the Backend on the same network
docker run -d \
  --name backend \
  --network two-tier \
  -p 5000:5000 \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=root \
  -e MYSQL_DB=devops \
  two-tier-backend:latest
```

> 🔑 **Why `MYSQL_HOST=mysql` works:** Within the same Docker network, a container's **name** acts as its DNS hostname. The backend resolves `mysql` → the MySQL container's IP automatically.

---

## 🎼 Docker Compose

Docker Compose lets you define and manage **multi-container applications** using a single `docker-compose.yml` file — no need to run long `docker run` commands manually.

### Example `docker-compose.yml`

```yaml
version: "3.8"

services:

  mysql:
    image: mysql:8
    container_name: mysql-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: devops
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - two-tier

  backend:
    image: two-tier-backend:latest
    container_name: backend
    ports:
      - "5000:5000"
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: root
      MYSQL_DB: devops
    depends_on:
      - mysql
    networks:
      - two-tier

volumes:
  mysql-data:

networks:
  two-tier:
```

### Docker Compose Commands

```bash
# Start all services in detached mode
docker compose up -d

# Build images before starting
docker compose up -d --build

# View running services
docker compose ps

# View logs for all services
docker compose logs -f

# View logs for a specific service
docker compose logs -f backend

# Stop all services (keeps containers and volumes)
docker compose stop

# Stop and remove containers + networks
docker compose down

# Stop and remove containers, networks, AND volumes
docker compose down -v
```

---

## 🚢 Image Distribution

Share your custom images publicly via Docker Hub or privately via a registry.

```bash
# Step 1: Log in to Docker Hub
docker login

# Step 2: Tag your image with your Docker Hub username
docker image tag myapp yourusername/myapp:v1.0

# Step 3: Push to Docker Hub
docker push yourusername/myapp:v1.0

# Pull it on another machine
docker pull yourusername/myapp:v1.0
```

> 💡 **Tip:** Use version tags (`v1.0`, `v1.1`) instead of `latest` so you can roll back to a specific version if needed.

---

## 🧹 Cleanup

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# ☢️ Nuclear option — remove EVERYTHING unused (containers, images, networks, build cache)
docker system prune -a

# Also remove volumes (use with caution!)
docker system prune -a --volumes
```

---

## ⚠️ Common Mistakes

### ❌ Anonymous Volume (Data Loss Risk)
```bash
# WRONG — creates an anonymous volume with a random ID, hard to reuse
docker run -v mysql-data mysql

# CORRECT — maps the named volume to the internal storage path
docker run -v mysql-data:/var/lib/mysql mysql
```

### ❌ Containers Can't Reach Each Other
```bash
# WRONG — containers on the default bridge can't resolve each other by name
docker run --name backend myapp
docker run --name mysql mysql

# CORRECT — put them on the same named network
docker network create app-net
docker run --name backend --network app-net myapp
docker run --name mysql --network app-net mysql
```

### ❌ Using `latest` in Production
```bash
# RISKY — image may change unexpectedly
FROM python:latest

# SAFE — locked to a specific version
FROM python:3.11-slim
```

### ❌ Storing Secrets in Dockerfiles
```dockerfile
# NEVER DO THIS — secrets are baked into the image layer
ENV DB_PASSWORD=supersecret
```
> Use Docker secrets, `.env` files (excluded from version control), or a secrets manager instead.

---

## 📌 Quick Reference Cheatsheet

```bash
docker build -t name .               # Build image
docker run -d -p host:cont --name n  # Run container
docker ps / docker ps -a             # List containers
docker logs -f <name>                # Stream logs
docker exec -it <name> bash          # Shell into container
docker stop / rm <name>              # Stop / delete container
docker volume create <name>          # Create volume
docker network create <name>         # Create network
docker compose up -d                 # Start Compose stack
docker compose down -v               # Tear down + delete volumes
docker system prune -a               # Clean everything
```

---

*Happy containerizing! 🐳*