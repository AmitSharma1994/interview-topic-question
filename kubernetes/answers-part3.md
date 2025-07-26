## Scaling and High Availability

### How to create a replica set in Kubernetes?

**Answer:**
A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, Deployments are preferred as they provide additional features like rolling updates.

**ReplicaSet specification:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: v1
  template:
    metadata:
      labels:
        app: myapp
        version: v1
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

**ReplicaSet management:**
```bash
# Create ReplicaSet
kubectl apply -f replicaset.yaml

# List ReplicaSets
kubectl get replicasets
kubectl get rs

# Describe ReplicaSet
kubectl describe replicaset myapp-replicaset

# Scale ReplicaSet
kubectl scale replicaset myapp-replicaset --replicas=5

# Delete ReplicaSet
kubectl delete replicaset myapp-replicaset

# Delete ReplicaSet but keep pods
kubectl delete replicaset myapp-replicaset --cascade=orphan
```

**ReplicaSet vs Deployment:**
```yaml
# Deployment (Recommended)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
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
        ports:
        - containerPort: 8080
```

**Why use Deployments over ReplicaSets:**
- **Rolling updates:** Gradual pod replacement
- **Rollback capability:** Revert to previous versions
- **Update strategies:** Configure update behavior
- **Revision history:** Track deployment changes
- **Pause/resume:** Control deployment process

### How does Kubernetes handle scaling?

**Answer:**
Kubernetes provides multiple scaling mechanisms to handle varying workloads:

**1. Manual Scaling:**
```bash
# Scale deployment
kubectl scale deployment myapp --replicas=5

# Scale ReplicaSet
kubectl scale replicaset myapp-rs --replicas=3

# Scale StatefulSet
kubectl scale statefulset mysql --replicas=3
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
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 4
        periodSeconds: 15
      selectPolicy: Max
```

**HPA management:**
```bash
# Create HPA
kubectl autoscale deployment myapp --cpu-percent=70 --min=2 --max=10

# List HPAs
kubectl get hpa

# Describe HPA
kubectl describe hpa myapp-hpa

# Delete HPA
kubectl delete hpa myapp-hpa
```

**3. Vertical Pod Autoscaler (VPA):**
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # Auto, Off, Initial
  resourcePolicy:
    containerPolicies:
    - containerName: myapp
      maxAllowed:
        cpu: 1
        memory: 500Mi
      minAllowed:
        cpu: 100m
        memory: 50Mi
      controlledResources: ["cpu", "memory"]
```

**4. Cluster Autoscaler:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/kubernetes
```

**5. Custom Metrics Scaling:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metric:
        name: queue_messages_ready
      target:
        type: AverageValue
        averageValue: "30"
  - type: External
    external:
      metric:
        name: pubsub.googleapis.com|subscription|num_undelivered_messages
        selector:
          matchLabels:
            resource.labels.subscription_id: myapp-subscription
      target:
        type: AverageValue
        averageValue: "30"
```

### How to ensure high availability in Kubernetes?

**Answer:**
High availability in Kubernetes involves multiple layers of redundancy and fault tolerance:

**1. Multi-Zone Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 6
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # Spread pods across zones
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
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 8080
```

**2. Pod Disruption Budget:**
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2  # or maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

**3. Resource Requests and Limits:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 3
```

**4. Multi-Region Setup:**
```yaml
# Primary region deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-primary
  namespace: primary-region
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      region: primary
  template:
    metadata:
      labels:
        app: myapp
        region: primary
    spec:
      nodeSelector:
        topology.kubernetes.io/region: us-east-1
      containers:
      - name: myapp
        image: myapp:1.0.0

---
# Secondary region deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-secondary
  namespace: secondary-region
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      region: secondary
  template:
    metadata:
      labels:
        app: myapp
        region: secondary
    spec:
      nodeSelector:
        topology.kubernetes.io/region: us-west-2
      containers:
      - name: myapp
        image: myapp:1.0.0
```

**5. Database High Availability:**
```yaml
# StatefulSet for database cluster
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-cluster
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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - mysql
            topologyKey: kubernetes.io/hostname
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
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
          storage: 20Gi
      storageClassName: fast-ssd
```

**6. Ingress and Load Balancing:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port:
              number: 80
```

**7. Monitoring and Alerting:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: myapp-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics

---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: myapp-alerts
spec:
  groups:
  - name: myapp.rules
    rules:
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: High error rate detected
    - alert: PodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: Pod is crash looping
```

## Application Servers

### Can we deploy a WebLogic server in Kubernetes?

**Answer:**
Yes, WebLogic Server can be deployed in Kubernetes using Oracle's WebLogic Kubernetes Operator, which provides enterprise-grade management capabilities.

**WebLogic Kubernetes Operator setup:**
```bash
# Install WebLogic Kubernetes Operator
kubectl create namespace weblogic-operator-ns
kubectl create serviceaccount -n weblogic-operator-ns weblogic-operator

# Create RBAC
kubectl apply -f https://raw.githubusercontent.com/oracle/weblogic-kubernetes-operator/main/kubernetes/charts/weblogic-operator/templates/main-rbac.yaml

# Install operator using Helm
helm repo add weblogic-operator https://oracle.github.io/weblogic-kubernetes-operator/charts
helm install weblogic-operator weblogic-operator/weblogic-operator \
  --namespace weblogic-operator-ns \
  --set image=ghcr.io/oracle/weblogic-kubernetes-operator:3.4.0
```

**WebLogic Domain configuration:**
```yaml
apiVersion: weblogic.oracle/v8
kind: Domain
metadata:
  name: sample-domain1
  namespace: sample-domain1-ns
  labels:
    weblogic.domainUID: sample-domain1
spec:
  domainUID: sample-domain1
  domainHome: /u01/oracle/user_projects/domains/sample-domain1
  image: "container-registry.oracle.com/middleware/weblogic:12.2.1.4"
  imagePullPolicy: IfNotPresent
  imagePullSecrets:
  - name: ocirsecret
  webLogicCredentialsSecret:
    name: sample-domain1-weblogic-credentials
  includeServerOutInPodLog: true
  serverStartPolicy: "IF_NEEDED"
  serverPod:
    env:
    - name: JAVA_OPTIONS
      value: "-Dweblogic.StdoutDebugEnabled=false"
    - name: USER_MEM_ARGS
      value: "-Djava.security.egd=file:/dev/./urandom -Xms256m -Xmx512m"
    resources:
      requests:
        cpu: "250m"
        memory: "768Mi"
      limits:
        cpu: "500m"
        memory: "1Gi"
  adminServer:
    serverStartState: "RUNNING"
    adminService:
      channels:
      - channelName: default
        nodePort: 30701
  clusters:
  - clusterName: cluster-1
    serverStartState: "RUNNING"
    replicas: 2
    serverPod:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: "weblogic.clusterName"
                  operator: In
                  values:
                  - $(CLUSTER_NAME)
              topologyKey: "kubernetes.io/hostname"
```

**WebLogic Secret configuration:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sample-domain1-weblogic-credentials
  namespace: sample-domain1-ns
  labels:
    weblogic.domainUID: sample-domain1
type: Opaque
data:
  username: d2VibG9naWM=  # weblogic (base64)
  password: V2VsY29tZTE=  # Welcome1 (base64)
```

**WebLogic Service configuration:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sample-domain1-admin-server
  namespace: sample-domain1-ns
  labels:
    weblogic.domainUID: sample-domain1
    weblogic.serverName: admin-server
spec:
  type: NodePort
  selector:
    weblogic.domainUID: sample-domain1
    weblogic.serverName: admin-server
  ports:
  - name: default
    port: 7001
    protocol: TCP
    targetPort: 7001
    nodePort: 30701

---
apiVersion: v1
kind: Service
metadata:
  name: sample-domain1-cluster-cluster-1
  namespace: sample-domain1-ns
  labels:
    weblogic.domainUID: sample-domain1
    weblogic.clusterName: cluster-1
spec:
  type: ClusterIP
  selector:
    weblogic.domainUID: sample-domain1
    weblogic.clusterName: cluster-1
  ports:
  - name: default
    port: 8001
    protocol: TCP
    targetPort: 8001
```

### What are the considerations for deploying application servers in Kubernetes?

**Answer:**
Deploying traditional application servers in Kubernetes requires careful consideration of several factors:

**1. Containerization Strategy:**
```dockerfile
# WebLogic Dockerfile example
FROM container-registry.oracle.com/middleware/weblogic:12.2.1.4

# Copy domain configuration
COPY --chown=oracle:root domain-home/ ${DOMAIN_HOME}/

# Copy applications
COPY --chown=oracle:root applications/ ${DOMAIN_HOME}/applications/

# Set environment variables
ENV JAVA_OPTIONS="-Dweblogic.StdoutDebugEnabled=false"
ENV USER_MEM_ARGS="-Xms512m -Xmx1024m -XX:+UseG1GC"

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5m \
  CMD curl -f http://localhost:7001/weblogic/ready || exit 1

EXPOSE 7001 8001
```

**2. Resource Management:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: weblogic-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: weblogic-server
  template:
    metadata:
      labels:
        app: weblogic-server
    spec:
      containers:
      - name: weblogic
        image: weblogic:12.2.1.4
        resources:
          requests:
            cpu: "1000m"      # Application servers need more CPU
            memory: "2Gi"     # Higher memory requirements
          limits:
            cpu: "2000m"
            memory: "4Gi"
        env:
        - name: JAVA_OPTIONS
          value: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
```

**3. Persistent Storage:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: weblogic-domain-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: weblogic-cluster
spec:
  serviceName: weblogic-headless
  replicas: 3
  selector:
    matchLabels:
      app: weblogic
  template:
    metadata:
      labels:
        app: weblogic
    spec:
      containers:
      - name: weblogic
        image: weblogic:12.2.1.4
        volumeMounts:
        - name: domain-storage
          mountPath: /u01/oracle/user_projects/domains
        - name: shared-storage
          mountPath: /shared
  volumeClaimTemplates:
  - metadata:
      name: domain-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 5Gi
```

**4. Configuration Management:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: weblogic-config
data:
  domain.properties: |
    domain.name=mydomain
    admin.server.name=AdminServer
    admin.server.port=7001
    cluster.name=MyCluster
    managed.server.port=8001
    
  datasource.properties: |
    datasource.url=jdbc:oracle:thin:@database:1521:XE
    datasource.driver=oracle.jdbc.OracleDriver
    connection.pool.initial=5
    connection.pool.max=20

---
apiVersion: v1
kind: Secret
metadata:
  name: weblogic-secrets
type: Opaque
stringData:
  admin.username: weblogic
  admin.password: Welcome123
  db.username: dbuser
  db.password: dbpassword
```

**5. Health Checks and Monitoring:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: weblogic-server
spec:
  template:
    spec:
      containers:
      - name: weblogic
        image: weblogic:12.2.1.4
        ports:
        - containerPort: 7001
          name: admin
        - containerPort: 8001
          name: managed
        
        # Startup probe for slow-starting servers
        startupProbe:
          httpGet:
            path: /weblogic/ready
            port: 7001
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 20  # 10 minutes max startup
        
        # Liveness probe
        livenessProbe:
          httpGet:
            path: /weblogic/ready
            port: 7001
          initialDelaySeconds: 300
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        # Readiness probe
        readinessProbe:
          httpGet:
            path: /weblogic/ready
            port: 7001
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
```

**6. Networking and Service Discovery:**
```yaml
# Headless service for StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: weblogic-headless
spec:
  clusterIP: None
  selector:
    app: weblogic
  ports:
  - name: admin
    port: 7001
    targetPort: 7001
  - name: managed
    port: 8001
    targetPort: 8001

---
# Load balancer service
apiVersion: v1
kind: Service
metadata:
  name: weblogic-lb
spec:
  type: LoadBalancer
  selector:
    app: weblogic
  ports:
  - name: http
    port: 80
    targetPort: 8001
  - name: admin
    port: 7001
    targetPort: 7001
```

**Key considerations:**
- **Licensing:** Ensure proper licensing for containerized deployments
- **State management:** Use StatefulSets for clustered application servers
- **Session affinity:** Configure sticky sessions if required
- **Security:** Implement proper RBAC and network policies
- **Monitoring:** Set up comprehensive monitoring and logging
- **Backup:** Implement backup strategies for persistent data
- **Performance:** Tune JVM settings for containerized environments
- **Migration:** Plan gradual migration from traditional deployments