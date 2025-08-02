# Kubernetes Interview Questions and Answers

## Core Concepts

### What is Kubernetes?

**Answer:**
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications across clusters of machines.

**Key characteristics:**
- **Container orchestration:** Manages containerized applications at scale
- **Declarative configuration:** Define desired state, Kubernetes maintains it
- **Self-healing:** Automatically replaces failed containers and nodes
- **Horizontal scaling:** Scale applications up or down based on demand
- **Service discovery:** Built-in networking and load balancing
- **Rolling updates:** Deploy new versions without downtime

### What problem does Kubernetes solve?

**Answer:**
Kubernetes solves several critical problems in modern application deployment and management:

**1. Container Orchestration at Scale:**
```yaml
# With Kubernetes: Declarative management
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3  # Kubernetes maintains 3 instances
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v1
```

**2. Service Discovery and Load Balancing:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

### What is a Kubernetes cluster?

**Answer:**
A Kubernetes cluster is a set of machines (nodes) that work together to run containerized applications, consisting of control plane components and worker nodes.

**Control Plane Components:**
- **API Server:** Central management entity
- **etcd:** Distributed key-value store
- **Controller Manager:** Runs controller processes
- **Scheduler:** Assigns pods to nodes

**Worker Node Components:**
- **kubelet:** Node agent that communicates with control plane
- **kube-proxy:** Network proxy on each node
- **Container Runtime:** Runs containers (Docker, containerd, CRI-O)

## Kubernetes Components

### What are Pods in Kubernetes?

**Answer:**
A Pod is the smallest deployable unit in Kubernetes, representing one or more containers that share storage, network, and lifecycle.

**Pod specification:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0.0
    ports:
    - containerPort: 8080
    resources:
      requests:
        cpu: "100m"
        memory: "128Mi"
      limits:
        cpu: "500m"
        memory: "256Mi"
```

### What are Deployments in Kubernetes?

**Answer:**
A Deployment is a higher-level abstraction that manages ReplicaSets and provides declarative updates to Pods, enabling rolling updates, rollbacks, and scaling.

**Deployment specification:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
```

### What are Services in Kubernetes?

**Answer:**
A Service is an abstraction that defines a logical set of Pods and provides stable network access to them, enabling service discovery and load balancing.

**Service types:**

**1. ClusterIP (Default):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

**2. NodePort:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-nodeport
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

### What are StatefulSets in Kubernetes?

**Answer:**
StatefulSet is a workload API object used to manage stateful applications, providing guarantees about ordering, uniqueness, and persistent storage.

**StatefulSet specification:**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

## Configuration

### Where are Pod and Deployment configurations defined in a project?

**Answer:**
Kubernetes configurations are typically organized in YAML files within a project structure:

**Common project structure:**
```
project/
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── configmap.yaml
│   └── overlays/
│       ├── development/
│       ├── staging/
│       └── production/
└── helm/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

### How to configure resource limits for containers?

**Answer:**
Resource limits in Kubernetes control CPU and memory allocation for containers:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: myapp
    image: myapp:1.0.0
    resources:
      requests:        # Minimum guaranteed resources
        cpu: "100m"    # 0.1 CPU core
        memory: "128Mi" # 128 MiB memory
      limits:          # Maximum allowed resources
        cpu: "500m"    # 0.5 CPU core
        memory: "256Mi" # 256 MiB memory
```

### How to manage configuration in Kubernetes?

**Answer:**
Kubernetes provides several mechanisms to manage application configuration:

**1. ConfigMaps for non-sensitive configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.host: "mysql.example.com"
  database.port: "3306"
  application.properties: |
    server.port=8080
    spring.datasource.url=jdbc:mysql://mysql.example.com:3306/mydb
```

**2. Secrets for sensitive data:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
stringData:
  database.password: "password123"
  api.key: "abcdefghijk"
```

## Kubernetes Tools

### What are the tools used for Kubernetes? (kubectl, kind, minikube)

**Answer:**
Kubernetes ecosystem includes various tools for development, deployment, and management:

**1. kubectl - Kubernetes CLI:**
```bash
kubectl version                    # Check kubectl and cluster version
kubectl get pods                   # List pods
kubectl apply -f deployment.yaml   # Apply configuration
kubectl scale deployment myapp --replicas=5  # Scale deployment
```

**2. minikube - Local Kubernetes:**
```bash
minikube start                    # Start local cluster
minikube dashboard               # Open Kubernetes dashboard
minikube addons enable ingress  # Enable ingress controller
```

**3. kind - Kubernetes in Docker:**
```bash
kind create cluster                    # Create default cluster
kind create cluster --name dev-cluster # Create named cluster
```

**4. Helm - Package Manager:**
```bash
helm install myapp ./chart/
helm upgrade myapp ./chart/
helm rollback myapp 1
```

### How to use kubectl to manage Kubernetes resources?

**Answer:**
kubectl is the primary command-line tool for interacting with Kubernetes clusters:

**Resource management:**
```bash
# Create resources
kubectl apply -f deployment.yaml
kubectl create deployment nginx --image=nginx:1.21

# View resources
kubectl get pods
kubectl get deployments
kubectl describe pod myapp-pod

# Scale and update
kubectl scale deployment myapp --replicas=5
kubectl set image deployment/myapp myapp=myapp:v2

# Debug and troubleshoot
kubectl logs myapp-pod
kubectl exec -it myapp-pod -- /bin/bash
kubectl port-forward pod/myapp-pod 8080:8080
```

### What is Helm and how is it used with Kubernetes?

**Answer:**
Helm is a package manager for Kubernetes that simplifies application deployment and management through templated YAML files called charts.

**Helm Chart structure:**
```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
└── templates/          # Kubernetes manifest templates
    ├── deployment.yaml
    ├── service.yaml
    └── configmap.yaml
```

**Helm commands:**
```bash
helm create myapp                    # Create new chart
helm install myapp ./myapp/          # Install chart
helm upgrade myapp ./myapp/          # Upgrade release
helm rollback myapp 1               # Rollback to revision 1
helm uninstall myapp                # Remove release
```

## Deployment Strategies

### What are the different deployment strategies in Kubernetes?

**Answer:**
Kubernetes supports various deployment strategies:

**1. Rolling Update (Default):**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

**2. Blue-Green Deployment:**
```yaml
# Switch service selector between blue and green deployments
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' for deployment
```

**3. Canary Deployment:**
```yaml
# Deploy small percentage of new version alongside stable version
# Stable: 9 replicas, Canary: 1 replica (10% traffic)
```

### How to implement rolling updates?

**Answer:**
Rolling updates provide zero-downtime deployments:

```bash
# Update image
kubectl set image deployment/myapp myapp=myapp:2.0.0

# Monitor rollout
kubectl rollout status deployment/myapp

# Rollback if needed
kubectl rollout undo deployment/myapp
```

### How to implement blue-green deployments?

**Answer:**
Blue-green deployment maintains two identical environments:

```bash
# 1. Deploy green environment
kubectl apply -f myapp-green-deployment.yaml

# 2. Test green environment
kubectl port-forward service/myapp-green-test 8080:80

# 3. Switch traffic to green
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

# 4. Rollback if needed
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

### How to implement canary deployments?

**Answer:**
Canary deployment gradually shifts traffic from stable to new version:

```yaml
# Stable deployment (90% of replicas)
spec:
  replicas: 9

# Canary deployment (10% of replicas)  
spec:
  replicas: 1
```

## Scaling and High Availability

### How to create a replica set in Kubernetes?

**Answer:**
A ReplicaSet ensures that a specified number of pod replicas are running:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
```

### How does Kubernetes handle scaling?

**Answer:**
Kubernetes provides multiple scaling mechanisms:

**1. Manual Scaling:**
```bash
kubectl scale deployment myapp --replicas=5
```

**2. Horizontal Pod Autoscaler (HPA):**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### How to ensure high availability in Kubernetes?

**Answer:**
High availability involves multiple layers of redundancy:

**1. Multi-Zone Deployment:**
```yaml
spec:
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - myapp
              topologyKey: topology.kubernetes.io/zone
```

**2. Pod Disruption Budget:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

## Application Servers

### Can we deploy a WebLogic server in Kubernetes?

**Answer:**
Yes, WebLogic Server can be deployed in Kubernetes using Oracle's WebLogic Kubernetes Operator:

```yaml
apiVersion: weblogic.oracle/v8
kind: Domain
metadata:
  name: sample-domain1
spec:
  domainUID: sample-domain1
  image: "container-registry.oracle.com/middleware/weblogic:12.2.1.4"
  serverStartPolicy: "IF_NEEDED"
  adminServer:
    serverStartState: "RUNNING"
  clusters:
  - clusterName: cluster-1
    serverStartState: "RUNNING"
    replicas: 2
```

### What are the considerations for deploying application servers in Kubernetes?

**Answer:**
Deploying traditional application servers requires careful consideration:

**Key considerations:**
- **Resource Management:** Application servers need more CPU and memory
- **Persistent Storage:** Use StatefulSets for clustered servers
- **Configuration Management:** External configuration via ConfigMaps/Secrets
- **Health Checks:** Implement proper startup, liveness, and readiness probes
- **Networking:** Configure proper service discovery and load balancing
- **Licensing:** Ensure proper licensing for containerized deployments
- **State Management:** Handle session affinity if required
- **Monitoring:** Set up comprehensive monitoring and logging