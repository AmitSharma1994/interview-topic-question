# Docker Interview Questions

## Docker Basics

- What is Docker?
- How is Docker different from virtual machines?
- What are the key components of Docker?

## Docker Files

- What are Dockerfiles?
- Have you written any Dockerfile in your project from scratch?
- What are the best practices for writing Dockerfiles?

## Docker Images

- What is a Docker image?
- How are Docker images layered?
- How to optimize Docker image size?

## Docker Containers

- What is a Docker container?
- What is the lifecycle of a Docker container?
- How to manage container state?

## Docker Commands

- How to pull an image from Docker Hub?
- How to push an image to a local registry or Docker Hub?
- How to push an image to Amazon ECR?

## Docker Configuration

- How to override properties in Docker files for different environments?
- How to manage environment variables in Docker?
- How to handle secrets in Docker?

## Docker Networking

- What are the different networking modes in Docker?
- How do containers communicate with each other?
- How to expose container ports?

## Docker Volumes

- What are Docker volumes?
- How to persist data in Docker containers?
- What are the different types of Docker volumes?

## Docker Compose

- What is Docker Compose?
- How to define multi-container applications with Docker Compose?

## Code Examples

```dockerfile
# Example: Dockerfile for a Spring Boot application
FROM openjdk:11-jre-slim as builder
WORKDIR /app
COPY target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM openjdk:11-jre-slim
WORKDIR /app
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./

# Environment variables with defaults
ENV SERVER_PORT=8080
ENV SPRING_PROFILES_ACTIVE=prod

# Expose the application port
EXPOSE ${SERVER_PORT}

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:${SERVER_PORT}/actuator/health || exit 1

# Run the application
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

```yaml
# Example: Docker Compose file
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db
    networks:
      - app-network

  db:
    image: postgres:13
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD=apppass
      - POSTGRES_DB=appdb
    ports:
      - "5432:5432"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres-data:
```

## Resources

- [Docker Resources](https://docs.google.com/document/d/1Xg2NbNd29_nZTtBKP4cXArmbk5S1Vw-ADerFsUUSUy4/edit)