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
ENV DB_PORT=5432

EXPOSE ${SERVER_PORT}
CMD ["java", "-jar", "/app.jar"]
```

```bash
# Override for development
docker run -e SPRING_PROFILES_ACTIVE=dev \
           -e DB_HOST=dev-db \
           -e SERVER_PORT=8081 \
           -p 8081:8081 myapp

# Override for production
docker run -e SPRING_PROFILES_ACTIVE=prod \
           -e DB_HOST=prod-db \
           -e DB_PORT=5432 \
           myapp
```

**2. Environment Files:**
```bash
# dev.env
SPRING_PROFILES_ACTIVE=dev
DB_HOST=dev-database
DB_PORT=5432
LOG_LEVEL=DEBUG

# prod.env
SPRING_PROFILES_ACTIVE=prod
DB_HOST=prod-database
DB_PORT=5432
LOG_LEVEL=INFO

# Use environment files
docker run --env-file dev.env myapp
docker run --env-file prod.env myapp
```

**3. Multi-stage Builds with Build Args:**
```dockerfile
# Multi-stage Dockerfile with build arguments
FROM node:16-alpine AS base
WORKDIR /app
COPY package*.json ./

# Development stage
FROM base AS development
ARG NODE_ENV=development
ENV NODE_ENV=${NODE_ENV}
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

# Production stage
FROM base AS production
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
RUN npm ci --only=production
COPY . .
CMD ["npm", "start"]

# Build for different environments
# docker build --target development -t myapp:dev .
# docker build --target production -t myapp:prod .
```

**4. Configuration File Mounting:**
```bash
# Mount different config files
docker run -v $(pwd)/config/dev.yml:/app/config.yml myapp
docker run -v $(pwd)/config/prod.yml:/app/config.yml myapp

# Mount entire config directory
docker run -v $(pwd)/config/dev:/app/config myapp
docker run -v $(pwd)/config/prod:/app/config myapp
```

**5. Docker Compose Override:**
```yaml
# docker-compose.yml (base)
version: '3.8'
services:
  app:
    build: .
    environment:
      - NODE_ENV=production
    ports:
      - "8080:8080"

# docker-compose.override.yml (development)
version: '3.8'
services:
  app:
    environment:
      - NODE_ENV=development
      - DEBUG=true
    volumes:
      - .:/app
    ports:
      - "3000:3000"

# docker-compose.prod.yml (production)
version: '3.8'
services:
  app:
    environment:
      - NODE_ENV=production
      - LOG_LEVEL=info
    restart: always
```

```bash
# Use different compose files
docker-compose up  # Uses base + override
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
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
ENV APP_NAME="My Application"

# Dynamic environment variables with defaults
ENV DB_HOST=${DB_HOST:-localhost}
ENV DB_PORT=${DB_PORT:-5432}

WORKDIR /app
COPY . .
EXPOSE ${PORT}
CMD ["npm", "start"]
```

**2. Runtime environment variables:**
```bash
# Single environment variable
docker run -e NODE_ENV=development myapp

# Multiple environment variables
docker run -e NODE_ENV=development \
           -e DB_HOST=localhost \
           -e DB_PORT=5432 \
           myapp

# Environment variables from host
docker run -e NODE_ENV \
           -e DB_HOST \
           myapp
```

**3. Environment files:**
```bash
# .env file
NODE_ENV=development
DB_HOST=localhost
DB_PORT=5432
API_KEY=your-api-key
DEBUG=true

# Use environment file
docker run --env-file .env myapp

# Multiple environment files
docker run --env-file base.env --env-file dev.env myapp
```

**4. Docker Compose environment management:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    environment:
      # Direct assignment
      - NODE_ENV=production
      - PORT=3000
      # From host environment
      - DB_PASSWORD
      # With default value
      - DB_HOST=${DB_HOST:-localhost}
    env_file:
      - .env
      - .env.local
```

### How to handle secrets in Docker?

**Answer:**
Handling secrets securely in Docker requires careful consideration of security best practices:

**1. Docker Secrets (Swarm Mode):**
```bash
# Create secret
echo "mysecretpassword" | docker secret create db_password -

# Or from file
docker secret create db_password ./password.txt

# Use secret in service
docker service create \
  --name myapp \
  --secret db_password \
  myapp:latest
```

```dockerfile
# Access secret in container (mounted at /run/secrets/)
FROM alpine
RUN apk add --no-cache curl
CMD ["sh", "-c", "DB_PASSWORD=$(cat /run/secrets/db_password) && echo $DB_PASSWORD"]
```

**2. Build-time secrets (BuildKit):**
```dockerfile
# syntax=docker/dockerfile:1
FROM alpine
RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) && \
    curl -H "Authorization: Bearer $API_KEY" https://api.example.com/data
```

```bash
# Build with secret
echo "secret-api-key" | docker build --secret id=api_key,src=- .
```

**3. External secret management:**
```bash
# AWS Secrets Manager
aws secretsmanager get-secret-value --secret-id prod/db/password --query SecretString --output text

# HashiCorp Vault
vault kv get -field=password secret/myapp/db

# Azure Key Vault
az keyvault secret show --name db-password --vault-name myvault --query value -o tsv
```

**4. Environment variable injection at runtime:**
```bash
# Script to inject secrets
#!/bin/bash
# start.sh
export DB_PASSWORD=$(aws secretsmanager get-secret-value \
  --secret-id prod/db/password \
  --query SecretString --output text)

export API_KEY=$(vault kv get -field=api_key secret/myapp)

# Start application with secrets
exec "$@"
```

```dockerfile
FROM myapp:base
COPY start.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/start.sh
ENTRYPOINT ["/usr/local/bin/start.sh"]
CMD ["myapp"]
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

# Containers can communicate using container names
docker exec app ping web

# Create custom bridge network
docker network create mynetwork
docker run -d --network mynetwork --name web nginx
docker run -d --network mynetwork --name app myapp
```

**Characteristics:**
- Isolated network segment
- Automatic DNS resolution between containers
- NAT for external communication
- Port mapping required for external access

**2. Host Network:**
```bash
# Use host network stack
docker run -d --network host nginx

# Container uses host's network interface directly
# No port mapping needed
# Less isolation
```

**3. None Network:**
```bash
# No network access
docker run -d --network none alpine sleep 3600

# Container has no network interface
# Complete network isolation
```

**4. Container Network:**
```bash
# Share network with another container
docker run -d --name web nginx
docker run -d --network container:web myapp

# Both containers share same network stack
# Same IP address and ports
```

### How do containers communicate with each other?

**Answer:**
Containers can communicate through various methods depending on the network configuration:

**1. Same Bridge Network Communication:**
```bash
# Create custom network
docker network create myapp-network

# Run containers on same network
docker run -d --network myapp-network --name database postgres:13
docker run -d --network myapp-network --name backend myapp-backend
docker run -d --network myapp-network --name frontend myapp-frontend

# Containers can communicate using container names
# backend can connect to database using hostname 'database'
# frontend can connect to backend using hostname 'backend'
```

**2. Service Discovery:**
```bash
# Automatic DNS resolution
docker exec backend nslookup database
# Returns IP address of database container

docker exec backend ping database
# Pings database container successfully
```

**3. Docker Compose Communication:**
```yaml
# docker-compose.yml
version: '3.8'
services:
  database:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: password
    networks:
      - backend-network

  backend:
    build: ./backend
    environment:
      - DB_HOST=database  # Service name as hostname
      - DB_PORT=5432
    depends_on:
      - database
    networks:
      - backend-network
      - frontend-network

  frontend:
    build: ./frontend
    environment:
      - API_URL=http://backend:8080  # Service name as hostname
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - frontend-network

networks:
  backend-network:
  frontend-network:
```

### How to expose container ports?

**Answer:**
Exposing container ports makes services accessible from outside the container:

**1. Port Publishing with -p flag:**
```bash
# Basic port mapping: host_port:container_port
docker run -p 8080:80 nginx
# Access nginx on http://localhost:8080

# Multiple port mappings
docker run -p 8080:80 -p 8443:443 nginx

# Bind to specific host interface
docker run -p 127.0.0.1:8080:80 nginx
# Only accessible from localhost

# Random host port
docker run -P nginx
# Docker assigns random available port
```

**2. EXPOSE instruction in Dockerfile:**
```dockerfile
# Dockerfile
FROM nginx:alpine

# Document which ports the container listens on
EXPOSE 80
EXPOSE 443

# EXPOSE is documentation only - doesn't publish ports
# Still need -p flag when running container
```

**3. Docker Compose port configuration:**
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      # Short syntax: host:container
      - "3000:3000"
      - "8080:8080"
      
      # Long syntax with protocol
      - target: 80
        published: 8080
        protocol: tcp
        mode: host
      
      # Bind to specific interface
      - "127.0.0.1:5432:5432"
      
      # Random host port
      - "3000"

  database:
    image: postgres:13
    ports:
      # Internal port only (not published to host)
      - "5432"
    # Or use expose for documentation
    expose:
      - "5432"
```

## Docker Volumes

### What are Docker volumes?

**Answer:**
Docker volumes are the preferred mechanism for persisting data generated and used by Docker containers. They provide a way to store data outside the container's filesystem.

**Key characteristics:**
- **Persistent:** Data survives container restarts and removal
- **Shareable:** Can be shared between multiple containers
- **Managed:** Docker manages the volume lifecycle
- **Performance:** Better performance than bind mounts on Docker Desktop
- **Backup-friendly:** Easy to backup and restore

**Volume types:**

**1. Named Volumes:**
```bash
# Create named volume
docker volume create mydata

# Use named volume
docker run -v mydata:/data alpine

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata
```

**2. Anonymous Volumes:**
```bash
# Create anonymous volume
docker run -v /data alpine

# Docker generates random name
# Removed when container is removed (unless --rm is not used)
```

**3. Bind Mounts:**
```bash
# Mount host directory
docker run -v /host/path:/container/path alpine
docker run -v $(pwd):/app node:alpine

# Read-only bind mount
docker run -v /host/path:/container/path:ro alpine
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

# MySQL with named volume
docker run -d \
  --name mysql-db \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  mysql:8.0

# MongoDB with named volume
docker run -d \
  --name mongo-db \
  -v mongo-data:/data/db \
  mongo:4.4
```

**2. Application Data Persistence:**
```dockerfile
# Dockerfile with volume declaration
FROM node:16-alpine
WORKDIR /app

# Create volume for uploads
VOLUME ["/app/uploads"]

# Create volume for logs
VOLUME ["/app/logs"]

COPY . .
CMD ["npm", "start"]
```

```bash
# Run with persistent volumes
docker run -d \
  -v app-uploads:/app/uploads \
  -v app-logs:/app/logs \
  myapp
```

**3. Docker Compose Persistence:**
```yaml
version: '3.8'
services:
  database:
    image: postgres:13
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    environment:
      POSTGRES_PASSWORD: password

  app:
    build: .
    volumes:
      - app-uploads:/app/uploads
      - app-logs:/app/logs
      - ./config:/app/config:ro
    depends_on:
      - database

volumes:
  postgres-data:
  app-uploads:
  app-logs:
```

### What are the different types of Docker volumes?

**Answer:**
Docker provides three main types of volume mounts:

**1. Volumes (Recommended):**
```bash
# Create and manage volumes
docker volume create myvolume
docker volume ls
docker volume inspect myvolume
docker volume rm myvolume

# Use volume in container
docker run -v myvolume:/data alpine
```

**Characteristics:**
- Managed by Docker
- Stored in Docker's directory (/var/lib/docker/volumes/)
- Can use volume drivers for remote storage
- Best performance on Docker Desktop
- Easy backup and migration

**2. Bind Mounts:**
```bash
# Absolute path bind mount
docker run -v /host/data:/container/data alpine

# Relative path bind mount
docker run -v $(pwd)/data:/container/data alpine

# Read-only bind mount
docker run -v /host/data:/container/data:ro alpine
```

**Characteristics:**
- File or directory on host filesystem
- Full path must be specified
- Host manages the data
- Good for development workflows
- Direct access from host

**3. tmpfs Mounts (Linux only):**
```bash
# tmpfs mount (memory-based)
docker run --tmpfs /tmp alpine

# tmpfs with options
docker run --mount type=tmpfs,destination=/tmp,tmpfs-size=100m alpine

# Multiple tmpfs mounts
docker run \
  --tmpfs /tmp \
  --tmpfs /var/cache \
  alpine
```

**Characteristics:**
- Stored in host memory
- Not persisted to disk
- Fast access
- Automatically cleaned up
- Good for temporary data

## Docker Compose

### What is Docker Compose?

**Answer:**
Docker Compose is a tool for defining and running multi-container Docker applications. It uses a YAML file to configure application services and manages the entire application lifecycle with simple commands.

**Key features:**
- **Multi-container orchestration:** Manage multiple containers as a single application
- **Service definition:** Define services, networks, and volumes in YAML
- **Environment management:** Different configurations for different environments
- **Dependency management:** Control startup order and dependencies
- **Scaling:** Scale services up or down easily
- **Development workflow:** Streamlined development and testing

**Basic Docker Compose file structure:**
```yaml
version: '3.8'  # Compose file format version

services:       # Define application services
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

volumes:        # Define named volumes
  db-data:

networks:       # Define custom networks
  default:
    driver: bridge
```

**Common Docker Compose commands:**
```bash
# Start services
docker-compose up
docker-compose up -d  # Detached mode

# Stop services
docker-compose down
docker-compose down -v  # Remove volumes

# Build services
docker-compose build
docker-compose build --no-cache

# Scale services
docker-compose up --scale web=3

# View logs
docker-compose logs
docker-compose logs web

# Execute commands
docker-compose exec web bash

# View running services
docker-compose ps
```

### How to define multi-container applications with Docker Compose?

**Answer:**
Docker Compose allows you to define complex multi-container applications with service dependencies, networking, and data persistence:

**1. Complete Web Application Stack:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Reverse Proxy / Load Balancer
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static-files:/usr/share/nginx/html
    depends_on:
      - web
    networks:
      - frontend
      - backend

  # Web Application
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@database:5432/myapp
      - REDIS_URL=redis://redis:6379
    volumes:
      - ./web:/app
      - static-files:/app/public
    depends_on:
      - database
      - redis
    networks:
      - backend
    restart: unless-stopped

  # Background Worker
  worker:
    build:
      context: ./web
      dockerfile: Dockerfile
    command: npm run worker
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@database:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - database
      - redis
    networks:
      - backend
    restart: unless-stopped

  # Database
  database:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    restart: unless-stopped

  # Cache / Session Store
  redis:
    image: redis:6-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped

volumes:
  postgres-data:
  redis-data:
  static-files:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

**2. Development Environment:**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  web:
    build:
      context: ./web
      target: development  # Multi-stage build target
    ports:
      - "3000:3000"
      - "9229:9229"  # Node.js debugger port
    environment:
      - NODE_ENV=development
      - DEBUG=myapp:*
    volumes:
      - ./web:/app
      - /app/node_modules  # Anonymous volume for node_modules
    command: npm run dev
    stdin_open: true
    tty: true

  database:
    image: postgres:13
    ports:
      - "5432:5432"  # Expose for external tools
    environment:
      - POSTGRES_PASSWORD=dev_password
    volumes:
      - ./database/dev-data:/var/lib/postgresql/data

  mailhog:  # Email testing
    image: mailhog/mailhog:latest
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI
```

**3. Environment-specific configurations:**
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

# Testing
docker-compose -f docker-compose.yml -f docker-compose.test.yml up
```

**4. Health checks and dependencies:**
```yaml
services:
  web:
    build: .
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    depends_on:
      database:
        condition: service_healthy

  database:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```