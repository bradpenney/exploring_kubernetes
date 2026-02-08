# Namespaces: Your Workspace in Kubernetes

!!! tip "Part of Level 1: Core Primitives"
    This article is part of [Level 1: Core Primitives](overview.md).

## Why You Need to Know About Namespaces

**Your actual problem:** Your platform team gave you kubectl access and said "you have access to the dev-yourteam namespace." Every kubectl command you run asks "which namespace?" You need to understand what namespaces are and why they matter.

**What you'll encounter:**
- Commands failing with "not found" (you're in the wrong namespace)
- `-n dev` or `--namespace=dev` flags everywhere
- Being told "you don't have access to that namespace" (prod, other teams)
- Resources that exist but `kubectl get` doesn't show them (different namespace)

**You need to know:** Namespaces are isolation boundaries. They're like folders for your Kubernetes resources. Your team has its own namespace—you can do whatever you want there. Other namespaces are off-limits.

**Why this matters for developers:**
- You deploy to YOUR namespace (not someone else's)
- Commands default to your current namespace (set it once, forget it)
- Production is in a different namespace (you probably can't see it—that's intentional)
- Accidentally deploying to the wrong namespace causes confusion (or worse)

---

## What Are Namespaces, Really?

You've been deploying resources to "your namespace." But what is a namespace, and why do you need one?

**Think of namespaces as folders for your Kubernetes resources.** Just like folders organize files on your computer, namespaces organize pods, services, and deployments in your cluster.

**Simple analogy:**
- Cluster = Computer
- Namespace = Folder
- Resources (pods, services) = Files

Multiple teams can have a file called `web-app`, as long as they're in different folders (namespaces).

---

## Why Namespaces Exist

### The Problem: Name Collisions

Without namespaces, if you name your deployment `web-app` and another team also names theirs `web-app`, one would overwrite the other.

**With namespaces:**
- Your team: `frontend` namespace, `web-app` deployment
- Other team: `backend` namespace, `web-app` deployment
- No conflict!

### Use Cases

**Multi-tenancy:**
- Different teams share one cluster
- Each team gets their own namespace
- Teams can't interfere with each other

**Environments:**
- `dev`, `staging`, `prod` namespaces in one cluster
- Same deployment names, different namespaces

**Resource Quotas:**
- Limit CPU/memory per namespace
- Prevent one team from consuming all resources

**Access Control:**
- RBAC policies per namespace
- Developers access `dev`, not `prod`

---

## Default Namespaces

Every cluster starts with these namespaces:

| Namespace | Purpose |
|-----------|---------|
| `default` | Where resources go if you don't specify |
| `kube-system` | Kubernetes system components |
| `kube-public` | Publicly accessible resources |
| `kube-node-lease` | Node heartbeats (ignore this) |

``` bash title="List Namespaces"
kubectl get namespaces
kubectl get ns  # Short form

# NAME              STATUS   AGE
# default           Active   30d
# kube-system       Active   30d
# kube-public       Active   30d
# kube-node-lease   Active   30d
```

!!! warning "Never Touch kube-system"
    The `kube-system` namespace contains cluster infrastructure. Deleting resources here can break your entire cluster. Your platform team manages this.

---

## Creating Namespaces

=== "Imperative"

    ``` bash title="Create Namespace"
    kubectl create namespace dev
    # namespace/dev created

    kubectl create namespace staging
    kubectl create namespace prod
    ```

=== "Declarative (YAML)"

    ``` yaml title="namespace.yaml" linenums="1"
    apiVersion: v1
    kind: Namespace
    metadata:
      name: dev
      labels:
        environment: development
        team: frontend
    ```

    ``` bash
    kubectl apply -f namespace.yaml
    ```

---

## Working with Namespaces

### View Resources in Namespace

``` bash title="Get Resources in Specific Namespace"
# Pods in 'dev' namespace
kubectl get pods -n dev
kubectl get pods --namespace=dev  # Long form

# All resources in 'staging'
kubectl get all -n staging

# Pods in ALL namespaces
kubectl get pods --all-namespaces
kubectl get pods -A  # Short form
```

### Set Default Namespace

Instead of typing `-n dev` every time:

``` bash title="Set Default Namespace for Current Context"
# Set default namespace
kubectl config set-context --current --namespace=dev

# Now all commands use 'dev' by default
kubectl get pods  # Looks in 'dev'
```

**Check your current namespace:**
``` bash
kubectl config view --minify | grep namespace:
```

---

## Creating Resources in Namespaces

### Method 1: Specify in YAML

``` yaml title="deployment-in-namespace.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: dev  # (1)!
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

1. Explicitly specify namespace

``` bash
kubectl apply -f deployment-in-namespace.yaml
# deployment.apps/web-app created (in 'dev' namespace)
```

### Method 2: Use -n Flag

``` bash title="Create in Specific Namespace"
kubectl apply -f deployment.yaml -n dev
```

### Method 3: Use Default Namespace

If you set default namespace, just apply:
``` bash
kubectl config set-context --current --namespace=dev
kubectl apply -f deployment.yaml  # Goes to 'dev'
```

---

## Cross-Namespace Communication

### Services Across Namespaces

Services can be accessed from other namespaces using fully qualified domain names (FQDN):

**Format:** `<service-name>.<namespace>.svc.cluster.local`

**Example:**

=== "Same Namespace"

    ``` yaml
    # In 'dev' namespace
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
      namespace: dev
    spec:
      containers:
      - name: app
        image: myapp
        env:
        - name: API_URL
          value: "http://backend-svc"  # (1)!
    ```

    1. Short name works within same namespace

=== "Different Namespace"

    ``` yaml
    # In 'dev' namespace, accessing service in 'staging'
    apiVersion: v1
    kind: Pod
    metadata:
      name: frontend
      namespace: dev
    spec:
      containers:
      - name: app
        image: myapp
        env:
        - name: API_URL
          value: "http://backend-svc.staging.svc.cluster.local"  # (1)!
    ```

    1. FQDN required for cross-namespace access

!!! tip "Namespace Security"
    By default, pods can access services in any namespace. Use **Network Policies** (Level 3) to restrict this.

---

## Resource Quotas

Limit resources per namespace to prevent one team from consuming everything:

``` yaml title="resource-quota.yaml" linenums="1"
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "10"  # (1)!
    requests.memory: "20Gi"
    limits.cpu: "20"
    limits.memory: "40Gi"
    pods: "50"  # (2)!
    services: "10"
    persistentvolumeclaims: "5"
```

1. Total CPU/memory requests allowed in namespace
2. Maximum number of pods

``` bash title="View Quota Usage"
kubectl get resourcequota -n dev
kubectl describe resourcequota dev-quota -n dev
```

---

## LimitRanges

Set default and max resources for pods:

``` yaml title="limit-range.yaml" linenums="1"
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev
spec:
  limits:
  - max:  # (1)!
      cpu: "2"
      memory: "4Gi"
    min:  # (2)!
      cpu: "100m"
      memory: "128Mi"
    default:  # (3)!
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:  # (4)!
      cpu: "250m"
      memory: "256Mi"
    type: Container
```

1. Maximum resources per container
2. Minimum resources per container
3. Default limits if not specified
4. Default requests if not specified

---

## Namespace Lifecycle

### Deleting Namespaces

!!! danger "Namespace Deletion is Destructive"
    Deleting a namespace deletes **ALL resources in it**. Pods, services, deployments—everything gone.

``` bash title="Delete Namespace (DANGER)"
# This deletes EVERYTHING in 'dev' namespace
kubectl delete namespace dev

# It will hang while terminating all resources
# Can take 30-60 seconds
```

**Stuck namespace?**
Sometimes namespaces get stuck in "Terminating" state. This usually means resources with finalizers are blocking deletion. Contact your platform team.

---

## Common Patterns

### Team-Based Namespaces

```
team-frontend-dev
team-frontend-staging
team-frontend-prod

team-backend-dev
team-backend-staging
team-backend-prod
```

### Environment-Based Namespaces

```
dev
staging
prod
```

### Feature-Based Namespaces (for testing)

```
feature-new-ui
feature-api-v2
feature-payment-gateway
```

---

## Working with Multiple Namespaces

``` bash title="Essential Namespace Commands"
# List all namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace my-namespace

# Delete namespace (DANGER)
kubectl delete namespace my-namespace

# View resources across all namespaces
kubectl get pods -A
kubectl get services -A
kubectl get deployments -A

# Set default namespace
kubectl config set-context --current --namespace=dev

# View current namespace
kubectl config view --minify | grep namespace

# Describe namespace (shows quotas, limits)
kubectl describe namespace dev
```

---

## Practice Exercises

??? question "Exercise 1: Multi-Environment Setup"
    Create three namespaces (dev, staging, prod) and deploy the same application to each.

    ??? tip "Solution"
        ``` bash
        # Create namespaces
        kubectl create namespace dev
        kubectl create namespace staging
        kubectl create namespace prod

        # Deploy to each
        kubectl apply -f deployment.yaml -n dev
        kubectl apply -f deployment.yaml -n staging
        kubectl apply -f deployment.yaml -n prod

        # Verify
        kubectl get deployments -A | grep web-app
        ```

??? question "Exercise 2: Cross-Namespace Communication"
    Deploy a service in `dev` namespace and access it from a pod in `test` namespace.

    ??? tip "Solution"
        ``` bash
        # Create namespaces
        kubectl create namespace dev
        kubectl create namespace test

        # Deploy service in dev
        kubectl apply -f service.yaml -n dev

        # Create test pod in test namespace
        kubectl run test-pod --image=busybox -n test -it --rm -- sh

        # Inside pod, access service:
        wget -O- http://my-service.dev.svc.cluster.local
        ```

---

## Quick Recap

| Concept | Purpose |
|---------|---------|
| **Namespace** | Logical resource grouping |
| **default** | Default namespace if none specified |
| **kube-system** | Cluster system components (don't touch!) |
| **Resource Quota** | Limit resources per namespace |
| **LimitRange** | Set default/max resource limits |
| **FQDN** | `service.namespace.svc.cluster.local` for cross-namespace access |

---

## Further Reading

### Official Documentation
- [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/)
- [Limit Ranges](https://kubernetes.io/docs/concepts/policy/limit-range/)

### Deep Dives
- [Namespace Best Practices](https://kubernetes.io/docs/tasks/administer-cluster/namespaces/)
- [Multi-Tenancy](https://kubernetes.io/blog/2021/04/15/three-tenancy-models-for-kubernetes/)

### Related Articles
- [Pods](pods.md) - Pods live in namespaces
- [Services](services.md) - Cross-namespace service access
- **RBAC** - Namespace-scoped permissions (coming in Level 5)
- **Resource Requests and Limits** - Resource management (coming in Level 5)

---

## What's Next?

You understand namespaces and resource organization. Next, learn **[Labels and Selectors](labels_selectors.md)** to organize and target resources within namespaces.

---

**Remember:** Namespaces provide isolation but not security by default. Use RBAC and Network Policies for true security boundaries.
