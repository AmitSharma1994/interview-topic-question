# Kubernetes Interview Questions

## Core Concepts

- What is Kubernetes?
- What problem does Kubernetes solve?
- What is a Kubernetes cluster?

## Kubernetes Components

- What are Pods in Kubernetes?
- What are Deployments in Kubernetes?
- What are Services in Kubernetes?
- What are StatefulSets in Kubernetes?

## Configuration

- Where are Pod and Deployment configurations defined in a project?
- How to configure resource limits for containers?
- How to manage configuration in Kubernetes?

## Kubernetes Tools

- What are the tools used for Kubernetes? (kubectl, kind, minikube)
- How to use kubectl to manage Kubernetes resources?
- What is Helm and how is it used with Kubernetes?

## Deployment Strategies

- What are the different deployment strategies in Kubernetes?
- How to implement rolling updates?
- How to implement blue-green deployments?
- How to implement canary deployments?

## Scaling and High Availability

- How to create a replica set in Kubernetes?
- How does Kubernetes handle scaling?
- How to ensure high availability in Kubernetes?

## Application Servers

- Can we deploy a WebLogic server in Kubernetes?
- What are the considerations for deploying application servers in Kubernetes?

## Code Examples

```yaml
# Example: Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
  labels:
    app: spring-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: myregistry/spring-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 20
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "prod"
        - name: DB_HOST
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: db.host
```

```yaml
# Example: Kubernetes Service
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  selector:
    app: spring-app
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

## Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
- [Kubernetes Patterns](https://www.oreilly.com/library/view/kubernetes-patterns/9781492050278/)