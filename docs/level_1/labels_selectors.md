# Labels and Selectors: How Kubernetes Connects Resources

!!! tip "Part of Level 1: Core Primitives"
    This is the final article in [Level 1: Core Primitives](overview.md).

## Why You Need to Know About Labels and Selectors

**Your actual problem:** You have 50 pods running in your namespace. You need to find just YOUR app's pods. Or you need to understand how a Service knows which pods to send traffic to. Or why a Deployment manages some pods but not others.

**What you'll encounter:**
- YAML with `labels:` and `selector:` sections (what's the difference?)
- Commands like `kubectl get pods -l app=myapp` (what's `-l` doing?)
- A Service that doesn't route traffic to your pods (selector mismatch!)
- Documentation saying "services use selectors to find pods" (how does that work?)

**You need to know:** Labels are tags you put on resources (like hashtags). Selectors are queries that find resources by their labels. This is how Kubernetes connects things—Services find Pods, Deployments manage Pods, you query resources.

**Why this matters for developers:**
- **Services route traffic** by matching labels (selector must match pod labels)
- **Deployments manage pods** by matching labels (it only controls pods with matching labels)
- **You debug issues** by querying labels (`kubectl get pods -l app=myapp`)
- **You organize resources** with meaningful labels (environment, version, team)

---

## What Are Labels and Selectors?

You've created pods, services, and deployments. But how does a Service know which Pods to route traffic to? How does a Deployment know which Pods it manages?

**Answer: Labels and Selectors.**

**Labels** are key-value tags you attach to resources (like `app: web` or `environment: dev`).

**Selectors** are queries that find resources by their labels (like "find all pods with `app: web`").

They're the glue that connects everything in Kubernetes. Services use selectors to find pods. Deployments use selectors to manage pods. You use selectors to filter resources.

---

## What Are Labels?

Labels are arbitrary key-value pairs you attach to resources:

``` yaml
metadata:
  labels:
    app: web
    environment: production
    tier: frontend
    version: v1.2.0
```

**Rules:**
- Keys and values are strings
- Keys can have optional prefix: `example.com/app: web`
- Maximum 63 characters for values
- Must start/end with alphanumeric character

**Think of labels like tags or hashtags.** They help you organize and find resources.

---

## Why Labels Matter

### 1. Services Find Pods

``` yaml
# Service
spec:
  selector:
    app: web  # (1)!

# Pod
metadata:
  labels:
    app: web  # (2)!
```

1. Service looks for this label
2. Pod has matching label → receives traffic

### 2. Deployments Manage Pods

``` yaml
# Deployment
spec:
  selector:
    matchLabels:
      app: web  # (1)!
  template:
    metadata:
      labels:
        app: web  # (2)!
```

1. Deployment manages pods with this label
2. Pods created have this label → managed by Deployment

### 3. You Query Resources

``` bash
# Get all web pods
kubectl get pods -l app=web

# Get production resources
kubectl get all -l environment=production
```

---

## Adding Labels

### At Creation (YAML)

``` yaml title="pod-with-labels.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
  labels:  # (1)!
    app: web
    tier: frontend
    environment: production
    version: "1.2.0"
    owner: team-frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

1. Labels in metadata section

### Imperatively (kubectl)

``` bash title="Add Labels to Existing Resources"
# Add single label
kubectl label pod web-pod tier=frontend

# Add multiple labels
kubectl label pod web-pod env=prod version=1.2.0

# Overwrite existing label
kubectl label pod web-pod env=staging --overwrite

# Remove label
kubectl label pod web-pod version-
```

---

## Querying with Selectors

### Equality-Based Selectors

``` bash title="Equality Selectors"
# Exact match
kubectl get pods -l app=web

# Not equal
kubectl get pods -l app!=web

# Multiple conditions (AND)
kubectl get pods -l app=web,environment=production

# Show labels in output
kubectl get pods -l app=web --show-labels
```

### Set-Based Selectors

``` bash title="Set-Based Selectors"
# app is web OR database
kubectl get pods -l 'app in (web,database)'

# environment is NOT production
kubectl get pods -l 'environment notin (production)'

# Has 'tier' label (any value)
kubectl get pods -l tier

# Does not have 'tier' label
kubectl get pods -l '!tier'
```

---

## Label Best Practices

### Recommended Labels

Kubernetes recommends these standard labels:

``` yaml
metadata:
  labels:
    app.kubernetes.io/name: mysql  # (1)!
    app.kubernetes.io/instance: mysql-abcxzy  # (2)!
    app.kubernetes.io/version: "5.7.21"  # (3)!
    app.kubernetes.io/component: database  # (4)!
    app.kubernetes.io/part-of: wordpress  # (5)!
    app.kubernetes.io/managed-by: helm  # (6)!
```

1. Application name
2. Unique instance identifier
3. Application version
4. Component within architecture
5. Higher-level application this is part of
6. Tool managing this resource

### Organization Patterns

**Environment-based:**
```yaml
environment: production
environment: staging
environment: development
```

**Tier-based:**
```yaml
tier: frontend
tier: backend
tier: database
tier: cache
```

**Team ownership:**
```yaml
owner: team-frontend
owner: team-backend
owner: team-platform
```

**Version tracking:**
```yaml
version: v1.2.0
version: v1.2.1
version: v2.0.0
```

---

## Selectors in Resources

### Service Selector

``` yaml title="service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:  # (1)!
    app: web
    tier: frontend
  ports:
  - port: 80
    targetPort: 8080
```

1. Matches pods with BOTH labels

**Matches:**
- Pod with `app: web` AND `tier: frontend` ✅
- Pod with `app: web` AND `tier: frontend` AND `version: 1.0` ✅

**Does not match:**
- Pod with only `app: web` ❌
- Pod with `app: api` ❌

### Deployment Selector

``` yaml title="deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:  # (1)!
      app: web
  template:
    metadata:
      labels:  # (2)!
        app: web
        version: "1.2.0"
    spec:
      containers:
      - name: web
        image: nginx:1.21
```

1. Deployment manages pods with `app: web`
2. Pods get these labels (must include `app: web`)

!!! warning "Selector Immutability"
    Deployment selectors are **immutable**. You can't change `selector.matchLabels` after creation. You must delete and recreate the Deployment.

### Advanced Selectors (matchExpressions)

``` yaml title="deployment-advanced-selector.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchExpressions:  # (1)!
    - key: app
      operator: In  # (2)!
      values:
      - web
      - api
    - key: environment
      operator: NotIn
      values:
      - production
    - key: tier
      operator: Exists  # (3)!
```

1. More flexible than matchLabels
2. Operators: In, NotIn, Exists, DoesNotExist
3. Checks if label exists (any value)

---

## Practical Examples

### Blue-Green Deployments

``` yaml
# Blue version (current)
metadata:
  labels:
    app: web
    version: blue

# Service points to blue
spec:
  selector:
    app: web
    version: blue
```

**Deploy green, then switch:**
``` bash
# Deploy green version
kubectl apply -f deployment-green.yaml

# Test green works
kubectl port-forward deployment/web-green 8080:80

# Switch traffic to green
kubectl patch service web-svc -p '{"spec":{"selector":{"version":"green"}}}'

# Roll back if needed
kubectl patch service web-svc -p '{"spec":{"selector":{"version":"blue"}}}'
```

### Canary Deployments

``` yaml
# Stable version (90%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-stable
spec:
  replicas: 9
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        version: stable

---
# Canary version (10%)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-canary
spec:
  replicas: 1  # (1)!
  selector:
    matchLabels:
      app: web
      version: canary
  template:
    metadata:
      labels:
        app: web
        version: canary

---
# Service routes to both
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  selector:
    app: web  # (2)!
  ports:
  - port: 80
```

1. 1 out of 10 pods = 10% traffic
2. Matches both stable and canary (both have `app: web`)

---

## Common Pitfalls

❌ **Selector doesn't match pod labels:**
``` yaml
# Service
selector:
  app: web

# Pod
labels:
  app: api  # ❌ No match!
```

❌ **Too many labels in selector:**
``` yaml
# Service
selector:
  app: web
  tier: frontend
  version: "1.0"

# Pod (missing version)
labels:
  app: web
  tier: frontend  # ❌ Missing version label!
```

❌ **Typos in label keys/values:**
``` yaml
# Service
selector:
  app: web  # (lowercase 'w')

# Pod
labels:
  app: Web  # ❌ Capital 'W' doesn't match!
```

---

## Working with Labels

``` bash title="Essential Label Commands"
# Show labels on resources
kubectl get pods --show-labels
kubectl get deployments --show-labels

# Filter by label
kubectl get pods -l app=web
kubectl get pods -l 'environment in (production,staging)'

# Filter by multiple labels
kubectl get pods -l app=web,tier=frontend

# Add label
kubectl label pod my-pod environment=production

# Update label
kubectl label pod my-pod environment=staging --overwrite

# Remove label
kubectl label pod my-pod environment-

# Label multiple resources
kubectl label pods --all environment=production

# Show specific label columns
kubectl get pods -L app,tier,version
```

---

## Practice Exercises

??? question "Exercise 1: Organize Pods with Labels"
    Create 5 pods with different labels and query them:
    - 2 pods: `tier=frontend`, `environment=production`
    - 2 pods: `tier=backend`, `environment=production`
    - 1 pod: `tier=frontend`, `environment=staging`

    Then query:
    1. All frontend pods
    2. All production pods
    3. Frontend production pods only

    ??? tip "Solution"
        ``` bash
        # Create pods (abbreviated)
        kubectl run frontend-prod-1 --image=nginx --labels="tier=frontend,environment=production"
        kubectl run frontend-prod-2 --image=nginx --labels="tier=frontend,environment=production"
        kubectl run backend-prod-1 --image=nginx --labels="tier=backend,environment=production"
        kubectl run backend-prod-2 --image=nginx --labels="tier=backend,environment=production"
        kubectl run frontend-staging-1 --image=nginx --labels="tier=frontend,environment=staging"

        # Queries
        kubectl get pods -l tier=frontend
        kubectl get pods -l environment=production
        kubectl get pods -l tier=frontend,environment=production
        ```

??? question "Exercise 2: Fix Broken Service"
    A Service isn't routing traffic to pods. The selector is `app: web` but pods have label `app: website`. Fix it.

    ??? tip "Solution"
        **Option 1: Fix pod labels**
        ``` bash
        kubectl label pods -l app=website app=web --overwrite
        kubectl label pods -l app=website app-  # Remove old label
        ```

        **Option 2: Fix service selector**
        ``` bash
        kubectl patch service my-svc -p '{"spec":{"selector":{"app":"website"}}}'
        ```

---

## Quick Recap

| Concept | Purpose |
|---------|---------|
| **Labels** | Key-value pairs attached to resources |
| **Selectors** | Query resources by labels |
| **Equality-based** | `app=web`, `app!=web` |
| **Set-based** | `app in (web,api)`, `tier notin (cache)` |
| **matchLabels** | Simple AND selector |
| **matchExpressions** | Advanced selector with operators |

---

## Further Reading

### Official Documentation
- [Labels and Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)
- [Recommended Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/)
- [Field Selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/field-selectors/)

### Deep Dives
- [Label Best Practices](https://kubernetes.io/docs/concepts/configuration/overview/#naked-pods-vs-replicasets-deployments-and-jobs)
- [Blue-Green Deployments](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/)

### Related Articles
- [Pods](pods.md) - Where labels are applied
- [Services](services.md) - Use selectors to find pods
- **Deployments** - Use selectors to manage pods (coming in Level 2)

---

## What's Next?

**Congratulations!** You've completed Level 1: Core Primitives. You understand:
- Pods (atomic units)
- Services (networking)
- ConfigMaps/Secrets (configuration)
- Namespaces (organization)
- Labels/Selectors (connecting resources)

**Level 2: Workload Management** (coming soon) will teach you about Deployments, StatefulSets, DaemonSets, and managing applications at scale.

---

**Remember:** Labels are metadata. They don't affect how resources run, only how you organize and select them. Use them liberally!
