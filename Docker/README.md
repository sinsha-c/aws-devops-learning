# Docker & Containers: Notes for Beginners and Interviews

A beginner-friendly guide to Docker and related concepts, with commands and interview questions.

---

## Table of Contents

1. [What is a Container?](#1-what-is-a-container)
2. [What is Docker?](#2-what-is-docker)
3. [Containers vs Virtual Machines](#3-containers-vs-virtual-machines)
4. [Docker Architecture](#4-docker-architecture)
5. [Core Concepts](#5-core-concepts)
6. [Dockerfile](#6-dockerfile)
7. [Essential Docker Commands](#7-essential-docker-commands)
8. [Docker Networking](#8-docker-networking)
9. [Docker Volumes and Storage](#9-docker-volumes-and-storage)
10. [Docker Compose](#10-docker-compose)
11. [Container Registries](#11-container-registries)
12. [Multi-Stage Builds](#12-multi-stage-builds)
13. [Best Practices](#13-best-practices)
14. [Security Basics](#14-security-basics)
15. [Container Orchestration](#15-container-orchestration)
16. [Troubleshooting](#16-troubleshooting)
17. [Interview Questions and Answers](#17-interview-questions-and-answers)
18. [Quick Cheat Sheet](#18-quick-cheat-sheet)

---

## 1. What is a Container?

A **container** is a lightweight, isolated package that contains an application and everything it needs to run: code, runtime, libraries, and configuration.

**Simple analogy:** A container is like a **lunchbox**. You pack the meal (app) and everything needed to eat it (dependencies). It works the same wherever you open it.

**Key points**

- Runs as an isolated process on the host OS.
- Shares the host's **kernel** (does not need its own full OS).
- Starts in seconds and uses few resources.
- Solves the "it works on my machine" problem.

---

## 2. What is Docker?

**Docker** is an open-source platform to **build, ship, and run** applications in containers.

**Simple analogy:** If a container is the lunchbox, Docker is the kitchen and packing system that makes, stores, and opens it.

**Why use Docker?**

| Benefit | Explanation |
|---|---|
| Consistency | Same environment in dev, test, and production |
| Portability | Runs on any machine with Docker installed |
| Speed | Containers start in seconds |
| Efficiency | Many containers share one host with low overhead |
| Isolation | Apps do not interfere with each other |
| Scalability | Easy to replicate containers |
| CI/CD friendly | Build once, run anywhere |

---

## 3. Containers vs Virtual Machines

| Feature | Container | Virtual Machine |
|---|---|---|
| Includes | App + dependencies | App + dependencies + full guest OS |
| Kernel | Shares host kernel | Own kernel |
| Size | Megabytes | Gigabytes |
| Startup time | Seconds or less | Minutes |
| Isolation | Process-level | Hardware-level (stronger) |
| Overhead | Low | High |
| Managed by | Container runtime (Docker) | Hypervisor (VMware, KVM, Hyper-V) |

```
Virtual Machines                     Containers
+-----+ +-----+ +-----+             +-----+ +-----+ +-----+
| App | | App | | App |             | App | | App | | App |
| Bins| | Bins| | Bins|             | Bins| | Bins| | Bins|
| OS  | | OS  | | OS  |             +-----+-+-----+-+-----+
+-----+-+-----+-+-----+             |    Container Runtime  |
|      Hypervisor      |             |       Host OS         |
|      Host / HW       |             |       Hardware        |
+----------------------+             +-----------------------+
```

---

## 4. Docker Architecture

Docker uses a **client-server** architecture.

- **Docker Client (`docker` CLI):** where you type commands.
- **Docker Daemon (`dockerd`):** background service that builds, runs, and manages containers.
- **Docker Registry:** storage for images (Docker Hub, Amazon ECR).
- **Docker Objects:** images, containers, networks, volumes.

```
Docker Client  --->  Docker Daemon  --->  Containers
 (docker CLI)         (dockerd)            Images / Networks / Volumes
                          |
                          v
                   Registry (Docker Hub / ECR)
```

**Under the hood, Docker uses Linux features:**

- **Namespaces:** isolation (process IDs, network, filesystem, users).
- **cgroups (control groups):** limit CPU, memory, and I/O.
- **Union file systems (OverlayFS):** layered images.

---

## 5. Core Concepts

### Image
A **read-only template** used to create containers. It is built in layers.

### Container
A **running instance** of an image. Writable layer on top of the image.

### Dockerfile
A **text file** with instructions to build an image.

### Registry
A place to **store and share** images (Docker Hub, Amazon ECR, GitHub Container Registry).

### Layer
Each Dockerfile instruction creates a layer. Layers are **cached and reused**, which speeds up builds.

### Tag
A label for an image version, such as `nginx:1.25`. If no tag is given, `latest` is used.

**Relationship**

```
Dockerfile --(docker build)--> Image --(docker run)--> Container
```

**Analogy:** Dockerfile = recipe, Image = packed meal kit, Container = the cooked meal being served.

---

## 6. Dockerfile

### Common Instructions

| Instruction | Purpose |
|---|---|
| `FROM` | Base image to start from |
| `WORKDIR` | Set working directory |
| `COPY` | Copy files from host into image |
| `ADD` | Like COPY, plus can extract tar files and fetch URLs |
| `RUN` | Run a command **during build** (creates a layer) |
| `CMD` | Default command **when container starts** (can be overridden) |
| `ENTRYPOINT` | Fixed command that always runs when container starts |
| `ENV` | Set environment variables |
| `ARG` | Build-time variables |
| `EXPOSE` | Document the port the app listens on |
| `VOLUME` | Create a mount point for persistent data |
| `USER` | Set the user to run as |
| `HEALTHCHECK` | Check if the container is healthy |

### Example: Node.js App

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Example: Python App

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

### CMD vs ENTRYPOINT

| | CMD | ENTRYPOINT |
|---|---|---|
| Purpose | Default command/arguments | Main executable |
| Override at runtime | Easily (`docker run image newcmd`) | Only with `--entrypoint` |
| Together | CMD supplies default arguments to ENTRYPOINT | |

```dockerfile
ENTRYPOINT ["ping"]
CMD ["localhost"]
# docker run image           -> ping localhost
# docker run image google.com -> ping google.com
```

### COPY vs ADD
Prefer `COPY`. Use `ADD` only when you need tar auto-extraction or a remote URL.

### .dockerignore
Excludes files from the build context (like `.gitignore`). Keeps images small and secrets out.

```
node_modules
.git
.env
*.log
```

---

## 7. Essential Docker Commands

### Images

```bash
docker build -t myapp:1.0 .          # build image from Dockerfile
docker images                        # list images
docker pull nginx:latest             # download image
docker push user/myapp:1.0           # upload image to registry
docker tag myapp:1.0 user/myapp:1.0  # tag an image
docker rmi <image>                   # remove image
docker history <image>               # show image layers
docker image prune                   # remove unused images
```

### Containers

```bash
docker run nginx                     # run a container
docker run -d nginx                  # run in background (detached)
docker run -d -p 8080:80 nginx       # map host port 8080 to container port 80
docker run -d --name web nginx       # give a name
docker run -it ubuntu bash           # interactive terminal
docker run --rm alpine echo hi       # auto-remove after exit
docker run -e MY_VAR=value nginx     # set environment variable
docker run -v mydata:/data nginx     # attach a volume
docker run --memory=512m --cpus=1 nginx   # limit resources

docker ps                            # running containers
docker ps -a                         # all containers
docker stop <container>              # graceful stop
docker start <container>             # start stopped container
docker restart <container>           # restart
docker kill <container>              # force stop
docker rm <container>                # remove container
docker rm -f <container>             # force remove
```

### Inspect and Debug

```bash
docker logs <container>              # view logs
docker logs -f <container>           # follow logs
docker exec -it <container> bash     # open shell inside running container
docker inspect <container>           # detailed JSON info
docker stats                         # live CPU/memory usage
docker top <container>               # processes in container
docker cp file.txt <container>:/tmp  # copy files in or out
```

### Cleanup

```bash
docker system prune                  # remove unused data
docker system prune -a --volumes     # remove everything unused (careful)
docker system df                     # disk usage
```

---

## 8. Docker Networking

### Network Drivers

| Driver | Description |
|---|---|
| `bridge` (default) | Private network on a single host. Containers talk via IP/name on user-defined bridges |
| `host` | Container shares the host's network (no isolation, no port mapping needed) |
| `none` | No networking |
| `overlay` | Multi-host networking (Docker Swarm) |
| `macvlan` | Gives container its own MAC/IP on the physical network |

```bash
docker network ls
docker network create mynet
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp   # app can reach db by name "db"
docker network inspect mynet
```

**Key points**

- Containers on a **user-defined bridge** can reach each other **by container name** (built-in DNS).
- The default `bridge` network does not provide name resolution.
- `-p HOST:CONTAINER` publishes a port to the outside world.

---

## 9. Docker Volumes and Storage

Container data is **ephemeral**: it is lost when the container is removed. Use storage options to persist data.

| Type | Description | Use case |
|---|---|---|
| **Volume** | Managed by Docker (`/var/lib/docker/volumes`) | Databases, production data (recommended) |
| **Bind mount** | Maps a host folder into the container | Development (live code reload) |
| **tmpfs** | Stored in host memory only | Temporary or sensitive data |

```bash
docker volume create mydata
docker run -d -v mydata:/var/lib/mysql mysql         # named volume
docker run -d -v $(pwd):/app node                    # bind mount
docker volume ls
docker volume inspect mydata
docker volume rm mydata
```

---

## 10. Docker Compose

**Docker Compose** defines and runs **multi-container applications** using a YAML file (`docker-compose.yml` or `compose.yaml`).

### Example: Web App + Database

```yaml
services:
  web:
    build: .
    ports:
      - "8080:5000"
    environment:
      - DB_HOST=db
    depends_on:
      - db

  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - dbdata:/var/lib/postgresql/data

volumes:
  dbdata:
```

### Commands

```bash
docker compose up -d          # start all services in background
docker compose down           # stop and remove containers and networks
docker compose down -v        # also remove volumes
docker compose ps             # list services
docker compose logs -f        # follow logs
docker compose build          # rebuild images
docker compose exec web bash  # shell into a service
```

---

## 11. Container Registries

A **registry** stores and distributes images.

| Registry | Notes |
|---|---|
| Docker Hub | Default public registry |
| Amazon ECR | AWS private registry |
| GitHub Container Registry (GHCR) | Integrated with GitHub |
| Google Artifact Registry | Google Cloud |
| Azure Container Registry | Azure |

### Push an Image (Docker Hub)

```bash
docker login
docker tag myapp:1.0 username/myapp:1.0
docker push username/myapp:1.0
```

### Push an Image (Amazon ECR)

```bash
aws ecr get-login-password --region <region> | \
  docker login --username AWS --password-stdin <account_id>.dkr.ecr.<region>.amazonaws.com

docker tag myapp:1.0 <account_id>.dkr.ecr.<region>.amazonaws.com/myapp:1.0
docker push <account_id>.dkr.ecr.<region>.amazonaws.com/myapp:1.0
```

---

## 12. Multi-Stage Builds

Use multiple `FROM` statements to keep the **final image small**: build in one stage, copy only the result to the next.

```dockerfile
# Stage 1: Build
FROM golang:1.22 AS builder
WORKDIR /src
COPY . .
RUN go build -o app

# Stage 2: Run
FROM alpine:3.19
COPY --from=builder /src/app /usr/local/bin/app
CMD ["app"]
```

**Benefits:** smaller image, faster pull, fewer vulnerabilities, no build tools in production.

---

## 13. Best Practices

- Use **small base images** (`alpine`, `slim`, distroless).
- Use **specific tags**, not `latest`.
- Use **multi-stage builds**.
- Order instructions from **least to most frequently changing** to maximize layer caching (copy dependency files first, source code later).
- Combine related `RUN` commands to reduce layers.
- Use **`.dockerignore`**.
- **One main process per container.**
- Do **not run as root**; use `USER`.
- Never put **secrets** in images or Dockerfiles.
- Add **`HEALTHCHECK`**.
- Keep containers **stateless**; store data in volumes or external services.
- Scan images for vulnerabilities.
- Log to **stdout/stderr** so `docker logs` works.

---

## 14. Security Basics

- Run containers as a **non-root user**.
- Use **trusted, minimal** base images and keep them updated.
- **Scan images** (Trivy, Docker Scout, Amazon ECR scanning).
- Use **read-only filesystem**: `docker run --read-only`.
- **Drop capabilities**: `--cap-drop ALL`, add only what is needed.
- Avoid `--privileged`.
- Use **secrets management** (AWS Secrets Manager, Docker secrets, Vault), not plain `ENV`.
- Limit resources with `--memory` and `--cpus`.
- Do not expose the Docker socket (`/var/run/docker.sock`) to containers.
- Sign and verify images (Docker Content Trust).

---

## 15. Container Orchestration

For running many containers across many servers, you need an **orchestrator** to handle scheduling, scaling, self-healing, load balancing, and rolling updates.

| Tool | Description |
|---|---|
| **Kubernetes (K8s)** | Industry-standard orchestrator |
| **Amazon EKS** | Managed Kubernetes on AWS |
| **Amazon ECS** | AWS-native container orchestration |
| **AWS Fargate** | Serverless compute for ECS/EKS (no servers to manage) |
| **Docker Swarm** | Docker's built-in, simpler orchestrator |

### Docker vs Kubernetes
Docker **builds and runs** containers. Kubernetes **manages** containers at scale.

---

## 16. Troubleshooting

| Problem | What to check |
|---|---|
| Container exits immediately | `docker logs <id>`; main process may have finished or crashed |
| Cannot reach app from browser | Port mapping (`-p`), app binding to `0.0.0.0` (not `127.0.0.1`), firewall or security group |
| `port is already allocated` | Another process or container uses that port; change host port |
| Container cannot reach another | Same network? Use container name on a user-defined network |
| Data lost after restart | No volume attached |
| Image too large | Use alpine/slim, multi-stage, `.dockerignore` |
| `permission denied` on docker.sock | Add user to `docker` group: `sudo usermod -aG docker $USER` |
| Build is slow | Reorder Dockerfile for caching; use `.dockerignore` |
| Out of disk space | `docker system df`, then `docker system prune` |
| Container keeps restarting | Check logs, restart policy, health check, memory limit (OOMKilled) |

**Useful debugging steps**

```bash
docker ps -a
docker logs <container>
docker inspect <container>
docker exec -it <container> sh
docker stats
```

---

## 17. Interview Questions and Answers

### Basic

**Q1. What is Docker?**
Docker is an open-source platform that packages applications and their dependencies into containers so they run consistently across environments.

**Q2. What is a container?**
A lightweight, isolated, running instance of an image that shares the host OS kernel.

**Q3. What is the difference between an image and a container?**
An image is a read-only template (blueprint). A container is a running instance of that image with a writable layer.

**Q4. What is a Dockerfile?**
A text file with step-by-step instructions to build a Docker image.

**Q5. Container vs virtual machine?**
Containers share the host kernel and are lightweight and fast. VMs include a full guest OS and run on a hypervisor, so they are heavier but offer stronger isolation.

**Q6. What is Docker Hub?**
The default public registry to store and share Docker images.

**Q7. What is the difference between `docker run`, `docker start`, and `docker exec`?**
`run` creates and starts a new container from an image. `start` restarts an existing stopped container. `exec` runs a command inside an already running container.

**Q8. What is the difference between `docker stop` and `docker kill`?**
`stop` sends SIGTERM, then SIGKILL after a grace period (graceful). `kill` sends SIGKILL immediately.

### Intermediate

**Q9. Difference between CMD and ENTRYPOINT?**
`CMD` gives a default command that is easily overridden. `ENTRYPOINT` defines the fixed executable. Together, CMD provides default arguments to ENTRYPOINT.

**Q10. Difference between COPY and ADD?**
Both copy files. `ADD` can also extract local tar archives and download URLs. Prefer `COPY` for clarity.

**Q11. Difference between RUN, CMD, and ENTRYPOINT?**
`RUN` executes during **build**. `CMD` and `ENTRYPOINT` execute when the **container starts**.

**Q12. What are Docker layers and why do they matter?**
Each instruction creates a read-only layer. Layers are cached and shared between images, which speeds up builds and saves storage.

**Q13. How do you reduce Docker image size?**
Use small base images (alpine/slim), multi-stage builds, `.dockerignore`, combine `RUN` commands, and remove caches and unnecessary packages.

**Q14. What is the difference between a volume and a bind mount?**
A volume is managed by Docker and is best for persistent data. A bind mount maps a specific host path and is common in development.

**Q15. How do containers communicate with each other?**
Through Docker networks. On a user-defined bridge network, containers resolve each other by name.

**Q16. What are Docker network types?**
bridge, host, none, overlay, macvlan.

**Q17. What is Docker Compose?**
A tool to define and run multi-container apps using a YAML file and a single command (`docker compose up`).

**Q18. What is the difference between `EXPOSE` and `-p`?**
`EXPOSE` only documents the port. `-p` actually publishes the port to the host.

**Q19. What is the difference between `ENV` and `ARG`?**
`ARG` is available only at build time. `ENV` persists into the running container.

**Q20. How do you persist data in Docker?**
Use volumes (or bind mounts) so data survives container deletion.

**Q21. What is a multi-stage build?**
Using multiple `FROM` stages so only the final artifacts are copied into a small runtime image.

**Q22. How do you pass environment variables to a container?**
`-e KEY=value`, `--env-file .env`, or `environment:` in Docker Compose.

### Advanced

**Q23. How does Docker isolate containers?**
Using Linux **namespaces** (isolation of PID, network, mount, user, etc.) and **cgroups** (resource limits).

**Q24. What is the Docker daemon?**
`dockerd`, the background service that manages images, containers, networks, and volumes.

**Q25. What happens when you run `docker run nginx`?**
1. Client sends the request to the daemon.
2. Daemon checks for the image locally; if missing, pulls it from the registry.
3. Daemon creates a container with a writable layer, network, and settings.
4. Daemon starts the container process.

**Q26. How do you limit container resources?**
`docker run --memory=512m --cpus=1 image`. Enforced by cgroups.

**Q27. What are restart policies?**
`no`, `on-failure`, `always`, `unless-stopped`. They control when Docker automatically restarts containers.

**Q28. How do you secure Docker containers?**
Non-root user, minimal images, image scanning, secrets managers, read-only filesystem, dropped capabilities, resource limits, and no privileged mode.

**Q29. What is a dangling image?**
An untagged image (shown as `<none>`) that is no longer referenced. Remove with `docker image prune`.

**Q30. How do you debug a container that exits immediately?**
Check `docker logs`, `docker inspect` for the exit code, and run it interactively (`docker run -it image sh`) to investigate.

**Q31. What is the difference between Docker and Kubernetes?**
Docker builds and runs containers on a host. Kubernetes orchestrates containers across a cluster (scaling, self-healing, rolling updates, service discovery).

**Q32. What is a container runtime?**
Software that actually runs containers. Examples: `containerd`, `runc`, CRI-O. Docker uses containerd under the hood.

**Q33. What is OCI?**
The Open Container Initiative: standards for container image format and runtime so tools are interoperable.

**Q34. How do you do zero-downtime updates with containers?**
Use rolling or blue-green deployments through an orchestrator (ECS, Kubernetes) behind a load balancer.

**Q35. Where does Docker CI/CD fit in?**
Pipeline builds the image, runs tests, scans it, pushes it to a registry (ECR/Docker Hub), then deploys it to ECS/EKS/servers.

### Scenario-Based

**Q36. Your container works locally but not on the server. What do you check?**
Image tag/version differences, environment variables, port mapping and security groups, network access, volume paths, architecture mismatch (ARM vs x86), and logs.

**Q37. A container keeps restarting. What do you do?**
Check `docker logs` and `docker inspect` (exit code, `OOMKilled`), review the restart policy, health check, memory limits, and config or secrets.

**Q38. Your image is 2 GB. How do you reduce it?**
Switch to a slim/alpine base, use multi-stage builds, add `.dockerignore`, remove build tools and package caches, and avoid copying unnecessary files.

---

## 18. Quick Cheat Sheet

```bash
# Build and run
docker build -t app:1.0 .
docker run -d -p 8080:80 --name web app:1.0

# Inspect
docker ps -a
docker logs -f web
docker exec -it web sh
docker inspect web
docker stats

# Stop and remove
docker stop web && docker rm web
docker rmi app:1.0

# Volumes and networks
docker volume create data
docker network create mynet
docker run -d -v data:/data --network mynet app:1.0

# Registry
docker login
docker tag app:1.0 user/app:1.0
docker push user/app:1.0

# Compose
docker compose up -d
docker compose down

# Cleanup
docker system prune
```

### One-Line Memory Aids

- **Image** = blueprint. **Container** = running instance.
- **Dockerfile** = recipe. **Registry** = store.
- **Volume** = keeps data. **Network** = lets containers talk.
- **Compose** = many containers, one file.
- **Kubernetes/ECS/EKS** = manage containers at scale.

---

## Further Learning

- Official docs: https://docs.docker.com
- Play with Docker (free browser lab): https://labs.play-with-docker.com
- Practice: containerize a small app, push it to Docker Hub or Amazon ECR, then deploy it on ECS or EKS.