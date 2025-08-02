# Docker Interview Questions and Answers

## Docker Basics

### What is Docker?

**Answer:**
Docker is a containerization platform that enables developers to package applications and their dependencies into lightweight, portable containers that can run consistently across different environments.

**Key characteristics:**
- **Containerization:** Packages applications with all dependencies
- **Portability:** Runs consistently across different environments
- **Lightweight:** Shares OS kernel, more efficient than VMs
- **Scalability:** Easy to scale applications horizontally
- **Isolation:** Applications run in isolated environments

**Core concepts:**
- **Container:** Runtime instance of a Docker image
- **Image:** Read-only template used to create containers
- **Dockerfile:** Text file with instructions to build images
- **Registry:** Repository for storing and sharing images
- **Docker Engine:** Runtime that manages containers

### How is Docker different from virtual machines?

**Answer:**
Docker containers and virtual machines solve similar problems but use fundamentally different approaches:

**Architecture differences:**

| Aspect | Docker Containers | Virtual Machines |
|--------|------------------|------------------|
| **Virtualization Level** | OS-level virtualization | Hardware-level virtualization |
| **OS Kernel** | Shares host OS kernel | Each VM has its own OS |
| **Resource Usage** | Lightweight, minimal overhead | Heavy, significant overhead |
| **Startup Time** | Seconds | Minutes |
| **Isolation** | Process-level isolation | Complete isolation |
| **Portability** | Highly portable | Less portable |

**Use cases:**
- **Use Docker when:** Microservices, CI/CD, development environments, application deployment
- **Use VMs when:** Complete isolation needed, different OS requirements, legacy applications

### What are the key components of Docker?

**Answer:**
Docker consists of several key components that work together to provide containerization:

**1. Docker Engine:**
- **Docker Daemon (dockerd):** Background service that manages containers
- **Docker CLI:** Command-line interface for interacting with Docker
- **REST API:** API for programmatic interaction

**2. Docker Images:**
- **Read-only templates** used to create containers
- **Layered filesystem** for efficient storage
- **Immutable** once created

**3. Docker Containers:**
- **Runtime instances** of Docker images
- **Isolated processes** running on host OS
- **Stateful** and can be started, stopped, moved, deleted

**4. Docker Registry:**
- **Repository** for storing and sharing images
- **Docker Hub** is the default public registry
- **Private registries** for enterprise use

**5. Dockerfile:**
- **Text file** with instructions to build images
- **Declarative** approach to image creation
- **Version controlled** with application code

**6. Docker Compose:**
- **Tool** for defining multi-container applications
- **YAML file** for configuration
- **Orchestrates** multiple containers

**7. Docker Networks:**
- **Virtual networks** for container communication
- **Built-in network drivers** (bridge, host, overlay)
- **Custom networks** for isolation

**8. Docker Volumes:**
- **Persistent storage** for containers
- **Data persistence** beyond container lifecycle
- **Shared storage** between containers

## Docker Files

### What are Dockerfiles?

**Answer:**
A Dockerfile is a text file containing a series of instructions used to build a Docker image automatically. It defines the environment, dependencies, and configuration needed to run an application.

**Key characteristics:**
- **Declarative:** Describes desired state rather than steps
- **Layered:** Each instruction creates a new layer
- **Cacheable:** Layers can be cached for faster builds
- **Version controlled:** Stored with application code
- **Reproducible:** Same Dockerfile produces same image

**Common Dockerfile instructions:**

| Instruction | Purpose | Example |
|-------------|---------|---------|
| **FROM** | Base image | `FROM node:14-alpine` |
| **RUN** | Execute commands | `RUN npm install` |
| **COPY** | Copy files from host | `COPY . /app` |
| **WORKDIR** | Set working directory | `WORKDIR /app` |
| **ENV** | Set environment variables | `ENV NODE_ENV=production` |
| **EXPOSE** | Document port usage | `EXPOSE 8080` |
| **CMD** | Default command | `CMD ["npm", "start"]` |

### Have you written any Dockerfile in your project from scratch?

**Answer:**
Yes, I have written Dockerfiles from scratch for various types of applications. Here are examples:

**Spring Boot Application Dockerfile:**
```dockerfile
# Multi-stage build for Spring Boot
FROM maven:3.8.4-openjdk-11 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM openjdk:11-jre-slim
WORKDIR /app

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Copy JAR from builder stage
COPY --from=builder /app/target/*.jar app.jar

# Change ownership
RUN chown -R appuser:appuser /app
USER appuser

# Environment variables
ENV JAVA_OPTS="-Xmx512m -Xms256m"
ENV SERVER_PORT=8080

EXPOSE $SERVER_PORT

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s \
  CMD curl -f http://localhost:$SERVER_PORT/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### What are the best practices for writing Dockerfiles?

**Answer:**
Following best practices ensures efficient, secure, and maintainable Docker images:

**1. Use Multi-stage Builds:**
```dockerfile
# Build stage
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
```

**2. Minimize Image Layers:**
```dockerfile
# Good - Single RUN instruction
RUN apt-get update && \
    apt-get install -y curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**3. Use Specific Base Images:**
```dockerfile
# Good - Specific version and variant
FROM node:16.14.2-alpine3.15
```

**4. Run as Non-root User:**
```dockerfile
# Create user
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app
USER appuser
```

## Docker Images

### What is a Docker image?

**Answer:**
A Docker image is a read-only template that contains the application code, runtime, system tools, libraries, and settings needed to run an application. It serves as the blueprint for creating Docker containers.

**Key characteristics:**
- **Immutable:** Once created, images cannot be changed
- **Layered:** Built using a layered filesystem
- **Portable:** Can run on any system with Docker
- **Versioned:** Tagged with versions for management
- **Shareable:** Can be stored and shared via registries

### How are Docker images layered?

**Answer:**
Docker images use a layered filesystem where each instruction in a Dockerfile creates a new layer. This layered approach provides efficiency, caching, and sharing benefits.

**Benefits of layering:**
1. **Storage efficiency:** Shared layers reduce disk usage
2. **Build speed:** Cached layers speed up builds
3. **Network efficiency:** Only changed layers are transferred
4. **Incremental updates:** Only modified layers need updating

### How to optimize Docker image size?

**Answer:**
Optimizing Docker image size improves deployment speed, reduces storage costs, and minimizes attack surface:

**1. Use Minimal Base Images:**
```dockerfile
# Smaller image (~100MB)
FROM alpine:3.15

# Distroless (security-focused, ~50MB)
FROM gcr.io/distroless/java:11
```

**2. Multi-stage Builds:**
```dockerfile
# Build stage - includes build tools
FROM node:16-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Production stage - minimal runtime
FROM node:16-alpine AS production
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
CMD ["npm", "start"]
```

**3. Remove Unnecessary Files:**
```dockerfile
# Clean up in same layer
RUN apt-get update && \
    apt-get install -y build-essential && \
    # ... build application ... && \
    apt-get remove -y build-essential && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

## Docker Containers

### What is a Docker container?

**Answer:**
A Docker container is a runtime instance of a Docker image - a lightweight, standalone, executable package that includes everything needed to run an application.

**Key characteristics:**
- **Isolated:** Runs in its own namespace with isolated processes
- **Lightweight:** Shares host OS kernel, minimal overhead
- **Portable:** Runs consistently across different environments
- **Ephemeral:** Can be easily created, started, stopped, and destroyed
- **Stateless:** Designed to be stateless by default

### What is the lifecycle of a Docker container?

**Answer:**
A Docker container goes through several states during its lifecycle:

**Container States:**
```
Created → Running → Paused → Stopped → Removed
```

**Lifecycle commands:**
```bash
# Container lifecycle management
docker run nginx                    # Create and start
docker create nginx                 # Create only
docker start container_id           # Start created/stopped container
docker restart container_id         # Restart container
docker pause container_id           # Pause running container
docker unpause container_id         # Resume paused container
docker stop container_id            # Stop running container
docker kill container_id            # Force kill container
docker rm container_id              # Remove stopped container
```

### How to manage container state?

**Answer:**
Managing container state involves monitoring, controlling, and persisting container data and configuration:

**1. Container State Monitoring:**
```bash
# Check container status
docker ps                           # Running containers
docker ps -a                        # All containers
docker stats                        # Real-time resource usage
docker logs container_id             # Container logs
docker inspect container_id         # Detailed container info
```

**2. Container Resource Management:**
```bash
# CPU limits
docker run --cpus="1.5" nginx       # Limit to 1.5 CPU cores

# Memory limits
docker run -m 512m nginx            # Limit to 512MB RAM
```

**3. Container Restart Policies:**
```bash
# Restart policies
docker run --restart=always nginx          # Always restart
docker run --restart=unless-stopped nginx  # Restart unless manually stopped
docker run --restart=on-failure:3 nginx    # Restart on failure, max 3 times
```

## Docker Commands

### How to pull an image from Docker Hub?

**Answer:**
Pulling images from Docker Hub is done using the `docker pull` command:

```bash
# Pull latest version of an image
docker pull nginx
docker pull nginx:latest  # Explicit latest tag

# Pull specific version
docker pull nginx:1.21-alpine
docker pull postgres:13.4

# Pull with platform specification
docker pull --platform linux/amd64 nginx
```

### How to push an image to a local registry or Docker Hub?

**Answer:**
Pushing images involves tagging them appropriately and using the `docker push` command:

**Push to Docker Hub:**
```bash
# 1. Build or tag image with your Docker Hub username
docker build -t username/myapp:latest .

# 2. Login to Docker Hub
docker login

# 3. Push the image
docker push username/myapp:latest
```

**Push to local registry:**
```bash
# 1. Start local registry
docker run -d -p 5000:5000 --name registry registry:2

# 2. Tag image for local registry
docker tag myapp:latest localhost:5000/myapp:latest

# 3. Push to local registry
docker push localhost:5000/myapp:latest
```

### How to push an image to Amazon ECR?

**Answer:**
Amazon Elastic Container Registry (ECR) requires AWS authentication and specific tagging format:

**Push to ECR steps:**

**1. Create ECR repository:**
```bash
# Create repository via AWS CLI
aws ecr create-repository --repository-name myapp --region us-west-2
```

**2. Get login token:**
```bash
# Get login command (AWS CLI v2)
aws ecr get-login-password --region us-west-2 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-west-2.amazonaws.com
```

**3. Tag and push image:**
```bash
# Tag image with ECR repository URI
docker tag myapp:latest \
  123456789012.dkr.ecr.us-west-2.amazonaws.com/myapp:latest

# Push to ECR
docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/myapp:latest
```

## Docker Configuration

### How to override properties in Docker files for different environments?

**Answer:**
Overriding properties for different environments can be achieved through various methods:

**1. Environment Variables:**
```dockerfile
# Dockerfile with environment variables
FROM openjdk:11-jre-slim
COPY app.jar /app.jar

# Default values
ENV SERVER_PORT=8080
ENV SPRING_PROFILES_ACTIVE=prod
ENV DB_HOST=localhost

EXPOSE ${SERVER_PORT}
CMD ["java", "-jar", "/app.jar"]
```

```bash
# Override for development
docker run -e SPRING_PROFILES_ACTIVE=dev \
           -e DB_HOST=dev-db \
           -e SERVER_PORT=8081 \
           -p 8081:8081 myapp
```

**2. Environment Files:**
```bash
# dev.env
SPRING_PROFILES_ACTIVE=dev
DB_HOST=dev-database
LOG_LEVEL=DEBUG

# Use environment files
docker run --env-file dev.env myapp
```

**3. Docker Compose Override:**
```yaml
# docker-compose.yml (base)
version: '3.8'
services:
  app:
    build: .
    environment:
      - NODE_ENV=production

# docker-compose.override.yml (development)
version: '3.8'
services:
  app:
    environment:
      - NODE_ENV=development
      - DEBUG=true
```

### How to manage environment variables in Docker?

**Answer:**
Environment variables in Docker can be managed through multiple approaches:

**1. Dockerfile ENV instruction:**
```dockerfile
# Set environment variables in Dockerfile
FROM node:16-alpine

# Static environment variables
ENV NODE_ENV=production
ENV PORT=3000

# Dynamic environment variables with defaults
ENV DB_HOST=${DB_HOST:-localhost}
```

**2. Runtime environment variables:**
```bash
# Single environment variable
docker run -e NODE_ENV=development myapp

# Multiple environment variables
docker run -e NODE_ENV=development \
           -e DB_HOST=localhost \
           myapp
```

**3. Environment files:**
```bash
# .env file
NODE_ENV=development
DB_HOST=localhost
API_KEY=your-api-key

# Use environment file
docker run --env-file .env myapp
```

### How to handle secrets in Docker?

**Answer:**
Handling secrets securely in Docker requires careful consideration of security best practices:

**1. Docker Secrets (Swarm Mode):**
```bash
# Create secret
echo "mysecretpassword" | docker secret create db_password -

# Use secret in service
docker service create \
  --name myapp \
  --secret db_password \
  myapp:latest
```

**2. External secret management:**
```bash
# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id prod/db/password

# HashiCorp Vault
vault kv get -field=password secret/myapp/db
```

## Docker Networking

### What are the different networking modes in Docker?

**Answer:**
Docker provides several networking modes to control how containers communicate:

**1. Bridge Network (Default):**
```bash
# Default bridge network
docker run -d --name web nginx
docker run -d --name app myapp

# Create custom bridge network
docker network create mynetwork
docker run -d --network mynetwork --name web nginx
```

**2. Host Network:**
```bash
# Use host network stack
docker run -d --network host nginx
# Container uses host's network interface directly
```

**3. None Network:**
```bash
# No network access
docker run -d --network none alpine sleep 3600
# Container has no network interface
```

### How do containers communicate with each other?

**Answer:**
Containers can communicate through various methods:

**1. Same Bridge Network Communication:**
```bash
# Create custom network
docker network create myapp-network

# Run containers on same network
docker run -d --network myapp-network --name database postgres:13
docker run -d --network myapp-network --name backend myapp-backend

# Containers can communicate using container names
```

**2. Docker Compose Communication:**
```yaml
version: '3.8'
services:
  database:
    image: postgres:13
    networks:
      - backend-network

  backend:
    build: ./backend
    environment:
      - DB_HOST=database  # Service name as hostname
    networks:
      - backend-network

networks:
  backend-network:
```

### How to expose container ports?

**Answer:**
Exposing container ports makes services accessible from outside the container:

**1. Port Publishing with -p flag:**
```bash
# Basic port mapping: host_port:container_port
docker run -p 8080:80 nginx

# Multiple port mappings
docker run -p 8080:80 -p 8443:443 nginx

# Bind to specific host interface
docker run -p 127.0.0.1:8080:80 nginx
```

**2. Docker Compose port configuration:**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - "3000:3000"
      - "8080:8080"
```

## Docker Volumes

### What are Docker volumes?

**Answer:**
Docker volumes are the preferred mechanism for persisting data generated and used by Docker containers.

**Key characteristics:**
- **Persistent:** Data survives container restarts and removal
- **Shareable:** Can be shared between multiple containers
- **Managed:** Docker manages the volume lifecycle
- **Performance:** Better performance than bind mounts
- **Backup-friendly:** Easy to backup and restore

**Volume types:**

**1. Named Volumes:**
```bash
# Create named volume
docker volume create mydata

# Use named volume
docker run -v mydata:/data alpine
```

**2. Bind Mounts:**
```bash
# Mount host directory
docker run -v /host/path:/container/path alpine
docker run -v $(pwd):/app node:alpine
```

### How to persist data in Docker containers?

**Answer:**
Data persistence ensures that important data survives container lifecycle events:

**1. Database Persistence:**
```bash
# PostgreSQL with named volume
docker run -d \
  --name postgres-db \
  -v postgres-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=password \
  postgres:13
```

**2. Docker Compose Persistence:**
```yaml
version: '3.8'
services:
  database:
    image: postgres:13
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password

volumes:
  postgres-data:
```

### What are the different types of Docker volumes?

**Answer:**
Docker provides three main types of volume mounts:

**1. Volumes (Recommended):**
- Managed by Docker
- Best performance on Docker Desktop
- Easy backup and migration

**2. Bind Mounts:**
- File or directory on host filesystem
- Good for development workflows
- Direct access from host

**3. tmpfs Mounts (Linux only):**
- Stored in host memory
- Not persisted to disk
- Good for temporary data

## Docker Compose

### What is Docker Compose?

**Answer:**
Docker Compose is a tool for defining and running multi-container Docker applications. It uses a YAML file to configure application services.

**Key features:**
- **Multi-container orchestration:** Manage multiple containers as a single application
- **Service definition:** Define services, networks, and volumes in YAML
- **Environment management:** Different configurations for different environments
- **Dependency management:** Control startup order and dependencies
- **Scaling:** Scale services up or down easily

**Basic Docker Compose file structure:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - database
    environment:
      - NODE_ENV=development

  database:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=password
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

### How to define multi-container applications with Docker Compose?

**Answer:**
Docker Compose allows you to define complex multi-container applications:

**Complete Web Application Stack:**
```yaml
version: '3.8'

services:
  # Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
    networks:
      - frontend

  # Web Application
  web:
    build: ./web
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@database:5432/myapp
    depends_on:
      - database
    networks:
      - frontend
      - backend

  # Database
  database:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend

volumes:
  postgres-data:

networks:
  frontend:
  backend:
```

**Environment-specific configurations:**
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```