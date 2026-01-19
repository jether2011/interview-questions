# Feature: Docker & Kubernetes Content

## What

Interview preparation content covering containerization and orchestration technologies that have become essential for modern application deployment. This feature provides comprehensive Q&A materials on:

- **Docker Fundamentals**: Understanding containers vs VMs, the Docker architecture (daemon, client, registry), image layers and the union filesystem, and how containers achieve isolation through Linux namespaces and cgroups.

- **Dockerfile Best Practices**: Writing efficient, secure, and maintainable Dockerfiles - multi-stage builds for smaller images, layer caching optimization, security scanning, and avoiding common pitfalls.

- **Container Networking**: How containers communicate - bridge networks, host networking, overlay networks for multi-host communication, and DNS-based service discovery within Docker.

- **Kubernetes Architecture**: The control plane components (API server, scheduler, controller manager, etcd) and worker node components (kubelet, kube-proxy, container runtime). How Kubernetes achieves its declarative, self-healing design.

- **Kubernetes Workloads**: Deployments for stateless apps, StatefulSets for databases, DaemonSets for node-level services, Jobs and CronJobs for batch processing. Understanding when to use each workload type.

- **Production Operations**: Service discovery, ingress routing, configuration management, secrets handling, resource limits, health probes, and monitoring in Kubernetes.

## Why

Container technology is now the standard for deploying Java applications:

1. **Industry Adoption**: Kubernetes is the de facto orchestration platform for enterprises
2. **DevOps Collaboration**: Developers must understand deployment targets
3. **Cloud-Native Development**: Building applications that leverage container benefits
4. **Operational Troubleshooting**: Debugging production issues requires container knowledge

This content helps developers understand both the "how" of containers and the "why" behind Kubernetes design decisions.

## Acceptance Criteria
- [ ] Covers Docker fundamentals and Dockerfile best practices
- [ ] Explains container networking and volumes
- [ ] Documents Kubernetes architecture components
- [ ] Covers deployment strategies (rolling, blue-green, canary)
- [ ] Explains service types and ingress
- [ ] Documents ConfigMaps and Secrets management
- [ ] Covers monitoring and logging in containers
- [ ] Includes Helm charts and package management

## Content Sections

### 1. Docker Fundamentals
Containers vs VMs: containers share the host kernel (lightweight, fast startup) vs VMs with full OS (stronger isolation). Docker architecture: the daemon (background service), CLI client, and registries (Docker Hub, private). Image layers: each Dockerfile instruction creates a layer, layers are cached and shared. Container lifecycle: create, start, stop, remove. Practical commands every developer should know.

### 2. Dockerfile Best Practices
**Multi-stage builds**: Separate build and runtime stages for smaller, more secure images. **Layer optimization**: Order instructions from least to most frequently changed; combine RUN commands to reduce layers. **Base image selection**: Alpine for size, Distroless for security, official images for reliability. **Security**: Run as non-root user, scan images for vulnerabilities, avoid secrets in build context. **Java-specific**: JVM memory in containers, choosing the right base image (eclipse-temurin, amazoncorretto).

### 3. Container Networking
Docker network types: **bridge** (default, isolated network on host), **host** (container uses host network directly), **none** (no networking), **overlay** (multi-host communication). Port mapping: exposing container ports to the host. DNS-based service discovery: containers can reach each other by name. Docker Compose networking: automatic network creation and service DNS.

### 4. Container Volumes and Storage
The container filesystem is ephemeral - data is lost when container stops. **Volumes**: Docker-managed storage, persists beyond container lifecycle. **Bind mounts**: Map host directory into container. **tmpfs**: In-memory storage for sensitive data. Use cases: database data, logs, configuration files. Volume drivers for cloud storage integration.

### 5. Kubernetes Architecture
**Control Plane**: 
- API Server: RESTful interface, all communication goes through it
- etcd: Distributed key-value store for cluster state
- Scheduler: Assigns pods to nodes based on resources and constraints
- Controller Manager: Runs controllers (Deployment, ReplicaSet, etc.)

**Worker Nodes**:
- kubelet: Ensures containers are running in pods
- kube-proxy: Network proxy and load balancer
- Container runtime: Docker, containerd, or CRI-O

### 6. Kubernetes Workloads
**Pods**: Smallest deployable unit, one or more containers sharing network/storage. **Deployments**: Declarative updates for stateless apps, rolling updates, rollbacks. **StatefulSets**: For stateful apps needing stable network identity and persistent storage. **DaemonSets**: Run on every node (monitoring agents, log collectors). **Jobs/CronJobs**: One-time or scheduled batch processing.

### 7. Services and Networking
**Service types**: ClusterIP (internal), NodePort (external via node ports), LoadBalancer (cloud provider LB). **Ingress**: HTTP/HTTPS routing, TLS termination, path-based routing. **Network Policies**: Firewall rules for pod communication. **Service discovery**: DNS-based (service-name.namespace.svc.cluster.local). **Service mesh**: Istio/Linkerd for advanced traffic management.

### 8. ConfigMaps and Secrets
**ConfigMaps**: Store non-sensitive configuration (environment variables, config files). Mounting as files or environment variables. **Secrets**: Base64-encoded sensitive data (passwords, API keys). Types: Opaque, docker-registry, TLS. Best practices: external secret management (Vault, AWS Secrets Manager), never commit secrets to Git.

### 9. Deployment Strategies
**Rolling update**: Gradually replace old pods with new ones (default). Configuring maxSurge and maxUnavailable. **Blue-Green**: Two identical environments, switch traffic instantly. **Canary**: Route small percentage of traffic to new version. Implementing with Kubernetes: multiple Deployments, weighted routing with service mesh or Ingress.

### 10. Health Probes and Resource Management
**Liveness probe**: Is the container alive? Restart if it fails. **Readiness probe**: Is the container ready for traffic? Remove from service if not. **Startup probe**: For slow-starting applications. **Resource requests**: Guaranteed resources for scheduling. **Resource limits**: Maximum resources, OOMKilled if exceeded. Right-sizing containers for efficiency.

### 11. Monitoring and Logging
**Logging**: stdout/stderr for container logs, kubectl logs, centralized logging (EFK stack, Loki). **Metrics**: Prometheus for metrics collection, Grafana for visualization. **Kubernetes-specific metrics**: kube-state-metrics, metrics-server. **Distributed tracing**: Correlation across services with Jaeger/Zipkin.

### 12. Helm and Package Management
**Helm**: The package manager for Kubernetes. **Charts**: Packages of pre-configured Kubernetes resources. **Values**: Customization parameters. **Releases**: Deployed instances of charts. **Templating**: Go templates for dynamic manifests. Best practices: version pinning, values validation, chart testing.

## Key Concepts to Master

Essential topics for container/Kubernetes interviews:

- **Pod lifecycle**: Pending → Running → Succeeded/Failed, and what triggers each
- **Service discovery**: How pods find and communicate with services
- **Resource limits**: What happens when limits are exceeded (OOMKilled, CPU throttling)
- **Rolling updates**: How Kubernetes ensures zero-downtime deployments
- **Persistent volumes**: PV, PVC, StorageClass relationship

## Study Tips

1. **Build and deploy** - Create a multi-container app, deploy to Kubernetes (minikube/kind)
2. **Break things** - Kill pods, simulate node failures, observe self-healing
3. **Read kubectl output** - Understand pod status, events, and logs
4. **Practice troubleshooting** - CrashLoopBackOff, ImagePullBackOff, pending pods
5. **Know the YAML** - Be able to write Deployment, Service, Ingress from memory

## Related
- [Project Intent](project-intent.md)
- [Feature: Microservices](feature-microservices.md)
- [Decision: Content Structure](../decisions/001-content-structure.md)
- [Pattern: Q&A Format](../knowledge/patterns/qa-format.md)

## Status
- **Created**: 2026-01-19 (Phase: Intent)
- **Updated**: 2026-01-19 - Enhanced content for better study experience
- **Status**: Active (already implemented)
- **File**: `docker-kubernetes.md`
