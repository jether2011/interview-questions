# Docker & Kubernetes Interview Questions

## Table of Contents
1. [Docker Fundamentals](#docker-fundamentals)
2. [Docker Images & Containers](#docker-images--containers)
3. [Docker Networking](#docker-networking)
4. [Docker Compose](#docker-compose)
5. [Kubernetes Architecture](#kubernetes-architecture)
6. [Kubernetes Objects](#kubernetes-objects)
7. [Kubernetes Networking](#kubernetes-networking)
8. [Kubernetes Storage](#kubernetes-storage)
9. [Kubernetes Security](#kubernetes-security)
10. [Deployment & Scaling](#deployment--scaling)
11. [Monitoring & Troubleshooting](#monitoring--troubleshooting)

## Docker Fundamentals

### Q1: What is Docker and how does it differ from virtualization?
**Answer:**
Docker is a containerization platform that packages applications with dependencies.

**Docker vs Virtual Machines:**
| Docker | Virtual Machine |
|--------|-----------------|
| OS-level virtualization | Hardware-level virtualization |
| Shares host kernel | Own OS kernel |
| Lightweight (MB) | Heavy (GB) |
| Starts in seconds | Starts in minutes |
| Less isolation | Complete isolation |
| Better resource utilization | More overhead |

### Q2: Explain Docker architecture.
**Answer:**
**Components:**
1. **Docker Client**: CLI or API calls
2. **Docker Daemon**: Manages Docker objects
3. **Docker Registry**: Stores images (Docker Hub)
4. **Docker Objects**:
   - Images: Read-only templates
   - Containers: Running instances
   - Networks: Container communication
   - Volumes: Persistent storage

### Q3: What is the difference between Docker image and container?
**Answer:**
- **Image**: Read-only template with application and dependencies
- **Container**: Running instance of an image with writable layer

```bash
# Image is like a class
docker pull nginx:latest

# Container is like an object/instance
docker run -d --name my-nginx nginx:latest
```

## Docker Images & Containers

### Q4: How to optimize Docker images?
**Answer:**
**Best Practices:**
1. **Multi-stage builds**: Reduce final image size
2. **Layer caching**: Order commands efficiently
3. **Minimal base images**: Alpine, distroless
4. **Combine RUN commands**: Reduce layers
5. **.dockerignore**: Exclude unnecessary files

**Example Dockerfile:**
```dockerfile
# Multi-stage build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# Final stage
FROM openjdk:17-jre-slim
WORKDIR /app
COPY --from=builder /app/target/app.jar app.jar
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Q5: Explain Docker layering system.
**Answer:**
Docker images consist of read-only layers:
- Each instruction creates a layer
- Layers are cached and reused
- Container adds writable layer on top

```dockerfile
FROM ubuntu:20.04        # Layer 1
RUN apt-get update      # Layer 2
RUN apt-get install -y curl  # Layer 3
COPY app.jar /app/      # Layer 4
CMD ["java", "-jar", "/app/app.jar"]  # Metadata, not layer
```

### Q6: What is the difference between CMD and ENTRYPOINT?
**Answer:**
- **CMD**: Default command, can be overridden
- **ENTRYPOINT**: Fixed command, CMD becomes arguments

```dockerfile
# CMD example
FROM ubuntu
CMD ["echo", "Hello"]
# docker run myimage → Hello
# docker run myimage echo "World" → World

# ENTRYPOINT example
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello"]
# docker run myimage → Hello
# docker run myimage "World" → World

# Combined
ENTRYPOINT ["java", "-jar"]
CMD ["app.jar"]
# docker run myimage → java -jar app.jar
# docker run myimage other.jar → java -jar other.jar
```

### Q7: How to manage container lifecycle?
**Answer:**
```bash
# Create and start
docker run -d --name myapp nginx

# Container states
docker ps           # Running containers
docker ps -a        # All containers

# Lifecycle commands
docker start myapp  # Start stopped container
docker stop myapp   # Graceful stop (SIGTERM)
docker kill myapp   # Force stop (SIGKILL)
docker restart myapp # Stop and start
docker pause myapp  # Freeze processes
docker unpause myapp # Resume processes
docker rm myapp     # Remove container

# Auto-restart policies
docker run -d --restart=always nginx
# Options: no, on-failure[:max-retries], always, unless-stopped
```

## Docker Networking

### Q8: Explain Docker networking modes.
**Answer:**
1. **Bridge** (default): Private network on host
2. **Host**: Share host's network stack
3. **None**: No networking
4. **Overlay**: Multi-host networking
5. **Macvlan**: Assign MAC address

```bash
# Bridge network
docker run -d --network bridge nginx

# Host network
docker run -d --network host nginx

# Custom network
docker network create mynetwork
docker run -d --network mynetwork --name app1 nginx
docker run -d --network mynetwork --name app2 nginx
# app1 and app2 can communicate by name
```

### Q9: How do containers communicate?
**Answer:**
**Same Network:**
- Use container names as hostnames
- Docker's embedded DNS server

**Different Networks:**
- Connect container to multiple networks
- Use host networking
- Port mapping

```bash
# Port mapping
docker run -d -p 8080:80 nginx
# Host port 8080 → Container port 80

# Link networks
docker network connect network2 container1
```

## Docker Compose

### Q10: What is Docker Compose?
**Answer:**
Docker Compose defines and runs multi-container applications using YAML.

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=database
    depends_on:
      - database
    networks:
      - app-network
    volumes:
      - ./app:/app
      - app-data:/data

  database:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

  redis:
    image: redis:alpine
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
  app-data:
```

### Q11: Docker Compose commands?
**Answer:**
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f service-name

# Scale service
docker-compose up -d --scale web=3

# Execute command in service
docker-compose exec web bash

# Build images
docker-compose build

# Pull images
docker-compose pull
```

## Kubernetes Architecture

### Q12: Explain Kubernetes architecture.
**Answer:**
**Control Plane Components:**
1. **API Server**: Central management point
2. **etcd**: Distributed key-value store
3. **Scheduler**: Assigns pods to nodes
4. **Controller Manager**: Runs controllers
5. **Cloud Controller Manager**: Cloud provider integration

**Node Components:**
1. **kubelet**: Manages pods on node
2. **kube-proxy**: Network proxy
3. **Container Runtime**: Docker/containerd

### Q13: What is a Pod in Kubernetes?
**Answer:**
Pod is the smallest deployable unit in Kubernetes.

**Characteristics:**
- One or more containers
- Shared network and storage
- Ephemeral by nature
- Single IP address

**Pod manifest:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo logs; sleep 5; done']
```

### Q14: Explain Kubernetes objects hierarchy.
**Answer:**
```
Cluster
├── Namespace
│   ├── Deployment
│   │   ├── ReplicaSet
│   │   │   └── Pod
│   │   │       └── Container
│   ├── Service
│   ├── ConfigMap
│   ├── Secret
│   └── PersistentVolumeClaim
└── PersistentVolume
```

## Kubernetes Objects

### Q15: What is a Deployment?
**Answer:**
Deployment manages ReplicaSets and provides declarative updates to Pods.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Q16: Explain different Service types.
**Answer:**
1. **ClusterIP** (default): Internal cluster IP
2. **NodePort**: Exposes on each node's IP
3. **LoadBalancer**: Cloud provider load balancer
4. **ExternalName**: Maps to external DNS

```yaml
# ClusterIP
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80

# NodePort
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # 30000-32767

# LoadBalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
```

### Q17: What are ConfigMaps and Secrets?
**Answer:**
**ConfigMap**: Store non-sensitive configuration
**Secret**: Store sensitive data (base64 encoded)

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database.url: "jdbc:postgresql://db:5432/mydb"
  app.properties: |
    server.port=8080
    logging.level=INFO

# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=

# Using in Pod
spec:
  containers:
  - name: app
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret
    volumeMounts:
    - name: config
      mountPath: /config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Q18: What is StatefulSet?
**Answer:**
StatefulSet manages stateful applications with:
- Stable network identities
- Ordered deployment/scaling
- Persistent storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### Q19: What is DaemonSet?
**Answer:**
DaemonSet ensures a pod runs on all (or selected) nodes.

Use cases:
- Node monitoring agents
- Log collectors
- Network plugins

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:latest
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

## Kubernetes Networking

### Q20: How does Kubernetes networking work?
**Answer:**
**Requirements:**
1. Every pod gets unique IP
2. Pods communicate without NAT
3. Nodes communicate with pods without NAT

**Components:**
- **CNI Plugins**: Calico, Flannel, Weave
- **Service Discovery**: CoreDNS
- **kube-proxy**: Load balancing

### Q21: What is Ingress?
**Answer:**
Ingress manages external access to services, typically HTTP/HTTPS.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Q22: Explain NetworkPolicy.
**Answer:**
NetworkPolicy controls traffic flow between pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-netpolicy
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
```

## Kubernetes Storage

### Q23: Explain Persistent Volumes and Claims.
**Answer:**
**PersistentVolume (PV)**: Cluster storage resource
**PersistentVolumeClaim (PVC)**: Request for storage

```yaml
# PersistentVolume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /mnt/data

# PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard

# Using in Pod
spec:
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: pvc-data
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
```

### Q24: What are StorageClasses?
**Answer:**
StorageClass provides dynamic provisioning of storage.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

## Kubernetes Security

### Q25: How to implement RBAC in Kubernetes?
**Answer:**
Role-Based Access Control manages permissions.

```yaml
# Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io

# ClusterRole (cluster-wide)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

### Q26: What are Pod Security Policies?
**Answer:**
Pod Security Policies control security-sensitive aspects of pods.

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'secret'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

### Q27: How to manage Secrets securely?
**Answer:**
**Best Practices:**
1. Use external secret managers (Vault, AWS Secrets Manager)
2. Enable encryption at rest
3. Use RBAC to limit access
4. Rotate secrets regularly
5. Use Sealed Secrets or External Secrets Operator

```yaml
# External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: app-secret
  data:
  - secretKey: password
    remoteRef:
      key: secret/data/database
      property: password
```

## Deployment & Scaling

### Q28: How to perform rolling updates?
**Answer:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Extra pods during update
      maxUnavailable: 1  # Pods that can be unavailable
  template:
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

```bash
# Update image
kubectl set image deployment/nginx nginx=nginx:1.22

# Check rollout status
kubectl rollout status deployment/nginx

# Rollback
kubectl rollout undo deployment/nginx

# History
kubectl rollout history deployment/nginx
```

### Q29: What is Horizontal Pod Autoscaler?
**Answer:**
HPA automatically scales pods based on metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
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
```

### Q30: What is Helm?
**Answer:**
Helm is a package manager for Kubernetes.

**Components:**
- **Chart**: Package of pre-configured resources
- **Release**: Instance of a chart
- **Repository**: Collection of charts

```bash
# Install chart
helm install myapp ./mychart

# Upgrade release
helm upgrade myapp ./mychart

# Values file
helm install myapp ./mychart -f values.yaml

# Rollback
helm rollback myapp 1
```

**Chart structure:**
```
mychart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl
└── charts/
```

## Monitoring & Troubleshooting

### Q31: How to monitor Kubernetes cluster?
**Answer:**
**Tools:**
1. **Prometheus**: Metrics collection
2. **Grafana**: Visualization
3. **ELK Stack**: Logging
4. **Jaeger**: Distributed tracing

**kubectl commands:**
```bash
# Resource usage
kubectl top nodes
kubectl top pods

# Events
kubectl get events --sort-by='.lastTimestamp'

# Describe resources
kubectl describe pod nginx-pod

# Logs
kubectl logs nginx-pod
kubectl logs nginx-pod -c container-name
kubectl logs -f nginx-pod --tail=100
kubectl logs -p nginx-pod  # Previous container

# Debug
kubectl exec -it nginx-pod -- /bin/bash
kubectl debug nginx-pod
```

### Q32: Common Kubernetes issues and solutions?
**Answer:**
**1. Pod Stuck in Pending:**
- Insufficient resources
- Node selector not matching
- PVC not bound

**2. CrashLoopBackOff:**
- Application error
- Missing dependencies
- Wrong command/args

**3. ImagePullBackOff:**
- Wrong image name/tag
- Private registry credentials
- Network issues

**4. OOMKilled:**
- Memory limit exceeded
- Memory leak

**Debugging:**
```bash
# Check pod status
kubectl get pod nginx-pod -o yaml

# Check node conditions
kubectl describe node node-1

# Check cluster resources
kubectl get nodes --show-labels
kubectl get all --all-namespaces
```

### Q33: Best practices for production Kubernetes?
**Answer:**
1. **Resource Management:**
   - Set resource requests and limits
   - Use namespaces for isolation
   - Implement ResourceQuotas

2. **High Availability:**
   - Multi-master setup
   - Pod disruption budgets
   - Anti-affinity rules

3. **Security:**
   - RBAC enabled
   - Network policies
   - Pod security policies
   - Image scanning

4. **Monitoring:**
   - Metrics collection
   - Log aggregation
   - Alerting

5. **Backup:**
   - etcd backups
   - Persistent volume backups
   - Disaster recovery plan