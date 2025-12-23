# K8

## Table of Contents

### Getting Started
- [Environment Setup](#environment-setup)
- [Helper/Utility Commands](#helperutility-commands)

### Core Resources
- [Pods](#pods)
  - [Basic Commands](#basic-commands)
  - [Inspection & Architecture](#inspection--architecture)
  - [Lifecycle & Manifests](#lifecycle--manifests)
  - [Editing Pods](#editing-pods)
  - [Commands and Arguments (Docker vs Kubernetes)](#commands-and-arguments-docker-vs-kubernetes)
  - [Multi-Container Pods](#multi-container-pods)
    - [Overview](#overview-multi-container)
    - [Pod Restart Behavior](#pod-restart-behavior)
    - [Init Containers](#init-containers)
    - [Sidecar Containers](#sidecar-containers)
    - [Co-Located Containers](#co-located-containers)
- [ReplicaSets](#replicasets)
  - [Overview](#overview)
  - [Manifest Examples](#manifest-examples)
  - [Basic Commands](#basic-commands-1)
  - [Scaling & Updates](#scaling--updates)
  - [Common Errors](#common-errors)
- [Kubernetes Deployments](#kubernetes-deployments)
  - [Overview](#overview-1)
  - [Basic Commands](#basic-commands-2)

### Networking
- [Services](#services)
  - [Overview](#overview-2)
  - [ClusterIP Service](#clusterip-service)
  - [NodePort Service](#nodeport-service)
  - [Service Manifest Example](#service-manifest-example)
  - [Creating Services Imperatively (Using kubectl expose)](#creating-services-imperatively-using-kubectl-expose)
  - [Basic Commands](#basic-commands-3)
- [Namespaces](#namespaces)
  - [Built-in Namespaces](#built-in-namespaces)
  - [Creating Namespaces](#creating-namespaces)
  - [Working with Namespaces](#working-with-namespaces)
  - [Service Discovery](#service-discovery)

### Resource Management
- [Resource Requests and Limits](#resource-requests-and-limits)
  - [What Happens When a Pod Exceeds CPU Limit](#what-happens-when-a-pod-exceeds-cpu-limit)
  - [What Happens When a Pod Exceeds Memory Limit](#what-happens-when-a-pod-exceeds-memory-limit)
- [LimitRange](#limitrange)
- [ResourceQuotas](#resourcequotas)

### Pod Scheduling and Node Selection
- [Taints and Tolerations](#taints-and-tolerations)
  - [Overview](#overview-4)
  - [Taint Effects](#taint-effects)
  - [Creating Taints](#creating-taints)
  - [Tolerations](#tolerations)
  - [Important Notes](#important-notes-taints-tolerations)
  - [Basic Commands](#basic-commands-5)
- [Node Labels](#node-labels)
- [NodeSelector](#nodeselector)
- [Node Affinity](#node-affinity)
  - [Node Affinity Types](#node-affinity-types)
  - [Operators](#operators)

### Configuration Management
- [Environment Variables](#environment-variables)
  - [Plain Key-Value Format](#1-plain-key-value-format)
  - [Using ConfigMaps](#2-using-configmaps)
  - [Using Secrets](#3-using-secrets)
- [ConfigMaps](#configmaps)
  - [Creating ConfigMaps (Imperative Methods)](#creating-configmaps-imperative-methods)
    - [Method 1: Using --from-literal](#method-1-using---from-literal)
    - [Method 2: Using --from-file](#method-2-using---from-file)
  - [Creating ConfigMaps (Declarative Method - YAML)](#creating-configmaps-declarative-method---yaml)
  - [Viewing ConfigMaps](#viewing-configmaps)
  - [Injecting ConfigMaps into Pods](#injecting-configmaps-into-pods)
    - [Method 1: Using envFrom](#method-1-using-envfrom-inject-all-key-value-pairs)
    - [Method 2: Using Single Environment Variable with valueFrom](#method-2-using-single-environment-variable-with-valuefrom)
    - [Method 3: Using Volumes](#method-3-using-volumes)
- [Secrets](#secrets)
  - [Viewing Secrets](#viewing-secrets)
  - [Injecting Secrets into Pods](#injecting-secrets-into-pods)
    - [Method 1: Using envFrom](#method-1-using-envfrom-inject-all-key-value-pairs-1)
    - [Method 2: Using Single Environment Variable with valueFrom](#method-2-using-single-environment-variable-with-valuefrom-1)
    - [Method 3: Using Volumes](#method-3-using-volumes-1)
  - [CSI Drivers to Manage Secrets](#csi-drivers-to-manage-secrets)
  - [Best Practices for Production Secret Management](#best-practices-for-production-secret-management)

### Security
- [Security Context](#security-context)
  - [Pod Level Security Context](#pod-level-security-context)
  - [Container Level Security Context](#container-level-security-context)
  - [Combining Pod and Container Level](#combining-pod-and-container-level)
  - [Capabilities (Container Level Only)](#capabilities-container-level-only)
- [Service Accounts](#service-accounts)
  - [Overview](#overview-3)
  - [Default Service Account](#default-service-account)
  - [Creating Service Accounts](#creating-service-accounts)
  - [Attaching Service Accounts to Pods](#attaching-service-accounts-to-pods)
  - [Service Account Tokens](#service-account-tokens)
  - [Dashboard Authentication Flow](#dashboard-authentication-flow)
  - [Using Tokens for API Authentication](#using-tokens-for-api-authentication)
  - [Basic Commands](#basic-commands-4)

---

## Getting Started

## Environment Setup

```bash
alias k=kubectl
```

## Helper/Utility Commands

**Find the Kind required for casings or to find shorthands:**
```bash
k api-resources | grep "replicasets"
```

**Understand the node definitions:**
```bash
k explain rs/po --recursive
```

## Pods

### Basic Commands

```bash
k get pods
k run nginx --image=nginx
```

### Inspection & Architecture

**Get details of a specific pod (containers, events, errors):**
```bash
k describe pod <pod-name>
```

**Get extra info (IP, Node assignment):**
```bash
k get pods -o wide
```
*Note: This shows the Node on which each pod is scheduled*

**Architecture Hierarchy:**
*Node has pods → pods have containers → containers are running instances of images.*

**Understanding 'Ready' state:**
*If 'Ready' is 1/2, it means the pod has 2 containers, of which only 1 is in the ready state. Use `k describe pod` to see container status and look at events to understand errors.*

### Lifecycle & Manifests

**Delete a pod:**
```bash
k delete pod webapp
```

**Generate a Pod manifest YAML without creating the pod:**
```bash
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml
```

**Create a resource from a file:**
```bash
kubectl create -f redis-definition.yaml
```

**Edit the pod manifest:**
```bash
vi redis-definition.yaml
```

**Apply changes to the file:**
```bash
k apply -f redis-definition.yaml
```

### Editing Pods

**Important limitation:** *You CANNOT edit most specifications of an existing Pod. Pods are immutable objects in Kubernetes.*

**What you CAN edit in a running Pod:**
- `spec.containers[*].image` - Container image
- `spec.initContainers[*].image` - Init container image
- `spec.activeDeadlineSeconds` - Active deadline seconds
- `spec.tolerations` - Tolerations

**What you CANNOT edit:**
- Environment variables
- Service accounts
- Resource limits and requests
- Container commands and arguments (after initial creation)
- Most other pod specifications

**Why this limitation exists:**
*Pods are designed to be immutable for reliability and consistency. Once created, their core specifications cannot be changed. This is different from Docker, where you can modify running containers.*

**Remember:** You CANNOT edit specifications of an existing POD other than the four fields listed above. For example, you cannot edit:
- Environment variables
- Service accounts  
- Resource limits and requests
- Container commands and arguments (after initial creation)
- Most other pod specifications

*But if you really need to modify these fields, you have two options:*

**If you need to modify a Pod's specifications, you have two options:**

#### Option 1: Using kubectl edit (Quick Method)

**Step 1: Edit the pod**
```bash
kubectl edit pod webapp
```
*This opens the pod specification in an editor (vi editor).*

**Step 2: Make your changes**
*Edit the required properties in the editor.*

**Step 3: Save and handle the error**
*When you try to save, you will get an error because you're attempting to edit a field that is not editable. The error message will look like:*

```
error: pods "ubuntu-sleeper-3" is invalid

A copy of your changes has been stored to "/tmp/kubectl-edit-266172180.yaml"

error: Edit cancelled, no valid changes were saved.
```

*However, a copy of the file with your changes is saved in a temporary location (the path shown in the error message, e.g., `/tmp/kubectl-edit-266172180.yaml`).*

**Step 4: Delete the existing pod**
```bash
kubectl delete pod ubuntu-sleeper-3
# or
kubectl delete pod webapp
```

**Step 5: Create a new pod with your changes**
```bash
kubectl create -f /tmp/kubectl-edit-266172180.yaml
# Use the exact path from the error message
```

**Complete example workflow:**
```bash
# Step 1: Try to edit the pod
k edit pod ubuntu-sleeper-3

# Step 2: Make changes in the editor, save and exit
# You'll see the error message

# Step 3: Delete the existing pod
k delete pod ubuntu-sleeper-3

# Step 4: Create new pod with the saved changes
k create -f /tmp/kubectl-edit-266172180.yaml
```

#### Option 2: Export, Edit, Delete, Recreate (Recommended)

**Step 1: Extract the pod definition to a file**
```bash
kubectl get pod webapp -o yaml > my-new-pod.yaml
```
*This exports the current pod definition to a YAML file.*

**Step 2: Edit the exported file**
```bash
vi my-new-pod.yaml
```
*Make your changes to the YAML file (remove metadata like uid, resourceVersion, etc., as they're auto-generated).*

**Step 3: Delete the existing pod**
```bash
kubectl delete pod webapp
```

**Step 4: Create a new pod with the edited file**
```bash
kubectl create -f my-new-pod.yaml
```

**Important cleanup notes:**
- Remove auto-generated fields like `uid`, `resourceVersion`, `creationTimestamp`, and `status` from the exported YAML before recreating
- The new pod will have a different name unless you explicitly specify the same name in the YAML
- You can remove or keep the exported file - it won't affect the pod once it's created

**When to use each option:**
- **Option 1 (kubectl edit)**: Quick method when you want to make changes on the fly
- **Option 2 (export/edit/recreate)**: Recommended when you want more control, need to review changes carefully, or want to keep a version-controlled copy of your pod definition

### Using Deployments (Better Approach)

*Instead of editing individual Pods, use Deployments. Deployments make it easy to edit any field/property of the Pod template.*

**Why Deployments are better:**
- **You can edit ANY field** in the Pod template (environment variables, resource limits, commands, arguments, etc.)
- The Deployment **automatically deletes the old pod and creates a new pod** with your changes
- No need to manually delete and recreate pods
- Provides a seamless way to update running applications
- Supports rolling updates with zero downtime

**Edit a Deployment:**
```bash
kubectl edit deployment my-deployment
```

**What happens when you edit a Deployment:**
1. You edit the Pod template in the Deployment spec (you can change any field)
2. Save the changes - **unlike pods, this will succeed!**
3. Kubernetes automatically:
   - Creates a new ReplicaSet with the updated Pod template
   - Gradually replaces old pods with new pods (rolling update)
   - Maintains the desired number of replicas
   - Ensures zero downtime during the update

**Key concept:**
- Since the pod template is a child of the deployment specification, with every change, the deployment will automatically delete and create a new pod with the new changes
- So, if you are asked to edit a property of a POD that is part of a deployment, you may do that simply by running `kubectl edit deployment my-deployment`

**Key difference:**
- **Pods**: Immutable, must delete and recreate to change most specs
- **Deployments**: Mutable, can edit Pod template and it automatically updates pods

**Best practice:**
*Always use Deployments instead of standalone Pods for applications that may need updates. This aligns with Kubernetes best practices and makes your infrastructure more maintainable.*

### Commands and Arguments (Docker vs Kubernetes)

*When creating a Pod using a Docker image, you can override the Docker image's default command and arguments using Kubernetes Pod manifest fields. This connects Docker's ENTRYPOINT/CMD concepts to Kubernetes Pod specifications.*

**Docker to Kubernetes Mapping:**

| Docker Instruction | Kubernetes Pod Field | Description |
|-------------------|---------------------|-------------|
| `ENTRYPOINT` | `command` | The base command that always runs |
| `CMD` | `args` | Default arguments (can be overridden) |

**Simple mapping:**
- **`ENTRYPOINT` in Docker** → **`command` in Kubernetes Pod manifest**
- **`CMD` in Docker** → **`args` in Kubernetes Pod manifest**

**How it works:**
- When you create a Pod using a Docker image, Kubernetes reads the image's ENTRYPOINT and CMD
- You can override these in the Pod manifest using `command` and `args` fields
- This works exactly like overriding ENTRYPOINT and CMD in Docker, but defined declaratively in YAML

**Docker Command vs Kubernetes YAML Comparison:**

| Docker Runtime Command | Kubernetes Pod Manifest |
|----------------------|------------------------|
| `docker run ubuntu-sleeper` | `image: ubuntu-sleeper` (uses default ENTRYPOINT/CMD) |
| `docker run ubuntu-sleeper 10` | `image: ubuntu-sleeper`<br>`args: ["10"]` (override CMD only) |
| `docker run --entrypoint sleep2.0 ubuntu-sleeper 10` | `image: ubuntu-sleeper`<br>`command: ["sleep2.0"]`<br>`args: ["10"]` (override both) |

**Basic Example - Pod using default image commands:**

If your Docker image has:
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

**Kubernetes Pod Definition (using defaults):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    # No command or args specified - uses image's ENTRYPOINT ["sleep"] and CMD ["5"]
    # This will run: sleep 5
```

**Kubernetes Pod Definition (overriding args only):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    args: ["10"]  # Overrides CMD ["5"] with ["10"], keeps ENTRYPOINT ["sleep"]
    # This will run: sleep 10
```

**Kubernetes Pod Definition (overriding both):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep2.0"]  # Overrides ENTRYPOINT ["sleep"]
    args: ["10"]           # Overrides CMD ["5"]
    # This will run: sleep2.0 10
```

**Dockerfile equivalent (for reference):**
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]  # Maps to Kubernetes 'command'
CMD ["5"]             # Maps to Kubernetes 'args'
```

**Key takeaway:**
- **`command`** in Kubernetes = **`ENTRYPOINT`** in Docker
- **`args`** in Kubernetes = **`CMD`** in Docker
- If you don't specify `command` or `args` in the Pod manifest, Kubernetes uses the image's ENTRYPOINT and CMD

**Key points:**
- `command` and `args` are both arrays (list format in YAML)
- If you don't specify `command`, the image's ENTRYPOINT is used
- If you don't specify `args`, the image's CMD is used (or no args if only ENTRYPOINT exists)
- You can override either or both fields independently
- This is equivalent to Docker's `--entrypoint` flag and runtime arguments

### Multi-Container Pods

#### Overview {#overview-multi-container}

*A Pod can contain multiple containers that share the same network namespace, storage, and lifecycle. There are three common patterns for multi-container Pods: Init Containers, Sidecar Containers, and Co-Located Containers.*

**Key Concept:**
- In a multi-container Pod, each container is expected to run a process that stays alive for the entire lifecycle of the Pod
- All containers in a Pod share:
  - Network namespace (same IP address, can communicate via localhost)
  - Storage volumes
  - Pod lifecycle

**Three Patterns:**

| Pattern | Purpose | Lifecycle |
|---------|---------|-----------|
| **Init Containers** | Run setup tasks before main containers start | Run to completion sequentially before main containers |
| **Sidecar Containers** | Support main container (logging, monitoring, etc.) | Run alongside main containers for entire Pod lifecycle |
| **Co-Located Containers** | Related containers working together | Run together, all expected to stay alive |

#### Pod Restart Behavior

*It's important to understand how restarts work in multi-container Pods:*

**Key Points:**
- If any **main container** (containers under `spec.containers`) exits and the Pod's `restartPolicy` is set to `Always` or `OnFailure`, **all containers in the Pod are restarted**
- Kubernetes does **NOT restart individual containers** within a Pod
- Kubernetes treats the Pod as a **single unit of execution** and restarts the entire Pod if needed
- This applies only to **main containers**, not init containers
- Init containers are always run to completion before main containers begin and are not restarted individually

**Example Scenario:**
- Pod has 2 containers: web application and logging agent
- Both containers are expected to remain active throughout the Pod's lifecycle
- If the web application container fails, the **entire Pod is restarted** (both containers restart)
- The logging agent container must stay alive as long as the web application is running

#### Init Containers

*Init containers are special containers that run before the main containers in a Pod. They are used for setup tasks that must complete before the main application starts.*

**Characteristics:**
- Run **before** main containers start
- Each init container must **succeed (exit 0)** before the next one is started
- Run **sequentially** (one after another)
- Once all init containers complete, **regular containers start simultaneously**
- If any init container fails, the entire Pod is restarted and all init containers are rerun from the beginning
- Init containers are configured in the `initContainers` section of the Pod spec

**Common Use Cases:**
- Pull code or binaries from a repository before the application starts
- Wait for an external service (like a database) to become available
- Initialize configuration files
- Run database migrations
- Set up network rules or security policies

**Example - Init Containers:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:
  - name: init-myservice
    image: busybox:1.31
    command: ["sh", "-c", "until nslookup myservice; do echo waiting for myservice; sleep 2; done;"]
  - name: init-mydb
    image: busybox:1.31
    command: ["sh", "-c", "until nslookup mydb; do echo waiting for mydb; sleep 2; done;"]
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ["sh", "-c", "echo The app is running! && sleep 3600"]
```

**How It Works:**
1. `init-myservice` runs first - waits for `myservice` to be available
2. Once `init-myservice` succeeds (exits 0), `init-mydb` runs - waits for `mydb` to be available
3. Once all init containers complete, `myapp-container` starts
4. If any init container fails, the Pod restarts and all init containers run again from the beginning

**Key Points:**
- Init containers run sequentially (not in parallel)
- All init containers must succeed before main containers start
- Init containers are not restarted individually - if one fails, the Pod restarts and all init containers rerun

#### Sidecar Containers

*Sidecar containers run alongside main containers to provide supporting functionality like logging, monitoring, or proxy services.*

**Characteristics:**
- Run **alongside** main containers for the entire Pod lifecycle
- Share the same network and storage with main containers
- Typically handle cross-cutting concerns (logging, monitoring, networking)
- Must stay alive as long as the main containers are running

**Native Sidecar Containers (Kubernetes 1.33+):**

Starting with Kubernetes v1.33, sidecar containers are natively supported. This allows sidecar containers to follow a defined lifecycle relative to the main containers without requiring entrypoint hacks.

**How Native Sidecars Work:**
- Declared using `restartPolicy: Always` field inside the `initContainers` block
- Kubernetes treats such containers as sidecars, ensuring they:
  - Start before main containers
  - Run alongside them
  - Shut down after the main containers complete

**Example - Native Sidecar Configuration:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  initContainers:
  - name: sidecar-logger
    image: busybox:1.31
    restartPolicy: Always  # Marks this as a sidecar container
    command: ["sh", "-c", "while true; do echo Sidecar running; sleep 10; done"]
  containers:
  - name: main-app
    image: busybox:1.31
    command: ["sh", "-c", "echo Main app starting; sleep 60"]
```

**In this setup:**
- The `sidecar-logger` container behaves like a sidecar, though declared in `initContainers`
- It uses `restartPolicy: Always` to stay alive throughout the Pod lifecycle
- Kubernetes starts the sidecar first, waits for readiness, then starts the main app
- The sidecar shuts down after the main container completes

**Traditional Sidecar Pattern (Pre-1.33):**

Before native sidecar support, sidecars were implemented as regular containers that run indefinitely:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  containers:
  - name: main-app
    image: myapp:latest
    command: ["./app"]
  - name: sidecar-logger
    image: logger:latest
    command: ["./logger"]
    # Runs alongside main-app for entire Pod lifecycle
```

**Common Sidecar Use Cases:**
- Logging agents (collect and forward logs)
- Monitoring agents (collect metrics)
- Network proxies (handle networking concerns)
- Config file watchers (reload configuration)
- Service mesh sidecars (handle service-to-service communication)

#### Co-Located Containers

*Co-located containers are multiple containers in the same Pod that work together as a cohesive unit. All containers are main containers and are expected to stay alive for the Pod's lifecycle.*

**Characteristics:**
- All containers are defined under `spec.containers` (not initContainers)
- All containers run simultaneously (not sequentially)
- All containers are expected to stay alive for the Pod's lifecycle
- Containers share network and storage
- If any container fails, the entire Pod restarts

**Example - Co-Located Containers:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: web-server
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: log-processor
    image: log-processor:latest
    # Both containers run together and must stay alive
```

**Common Use Cases:**
- Web server with log processor
- Application with caching service
- Frontend and backend services that must be together
- Main application with helper service

**Key Differences:**

| Aspect | Init Containers | Sidecar Containers | Co-Located Containers |
|--------|----------------|-------------------|----------------------|
| **Location in Spec** | `spec.initContainers` | `spec.initContainers` (1.33+) or `spec.containers` | `spec.containers` |
| **Execution Order** | Sequential (one after another) | Before main, then alongside | Simultaneous |
| **Lifecycle** | Run to completion before main | Run alongside main containers | Run together for Pod lifecycle |
| **Restart Behavior** | Rerun from beginning if Pod restarts | Restarted with Pod | Restarted with Pod |
| **Use Case** | Setup/preparation tasks | Supporting functionality | Related services working together |

**Key Takeaways:**
- Multi-container Pods share network, storage, and lifecycle
- Init containers run sequentially before main containers
- Sidecar containers (native in 1.33+) support main containers throughout lifecycle
- Co-located containers run together as a unit
- Pod restarts apply to all containers - individual containers are not restarted separately

### Resource Requests and Limits

*Kubernetes allows you to specify both resource requests and limits for containers. These help the scheduler place pods and prevent resource exhaustion.*

**Requests vs Limits:**

- **Requests**: Minimum resources guaranteed to the container (used by scheduler)
- **Limits**: Maximum resources the container can use (enforced by kubelet)

**Example Pod with Resource Requests and Limits:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"    # Minimum memory guaranteed
        cpu: 2            # Minimum CPU guaranteed
      limits:
        memory: "2Gi"     # Maximum memory allowed
        cpu: 2            # Maximum CPU allowed
```

**Key points:**
- `resources.requests` specifies the minimum resources required (used by scheduler)
- `resources.limits` specifies the maximum resources allowed (enforced at runtime)
- Requests help the scheduler find a suitable node
- Limits prevent containers from consuming too many resources
- If limits are exceeded, the container may be throttled (CPU) or terminated (memory)

**How kube-scheduler Uses Resource Requests:**

1. **Pod Creation**: When a pod is created with resource requests, the kube-scheduler receives the scheduling request
2. **Node Evaluation**: The kube-scheduler evaluates all nodes in the cluster to find one with sufficient available resources
3. **Resource Check**: For each node, it checks if:
   - Available CPU ≥ requested CPU
   - Available Memory ≥ requested Memory
4. **Scheduling Decision**:
   - **If resources are available**: The pod is scheduled to that node and starts running
   - **If no node has sufficient resources**: The pod remains in `Pending` state until resources become available

**Note:** The scheduler only considers `requests`, not `limits`, when making scheduling decisions.

**How Resource Limits Are Enforced:**

*Once a pod is running, the kubelet on the node enforces resource limits. The behavior differs significantly between CPU and memory:*

#### What Happens When a Pod Exceeds CPU Limit

*When a container tries to consume more CPU than its limit:*

1. **CPU Throttling**: The container's CPU usage is throttled (restricted) to stay within the limit
2. **No Termination**: The container **continues running** - it is NOT terminated
3. **Performance Impact**: The application may run slower because it cannot use more CPU than allowed
4. **Graceful Handling**: CPU is a "compressible" resource - Kubernetes can restrict it without killing the process

**Example:**
- Container has CPU limit: `2` cores
- Container tries to use: `3` cores
- **Result**: Container is throttled to use maximum `2` cores, continues running but may be slower

**Checking CPU Throttling:**
```bash
kubectl top pod <pod-name>
# Shows actual CPU usage vs limits
```

#### What Happens When a Pod Exceeds Memory Limit

*When a container tries to consume more memory than its limit:*

1. **OOMKilled**: The container is **terminated** by the kernel with status `OOMKilled` (Out Of Memory)
2. **Pod Restart**: If the pod has a restart policy, Kubernetes will attempt to restart the container
3. **No Graceful Handling**: Memory is a "non-compressible" resource - once allocated, it cannot be easily reclaimed
4. **Immediate Action**: The termination happens immediately when memory limit is exceeded

**Example:**
- Container has memory limit: `1Gi`
- Container tries to use: `1.5Gi`
- **Result**: Container is terminated with `OOMKilled` status, pod may restart if configured

**Checking Memory Issues:**
```bash
kubectl describe pod <pod-name>
# Look for events showing OOMKilled
kubectl get pod <pod-name>
# Status will show "OOMKilled" or "CrashLoopBackOff" if restarting
```

**Key Differences:**

| Resource | Behavior When Limit Exceeded | Container Status | Can Recover? |
|----------|----------------------------|------------------|--------------|
| **CPU** | Throttled (restricted) | Continues running | Yes - automatically throttled |
| **Memory** | Terminated (OOMKilled) | Terminated | Yes - if restart policy allows |

**Important Notes:**
- **CPU**: Compressible resource - can be throttled without termination
- **Memory**: Non-compressible resource - must be terminated if limit exceeded
- **Enforcement**: Both are enforced by the kubelet on the node, not the scheduler
- **Monitoring**: Always monitor resource usage to avoid unexpected throttling or terminations

**What Happens When Resources Are Not Available:**

*If the kube-scheduler cannot find a node with sufficient resources:*
- The pod status remains `Pending`
- The pod will not be scheduled to any node
- The pod waits until:
  - A node with sufficient resources becomes available (e.g., other pods are terminated, nodes are added)
  - Resources are freed up on existing nodes

**Checking Pod Status:**

```bash
kubectl get pods
```

*If a pod is stuck in `Pending` state, you can check why:*

```bash
kubectl describe pod <pod-name>
```

*Look for events or conditions that indicate resource constraints, such as:*
- `Insufficient cpu`
- `Insufficient memory`
- `0/3 nodes are available: 3 Insufficient cpu`

**Resource Request Formats:**

| Resource | Format | Example |
|----------|--------|---------|
| **CPU** | Integer or millicores | `cpu: 2` or `cpu: "2000m"` |
| **Memory** | Bytes, KiB, Mi, Gi, etc. | `memory: "4Gi"` or `memory: "512Mi"` |

**Common Memory Units:**
- `Ki` - Kibibytes (1024 bytes)
- `Mi` - Mebibytes (1024 KiB)
- `Gi` - Gibibytes (1024 Mi)
- `Ti` - Tebibytes (1024 Gi)

**Common CPU Formats:**
- `1` = 1 CPU core
- `"1000m"` = 1000 millicores = 1 CPU core
- `"500m"` = 0.5 CPU cores
- `2` = 2 CPU cores

**Requests vs Limits - Key Differences:**

| Aspect | Requests | Limits |
|--------|----------|--------|
| **Purpose** | Minimum resources guaranteed | Maximum resources allowed |
| **Used By** | kube-scheduler (for scheduling) | kubelet (for enforcement) |
| **Enforcement** | Guaranteed availability | Hard limit enforcement |
| **CPU Exceeded** | N/A (guaranteed minimum) | Container gets throttled |
| **Memory Exceeded** | N/A (guaranteed minimum) | Container may be terminated (OOMKilled) |
| **Scheduling** | Required for scheduling decision | Not considered by scheduler |

### LimitRange

*LimitRange is a Kubernetes resource that sets default resource requests and limits for containers in a namespace. It ensures that all containers have resource constraints defined, even if they're not explicitly specified in the pod definition.*

**What LimitRange Does:**
- Sets **default** resource limits and requests for containers
- Sets **minimum** and **maximum** resource constraints
- **Applies to newer pods** being created in the namespace
- Validates that pod resource specifications fall within the defined range

**Creating a LimitRange:**

**Example - CPU LimitRange:**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m        # Default limit (applied if not specified)
    defaultRequest:
      cpu: 500m        # Default request (applied if not specified)
    max:
      cpu: "1"         # Maximum CPU allowed
    min:
      cpu: 100m        # Minimum CPU required
    type: Container
```

**Example - Memory LimitRange:**

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi      # Default limit (applied if not specified)
    defaultRequest:
      memory: 1Gi      # Default request (applied if not specified)
    max:
      memory: 1Gi      # Maximum memory allowed
    min:
      memory: 500Mi    # Minimum memory required
    type: Container
```

**LimitRange Fields Explained:**

| Field | Description | Applies To |
|-------|-------------|------------|
| **`default`** | Default resource **limit** applied to containers that don't specify limits | Limits |
| **`defaultRequest`** | Default resource **request** applied to containers that don't specify requests | Requests |
| **`max`** | Maximum resource **limit** allowed - containers cannot exceed this | Limits |
| **`min`** | Minimum resource **request** required - containers must request at least this | Requests |
| **`type`** | Scope of the limit range - `Container`, `Pod`, or `PersistentVolumeClaim` | All |

**How LimitRange Works:**

1. **Default Values**: If a pod is created without specifying `resources.requests` or `resources.limits`, the LimitRange `defaultRequest` and `default` values are automatically applied
2. **Validation**: If a pod specifies resources, they must fall within the `min` and `max` constraints
3. **Namespace Scope**: LimitRange applies to all pods created in the namespace where it's defined
4. **New Pods Only**: LimitRange **applies to newer pods being created** - existing pods are not affected

**Example Scenario:**

*If you create a LimitRange with:*
- `default: cpu: 500m` and `defaultRequest: cpu: 500m`
- `min: cpu: 100m` and `max: cpu: "1"`

*And then create a pod without specifying resources:*
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
    # No resources specified
```

*Kubernetes will automatically apply:*
- `resources.requests.cpu: 500m` (from `defaultRequest`)
- `resources.limits.cpu: 500m` (from `default`)

**Creating a LimitRange:**

```bash
kubectl create -f limit-range-cpu.yaml
kubectl create -f limit-range-memory.yaml
```

**Viewing LimitRanges:**

```bash
kubectl get limitrange
# or
kubectl get limits
```

**Describing a LimitRange:**

```bash
kubectl describe limitrange cpu-resource-constraint
```

### ResourceQuotas

*ResourceQuotas define hard restrictions on resource consumption for a namespace. Unlike LimitRange which applies to individual containers, ResourceQuotas set aggregate limits for all resources in a namespace across multiple nodes.*

**What ResourceQuotas Do:**
- Set **hard limits** on total resource consumption in a namespace
- Apply to **all pods** in the namespace combined (aggregate limits)
- **Span across multiple nodes** - quotas are namespace-wide, not node-specific
- Prevent a namespace from consuming more than its allocated share of cluster resources

**Creating a ResourceQuota:**

**Example ResourceQuota:**

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4        # Total CPU requests across all pods
    requests.memory: 4Gi   # Total memory requests across all pods
    limits.cpu: 10         # Total CPU limits across all pods
    limits.memory: 10Gi    # Total memory limits across all pods
```

**ResourceQuota Fields Explained:**

| Field | Description | Example |
|-------|-------------|---------|
| **`requests.cpu`** | Total CPU requests allowed across all pods in namespace | `requests.cpu: 4` |
| **`requests.memory`** | Total memory requests allowed across all pods in namespace | `requests.memory: 4Gi` |
| **`limits.cpu`** | Total CPU limits allowed across all pods in namespace | `limits.cpu: 10` |
| **`limits.memory`** | Total memory limits allowed across all pods in namespace | `limits.memory: 10Gi` |

**How ResourceQuotas Work:**

1. **Namespace Scope**: ResourceQuota applies to the entire namespace where it's created
2. **Aggregate Limits**: Limits are calculated by summing resources from **all pods** in the namespace
3. **Cross-Node**: Resources are counted across **all nodes** - pods can be on different nodes
4. **Hard Restrictions**: When the quota is reached, **no new pods can be created** that would exceed the quota
5. **Validation**: Kubernetes validates resource requests/limits against the quota before allowing pod creation

**Example Scenario:**

*If you create a ResourceQuota with:*
- `requests.cpu: 4`
- `requests.memory: 4Gi`
- `limits.cpu: 10`
- `limits.memory: 10Gi`

*And you have pods in the namespace:*
- Pod 1: `requests.cpu: 1`, `requests.memory: 1Gi`
- Pod 2: `requests.cpu: 2`, `requests.memory: 2Gi`
- Pod 3: `requests.cpu: 1`, `requests.memory: 1Gi`

*Total used:*
- `requests.cpu: 4` (1 + 2 + 1) ✅ Within quota
- `requests.memory: 4Gi` (1Gi + 2Gi + 1Gi) ✅ Within quota

*If you try to create Pod 4 with `requests.cpu: 1`:*
- Total would be: `requests.cpu: 5` ❌ **Exceeds quota - pod creation will be denied**

**Creating a ResourceQuota:**

```bash
kubectl create -f resource-quota.yaml
```

**Or create imperatively:**

```bash
kubectl create quota my-resource-quota \
  --hard=requests.cpu=4,requests.memory=4Gi,limits.cpu=10,limits.memory=10Gi \
  --namespace=my-namespace
```

**Viewing ResourceQuotas:**

```bash
kubectl get resourcequota
# or shorthand
kubectl get quota
```

**Viewing ResourceQuota details:**

```bash
kubectl describe resourcequota my-resource-quota
```

*This shows:*
- The hard limits set
- Current usage (how much of the quota is consumed)
- Remaining quota

**ResourceQuota vs LimitRange:**

| Aspect | ResourceQuota | LimitRange |
|--------|---------------|------------|
| **Scope** | Namespace-wide (all pods combined) | Individual containers |
| **Purpose** | Set aggregate limits for namespace | Set defaults/constraints per container |
| **Level** | Namespace level | Container/Pod level |
| **Enforcement** | Prevents namespace from exceeding total quota | Sets defaults and validates per-container resources |
| **Cross-Node** | Yes - spans across all nodes | No - applies to individual containers |

**Important Notes:**
- **Hard Restrictions**: ResourceQuotas are hard limits - they cannot be exceeded
- **Namespace Isolation**: Prevents one namespace from consuming all cluster resources
- **Multi-Node**: Quotas span across all nodes - pods can be distributed anywhere
- **Aggregate Calculation**: All pod resources in the namespace are summed together
- **Pod Creation**: New pods are rejected if they would cause the namespace to exceed its quota
- **Existing Pods**: ResourceQuotas apply to all pods in the namespace, including existing ones
- **Requests**: Kubernetes guarantees that the requested resources are available on the node
- **Limits**: Prevent containers from consuming more than specified, protecting other pods on the node
- **Overcommitment**: Nodes can have pods with total requests exceeding node capacity (but limits prevent overuse)
- **Pending State**: Pods in Pending state are waiting for resources - this is normal behavior when cluster is at capacity
- **Best Practice**: Always set both requests and limits to ensure proper scheduling and resource protection
- **Equal Values**: You can set requests and limits to the same value for guaranteed resources

---

## Resource Management

*This section covers resource allocation, limits, and quotas. Resource Requests and Limits are defined in pod specifications. LimitRange sets defaults per container. ResourceQuotas set namespace-wide limits.*

---

## Core Resources

## ReplicaSets

| Docker Command | Kubernetes Pod Field |
|---------------|---------------------|
| `docker run --user=1001 ubuntu sleep 3600` | `securityContext.runAsUser: 1001` |
| `docker run --cap-add MAC_ADMIN ubuntu` | `securityContext.capabilities.add: ["MAC_ADMIN"]` |

**Security Context Levels:**

Security Context can be configured at two levels:
1. **Pod Level** - `spec.securityContext` - Applies to all containers in the pod
2. **Container Level** - `spec.containers[].securityContext` - Applies to a specific container

**Important Rule:**
- **Pod level configuration overrides container level configuration** for settings that can be set at both levels
- However, some settings (like capabilities) can **only** be set at the container level

#### Pod Level Security Context

*Configure security settings that apply to all containers in the pod.*

**Example - Pod Level:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
```

**Key points:**
- `spec.securityContext` applies to all containers in the pod
- `runAsUser: 1000` sets the user ID that all containers will run as
- This overrides the default user from the container image

#### Container Level Security Context

*Configure security settings for a specific container. Move the security context into the container definition for container-level security.*

**Example - Container Level:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
```

**Key points:**
- `spec.containers[].securityContext` applies only to that specific container
- Container-level settings can be overridden by pod-level settings (if both are specified)
- Each container can have its own security context

#### Combining Pod and Container Level

*When both pod-level and container-level security contexts are specified, pod-level settings override container-level settings.*

**Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000  # Pod level - applies to all containers
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 2000  # Container level - but will be overridden by pod level (1000)
```

*In this case, the container will run as user 1000 (pod level), not 2000 (container level).*

#### Capabilities (Container Level Only)

*Linux capabilities allow fine-grained control over what privileges a process can use. Capabilities can **only** be configured at the container level, not at the pod level.*

**Example - Adding Capabilities:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

**Key points:**
- `capabilities` can **only** be set at `spec.containers[].securityContext`, not at pod level
- `capabilities.add` adds specific capabilities to the container
- `capabilities.drop` removes capabilities (drops all by default, then adds back what you specify)
- Common capabilities include: `NET_ADMIN`, `SYS_ADMIN`, `MAC_ADMIN`, etc.

**Example - Dropping Capabilities:**

```yaml
securityContext:
  capabilities:
    drop: ["ALL"]  # Drop all capabilities
    add: ["NET_BIND_SERVICE"]  # Then add only what's needed
```

**Common Security Context Settings:**

| Setting | Pod Level | Container Level | Description |
|---------|-----------|----------------|-------------|
| `runAsUser` | ✅ | ✅ | User ID to run as (pod level overrides container) |
| `runAsGroup` | ✅ | ✅ | Group ID to run as |
| `runAsNonRoot` | ✅ | ✅ | Require non-root user |
| `fsGroup` | ✅ | ❌ | Group ID for volumes |
| `capabilities` | ❌ | ✅ | Linux capabilities (container level only) |
| `privileged` | ❌ | ✅ | Run container in privileged mode |
| `readOnlyRootFilesystem` | ❌ | ✅ | Mount root filesystem as read-only |

**Best Practices:**

1. **Run as non-root**: Always use `runAsNonRoot: true` or specify a non-root `runAsUser`
2. **Drop all capabilities**: Start with `capabilities.drop: ["ALL"]` and add only what's needed
3. **Read-only root filesystem**: Use `readOnlyRootFilesystem: true` when possible
4. **Least privilege**: Grant only the minimum permissions and capabilities required
5. **Container-level for capabilities**: Remember that capabilities can only be set at container level

---

## Security

### Security Context

*Security Context defines privilege and access control settings for a Pod or Container. It allows you to control various security-related aspects like user IDs, capabilities, and permissions.*

**Docker vs Kubernetes:**

### Overview

*ReplicaSets is a process that spans across multiple Nodes and ensures that a specified number of pods are always available.*

### Manifest Examples

**Correct ReplicaSet manifest:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-1
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Incorrect ReplicaSet manifest (label mismatch):**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-2
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend    # ← Selector looking for 'tier: frontend'
  template:
    metadata:
      labels:
        tier: nginx      # ← Template has 'tier: nginx' (MISMATCH!)
    spec:
      containers:
      - name: nginx
        image: nginx
```

**Key Difference:**
*The `selector.matchLabels` (e.g., `tier: frontend`) **MUST match** `template.metadata.labels` (e.g., `tier: frontend`). If they don't match, the ReplicaSet will fail to create pods because it cannot find pods with the labels it's looking for.*

### Basic Commands

```bash
k get replicasets
```

**Get details of a specific ReplicaSet:**
```bash
k describe replicasets <replicasetName>
```
*This shows the number of replicas desired in this ReplicaSet.*

**Delete ReplicaSets:**
```bash
k delete rs replicaset-1 replicaset-2
```

### Scaling & Updates

**Option 1: Use kubectl edit (recommended):**
```bash
kubectl edit replicaset new-replica-set
```
*Modify the replicas in the editor and save the file.*

**Option 2: Use kubectl scale:**
```bash
kubectl scale rs new-replica-set --replicas=5
```
*This scales the ReplicaSet up to 5 pods directly.*

### Common Errors

**Create/Edit/Apply Conflict Error:**
*When you create a ReplicaSet using `k create -f`, then edit the file with `vi` to change the replica count, and try to apply with `k apply -f`, it may fail with a Conflict error. This happens because:*
- *The resource was created with `kubectl create`, which doesn't add the `kubectl.kubernetes.io/last-applied-configuration` annotation*
- *The object on the server may have been modified externally*
- *`kubectl apply` requires the resource to have been created declaratively with `--save-config` or `kubectl apply`*

## Kubernetes Deployments

### Overview

*Deployments manage updates on a rolling basis. When you upgrade a version or make changes to the current deployment, the changes happen one by one - first the pods get deleted from the existing ReplicaSet and a new pod gets created in a new ReplicaSet to reflect these changes - on a rolling basis.*

### Basic Commands

**View all Kubernetes objects:**
```bash
k get all
```

**Generate a Deployment manifest YAML without creating the deployment:**
```bash
k create deploy http-frontend --image=http:2.4-alpine --dry-run=client -o yaml > new-deployment.yaml
```

**Edit a deployment:**
```bash
k edit deploy http-frontend
```

## Namespaces

*By default, we are in the `default` namespace. Namespaces provide a way to organize and isolate resources within a Kubernetes cluster.*

### Built-in Namespaces

**Default Namespace:**
*All resources are created in the `default` namespace unless otherwise specified.*

**kube-system Namespace:**
*Kubernetes creates DNS service and networking solutions in the `kube-system` namespace at cluster startup. This namespace contains system components that should not be modified.*

**kube-public Namespace:**
*This namespace is used to share resources across the cluster. Resources in this namespace are accessible to all users.*

### Creating Namespaces

**Method 1: Using YAML manifest**

**namespace-dev.yml:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

**Create namespace from file:**
```bash
kubectl create -f namespace-dev.yml
```

**Method 2: Using command directly**
```bash
kubectl create namespace dev
```

### Working with Namespaces

**List all namespaces:**
```bash
k get namespace
```

**View pods in a specific namespace:**
```bash
k get pods -n <namespace_name>
```

**View pods in all namespaces:**
```bash
k get pods --all-namespaces
```

**Create a resource in a specific namespace:**
```bash
kubectl create -f pod-definition.yml --namespace=dev
```

*Or specify the namespace in the YAML file:*
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  namespace: dev
  labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
```

**Change the default namespace for the current context:**
```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```
*This command first gets the current context and then sets the namespace to this context. After this, you don't need to run `--namespace` or `-n` every time - commands will default to the specified namespace.*

**Explicitly specify namespace (works regardless of context):**
```bash
kubectl get pods --namespace=dev
kubectl get pods --namespace=default
kubectl get pods --namespace=prod
```

### Service Discovery

**Within the Same Namespace:**
*Applications within a namespace can talk to each other by their service names directly:*
```python
mysql.connect("db-service")
```

**Across Different Namespaces:**
*To connect to a service present in another namespace, use the fully qualified domain name (FQDN) format:*
```
<service-name>.<namespace>.svc.cluster.local
```

*Example: From the `default` namespace, a web-pod can connect to a service in the `dev` namespace:*
```python
mysql.connect("db-service.dev.svc.cluster.local")
```

### Resource Quotas

*Each namespace can have a quota of resources allocated to it. This helps manage resource consumption and prevents one namespace from consuming all cluster resources.*

**compute-quota.yaml:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

**Create the ResourceQuota:**
```bash
kubectl create -f compute-quota.yaml
```

*This ResourceQuota limits the `dev` namespace to:*
- *Maximum 10 pods*
- *Total CPU requests: 4 cores*
- *Total memory requests: 5Gi*
- *Total CPU limits: 10 cores*
- *Total memory limits: 10Gi*

---

## Networking

## Services

### Overview

*Services enable end users to access services hosted on pods. They also enable different pods to communicate with each other within the cluster using ClusterIP in a distributed microservice architecture.*

*The Service acts as a stable endpoint that maps to the pods behind it. It provides:*
- *A stable IP address (ClusterIP) for internal communication*
- *Load balancing across multiple pods*
- *Service discovery within the cluster*
- *External access (with NodePort or LoadBalancer types)*

### ClusterIP Service

*ClusterIP is the default service type in Kubernetes. It provides a stable, internal IP address that is only accessible within the cluster. This is the primary method for inter-service communication in a microservices architecture.*

**Why ClusterIP is needed:**

*In a full-stack application, your front-end needs to talk to the backend, and the backend needs to talk to the database. Each service may have multiple instances (pods) running, and you can't use individual pod IPs for communication because:*
- *Pod IPs are ephemeral (they change when pods are recreated)*
- *You have multiple instances of each service (multiple front-end pods, multiple backend pods, multiple database pods)*
- *You need a single, stable endpoint to access a service regardless of how many pods are running*

*ClusterIP solves this by providing a global, stable IP address for each service (front-end, backend, database). All pods within the cluster can communicate with each other using these ClusterIP addresses, and Kubernetes automatically load balances the traffic across all pods behind the service.*

**Key Characteristics:**
- *Only accessible from within the cluster (not accessible from outside)*
- *Provides a stable IP address that doesn't change*
- *Default service type (if you don't specify `type`, it defaults to ClusterIP)*
- *Enables service discovery using DNS (services can be reached by their name)*

### NodePort Service

*NodePort is one of the service types that exposes the service on a static port on each Node. This allows external applications to access the service using `<NodeIP>:<NodePort>`.*

**The Three Ports in a NodePort Service:**

1. **`targetPort`** - The port on the pod where the application is running (e.g., port 80 on the container)
2. **`port`** - The port on the Service object in Kubernetes (e.g., port 80 on the service)
3. **`nodePort`** - The port on the Node itself (range: 30000-32767)

**How it works:**
*An external application connects to `<NodeIP>:<NodePort>` (e.g., `192.168.0.1:32333`). The Service intercepts this traffic and forwards it to the respective pod(s) on the `targetPort` (e.g., port 80).*

**The Selector:**
*The `selector` field in the service definition is responsible for mapping the service to pods using the labels defined in the pod YAML. The service finds all pods that match the selector labels and routes traffic to them.*

**Load Balancing:**
*The Service acts like a LoadBalancer itself. When there are multiple pods with the same labels running, the incoming request is intercepted by the service. The service detects that it has multiple pods running the same instance (e.g., 3 pods) and uses a random algorithm to pick one of them to process the request. This provides automatic load distribution across all matching pods.*

*What if your services are distributed across multiple Nodes? For instance, if the web-service pods are running on 3 different Nodes, the service ensures that it spans across multiple nodes in the cluster and forwards the request to a pod in a random manner, regardless of which node the pod is running on. This means the service provides true cluster-wide load balancing.*

### Service Manifest Example

**ClusterIP Service Example (back-end service):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP        # Default type (can be omitted)
  ports:
  - targetPort: 80       # Port on the pod/container
    port: 80             # Port on the Service (ClusterIP)
  selector:
    app: myapp           # Must match pod labels
    type: back-end       # Must match pod labels
```

**NodePort Service Example:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80      # Port on the pod/container
    port: 80            # Port on the Service
    nodePort: 30008     # Port on the Node (30000-32767)
  selector:
    app: myapp          # Must match pod labels
    type: front-end     # Must match pod labels
```

**Important Notes:**
- *The `selector` labels (e.g., `app: myapp`, `type: front-end`) **MUST match** the labels defined in your pod YAML*
- *If you don't specify a `nodePort`, Kubernetes will automatically assign one from the 30000-32767 range*
- *The `port` and `targetPort` can be different (e.g., service port 80 → pod port 8080)*

**Example Pod YAML that would match the above service:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp          # ← Matches service selector
    type: front-end     # ← Matches service selector
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80  # ← This is the targetPort for the service
```

### Creating Services Imperatively (Using kubectl expose)

*You can create services on the fly using the imperative `kubectl expose` command. This automatically uses the pod's labels as selectors.*

**Create a ClusterIP Service:**
```bash
k expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml > redis-service.yaml
```

**Create a NodePort Service:**
```bash
k expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml > nginx-service.yaml
```

**Important Notes about `kubectl expose`:**

- *The `--port` parameter specifies the **port on the Service** (the service port), **not** the targetPort*
- *Kubernetes automatically uses the pod's container port as the `targetPort`. If the container is listening on port 80, that becomes the targetPort*
- *This command **automatically uses the pod's labels as selectors** - you don't need to specify them manually*
- *For NodePort services: **You cannot specify the nodePort in the command**. You must generate the definition file and then manually add the `nodePort` field in the YAML before creating the service*
- *The `--dry-run=client -o yaml` flags generate the YAML file without creating the service, so you can review and modify it first*

**Example workflow for NodePort:**
1. Generate the service YAML: `k expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml > nginx-service.yaml`
2. Edit the file to add `nodePort: 30080` in the ports section
3. Create the service: `k create -f nginx-service.yaml`

### Basic Commands

**List all services:**
```bash
k get services
# or shorthand
k get svc
```

**Get detailed information about a service:**
```bash
k describe svc <service-name>
```
*This shows the service endpoints (pods it's connected to), ports, and selectors.*

**View services with endpoints:**
```bash
k get svc -o wide
```
*Shows the ClusterIP and external endpoints.*

**Generate a Service manifest YAML without creating it:**
```bash
k create service nodeport myapp-service --tcp=80:80 --dry-run=client -o yaml > service-definition.yml
```

**Create a service from a file:**
```bash
k create -f service-definition.yml
```

**Edit a service:**
```bash
k edit svc <service-name>
```

**Delete a service:**
```bash
k delete svc <service-name>
```

**Check which pods are selected by a service:**
```bash
k get pods -l app=myapp,type=front-end
```
*This uses the same label selector that the service uses to find matching pods.*

## Service Accounts

### Overview

*Service Accounts are Kubernetes identities used by applications, pods, and services to authenticate and interact with the Kubernetes API server. They provide a way for applications running in pods to prove their identity when making API requests.*

**What Service Accounts Do:**
- Provide identity for pods and applications
- Enable authentication to the Kubernetes API
- Allow fine-grained access control through RBAC (Role-Based Access Control)
- Automatically manage tokens for API authentication

**Key Concepts:**
- Service Accounts are namespace-scoped resources
- Each pod can have one service account attached
- Tokens are automatically created and mounted for service accounts
- Tokens are used to authenticate API requests

### Default Service Account

*Every namespace in Kubernetes has a default service account that is automatically created.*

**Characteristics:**
- **Name**: `default`
- **Automatic Creation**: Created automatically when a namespace is created
- **Automatic Attachment**: Automatically attached to pods if no service account is specified
- **Limited Permissions**: By default, has minimal permissions (may not be able to access the API)

**Viewing the Default Service Account:**

```bash
kubectl get serviceaccount
# or shorthand
kubectl get sa
```

*Output example:*
```
NAME      SECRETS   AGE
default   1         218d
```

**Key Points:**
- The default service account exists in every namespace
- If you don't specify a service account in your pod, it uses the `default` service account
- The default service account may have limited or no API permissions depending on your cluster configuration

### Creating Service Accounts

*You can create custom service accounts for your applications to have specific identities and permissions.*

**Method 1: Using kubectl (Imperative)**

```bash
kubectl create serviceaccount dashboard-sa
```

*Output:*
```
serviceaccount "dashboard-sa" created
```

**Method 2: Using YAML (Declarative)**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default  # Optional - defaults to current namespace
```

**Create from YAML:**
```bash
kubectl create -f serviceaccount.yaml
```

**Key Points:**
- Service accounts are namespace-scoped
- You can create multiple service accounts in the same namespace
- Custom service accounts are useful for applications that need specific API permissions
- Service accounts are used by pods, but can also be used by external applications (CI/CD, monitoring tools)

### Attaching Service Accounts to Pods

*To use a specific service account in a pod, specify it using the `serviceAccountName` field in the pod definition.*

**Pod Definition with Service Account:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
  - name: my-kubernetes-dashboard
    image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa  # Attach custom service account
```

**What Happens When a Service Account is Attached:**

When a service account is attached to a pod, Kubernetes automatically:

1. **Creates a Token**: Generates a JWT (JSON Web Token) for the service account
2. **Mounts as Projected Volume**: Mounts the token and related files into the pod at:
   ```
   /var/run/secrets/kubernetes.io/serviceaccount/
   ```
3. **Automatic Token Rotation**: Periodically rotates the token for security
4. **Automatic Expiration**: Expires the token when the pod is deleted

**Files Mounted in Pod:**

The following files are automatically mounted in every pod:

| File | Description |
|------|-------------|
| `token` | JWT token for API authentication |
| `ca.crt` | Certificate Authority certificate |
| `namespace` | The namespace the service account belongs to |

**If No Service Account is Specified:**

If you don't specify `serviceAccountName` in your pod definition:
- Kubernetes automatically uses the `default` service account
- The default service account token is still mounted at the same location
- The pod may have limited or no API permissions

**Example - Pod Using Default Service Account:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx
  # No serviceAccountName specified - uses 'default' service account
```

### Service Account Tokens

*Tokens are JWT (JSON Web Token) credentials that prove the identity of a service account when making API requests.*

**Token Structure:**

A JWT token has three parts:
```
header.payload.signature
```

**Token Claims (Payload):**

When decoded, a service account token contains:

```json
{
  "iss": "https://kubernetes.default.svc.cluster.local",
  "sub": "system:serviceaccount:default:dashboard-sa",
  "aud": ["https://kubernetes.default.svc.cluster.local"],
  "exp": 1664037763,
  "iat": 1664034163,
  "kubernetes.io": {
    "namespace": "default",
    "serviceaccount": {
      "name": "dashboard-sa",
      "uid": "7d0fdfe8-fbf3-4ff9-b362-d004297a73f6"
    }
  }
}
```

**Key Token Fields:**

| Field | Description |
|------|-------------|
| `iss` | Issuer - who created the token (Kubernetes API) |
| `sub` | Subject - the service account identity |
| `aud` | Audience - who the token is intended for |
| `exp` | Expiration time (Unix timestamp) |
| `iat` | Issued at time (Unix timestamp) |
| `kubernetes.io` | Kubernetes-specific claims (namespace, service account details) |

**Automatic Token Management:**

- **Creation**: Tokens are automatically created when a service account is attached to a pod
- **Rotation**: Tokens are automatically rotated periodically for security
- **Expiration**: Tokens expire when the pod is deleted
- **Location**: Tokens are mounted at `/var/run/secrets/kubernetes.io/serviceaccount/token`

**Creating Tokens for External Applications:**

For external applications (CI/CD pipelines, monitoring tools) that need to authenticate to the Kubernetes API:

```bash
kubectl create token dashboard-sa --duration=2h
```

*This command:*
- Creates a new token for the `dashboard-sa` service account
- Sets token expiration to 2 hours (`--duration=2h`)
- Outputs the token (JWT string) that can be used in API requests

**Token Duration Options:**

```bash
# 1 hour token
kubectl create token dashboard-sa --duration=1h

# 2 hour token
kubectl create token dashboard-sa --duration=2h

# 24 hour token
kubectl create token dashboard-sa --duration=24h

# Default duration (usually 1 hour)
kubectl create token dashboard-sa
```

**Token Signing:**

- **Signing**: Tokens are signed using the Kubernetes API server's **private key** (`sa.key`)
- **Verification**: Tokens are verified using the Kubernetes API server's **public key** (`sa.pub`)
- **Key Location**: 
  - Kubeadm clusters: `/etc/kubernetes/pki/sa.key` (private) and `/etc/kubernetes/pki/sa.pub` (public)
  - Cloud providers: Managed by the cloud provider
- **Security**: The private key must remain secret - only the API server can create valid tokens

### Dashboard Authentication Flow

*When you paste a service account token into the Kubernetes Dashboard, here's the step-by-step authentication flow:*

**Step-by-Step Authentication Flow:**

```
┌─────────────────────────────────────────────────────┐
│  STEP 1: You Generate Token                         │
└─────────────────────────────────────────────────────┘
$ kubectl create token dashboard-sa --duration=2h
eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJ1cm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC4uLi4=

                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│  STEP 2: You Paste Token in Dashboard               │
│                                                      │
│  Dashboard UI:                                      │
│  ┌──────────────────────────────────────┐          │
│  │  Authentication                      │          │
│  │  ┌────────────────────────────────┐  │          │
│  │  │ Token: [paste here]            │  │          │
│  │  └────────────────────────────────┘  │          │
│  │  [Login Button]                      │          │
│  └──────────────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│  STEP 3: Dashboard Sends Token to API Server       │
│                                                      │
│  HTTP Request:                                       │
│  POST https://<API_SERVER>:6443/api/v1/...         │
│  Headers:                                            │
│    Authorization: Bearer eyJhbGciOiJSUzI1NiIs...    │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│  STEP 4: API Server Validates Token                 │
│                                                      │
│  API Server Process:                                 │
│  1. Receives token from dashboard                    │
│  2. Extracts header.payload.signature               │
│  3. Reads PUBLIC KEY (sa.pub)                       │
│  4. Verifies signature using PUBLIC KEY             │
│     - Decrypts signature with public key             │
│     - Compares with hash of header+payload          │
│  5. Checks token claims:                            │
│     - exp: Not expired? ✅                           │
│     - iss: Issuer is Kubernetes? ✅                 │
│     - sub: serviceaccount:default:dashboard-sa ✅   │
│  6. If all valid → Authentication SUCCESS ✅         │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│  STEP 5: API Server Checks Authorization (RBAC)    │
│                                                      │
│  RBAC Check:                                         │
│  - Does serviceaccount:default:dashboard-sa          │
│    have permission to access /api/v1/...?           │
│  - Checks Role/RoleBinding/ClusterRole              │
│  - If authorized → Returns data ✅                  │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│  STEP 6: Dashboard Receives Response                │
│                                                      │
│  Dashboard:                                          │
│  - Token is valid ✅                                 │
│  - User is authenticated ✅                         │
│  - Shows Kubernetes resources                        │
│  - Can now make API calls using this token          │
└─────────────────────────────────────────────────────┘
```

**Key Points:**

1. **Token Generation**: Token is created and signed by the API server using its private key
2. **Token Pasting**: You paste the token into the dashboard's authentication form
3. **API Request**: Dashboard sends the token in the `Authorization: Bearer` header
4. **Token Verification**: API server verifies the token signature using its public key
5. **Authentication**: If token is valid (signature matches, not expired, issuer correct) → authenticated
6. **Authorization**: RBAC checks if the service account has permissions for the requested action
7. **Access Granted**: If authenticated and authorized, dashboard receives data and displays resources

**What Happens Internally:**

- **Token Signing**: API server uses its **private key** (`sa.key`) to sign the token
- **Token Verification**: API server uses its **public key** (`sa.pub`) to verify the token signature
- **Digital Signature Pattern**: 
  - Private key signs (only API server can create valid tokens)
  - Public key verifies (anyone can verify, but only API server can sign)
- **Security**: This ensures only the API server can create valid tokens, preventing token forgery

### Using Tokens for API Authentication

*Service account tokens are used to authenticate requests to the Kubernetes API server.*

**How Authentication Works:**

1. **Pod Reads Token**: Application in pod reads token from `/var/run/secrets/kubernetes.io/serviceaccount/token`
2. **Sends in Request**: Token is sent in the `Authorization: Bearer` header
3. **API Server Validates**: API server validates token signature, expiration, and issuer
4. **Authorization Check**: RBAC checks if the service account has permissions for the requested action
5. **Response**: API server returns response if authenticated and authorized

**Example - Reading Token from Pod:**

```bash
# Read token from inside a pod
kubectl exec -it my-kubernetes-dashboard -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

**Example - Using Token in curl Command:**

```bash
# Get the token
TOKEN=$(kubectl exec my-kubernetes-dashboard -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Use token to authenticate API request
curl https://<API_SERVER_IP>:6443/api \
  --insecure \
  --header "Authorization: Bearer $TOKEN"
```

**Example - Decoding Token (for inspection):**

```bash
# Get token
TOKEN=$(kubectl exec my-kubernetes-dashboard -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)

# Decode and view token claims
echo $TOKEN | jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64d | fromjson'
```

**Token in API Request Header:**

```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJ1cm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC4uLi4=
```

**Important Notes:**
- Tokens are sensitive credentials - treat them like passwords
- Tokens expire - check the `exp` field for expiration time
- Use HTTPS in production - never use `--insecure` in production
- RBAC controls what the service account can do - tokens only prove identity

### Basic Commands

**List all service accounts:**
```bash
kubectl get serviceaccount
# or shorthand
kubectl get sa
```

**Get details of a specific service account:**
```bash
kubectl describe serviceaccount dashboard-sa
# or
kubectl describe sa dashboard-sa
```

**Create a service account:**
```bash
kubectl create serviceaccount dashboard-sa
```

**Create a token for a service account:**
```bash
kubectl create token dashboard-sa --duration=2h
```

**Delete a service account:**
```bash
kubectl delete serviceaccount dashboard-sa
# or
kubectl delete sa dashboard-sa
```

**View service accounts in a specific namespace:**
```bash
kubectl get sa -n <namespace>
```

**Read token from inside a pod:**
```bash
kubectl exec -it <pod-name> -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

**List all files in service account mount:**
```bash
kubectl exec -it <pod-name> -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

**Key Takeaways:**
- Service accounts provide identity for pods and applications
- Every namespace has a default service account
- Tokens are automatically created and mounted when a service account is attached to a pod
- Tokens are used in the `Authorization: Bearer` header for API authentication
- RBAC controls what service accounts can do (authorization)
- Tokens are automatically rotated and expire when pods are deleted
- Tokens are signed with the API server's private key and verified with the public key

## Pod Scheduling and Node Selection

*Kubernetes provides several mechanisms to control where pods are scheduled: Taints and Tolerations, NodeSelector, and Node Affinity. These mechanisms allow you to restrict pod placement based on node characteristics.*

### Taints and Tolerations

#### Overview

*Taints and Tolerations are Kubernetes mechanisms that allow nodes to repel or evict pods that don't have matching tolerations. Taints are applied to nodes, and tolerations are applied to pods.*

**Key Concept:**
- **Taints** tell nodes: "Only accept pods with matching tolerations"
- **Tolerations** tell pods: "I can tolerate certain taints"
- **This does NOT guarantee pod placement** - the scheduler still decides where pods go
- **For pod-to-node affinity**, use Node Affinity instead

**How It Works:**
- Taints are set on nodes to repel pods
- Pods must have matching tolerations to be scheduled on tainted nodes
- If a pod doesn't tolerate a node's taint, it cannot be scheduled on that node
- Tolerations allow pods to be scheduled, but the scheduler makes the final decision

#### Taint Effects

*There are three types of taint effects that determine what happens to pods that don't tolerate the taint:*

| Taint Effect | Behavior for Non-Tolerating Pods |
|--------------|----------------------------------|
| **NoSchedule** | Pods that do not tolerate this taint will **not be scheduled** on the node |
| **PreferNoSchedule** | Kubernetes will try to avoid scheduling pods without matching toleration, but it's not guaranteed |
| **NoExecute** | Pods that do not tolerate this taint will be **evicted** from the node (if already running) and new pods **will not be scheduled** |

**Important Notes:**
- **NoSchedule**: Prevents new pods from being scheduled on the node
- **PreferNoSchedule**: Soft preference - scheduler tries to avoid the node but may schedule if needed
- **NoExecute**: Most aggressive - **evicts existing pods** that don't tolerate the taint, in addition to preventing new scheduling

#### Creating Taints

*Apply a taint to a node using the `kubectl taint` command.*

**Basic Syntax:**

```bash
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>
```

**Example - NoSchedule Taint:**

```bash
kubectl taint nodes node-name app=special:NoSchedule
```

**Example - NoExecute Taint:**

```bash
kubectl taint nodes node-name app=database:NoExecute
```

**Viewing Taints on a Node:**

```bash
kubectl describe node <node-name> | grep Taint
```

*Output example:*
```
Taints: node-role.kubernetes.io/master:NoSchedule
```

**Master Node Default Taint:**

- Master/Control Plane nodes have a **default taint** set automatically
- This prevents regular pods from being scheduled on master nodes
- Master nodes typically have: `node-role.kubernetes.io/master:NoSchedule`
- This reserves master nodes for Kubernetes control plane components

**Removing a Taint:**

```bash
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>-
```

*Note the `-` at the end to remove the taint.*

#### Tolerations

*Tolerations are added to pod specifications to allow pods to be scheduled on nodes with matching taints.*

**Pod Definition with Toleration:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
  tolerations:
  - key: app
    operator: Equal
    value: special
    effect: NoSchedule
```

**Toleration Fields:**

| Field | Description | Values |
|-------|-------------|--------|
| **key** | The taint key to match | Any string (e.g., `app`, `node-role.kubernetes.io/master`) |
| **operator** | How to match the value | `Equal` (exact match) or `Exists` (key exists, ignore value) |
| **value** | The taint value (required if operator is `Equal`) | Any string |
| **effect** | The taint effect to tolerate | `NoSchedule`, `PreferNoSchedule`, or `NoExecute` |

**Example - Tolerating Master Node Taint:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: master-pod
spec:
  containers:
  - name: app
    image: nginx
  tolerations:
  - key: node-role.kubernetes.io/master
    operator: Exists
    effect: NoSchedule
```

**Example - Tolerating Multiple Taints:**

```yaml
tolerations:
- key: app
  operator: Equal
  value: special
  effect: NoSchedule
- key: environment
  operator: Equal
  value: production
  effect: NoExecute
```

#### Important Notes

**Tolerations Do NOT Guarantee Placement:**

- **Tolerations allow pods to be scheduled** on tainted nodes, but **do not force** placement
- The scheduler may place a pod with matching tolerations on a different node
- The scheduler considers resource availability, node affinity, and other factors

**Taints and Tolerations vs Node Affinity:**

- **Taints/Tolerations**: Tell nodes to **repel** pods without matching tolerations
- **Node Affinity**: Tell pods to **prefer** or **require** specific nodes
- **For pod-to-node restriction**, use Node Affinity, not Taints/Tolerations

**NoExecute Effect:**

- **Evicts existing pods** that don't tolerate the taint
- Happens immediately when taint is applied
- Only pods with matching tolerations can remain on the node

**Master Node Taint:**

- Master nodes have a **default taint**: `node-role.kubernetes.io/master:NoSchedule`
- This prevents regular workloads from being scheduled on master nodes
- Only system pods and pods with matching tolerations can run on master nodes

#### Basic Commands

**Apply a taint to a node:**

```bash
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>
```

**Remove a taint from a node:**

```bash
kubectl taint nodes <node-name> <key>=<value>:<taint-effect>-
```

**View taints on a node:**

```bash
kubectl describe node <node-name> | grep Taint
```

**View all nodes and their taints:**

```bash
kubectl get nodes
kubectl describe node <node-name>
```

**Key Takeaways:**
- Taints repel pods; tolerations allow pods to be scheduled on tainted nodes
- Three taint effects: `NoSchedule`, `PreferNoSchedule`, `NoExecute`
- `NoExecute` evicts existing pods that don't tolerate the taint
- Tolerations allow scheduling but don't guarantee placement - scheduler decides
- Master nodes have default taints to prevent regular workloads
- Use Node Affinity for pod-to-node placement requirements

### Node Labels

*Labels are key-value pairs attached to nodes that can be used to identify node characteristics (e.g., size, environment, hardware type).*

**Labeling a Node:**

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

**Example - Label Node by Size:**

```bash
kubectl label nodes node-1 size=Large
```

**Viewing Node Labels:**

```bash
kubectl get nodes --show-labels
# or
kubectl describe node <node-name>
```

**Use Case:**
- Label nodes with characteristics like `size=Large`, `size=Medium`, `size=Small`
- Label nodes with environment: `environment=production`, `environment=staging`
- Use labels to select specific nodes for pod scheduling

### NodeSelector

*NodeSelector is a simple way to restrict a pod to only run on nodes with specific labels. It uses exact key-value matching.*

**Use Case:**
- When you want to schedule resource-intensive workloads only on nodes with sufficient resources
- When you need exact label matching (key and value must match exactly)

**Limitations:**
- Only supports exact key-value matching
- Cannot specify multiple values (e.g., "Medium OR Large")
- Cannot use operators like `In`, `NotIn`, etc.

**Pod Definition with NodeSelector:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large
```

**How It Works:**
1. **Label the node**: `kubectl label nodes node-1 size=Large`
2. **Pod specifies nodeSelector**: `nodeSelector: size: Large`
3. **Scheduler matches**: Pod can only be scheduled on nodes with `size=Large` label
4. **If no matching node**: Pod remains in `Pending` state

**Example Workflow:**

```bash
# Step 1: Label nodes
kubectl label nodes node-1 size=Large
kubectl label nodes node-2 size=Small

# Step 2: Create pod with nodeSelector
kubectl create -f pod-definition.yaml
# Pod will only be scheduled on node-1 (has size=Large label)
```

**Key Points:**
- Simple and straightforward for exact label matching
- Limited capability - cannot handle "OR" conditions
- For more complex selection, use Node Affinity

### Node Affinity

*Node Affinity provides more expressive and flexible ways to select nodes compared to NodeSelector. It supports operators, multiple values, and both hard and soft requirements.*

**Why Node Affinity?**

NodeSelector limitations:
- Cannot schedule pods on "Medium OR Large" nodes (only exact match)
- Cannot use operators like `NotIn`, `Exists`, etc.
- No support for soft preferences (required vs preferred)

Node Affinity solves these limitations.

#### Node Affinity Types

*Node Affinity has two types that determine what happens if no matching node is found and what happens if node labels change after scheduling:*

| Type | Behavior When No Match Found | Behavior If Labels Change After Scheduling |
|------|------------------------------|--------------------------------------------|
| **requiredDuringSchedulingIgnoredDuringExecution** | Pod **cannot be scheduled** (stays Pending) | Pod **continues running** (ignored) |
| **preferredDuringSchedulingIgnoredDuringExecution** | Pod can be scheduled on any node (soft preference) | Pod **continues running** (ignored) |

**Understanding the Names:**
- **requiredDuringScheduling**: Must satisfy rule to schedule pod (hard requirement)
- **preferredDuringScheduling**: Try to satisfy rule, but not mandatory (soft preference)
- **IgnoredDuringExecution**: After pod is scheduled, if node labels change and no longer match, the pod keeps running (not evicted)

**Example - Required (Hard Requirement):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

**Example - Preferred (Soft Preference):**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

#### Operators

*Node Affinity supports various operators for flexible node selection:*

| Operator | Description | Example |
|----------|-------------|---------|
| **In** | Node label value must be in the provided list | `values: [Large, Medium]` - matches Large OR Medium |
| **NotIn** | Node label value must NOT be in the provided list | `values: [Small]` - excludes Small nodes |
| **Exists** | Node must have the label key (any value) | No `values` field needed |
| **DoesNotExist** | Node must NOT have the label key | No `values` field needed |
| **Gt** | Node label value must be greater than specified (numeric) | `values: ["3"]` - value > 3 |
| **Lt** | Node label value must be less than specified (numeric) | `values: ["5"]` - value < 5 |

**Example - Using In Operator (Multiple Values):**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: In
          values:
          - Large
          - Medium
```

*This pod can be scheduled on nodes with `size=Large` OR `size=Medium`.*

**Example - Using NotIn Operator (Anti-Affinity):**

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: size
          operator: NotIn
          values:
          - Small
```

*This pod will NOT be scheduled on nodes with `size=Small`.*

**What Happens If No Matching Node is Found?**

- **requiredDuringSchedulingIgnoredDuringExecution**: Pod remains in `Pending` state until a matching node becomes available
- **preferredDuringSchedulingIgnoredDuringExecution**: Pod can be scheduled on any available node (preference ignored)

**What Happens If Node Labels Change After Pod is Scheduled?**

- Both types: Pod **continues running** (the "IgnoredDuringExecution" part)
- The scheduler does NOT evict the pod
- Labels are only checked during scheduling, not during execution

**Key Takeaways:**
- Node Affinity provides more flexibility than NodeSelector
- Use `required` for hard requirements, `preferred` for soft preferences
- `IgnoredDuringExecution` means pods won't be evicted if labels change
- Operators like `In`, `NotIn` allow complex selection logic
- Use `In` operator to schedule on "Medium OR Large" nodes

---

## Configuration Management

### Environment Variables

*Environment variables in Kubernetes Pods allow you to pass configuration data to containers. The `env` field is an array that contains environment variable definitions, each with a `name` and a `value` (or `valueFrom` for external sources).*

**Docker vs Kubernetes:**

| Docker Command | Kubernetes Pod Field |
|---------------|---------------------|
| `docker run -e APP_COLOR=pink simple-webapp-color` | `env:` array in Pod spec |

**There are 3 ways to specify environment variables in Kubernetes:**

#### 1. Plain Key-Value Format

*The simplest way - directly specify the environment variable name and value in the Pod definition.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      value: pink
```

**Key points:**
- `env` is an array field (notice the `-` before `name`)
- Each entry has `name` and `value` fields
- The value is directly embedded in the Pod definition
- Simple and straightforward for static values

#### 2. Using ConfigMaps

*Instead of hardcoding the value, reference it from a ConfigMap. Use `valueFrom` with `configMapKeyRef`.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
```

**Key points:**
- Use `valueFrom` instead of `value`
- `configMapKeyRef` references a ConfigMap
- `name` is the ConfigMap name
- `key` is the key in the ConfigMap to retrieve the value from
- Useful for configuration that might change or be shared across multiple pods

#### 3. Using Secrets

*For sensitive data like passwords or API keys, use `valueFrom` with `secretKeyRef` to reference a Secret.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    env:
    - name: APP_COLOR
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: APP_COLOR
```

**Key points:**
- Use `valueFrom` with `secretKeyRef` (instead of `configMapKeyRef`)
- `name` is the Secret name
- `key` is the key in the Secret to retrieve the value from
- Used for sensitive information like passwords, tokens, API keys
- Values are base64 encoded in Secrets (though Kubernetes handles encoding/decoding automatically)

**Summary of the three methods:**

| Method | Field Used | Use Case |
|--------|-----------|----------|
| **Plain Key-Value** | `value: "pink"` | Static values that don't change |
| **ConfigMap** | `valueFrom.configMapKeyRef` | Configuration data that might change or be shared |
| **Secret** | `valueFrom.secretKeyRef` | Sensitive data like passwords, API keys |

**Important Notes:**
- `env` is an **array field** - each environment variable entry starts with `-`
- Each entry has a `name` field (the environment variable name)
- Use `value` for direct values, or `valueFrom` for referencing external sources
- You can mix different methods in the same `env` array (some with `value`, some with `valueFrom`)

### ConfigMaps

*ConfigMaps are used to handle configuration data separately from Pod definitions. This allows you to manage configuration data independently, making it easier to update and share configuration across multiple pods.*

**Why use ConfigMaps:**
- Separates configuration from application code
- Makes it easy to update configuration without rebuilding images
- Allows sharing configuration across multiple pods
- Provides a clean way to manage configuration data

**ConfigMaps can be created in 3 ways:**

### Creating ConfigMaps (Imperative Methods)

#### Method 1: Using --from-literal

*Create a ConfigMap directly from key-value pairs specified in the command line.*

**Basic syntax:**
```bash
k create configmap <config-name> --from-literal=key=value
```

**Example - Single key-value:**
```bash
k create configmap app-config --from-literal=APP_COLOR=blue
```

**Example - Multiple key-value pairs:**
```bash
k create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
```

**Key points:**
- Use `--from-literal` flag to specify key-value pairs directly
- Can specify multiple key-value pairs by repeating `--from-literal`
- Simple for small amounts of configuration data
- Gets messy when configuration grows (too many `--from-literal` flags)

#### Method 2: Using --from-file

*When configuration grows, it becomes messy to manage with multiple `--from-literal` flags. Use `--from-file` to load configuration from a file instead.*

**Basic syntax:**
```bash
k create configmap <config-name> --from-file=<path-to-file>
```

**Example:**
```bash
k create configmap app-config --from-file=/path/to/app.properties
```

**Key points:**
- Use `--from-file` flag to load configuration from a file
- Better for larger configurations
- Keeps configuration data in version-controlled files
- The file name becomes the key in the ConfigMap (unless you specify a custom key)
- You can specify multiple files: `--from-file=file1 --from-file=file2`

### Creating ConfigMaps (Declarative Method - YAML)

*Instead of using `spec:`, ConfigMaps use the `data:` field to define key-value pairs.*

**config-map.yaml:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

**Create the ConfigMap from YAML:**
```bash
kubectl create -f config-map.yaml
```

**Key points:**
- Use `data:` field instead of `spec:` in ConfigMaps
- Define key-value pairs directly in the YAML file
- Declarative approach - version controlled and repeatable
- Same result as imperative methods, but defined in code

### Viewing ConfigMaps

**List all ConfigMaps:**
```bash
k get configmaps
# or shorthand
k get cm
```

**Get detailed information about a ConfigMap:**
```bash
k describe configmap <config-name>
# or
k describe cm <config-name>
```

*This shows the ConfigMap's data, including all key-value pairs.*

### Injecting ConfigMaps into Pods

**There are 3 different ways to inject ConfigMaps into Pods:**

#### Method 1: Using envFrom (Inject All Key-Value Pairs)

*Inject all key-value pairs from a ConfigMap as environment variables. This is useful when you want all configuration values from a ConfigMap to be available as environment variables.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    envFrom:
    - configMapRef:
        name: app-config
```

**Key points:**
- `envFrom` is an array that can contain multiple ConfigMap references
- All key-value pairs in the ConfigMap become environment variables
- Each key becomes an environment variable name
- Each value becomes the environment variable value
- Useful when you want to inject entire ConfigMaps

#### Method 2: Using Single Environment Variable with valueFrom

*Inject a specific key's value from a ConfigMap as a single environment variable. This gives you granular control over which ConfigMap keys to use.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config    # ConfigMap name
          key: APP_COLOR      # Specific key in the ConfigMap
```

**Key points:**
- Use `env` array with `valueFrom.configMapKeyRef`
- `name` in `env` is the environment variable name in the container
- `configMapKeyRef.name` is the ConfigMap name
- `configMapKeyRef.key` is the specific key to retrieve from the ConfigMap
- You can use multiple `env` entries to inject multiple specific keys

#### Method 3: Using Volumes

*Mount a ConfigMap as a volume. Each key in the ConfigMap becomes a file, and the value becomes the file's content. Useful when applications expect configuration files rather than environment variables.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: app-config-volume
      mountPath: /etc/config
  volumes:
  - name: app-config-volume
    configMap:
      name: app-config
```

**Key points:**
- Define a volume using `volumes` array with `configMap` source
- Mount the volume using `volumeMounts` in the container spec
- Each key in the ConfigMap becomes a file in the mounted directory
- Each value becomes the content of that file
- Useful for applications that read configuration from files
- Mount path (e.g., `/etc/config`) is where the files will appear in the container

**Summary of the three methods:**

| Method | Field | Use Case |
|--------|-------|----------|
| **envFrom** | `envFrom.configMapRef` | Inject all key-value pairs as environment variables |
| **Single ENV** | `env[].valueFrom.configMapKeyRef` | Inject specific keys as individual environment variables |
| **Volume** | `volumes[].configMap` + `volumeMounts` | Mount ConfigMap as files in the container |

**When to use each:**
- **envFrom**: When you want all ConfigMap values as environment variables
- **Single ENV (valueFrom)**: When you need selective injection or custom environment variable names
- **Volume**: When your application expects configuration files rather than environment variables

### Secrets

*Secrets are used to store sensitive data like passwords, API keys, tokens, and certificates. Unlike ConfigMaps which store plain text, Secrets store base64-encoded data (though Kubernetes handles encoding/decoding automatically).*

**Why use Secrets:**
- Store sensitive information securely
- Separate sensitive data from application code
- Share secrets across multiple pods
- Base64 encoding provides basic obfuscation (not encryption - for production, use external secret management)

**Encoding Secrets with base64:**

*Before storing secrets in a Secret manifest, you need to encode them using base64:*

```bash
echo -n "password123" | base64
# Output: cGFzc3dvcmQxMjM=
```

**Key points:**
- Use `echo -n` to avoid adding a newline character
- The encoded value is what you store in the Secret manifest
- Kubernetes automatically decodes the value when injecting it into pods
- Base64 encoding is NOT encryption - it's just encoding (obfuscation)

**Example Secret Manifest:**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=  # base64 encoded "password123"
  API_KEY: bXlhcGlrZXk=           # base64 encoded "myapikey"
```

**Create the Secret from YAML:**
```bash
kubectl create -f secret.yaml
```

**Or create imperatively:**
```bash
kubectl create secret generic app-secret --from-literal=DB_PASSWORD=password123
```

### Viewing Secrets

**List all Secrets:**
```bash
k get secrets
# or shorthand
k get secret
```

**Get detailed information about a Secret:**
```bash
k describe secret <secret-name>
```

*Note: The actual values are not shown in `kubectl describe` for security reasons. Use `kubectl get secret <name> -o yaml` to see the base64-encoded values.*

### Injecting Secrets into Pods

**Just like ConfigMaps, there are 3 different ways to inject Secrets into Pods:**

#### Method 1: Using envFrom (Inject All Key-Value Pairs)

*Inject all key-value pairs from a Secret as environment variables. This is useful when you want all secret values from a Secret to be available as environment variables.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: myapp:latest
    ports:
    - containerPort: 8080
    envFrom:
    - secretRef:
        name: app-secret
```

**Key points:**
- `envFrom` is an array that can contain multiple Secret references
- Use `secretRef` (instead of `configMapRef` for ConfigMaps)
- `name` is the Secret name
- All key-value pairs in the Secret become environment variables
- Each key becomes an environment variable name
- Each value becomes the environment variable value (automatically decoded from base64)
- Useful when you want to inject entire Secrets

#### Method 2: Using Single Environment Variable with valueFrom

*Inject a specific key's value from a Secret as a single environment variable. This gives you granular control over which Secret keys to use.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: myapp:latest
    ports:
    - containerPort: 8080
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret    # Secret name
          key: DB_PASSWORD   # Specific key in the Secret
```

**Key points:**
- Use `env` array with `valueFrom.secretKeyRef`
- `name` in `env` is the environment variable name in the container
- `secretKeyRef.name` is the Secret name
- `secretKeyRef.key` is the specific key to retrieve from the Secret
- The value is automatically decoded from base64 when injected
- You can use multiple `env` entries to inject multiple specific keys

#### Method 3: Using Volumes

*Mount a Secret as a volume. Each key in the Secret becomes a file, and the value becomes the file's content. Useful when applications expect secret files (like certificates or keys) rather than environment variables.*

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: myapp:latest
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: app-secret-volume
      mountPath: /etc/secrets
  volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

**Key points:**
- Define a volume using `volumes` array with `secret` source
- Use `secret.secretName` to specify the Secret name
- Mount the volume using `volumeMounts` in the container spec
- Each key in the Secret becomes a file in the mounted directory
- Each value becomes the content of that file (automatically decoded from base64)
- Useful for applications that read secrets from files (e.g., SSL certificates, private keys)
- Mount path (e.g., `/etc/secrets`) is where the files will appear in the container

**Summary of the three methods:**

| Method | Field | Use Case |
|--------|-------|----------|
| **envFrom** | `envFrom.secretRef` | Inject all key-value pairs as environment variables |
| **Single ENV** | `env[].valueFrom.secretKeyRef` | Inject specific keys as individual environment variables |
| **Volume** | `volumes[].secret` + `volumeMounts` | Mount Secret as files in the container |

**When to use each:**
- **envFrom**: When you want all Secret values as environment variables
- **Single ENV (valueFrom)**: When you need selective injection or custom environment variable names
- **Volume**: When your application expects secret files (certificates, keys) rather than environment variables

**Important Security Notes:**
- Secrets are base64-encoded, NOT encrypted
- Anyone with access to the Secret can decode the values
- For production environments, consider using external secret management solutions
- Never commit Secret manifests with real credentials to version control
- Use RBAC to restrict access to Secrets

#### CSI Drivers to Manage Secrets

##### Why Native Kubernetes Secrets Are Insufficient for Production

*Native Kubernetes Secrets have significant limitations for production use:*
- **Base64 encoding is NOT encryption** - it's just obfuscation, not real encryption
- Secrets are stored in etcd (Kubernetes' database) without encryption by default
- Anyone with cluster access can easily decode and view secret values
- No automatic rotation or versioning capabilities
- No integration with enterprise secret management systems

*For production environments, you need proper encryption and integration with external secret management systems.*

##### External Secret Management Solutions

*Several tools integrate Kubernetes with external secret management systems:*
- **Secret Store CSI Driver** (Preferred method)
- **External Secrets Operator (ESO)**
- **Sealed Secrets**
- **HashiCorp Vault**
- **AWS Secrets Manager integration**
- And other external secret management solutions

###### Secret Store CSI Driver (Preferred Method)

*The Secret Store CSI Driver is the preferred solution for managing secrets in production. It provides a Container Storage Interface (CSI) driver that synchronizes secrets from external secret management stores directly into Kubernetes pods via volume mounts.*

**Key Advantages:**
- **No Kubernetes Secret Objects**: Secrets never touch Kubernetes etcd - they're synced directly from external stores
- **Volume Mount Integration**: Uses standard Kubernetes volume mounts, making it transparent to applications
- **Multiple Providers**: Supports AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, Google Secret Manager, and more
- **Automatic Sync**: Secrets are automatically synchronized from external stores when mounted
- **Better Security**: Secrets are encrypted at rest in the external secret management system

**How Secret Store CSI Driver Works - Architecture Flow:**

1. **User/Helm Deployment**: A user deploys a Custom Resource Definition (CRD) called `SecretProviderClass` using Helm or kubectl
2. **SecretProviderClass CRD**: Defines which external secrets to fetch and how to access them (from AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, etc.)
3. **Kubelet**: The Kubelet on the node recognizes the need for secrets defined in the SecretProviderClass
4. **CSI Driver**: The Container Storage Interface (CSI) Driver intercepts the volume mount request
5. **Secret Provider**: The Secret Provider component communicates with the external secret management store (e.g., AWS Secrets Manager, HashiCorp Vault)
6. **Secret Retrieval**: Secrets are retrieved from the external store (e.g., `DB_PASSWORD: mypass`, `DB_USERNAME: myuser`)
7. **Volume Mount**: The CSI Driver mounts the retrieved secrets as files in the pod via a volume mount
8. **Pod Access**: The pod can access secrets from the mounted volume at the specified mount path

**How Pods Use Secrets:**
- The pod defines a volume mount that references a `SecretProviderClass`
- The CSI Driver retrieves secrets from the external secret management store
- Secrets are mounted as files in the pod at the specified mount path (e.g., `/mnt/secrets-store`)
- The pod can read secrets from these mounted files
- Secrets never appear in Kubernetes Secret objects or etcd

**Supported External Secret Stores:**
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager
- IBM Cloud Secrets Manager
- And more...

######### External Secrets Operator (ESO)

*External Secrets Operator is another approach that syncs secrets from external stores into Kubernetes Secret objects. Unlike CSI Driver which mounts directly, ESO creates Kubernetes Secret resources that can then be used by pods.*

**How ESO Works:**
1. ESO watches for `ExternalSecret` custom resources
2. Retrieves secrets from external stores (AWS Secrets Manager, HashiCorp Vault, etc.)
3. Creates or updates Kubernetes Secret objects
4. Pods can then use these Secrets normally (via env vars, volumes, etc.)

**Key Difference from CSI Driver:**
- **CSI Driver**: Mounts secrets directly as files, no Kubernetes Secret objects created
- **ESO**: Creates Kubernetes Secret objects from external stores, which pods then use with standard Secret injection methods

**Comparison: CSI Driver vs External Secrets Operator**

| Feature | Secret Store CSI Driver | External Secrets Operator |
|---------|------------------------|--------------------------|
| **Storage** | Secrets never in etcd | Creates Kubernetes Secret objects in etcd |
| **Mount Method** | Volume mounts directly | Standard Secret injection methods (env vars, volumes) |
| **Sync Direction** | Direct from external store → Pod | External store → Secret → Pod |
| **Use Case** | When you want secrets completely out of Kubernetes | When you need standard Secret objects |
| **Security** | Higher - secrets never stored in Kubernetes | Lower - secrets stored as Secret objects |

### Best Practices for Production Secret Management

1. **Use External Secret Management**: Never store sensitive data directly in Kubernetes Secrets
2. **Enable Encryption at Rest**: Ensure etcd encryption is enabled (for any Secrets that do exist)
3. **RBAC**: Implement strict RBAC policies to limit access to Secrets
4. **Secret Rotation**: Use external stores that support automatic secret rotation
5. **Audit Logging**: Enable audit logging for secret access
6. **Network Policies**: Use network policies to restrict pod-to-pod communication
7. **Prefer CSI Driver**: Use Secret Store CSI Driver for production as it keeps secrets out of etcd entirely

##### Summary

*For production environments, native Kubernetes Secrets are insufficient. Use external secret management solutions like Secret Store CSI Driver or External Secrets Operator to:*
- Keep secrets encrypted in external secret management systems
- Avoid storing secrets in Kubernetes etcd (or minimize exposure if using ESO)
- Enable automatic rotation and versioning
- Integrate with enterprise secret management systems
- Maintain better security and compliance
- Meet regulatory requirements for secret management