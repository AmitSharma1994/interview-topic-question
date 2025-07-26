## Kubernetes Tools

### What are the tools used for Kubernetes? (kubectl, kind, minikube)

**Answer:**
Kubernetes ecosystem includes various tools for development, deployment, and management:

**1. kubectl - Kubernetes CLI:**
```bash
# Installation
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Basic commands
kubectl version                    # Check kubectl and cluster version
kubectl cluster-info              # Display cluster information
kubectl config view               # View kubeconfig
kubectl config current-context   # Show current context
kubectl config use-context prod  # Switch context
```

**2. minikube - Local Kubernetes:**
```bash
# Installation and setup
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Cluster management
minikube start                    # Start local cluster
minikube start --cpus=4 --memory=8192  # Start with specific resources
minikube stop                     # Stop cluster
minikube delete                   # Delete cluster
minikube status                   # Check cluster status

# Features
minikube dashboard               # Open Kubernetes dashboard
minikube addons list            # List available addons
minikube addons enable ingress  # Enable ingress controller
minikube tunnel                 # Expose LoadBalancer services
```

**3. kind - Kubernetes in Docker:**
```bash
# Installation
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Cluster management
kind create cluster                    # Create default cluster
kind create cluster --name dev-cluster # Create named cluster
kind get clusters                      # List clusters
kind delete cluster --name dev-cluster # Delete cluster

# Multi-node cluster configuration
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker

kind create cluster --config kind-config.yaml
```

**4. Helm - Package Manager:**
```bash
# Installation
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Repository management
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm search repo nginx

# Chart management
helm install my-nginx stable/nginx-ingress
helm list
helm upgrade my-nginx stable/nginx-ingress
helm rollback my-nginx 1
helm uninstall my-nginx
```

**5. kustomize - Configuration Management:**
```bash
# Built into kubectl
kubectl apply -k ./overlays/production

# Standalone installation
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash

# Usage
kustomize build ./overlays/production | kubectl apply -f -
```

**6. Development Tools:**
```bash
# Skaffold - Continuous development
skaffold dev                     # Watch and deploy changes
skaffold run                     # Build and deploy once

# Telepresence - Local development
telepresence connect            # Connect to cluster
telepresence intercept myservice # Intercept service traffic

# Stern - Multi-pod log tailing
stern myapp                     # Tail logs from all myapp pods
stern --selector app=myapp      # Tail logs by label selector
```

### How to use kubectl to manage Kubernetes resources?

**Answer:**
kubectl is the primary command-line tool for interacting with Kubernetes clusters:

**Basic kubectl syntax:**
```bash
kubectl [command] [TYPE] [NAME] [flags]

# Examples
kubectl get pods
kubectl describe deployment myapp
kubectl delete service myapp-service
```

**Resource management:**
```bash
# Create resources
kubectl create deployment nginx --image=nginx:1.21
kubectl apply -f deployment.yaml
kubectl apply -k ./kustomize/

# View resources
kubectl get pods                          # List pods
kubectl get pods -o wide                  # Detailed pod info
kubectl get pods --show-labels           # Show labels
kubectl get pods -l app=myapp            # Filter by labels
kubectl get all                          # All resources in namespace

# Describe resources
kubectl describe pod myapp-pod
kubectl describe deployment myapp
kubectl describe service myapp-service

# Edit resources
kubectl edit deployment myapp
kubectl patch deployment myapp -p '{"spec":{"replicas":5}}'

# Delete resources
kubectl delete pod myapp-pod
kubectl delete deployment myapp
kubectl delete -f deployment.yaml
```

**Namespace management:**
```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace myapp
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
EOF

# Set default namespace
kubectl config set-context --current --namespace=myapp

# Work with specific namespace
kubectl get pods -n kube-system
kubectl apply -f deployment.yaml -n myapp
```

**Scaling and updates:**
```bash
# Scale deployment
kubectl scale deployment myapp --replicas=5
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80

# Rolling updates
kubectl set image deployment/myapp container=myapp:v2
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp
```

**Debugging and troubleshooting:**
```bash
# Logs
kubectl logs myapp-pod
kubectl logs myapp-pod -c container-name    # Multi-container pod
kubectl logs -f myapp-pod                   # Follow logs
kubectl logs --previous myapp-pod           # Previous container logs

# Execute commands
kubectl exec -it myapp-pod -- /bin/bash
kubectl exec myapp-pod -- ls /app
kubectl exec myapp-pod -c container-name -- env

# Port forwarding
kubectl port-forward pod/myapp-pod 8080:8080
kubectl port-forward service/myapp-service 8080:80
kubectl port-forward deployment/myapp 8080:8080

# Copy files
kubectl cp myapp-pod:/app/config.json ./config.json
kubectl cp ./config.json myapp-pod:/app/config.json
```

**Advanced kubectl usage:**
```bash
# JSONPath queries
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{.items[*].status.podIP}'

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase,IP:.status.podIP

# Watch resources
kubectl get pods --watch
kubectl get events --watch

# Dry run
kubectl apply -f deployment.yaml --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --dry-run=server -o yaml

# Resource usage
kubectl top nodes
kubectl top pods
kubectl top pods --containers
```

### What is Helm and how is it used with Kubernetes?

**Answer:**
Helm is a package manager for Kubernetes that simplifies application deployment and management through templated YAML files called charts.

**Key concepts:**
- **Chart:** Package of pre-configured Kubernetes resources
- **Release:** Instance of a chart deployed to cluster
- **Repository:** Collection of charts
- **Values:** Configuration parameters for charts

**Helm Chart structure:**
```
mychart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── charts/             # Chart dependencies
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Template helpers
│   └── NOTES.txt       # Post-install notes
└── .helmignore         # Files to ignore
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: A Helm chart for my application
type: application
version: 0.1.0          # Chart version
appVersion: "1.0.0"     # Application version
dependencies:
- name: postgresql
  version: 11.6.12
  repository: https://charts.bitnami.com/bitnami
  condition: postgresql.enabled
```

**values.yaml:**
```yaml
# Default values for myapp
replicaCount: 2

image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: myapp.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 100m
    memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

postgresql:
  enabled: true
  auth:
    postgresPassword: "password"
    database: "myapp"
```

**Template example (deployment.yaml):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.postgresql.enabled }}
          env:
            - name: DB_HOST
              value: {{ include "myapp.fullname" . }}-postgresql
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "myapp.fullname" . }}-postgresql
                  key: postgres-password
          {{- end }}
```

**Helm commands:**
```bash
# Chart development
helm create myapp                    # Create new chart
helm lint myapp/                     # Validate chart
helm template myapp ./myapp/         # Render templates locally
helm package myapp/                  # Package chart

# Installation and management
helm install myapp ./myapp/                    # Install from local chart
helm install myapp bitnami/nginx              # Install from repository
helm install myapp ./myapp/ -f custom-values.yaml  # Custom values
helm install myapp ./myapp/ --set replicaCount=3   # Override values

# Release management
helm list                           # List releases
helm status myapp                   # Show release status
helm get values myapp              # Show values used
helm get manifest myapp            # Show generated manifests

# Updates and rollbacks
helm upgrade myapp ./myapp/ -f new-values.yaml
helm rollback myapp 1              # Rollback to revision 1
helm history myapp                 # Show release history

# Cleanup
helm uninstall myapp               # Remove release
helm uninstall myapp --keep-history  # Keep release history
```

**Environment-specific values:**
```yaml
# values-dev.yaml
replicaCount: 1
image:
  tag: "dev"
resources:
  limits:
    cpu: 200m
    memory: 256Mi

# values-prod.yaml
replicaCount: 5
image:
  tag: "1.0.0"
ingress:
  enabled: true
  hosts:
    - host: myapp.company.com
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
```

```bash
# Deploy to different environments
helm install myapp-dev ./myapp/ -f values-dev.yaml
helm install myapp-prod ./myapp/ -f values-prod.yaml
```

## Deployment Strategies

### What are the different deployment strategies in Kubernetes?

**Answer:**
Kubernetes supports various deployment strategies to minimize downtime and risk during application updates:

**1. Rolling Update (Default):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Max pods above desired count
      maxUnavailable: 1  # Max pods below desired count
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

**Characteristics:**
- Gradual replacement of old pods with new ones
- Zero downtime deployment
- Configurable surge and unavailability limits
- Automatic rollback on failure

**2. Recreate Strategy:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2
```

**Characteristics:**
- Terminates all old pods before creating new ones
- Causes downtime during deployment
- Useful for applications that can't run multiple versions
- Faster deployment than rolling update

**3. Blue-Green Deployment:**
```yaml
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1

---
# Green deployment (new)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2

---
# Service switches between blue and green
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
    version: blue  # Switch to 'green' for deployment
  ports:
  - port: 80
    targetPort: 8080
```

**4. Canary Deployment:**
```yaml
# Stable deployment (90% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: myapp
        image: myapp:v1

---
# Canary deployment (10% traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: myapp
        image: myapp:v2

---
# Service routes to both versions
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp  # Routes to both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

**5. A/B Testing Deployment:**
```yaml
# Using Istio for traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  http:
  - match:
    - headers:
        user-type:
          exact: premium
    route:
    - destination:
        host: myapp-service
        subset: v2
  - route:
    - destination:
        host: myapp-service
        subset: v1
      weight: 80
    - destination:
        host: myapp-service
        subset: v2
      weight: 20
```

### How to implement rolling updates?

**Answer:**
Rolling updates are the default deployment strategy in Kubernetes, providing zero-downtime deployments:

**Rolling update configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Can have 7 pods total (5 + 2)
      maxUnavailable: 1  # Minimum 4 pods available (5 - 1)
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

**Performing rolling updates:**
```bash
# Update image
kubectl set image deployment/myapp myapp=myapp:2.0.0

# Update using patch
kubectl patch deployment myapp -p '{"spec":{"template":{"spec":{"containers":[{"name":"myapp","image":"myapp:2.0.0"}]}}}}'

# Update using apply
kubectl apply -f updated-deployment.yaml

# Monitor rollout
kubectl rollout status deployment/myapp
kubectl get pods -w  # Watch pod changes
```

**Rollout management:**
```bash
# Check rollout status
kubectl rollout status deployment/myapp

# View rollout history
kubectl rollout history deployment/myapp
kubectl rollout history deployment/myapp --revision=2

# Pause rollout
kubectl rollout pause deployment/myapp

# Resume rollout
kubectl rollout resume deployment/myapp

# Rollback to previous version
kubectl rollout undo deployment/myapp

# Rollback to specific revision
kubectl rollout undo deployment/myapp --to-revision=2
```

**Rolling update with health checks:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # Ensure no downtime
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
        ports:
        - containerPort: 8080
        
        # Readiness probe ensures pod is ready before receiving traffic
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        
        # Liveness probe restarts unhealthy pods
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Startup probe for slow-starting applications
        startupProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30  # 5 minutes max startup time
```

### How to implement blue-green deployments?

**Answer:**
Blue-green deployment maintains two identical production environments, switching traffic between them for zero-downtime deployments:

**Blue-green setup:**
```yaml
# Blue environment (current production)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: VERSION
          value: "blue"

---
# Green environment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
        ports:
        - containerPort: 8080
        env:
        - name: VERSION
          value: "green"

---
# Production service (points to active environment)
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  labels:
    app: myapp
spec:
  selector:
    app: myapp
    version: blue  # Currently pointing to blue
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer

---
# Testing service (for green environment validation)
apiVersion: v1
kind: Service
metadata:
  name: myapp-green-test
  labels:
    app: myapp
spec:
  selector:
    app: myapp
    version: green
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

**Blue-green deployment process:**
```bash
# 1. Deploy green environment
kubectl apply -f myapp-green-deployment.yaml

# 2. Wait for green deployment to be ready
kubectl rollout status deployment/myapp-green

# 3. Test green environment
kubectl port-forward service/myapp-green-test 8080:80
# Test application on localhost:8080

# 4. Switch traffic to green (update service selector)
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"green"}}}'

# 5. Verify traffic switch
kubectl get service myapp-service -o yaml

# 6. Monitor application health
kubectl get pods -l version=green
kubectl logs -l version=green

# 7. If issues occur, rollback to blue
kubectl patch service myapp-service -p '{"spec":{"selector":{"version":"blue"}}}'

# 8. After successful deployment, clean up blue environment
kubectl delete deployment myapp-blue
```

**Automated blue-green with Argo Rollouts:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollout
spec:
  replicas: 3
  strategy:
    blueGreen:
      activeService: myapp-active
      previewService: myapp-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: myapp-preview
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: myapp-active
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
        image: myapp:2.0.0
        ports:
        - containerPort: 8080
```

### How to implement canary deployments?

**Answer:**
Canary deployment gradually shifts traffic from the stable version to the new version, allowing for risk mitigation:

**Basic canary deployment:**
```yaml
# Stable version (90% of replicas)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-stable
  labels:
    app: myapp
    version: stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080

---
# Canary version (10% of replicas)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-canary
  labels:
    app: myapp
    version: canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: myapp
        image: myapp:2.0.0
        ports:
        - containerPort: 8080

---
# Service routes to both versions based on replica count
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp  # Selects both stable and canary pods
  ports:
  - port: 80
    targetPort: 8080
```

**Advanced canary with Istio:**
```yaml
# Virtual Service for traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp-service
        subset: canary
  - route:
    - destination:
        host: myapp-service
        subset: stable
      weight: 95
    - destination:
        host: myapp-service
        subset: canary
      weight: 5

---
# Destination Rule for subsets
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-dr
spec:
  host: myapp-service
  subsets:
  - name: stable
    labels:
      version: stable
  - name: canary
    labels:
      version: canary
```

**Canary deployment with Argo Rollouts:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp-rollout
spec:
  replicas: 10
  strategy:
    canary:
      steps:
      - setWeight: 10    # 10% traffic to canary
      - pause: {duration: 1m}
      - setWeight: 20    # 20% traffic to canary
      - pause: {duration: 1m}
      - setWeight: 50    # 50% traffic to canary
      - pause: {duration: 1m}
      - setWeight: 100   # 100% traffic to canary
      canaryService: myapp-canary
      stableService: myapp-stable
      trafficRouting:
        istio:
          virtualService:
            name: myapp-vs
          destinationRule:
            name: myapp-dr
            canarySubsetName: canary
            stableSubsetName: stable
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2
        args:
        - name: service-name
          value: myapp-canary
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
        image: myapp:2.0.0
```

**Canary deployment process:**
```bash
# 1. Deploy canary version
kubectl apply -f myapp-canary.yaml

# 2. Monitor canary metrics
kubectl get pods -l version=canary
kubectl logs -l version=canary

# 3. Gradually increase canary traffic
kubectl patch deployment myapp-canary -p '{"spec":{"replicas":2}}'  # 20%
kubectl patch deployment myapp-stable -p '{"spec":{"replicas":8}}'

# 4. Monitor application metrics and errors
# Check error rates, response times, business metrics

# 5. If successful, promote canary to stable
kubectl patch deployment myapp-stable -p '{"spec":{"template":{"spec":{"containers":[{"name":"myapp","image":"myapp:2.0.0"}]}}}}'
kubectl delete deployment myapp-canary

# 6. If issues detected, rollback
kubectl delete deployment myapp-canary
# Traffic automatically goes back to stable version
```