# CKA Certification Course: Days 1-20 Master Practice Guide
**Comprehensive Command Reference & Explanations**

---

## TABLE OF CONTENTS
1. [Days 1-2: Docker Basics](#days-1-2-docker-basics)
2. [Days 3-4: Docker Deep Dive](#days-3-4-docker-deep-dive)
3. [Day 5: Docker Multi-Stage Builds](#day-5-docker-multi-stage-builds)
4. [Day 6: Kubernetes Fundamentals](#day-6-kubernetes-fundamentals)
5. [Day 7: Kubernetes Architecture](#day-7-kubernetes-architecture)
6. [Day 8: Kind Cluster & Contexts](#day-8-kind-cluster--contexts)
7. [Day 9: YAML & Imperative vs Declarative](#day-9-yaml--imperative-vs-declarative)
8. [Day 10: ReplicationController, ReplicaSets, Deployments](#day-10-replicationcontroller-replicasets-deployments)
9. [Day 11: Microservices & 3-Tier Architecture](#day-11-microservices--3-tier-architecture)
10. [Day 12: Kubernetes Services](#day-12-kubernetes-services)
11. [Day 13: Imperative Commands](#day-13-imperative-commands)
12. [Day 14: Kubernetes Namespaces](#day-14-kubernetes-namespaces)
13. [Day 15: Manual Scheduling & Static Pods](#day-15-manual-scheduling--static-pods)
14. [Day 16: Taints & Tolerations](#day-16-taints--tolerations)
15. [Day 17: Node Selector & Node Affinity](#day-17-node-selector--node-affinity)
16. [Day 18: Taints & Tolerations vs Node Affinity](#day-18-taints--tolerations-vs-node-affinity)
17. [Day 19: Requests, Limits & LimitRange](#day-19-requests-limits--limitrange)
18. [Day 20: HPA & VPA Autoscaling](#day-20-hpa--vpa-autoscaling)

---

## DAYS 1-2: DOCKER BASICS

### Key Concepts
- **Container**: Lightweight, isolated application environment with its own filesystem, processes, and networking
- **Image**: Read-only template containing code, dependencies, and runtime
- **Docker Daemon**: Background service managing containers and images

### Essential Commands

```bash
# Check Docker installation and version
docker --version
docker info

# Pull and run images
docker pull ubuntu                          # Download image from Docker Hub
docker run -it ubuntu /bin/bash            # Interactive container with terminal access
docker run -d nginx                        # Run container in detached (background) mode
docker run --name my-container nginx       # Assign custom container name

# Container management
docker ps                                   # List running containers
docker ps -a                                # List all containers (including stopped)
docker stop <container_id>                 # Stop running container gracefully
docker start <container_id>                # Start stopped container
docker restart <container_id>              # Restart container
docker rm <container_id>                   # Delete stopped container
docker rm -f <container_id>                # Force delete running container

# Container inspection
docker logs <container_id>                 # View container output/logs
docker logs -f <container_id>              # Stream live container logs
docker inspect <container_id>              # View detailed container metadata
docker top <container_id>                  # List processes running inside container
docker exec -it <container_id> /bin/bash   # Execute command inside running container

# Image management
docker images                               # List all downloaded images
docker rmi <image_id>                      # Delete image
docker tag <image_id> <new_name>           # Create image alias/tag
docker history <image>                     # View image layer history
```

---

## DAYS 3-4: DOCKER DEEP DIVE

### Important Docker Flags

```bash
# Port mapping and naming
docker run -d -p 8080:80 --name my-nginx nginx
# -d: Detached mode (run in background)
# -p 8080:80: Map host port 8080 → container port 80
# --name: Assign readable container name

# Volume mounting
docker run -v /host/path:/container/path nginx    # Mount host directory into container
docker run -v my-volume:/app/data nginx           # Mount named volume

# Environment variables
docker run -e APP_ENV=production nginx            # Pass environment variable
docker run -e VAR1=value1 -e VAR2=value2 nginx   # Multiple environment variables

# Resource limits
docker run --memory="512m" --cpus="0.5" nginx    # Limit memory to 512MB, CPU to 0.5 cores
```

### Dockerfile Instructions

```dockerfile
# Base image (every Dockerfile starts with this)
FROM ubuntu:latest

# Set working directory inside container
WORKDIR /app

# Copy files from host to container
COPY app.py /app/app.py

# Add files (with auto-extraction of archives)
ADD app.tar.gz /app

# Run commands during build (creates layers)
RUN apt-get update && apt-get install -y python3

# Expose port (documentation only, doesn't publish)
EXPOSE 5000

# Set default command (can be overridden at runtime)
CMD ["python3", "app.py"]

# Set entrypoint (cannot be overridden without --entrypoint flag)
ENTRYPOINT ["python3", "app.py"]

# Environment variable
ENV APP_ENV=production

# Add metadata/labels
LABEL version="1.0" author="DevOps Team"
```

### CMD vs ENTRYPOINT

| Feature | CMD | ENTRYPOINT |
|---------|-----|-----------|
| **Purpose** | Default command to run | Command that ALWAYS runs |
| **Override** | Easily overridden at runtime | Requires `--entrypoint` flag |
| **Best For** | Default parameters | Wrapper scripts, fixed entry points |
| **Form** | Shell form & Exec form | Exec form only |
| **Example** | `CMD ["python", "app.py"]` | `ENTRYPOINT ["python", "app.py"]` |

### Building Images

```bash
# Basic build
docker build -t my-image:1.0 .
# -t: Tag image with name and version
# .: Build context (current directory)

# Build with custom Dockerfile name
docker build -t my-image -f Dockerfile.prod .
# -f: Specify custom Dockerfile path

# Build without cache
docker build --no-cache -t my-image .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t my-app .
```

---

## DAY 5: DOCKER MULTI-STAGE BUILDS

### Why Multi-Stage Builds?
- **Problem**: Build tools (compilers, dependencies) bloat final image
- **Solution**: Use multiple `FROM` statements to separate build and runtime stages

### Multi-Stage Dockerfile Example

```dockerfile
# Stage 1: BUILD (includes build tools)
FROM golang:1.19 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp .                    # Compile application

# Stage 2: RUNTIME (lightweight final image)
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .           # Copy only compiled binary
EXPOSE 8080
CMD ["./myapp"]
```

### Key Benefits
1. **Smaller images** (final image excludes build tools)
2. **Faster deployments** (smaller images = faster pull/push)
3. **Improved security** (fewer attack surfaces)

### Layer-Creating vs Non-Creating Instructions

| **Creates Layers** | **Does NOT Create Layers** |
|-------------------|--------------------------|
| `FROM`, `RUN`, `COPY`, `ADD` | `CMD`, `ENTRYPOINT`, `WORKDIR`, `EXPOSE`, `ENV`, `LABEL`, `USER` |

**Best Practice**: Combine commands with `&&` to reduce layers
```dockerfile
RUN apt-get update && apt-get install -y curl python3 && rm -rf /var/lib/apt/lists/*
```

---

## DAY 6: KUBERNETES FUNDAMENTALS

### Challenges Docker Solves (vs VMs)
- Lightweight and fast startup
- Efficient resource usage
- Consistent environment across machines

### Limitations of Docker (Kubernetes Solves)
- **No auto-scaling** → Kubernetes HPA
- **No load balancing** → Kubernetes Services
- **No self-healing** → Kubernetes ReplicaSets
- **No rolling updates** → Kubernetes Deployments

### Kubernetes vs Docker Swarm vs Docker Compose

| Feature | Kubernetes | Docker Swarm | Docker Compose |
|---------|-----------|-------------|-----------------|
| **Scope** | Multi-cluster orchestration | Single cluster management | Single-host orchestration |
| **Scalability** | Large-scale (hundreds of nodes) | Moderate scaling | Limited (single host) |
| **Self-healing** | ✅ Yes (robust) | ⚠️ Limited | ❌ No |
| **Learning Curve** | Steep | Moderate | Easy |
| **Community** | Massive ecosystem | Smaller | Development-focused |
| **Use Case** | Production microservices | Smaller deployments | Local development |

---

## DAY 7: KUBERNETES ARCHITECTURE

### Core Concepts

```
┌─────────────────────────────────────────────┐
│       KUBERNETES CLUSTER                    │
├──────────────────┬──────────────────────────┤
│  CONTROL PLANE   │     WORKER NODES         │
│  (Master)        │  (Run Applications)      │
│                  │                          │
│ • API Server     │ • Kubelet (agent)        │
│ • etcd (DB)      │ • Kube-proxy (routing)   │
│ • Scheduler      │ • Container Runtime      │
│ • Controller Mgr │   (containerd, CRI-O)    │
└──────────────────┴──────────────────────────┘
```

### Control Plane Components

| Component | Role |
|-----------|------|
| **API Server** | Front-end for Kubernetes API; validates and processes requests |
| **etcd** | Distributed key-value store; stores all cluster configuration |
| **Scheduler** | Assigns unscheduled pods to suitable worker nodes |
| **Controller Manager** | Runs control loops (Deployment controller, ReplicaSet controller, etc.) |

### Worker Node Components

| Component | Role |
|-----------|------|
| **Kubelet** | Agent on each node; ensures containers run as expected |
| **Kube-proxy** | Network proxy; handles service-to-pod routing and load balancing |
| **Container Runtime** | Software that runs containers (containerd, CRI-O, Podman) |

### Pod Deployment Workflow

```
1. kubectl apply -f deployment.yaml
   ↓
2. API Server: Authenticate → Validate → Store in etcd
   ↓
3. Deployment Controller: Create ReplicaSet object
   ↓
4. ReplicaSet Controller: Create Pod objects
   ↓
5. Scheduler: Assign pods to nodes (set nodeName)
   ↓
6. Kubelet: Pull image → Create container → Monitor health
   ↓
7. Pod is RUNNING
```

---

## DAY 8: KIND CLUSTER & CONTEXTS

### Creating Kind Cluster

```bash
# Install Kind (Kubernetes IN Docker)
# Visit: https://kind.sigs.k8s.io/docs/user/quick-start/

# Create cluster with config file
kind create cluster --name my-cluster --config kind-cluster.yaml

# Example config file (kind-cluster.yaml)
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.28.0
  - role: worker
    image: kindest/node:v1.28.0
  - role: worker
    image: kindest/node:v1.28.0

# Create cluster without config (uses defaults)
kind create cluster --name simple-cluster

# List Kind clusters
kind get clusters

# Delete cluster
kind delete cluster --name my-cluster
```

### Kubernetes Contexts

```bash
# View kubeconfig file location
echo $KUBECONFIG                            # Default: ~/.kube/config

# List all contexts
kubectl config get-contexts

# Get current context
kubectl config current-context

# Switch contexts
kubectl config use-context <context-name>

# Set default namespace for current context
kubectl config set-context --current --namespace=app-ns

# View full kubeconfig
kubectl config view
```

### kubeconfig Structure

```yaml
apiVersion: v1
clusters:
  - name: cluster-1
    cluster:
      server: https://cluster-1-api-server
      certificate-authority-data: <cert-data>

users:
  - name: admin
    user:
      client-certificate-data: <cert-data>
      client-key-data: <key-data>

contexts:
  - name: dev-context
    context:
      cluster: cluster-1
      user: admin
      namespace: dev

current-context: dev-context
```

---

## DAY 9: YAML & IMPERATIVE VS DECLARATIVE

### Imperative vs Declarative

| Aspect | Imperative | Declarative |
|--------|-----------|------------|
| **Definition** | HOW to achieve desired state (step-by-step) | WHAT desired state should be |
| **Example** | `kubectl run pod --image=nginx` | `kubectl apply -f pod.yaml` |
| **Idempotent** | ❌ No (running twice = duplicates) | ✅ Yes (same result every time) |
| **Best For** | Quick testing, one-off tasks | Production, version control, repeatability |

### YAML Syntax Basics

```yaml
# Scalars (simple values)
name: John                 # String
age: 30                    # Integer
height: 5.9                # Float
is_engineer: true          # Boolean
pending: null              # Null value

# Lists (ordered collections)
hobbies:
  - singing
  - dancing
  - reading

# Dictionaries (key-value pairs)
address:
  street: "Main St"
  city: "New York"
  zip: 10001

# Nested structures
education:
  degrees:
    - name: "Bachelors"
      field: "Computer Science"
    - name: "Masters"
      field: "Data Science"
```

### Pod YAML Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: web
    env: prod
spec:
  containers:
    - name: app-container
      image: nginx:latest
      ports:
        - containerPort: 80
      env:
        - name: DEBUG
          value: "false"
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "512Mi"
```

### Essential kubectl Commands

```bash
# Apply declarative configuration
kubectl apply -f pod.yaml                  # Create or update resource
kubectl apply -f deployment.yaml           # Deploy application

# View resources
kubectl get pods                           # List pods
kubectl get pods -o wide                   # List pods with extra details
kubectl get all                            # List all resources
kubectl get pods --all-namespaces (-A)    # List pods across all namespaces

# Detailed information
kubectl describe pod <pod-name>            # Show pod details
kubectl describe node <node-name>          # Show node details

# View logs
kubectl logs <pod-name>                    # View pod logs
kubectl logs <pod-name> -c <container>    # View specific container logs
kubectl logs -f <pod-name>                 # Stream live logs

# Execute commands
kubectl exec -it <pod-name> -- /bin/bash   # Execute command in pod
kubectl exec <pod-name> -- cat /etc/hosts  # Run single command

# Delete resources
kubectl delete pod <pod-name>              # Delete pod
kubectl delete -f pod.yaml                 # Delete resource from file
kubectl delete pods --all                  # Delete all pods
```

---

## DAY 10: REPLICATIONCONTROLLER, REPLICASETS, DEPLOYMENTS

### Checking API Resources

```bash
# View all Kubernetes object types
kubectl api-resources

# Key shortnames:
# rc = ReplicationController
# rs = ReplicaSet
# deploy = Deployment
```

### ReplicationController (rc) - DEPRECATED

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3
  selector:
    app: nginx              # Pods with label app=nginx are managed
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
```

**Note**: Largely replaced by ReplicaSet and Deployments

### ReplicaSet (rs)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
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
```

**Advantages over ReplicationController**:
- Supports set-based selectors (`matchExpressions`)
- Part of apps/v1 API (more stable)

### Labels and Selectors

```yaml
# Labels: Key-value pairs attached to objects
labels:
  app: web-server
  environment: production
  tier: frontend

# Equality-based selectors (simple matching)
selector:
  app: web-server         # Exact match: app=web-server

# Set-based selectors (flexible matching)
matchExpressions:
  - key: app
    operator: In          # Values: In, NotIn, Exists, DoesNotExist
    values:
      - web-server
      - api-server
```

### Deployments

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 3
  strategy:
    type: RollingUpdate   # Gradual replacement of pods
    rollingUpdate:
      maxSurge: 1         # Max new pods during update
      maxUnavailable: 1   # Max unavailable pods during update
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
```

### Deployment Commands

```bash
# Create deployment
kubectl create deployment nginx-deploy --image=nginx:1.21 --replicas=3

# Scale deployment
kubectl scale deployment nginx-deploy --replicas=5

# Update image (rolling update)
kubectl set image deployment/nginx-deploy nginx=nginx:1.22

# View deployment status
kubectl get deployments
kubectl describe deployment nginx-deploy
kubectl rollout status deployment/nginx-deploy

# Rollout history
kubectl rollout history deployment/nginx-deploy
kubectl rollout undo deployment/nginx-deploy              # Rollback to previous
kubectl rollout undo deployment/nginx-deploy --to-revision=2

# Pause/resume updates
kubectl rollout pause deployment/nginx-deploy
kubectl rollout resume deployment/nginx-deploy
```

---

## DAY 11: MICROSERVICES & 3-TIER ARCHITECTURE

### Traditional 3-Tier Architecture

```
┌──────────────────┐
│   FRONTEND       │  (Web Tier)
│  HTML/CSS/JS     │  User Interface
├──────────────────┤
│   BACKEND        │  (Application Tier)
│  Business Logic  │  Processing
├──────────────────┤
│   DATABASE       │  (Data Tier)
│  Data Storage    │  Persistence
└──────────────────┘
```

### Monolithic vs Microservices

| Aspect | Monolithic | Microservices |
|--------|-----------|---------------|
| **Structure** | Single large codebase | Multiple independent services |
| **Scaling** | Scale entire app | Scale services independently |
| **Deployment** | Full app restart | Deploy individual services |
| **Failure Impact** | One bug crashes all | Isolated failures |
| **Tech Stack** | Single technology | Mix of technologies per service |

### Microservices in Kubernetes

```yaml
# Frontend Service Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: nginx:latest

---

# Backend Service Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deploy
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: my-api:latest

---

# Backend Service (for internal communication)
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 9090
      targetPort: 5678
```

### When to Host Databases in Kubernetes?
- ✅ Development/testing environments
- ✅ Using Kubernetes Operators (MySQL Operator, PostgreSQL Operator)
- ❌ Production (prefer cloud-managed: AWS RDS, GCP Cloud SQL, Azure Database)

---

## DAY 12: KUBERNETES SERVICES

### Why Services?
- Pods have ephemeral IPs (change on restart)
- Services provide stable, discoverable endpoints
- Enable load balancing across pod replicas

### ClusterIP Service (Default)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP          # Internal-only access
  selector:
    app: backend           # Select pods with this label
  ports:
    - port: 9090           # Service port (ClusterIP)
      targetPort: 5678     # Pod port
      protocol: TCP
```

**Usage**: `curl backend-svc:9090` from within cluster

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80             # Service port (ClusterIP)
      targetPort: 80       # Pod port
      nodePort: 31000      # External node port (30000-32767)
```

**Usage**: `curl <node-ip>:31000` from outside cluster

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-lb
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

**Usage**: Cloud provider creates external load balancer (AWS ELB, Azure LB, GCP Load Balancer)

### ExternalName Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: my-database.example.com
  ports:
    - port: 5432
```

**Usage**: Maps Kubernetes service to external hostname

### Service Commands

```bash
# Create service
kubectl expose deployment nginx-deploy --type=ClusterIP --port=80 --target-port=8080

# View services
kubectl get svc
kubectl get service -A                     # All namespaces
kubectl describe svc backend-svc           # Service details

# Get service endpoints (backing pods)
kubectl get endpoints backend-svc

# Port forward (local access to cluster resource)
kubectl port-forward svc/backend-svc 9090:9090
kubectl port-forward pod/my-pod 8080:8080
```

---

## DAY 13: IMPERATIVE COMMANDS

### Creating Deployments Imperatively

```bash
# Create deployment with image
kubectl create deployment backend-deploy \
  --image=hashicorp/http-echo \
  --replicas=3 \
  --port=5678 \
  --dry-run=client -o yaml > backend-deploy.yaml

# Create service imperatively
kubectl expose deployment backend-deploy \
  --type=ClusterIP \
  --port=9090 \
  --target-port=5678 \
  --name=backend-svc \
  --dry-run=client -o yaml >> backend-deploy.yaml
```

### Useful Flags

```bash
# Dry-run: Validate without creating
--dry-run=client                # Local validation (no API server call)
--dry-run=server                # Server-side validation (API server validation)

# Output formats
-o yaml                         # Output YAML format
-o json                         # Output JSON format
-o wide                         # Show extra columns

# Other flags
--labels="app=web,env=prod"    # Add labels
--restart=Never                 # Don't restart on failure
--rm                            # Delete pod after use (for testing)
-it                             # Interactive terminal
```

### Creating Test Pods

```bash
# Create test pod imperatively
kubectl run test-pod \
  --image=busybox \
  --restart=Never \
  --labels="app=test" \
  --dry-run=client -o yaml > test-pod.yaml

# Interactive test pod (auto-delete on exit)
kubectl run test-pod \
  --image=busybox \
  -it --rm \
  --restart=Never \
  -- /bin/sh
```

---

## DAY 14: KUBERNETES NAMESPACES

### What are Namespaces?
- Logical partitions within a cluster
- Isolate resources and access
- Enable multi-tenancy
- Avoid naming conflicts

### Default Namespaces

| Namespace | Purpose |
|-----------|---------|
| **default** | Used when no namespace specified |
| **kube-system** | Kubernetes system components (API server, scheduler, etc.) |
| **kube-public** | Publicly readable cluster information |
| **kube-node-lease** | Node heartbeat data |

### Namespace Commands

```bash
# Create namespace imperatively
kubectl create namespace app-ns

# Create namespace declaratively
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: app-ns
EOF

# List namespaces
kubectl get namespaces
kubectl get ns

# Set default namespace for context
kubectl config set-context --current --namespace=app-ns

# Deploy to specific namespace
kubectl apply -f deployment.yaml -n app-ns

# View resources in namespace
kubectl get pods -n app-ns
kubectl get all -n app-ns

# View resources across all namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# Delete namespace (CAUTION: deletes all resources in it)
kubectl delete namespace app-ns
```

### Cross-Namespace Communication

```bash
# Service DNS format for cross-namespace communication
<service-name>.<namespace-name>.svc.cluster.local

# Example
curl http://backend-svc.app-ns.svc.cluster.local:9090

# Short form (same cluster)
curl http://backend-svc.app-ns:9090
```

---

## DAY 15: MANUAL SCHEDULING & STATIC PODS

### Manual Scheduling (Bypass Scheduler)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  nodeName: worker-node-1    # Explicitly assign to node
  containers:
    - name: nginx
      image: nginx
```

**Key Points**:
- Scheduler is completely bypassed
- Pod is guaranteed to run on specified node (if node exists)
- Useful for debugging, testing node-specific workloads

### Static Pods

```bash
# Static pods are defined in /etc/kubernetes/manifests/
# Kubelet reads YAML files directly from this directory

# Create static pod
sudo cat <<EOF > /etc/kubernetes/manifests/static-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
EOF

# Pod appears in kubectl (as mirror pod)
kubectl get pods

# Static pod naming: <pod-name>-<node-name>
# Example: static-nginx-my-node

# Delete static pod (permanently)
sudo rm /etc/kubernetes/manifests/static-nginx.yaml
# Deleting via kubectl only removes mirror pod; kubelet recreates it
```

### Mirror Pods
- Kubernetes creates read-only mirror pod on API server
- Visible in `kubectl get pods`
- Cannot be modified via kubectl
- Shows status of actual static pod

---

## DAY 16: TAINTS & TOLERATIONS

### Taints (Applied to Nodes)

```bash
# Apply taint to node
kubectl taint nodes worker-1 storage=ssd:NoSchedule

# Taint syntax: key=value:effect
# Effects: NoSchedule, PreferNoSchedule, NoExecute
```

| Effect | Behavior |
|--------|----------|
| **NoSchedule** | New pods NOT scheduled unless tolerated |
| **PreferNoSchedule** | Kubernetes tries to avoid scheduling (not enforced) |
| **NoExecute** | Existing pods evicted if not tolerated |

### Tolerations (Applied to Pods)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerated-pod
spec:
  tolerations:
    - key: "storage"
      operator: "Equal"
      value: "ssd"
      effect: "NoSchedule"
  containers:
    - name: app
      image: app:latest
```

### Toleration Operators

```yaml
# Equality operator (exact match)
tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"

# Exists operator (key must exist, any value)
tolerations:
  - key: "storage"
    operator: "Exists"
    effect: "NoSchedule"
```

### Taint Commands

```bash
# List node taints
kubectl describe node worker-1 | grep Taints

# Remove taint (add '-' at end)
kubectl taint nodes worker-1 storage=ssd:NoSchedule-

# Remove all taints for a key
kubectl taint nodes worker-1 storage-
```

---

## DAY 17: NODE SELECTOR & NODE AFFINITY

### Node Selector (Simple)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  nodeSelector:
    storage: ssd         # Pod only scheduled on nodes with label storage=ssd
  containers:
    - name: web
      image: nginx
```

**Limitations**:
- Only supports equality matching
- No "OR" conditions
- No soft preferences

### Node Affinity (Advanced)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      # Hard requirement
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: storage
                operator: In
                values:
                  - ssd
                  - nvme

      # Soft preference
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: cpu
                operator: In
                values:
                  - high-performance
  containers:
    - name: app
      image: app:latest
```

### Node Affinity Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| **In** | Value in list | `In [ssd, nvme]` |
| **NotIn** | Value not in list | `NotIn [hdd]` |
| **Exists** | Label key exists (any value) | `Exists storage` |
| **DoesNotExist** | Label key does not exist | `DoesNotExist storage` |
| **Gt** | Greater than (for integers) | `Gt 5` |
| **Lt** | Less than | `Lt 10` |

### Node Labeling

```bash
# Add label to node
kubectl label nodes worker-1 storage=ssd

# Add multiple labels
kubectl label nodes worker-1 storage=ssd env=prod cpu=high

# View labels
kubectl get nodes --show-labels

# Remove label (add '-' at end)
kubectl label nodes worker-1 storage-
```

---

## DAY 18: TAINTS & TOLERATIONS VS NODE AFFINITY

### When to Use Each

| Scenario | Use |
|----------|-----|
| **Prevent pods from landing on nodes** | Taints & Tolerations |
| **Force pods onto specific nodes** | Node Affinity |
| **Both requirements** | Combine Taints + Tolerations + Affinity |

### Combined Scheduling Strategy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-workload
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      # Tolerate the GPU node taint
      tolerations:
        - key: "gpu"
          operator: "Equal"
          value: "true"
          effect: "NoSchedule"
      
      # Require GPU label
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: accelerator
                    operator: In
                    values:
                      - gpu
      
      containers:
        - name: gpu-app
          image: gpu-workload:latest
```

**Result**: Pods scheduled ONLY on GPU nodes with matching labels and taints

---

## DAY 19: REQUESTS, LIMITS & LIMITRANGE

### Why Requests and Limits?
- **Requests**: Guarantee minimum resources for pod (used by scheduler)
- **Limits**: Maximum resources pod can consume (enforced by kernel)
- Prevent "noisy neighbor" problem (one pod consuming all resources)

### Defining Requests and Limits

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
    - name: app
      image: app:latest
      resources:
        # Minimum resources guaranteed
        requests:
          cpu: "500m"        # 0.5 cores (millicores)
          memory: "256Mi"    # 256 megabytes
        
        # Maximum resources allowed
        limits:
          cpu: "1"           # 1 core
          memory: "512Mi"    # 512 megabytes
```

### Resource Units

| Resource | Unit | Examples |
|----------|------|----------|
| **CPU** | Millicores (m) | 100m = 0.1 cores, 1000m = 1 core |
| **Memory** | Bytes (Mi, Gi) | 256Mi, 1Gi |
| **Storage** | Bytes (Mi, Gi) | 500Mi, 2Gi |

### Behavior When Limits Exceeded

```
CPU: THROTTLED (compressed resource)
  └─ Container slows down
  └─ No termination

Memory: OOM KILLED (incompressible resource)
  └─ Container may be terminated
  └─ Pod restarts (depending on restart policy)
```

### LimitRange (Namespace-level Constraints)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: app-ns
spec:
  limits:
    # Pod-level limits
    - type: Pod
      max:
        cpu: "2"
        memory: "1Gi"
      min:
        cpu: "100m"
        memory: "128Mi"
    
    # Container-level limits
    - type: Container
      max:
        cpu: "1"
        memory: "512Mi"
      min:
        cpu: "50m"
        memory: "64Mi"
      default:
        cpu: "500m"
        memory: "256Mi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
```

### Monitoring Resources

```bash
# View node resource usage
kubectl top nodes

# View pod resource usage
kubectl top pods
kubectl top pods -n kube-system

# Per-container usage
kubectl top pod <pod-name> --containers

# Install Metrics Server (required for kubectl top)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verify Metrics Server
kubectl get deployment metrics-server -n kube-system
```

---

## DAY 20: HPA & VPA AUTOSCALING

### Types of Scaling

```
HORIZONTAL SCALING          VERTICAL SCALING
├─ Add more pods            ├─ Add more CPU/Memory
├─ Scale OUT/IN             ├─ Scale UP/DOWN
├─ Stateless-friendly       ├─ Requires pod restart
└─ HPA automates this       └─ VPA automates this
```

### Horizontal Pod Autoscaler (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
    # Scale based on CPU utilization
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50    # Scale up when CPU > 50%
    
    # Scale based on memory utilization
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 70    # Scale up when Memory > 70%
    
    # Scale based on custom metrics
    - type: Pods
      pods:
        metric:
          name: requests_per_second
        target:
          type: AverageValue
          averageValue: "1k"        # Scale based on custom metric
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100               # Double replicas on scale-up
          periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 50                # Reduce by 50% on scale-down
          periodSeconds: 15
```

### HPA Commands

```bash
# Create HPA imperatively
kubectl autoscale deployment web-deploy --min=2 --max=10 --cpu-percent=50

# View HPA status
kubectl get hpa
kubectl describe hpa web-hpa
kubectl get hpa web-hpa --watch

# Check HPA events
kubectl describe hpa web-hpa | grep Events -A 10
```

### Vertical Pod Autoscaler (VPA)

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: web-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: web-deploy
  
  updatePolicy:
    updateMode: "Auto"           # Auto, Recreate, Initial, Off
  
  resourcePolicy:
    containerPolicies:
      - containerName: "*"       # Apply to all containers
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 1
          memory: 1Gi
        controlledValues:
          - RequestsAndLimits    # Update both requests and limits
```

### VPA Update Modes

| Mode | Behavior |
|------|----------|
| **Auto** | Update requests and restart pods as needed |
| **Recreate** | Only update on pod restart (manual restart triggers update) |
| **Initial** | Update only on pod creation (no updates to running pods) |
| **Off** | No automatic updates (recommendations only) |

### HPA vs VPA Comparison

| Aspect | HPA | VPA |
|--------|-----|-----|
| **What it does** | Adjusts pod count | Adjusts pod resources |
| **Trigger** | Metrics (CPU, memory, custom) | Historical usage patterns |
| **Pod restart** | ❌ No | ✅ Yes (usually) |
| **Use Case** | Variable load patterns | Resource optimization |
| **Best For** | Stateless applications | Long-running apps |

### Key Requirements

```
HPA REQUIREMENTS:
├─ Resource requests MUST be defined
├─ Metrics Server must be installed
└─ Target metrics must be available

VPA REQUIREMENTS:
├─ History data (time to analyze usage)
├─ VPA controller running in cluster
└─ Pod restart permissions
```

### Monitoring Autoscaling

```bash
# Watch HPA decisions
kubectl get hpa --watch

# View HPA status details
kubectl describe hpa web-hpa

# Check Metrics Server health
kubectl get deployment metrics-server -n kube-system

# View pod resource recommendations (VPA)
kubectl describe vpa web-vpa
```

---

## QUICK REFERENCE: MOST-USED COMMANDS

```bash
# Cluster Info
kubectl cluster-info
kubectl get nodes
kubectl get namespaces

# Deployments
kubectl create deployment myapp --image=myapp:1.0 --replicas=3
kubectl apply -f deployment.yaml
kubectl scale deployment myapp --replicas=5
kubectl set image deployment/myapp myapp=myapp:2.0
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp

# Pods
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash
kubectl delete pod <pod-name>

# Services
kubectl expose deployment myapp --type=NodePort --port=80 --target-port=8080
kubectl get svc
kubectl port-forward svc/myapp 8080:80

# Namespaces
kubectl create namespace myns
kubectl apply -f resource.yaml -n myns
kubectl get all -n myns

# Labels & Selectors
kubectl label nodes worker-1 storage=ssd
kubectl label pods myapp-123 env=prod

# Taints & Tolerations
kubectl taint nodes worker-1 gpu=true:NoSchedule
kubectl taint nodes worker-1 gpu-

# Autoscaling
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=50
kubectl get hpa

# Resources
kubectl top nodes
kubectl top pods
kubectl describe resourcequota -n myns
```

---

## ESSENTIAL TIPS FOR CKA EXAM

✅ **Use imperative commands to generate YAML, then edit as needed**
```bash
kubectl create deployment myapp --image=myapp:1.0 --dry-run=client -o yaml > deploy.yaml
```

✅ **Always define resource requests/limits** (especially for HPA)

✅ **Use contexts for multi-cluster setups**
```bash
kubectl config use-context <cluster-name>
```

✅ **Combine taints, tolerations, and node affinity for complete scheduling control**

✅ **Namespace isolation for multi-tenancy and resource organization**

✅ **Rolling updates for zero-downtime deployments**
```bash
kubectl set image deployment/myapp myapp=myapp:2.0 --record
```

✅ **Test with temporary pods**
```bash
kubectl run test --image=busybox -it --rm --restart=Never -- /bin/sh
```

---

## PRACTICE SCENARIOS

### Scenario 1: Deploy Multi-Tier Application
1. Create namespace: `production`
2. Deploy frontend (3 replicas, NodePort service)
3. Deploy backend (3 replicas, ClusterIP service)
4. Frontend pods → backend-svc:9090
5. Test cross-service communication

### Scenario 2: Resource Management & Scaling
1. Deploy app with CPU requests (500m) and limits (1000m)
2. Setup HPA: min=2, max=10, CPU threshold=70%
3. Monitor scaling with `kubectl top pods`
4. Generate load and observe scale-up

### Scenario 3: Scheduling Control
1. Label nodes: storage=ssd, storage=hdd
2. Taint SSD nodes: gpu=true:NoSchedule
3. Deploy GPU workload with toleration + affinity
4. Verify pod lands on correct SSD node

---

**Last Updated**: CKA Days 1-20 Compilation
**For Latest Updates**: Visit https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025
