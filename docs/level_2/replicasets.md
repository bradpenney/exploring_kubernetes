# ReplicaSets Under the Hood

!!! tip "Part of Level 2: Workload Management"
    This article is part of [Level 2: Workload Management](overview.md). Read [Deployments Explained](deployments.md) first.

You've been creating Deployments, which automatically create ReplicaSets. But what exactly is a ReplicaSet, and why does it exist?

**Short answer:** ReplicaSets ensure the right number of Pods are running. That's it. One job, done well.

---

## What ReplicaSets Do

A ReplicaSet has one responsibility: **maintain a stable set of replica Pods**.

- You want 3 Pods running? ReplicaSet makes it happen.
- A Pod crashes? ReplicaSet creates a replacement.
- Too many Pods? ReplicaSet deletes extras.

**Think of it like a thermostat:** You set the desired temperature (replica count), and it automatically adjusts heating/cooling (creating/deleting pods) to maintain that temperature.

---

## ReplicaSet vs Deployment

| Resource | Purpose | When to Use |
|----------|---------|-------------|
| **ReplicaSet** | Maintains pod count | Almost never directly |
| **Deployment** | Manages ReplicaSets + updates | Always (in production) |

**You almost never create ReplicaSets directly.** Deployments create and manage them for you.

**Why the separation?**
- ReplicaSets handle scaling
- Deployments handle versioning and updates
- When you update a Deployment, it creates a NEW ReplicaSet for the new version while scaling down the old one

---

## ReplicaSet Anatomy

``` yaml title="replicaset.yaml" linenums="1"
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: web-rs
  labels:
    app: web
spec:
  replicas: 3  # (1)!
  selector:  # (2)!
    matchLabels:
      app: web
  template:  # (3)!
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

1. Desired number of pods
2. How to find pods to manage
3. Pod template to create

!!! warning "Selector Must Match Template Labels"
    The `selector` must match the labels in `template.metadata.labels`. If they don't match, ReplicaSet can't manage the pods it creates.

---

## How Deployments Use ReplicaSets

When you create a Deployment:

``` bash
kubectl create deployment web --image=nginx:1.21 --replicas=3
```

Kubernetes creates:
1. **Deployment** (`web`)
2. **ReplicaSet** (`web-7c5ddbdf54`)
3. **3 Pods** (`web-7c5ddbdf54-xxxxx`)

``` bash
kubectl get replicasets
# NAME              DESIRED   CURRENT   READY   AGE
# web-7c5ddbdf54    3         3         3       1m
```

**The ReplicaSet name** is deployment name + pod template hash.

---

## Rolling Updates: Two ReplicaSets

When you update a Deployment's image:

``` bash
kubectl set image deployment/web nginx=nginx:1.22
```

Deployment creates a NEW ReplicaSet:

``` bash
kubectl get replicasets
# NAME              DESIRED   CURRENT   READY   AGE
# web-7c5ddbdf54    0         0         0       10m  ← Old version (scaled to 0)
# web-5d9c8b9f7b    3         3         3       1m   ← New version
```

**Why keep the old ReplicaSet?**
For rollbacks! If new version fails:

``` bash
kubectl rollout undo deployment/web
# Scales up old ReplicaSet, scales down new one
```

---

## When You'd Use ReplicaSet Directly

**Almost never in production.** But here are the rare cases:

### 1. Simple Pod Replication (No Updates)

If you NEVER update your app (rare):

``` yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: static-web
spec:
  replicas: 5
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

**Problem:** To update, you must delete and recreate. Deployments handle this automatically.

### 2. Learning/Debugging

Understanding ReplicaSets helps debug Deployment issues.

---

## ReplicaSet Behavior

### Self-Healing

``` bash
# Delete a pod
kubectl delete pod web-7c5ddbdf54-xxxxx

# Immediately check
kubectl get pods
# A new pod is being created to maintain replica count!
```

### Manual Scaling

``` bash
# Scale ReplicaSet directly (don't do this in production!)
kubectl scale replicaset web-7c5ddbdf54 --replicas=5

# Better: Scale the Deployment
kubectl scale deployment web --replicas=5
```

### Orphaned Pods

If you delete a ReplicaSet with `--cascade=orphan`, Pods remain:

``` bash
kubectl delete replicaset web-rs --cascade=orphan
# ReplicaSet deleted, but Pods still running (orphaned)
```

**When useful:** Advanced scenarios like transferring pod ownership.

---

## Troubleshooting ReplicaSets

### Problem: Pods Not Starting

``` bash
kubectl describe replicaset web-7c5ddbdf54
# Check Events section for errors
```

**Common issues:**
- Image pull errors
- Insufficient cluster resources
- Invalid pod template

### Problem: Wrong Number of Pods

``` bash
kubectl get replicaset web-7c5ddbdf54
# DESIRED   CURRENT   READY
# 3         2         2

kubectl describe replicaset web-7c5ddbdf54
# Check why it can't create the 3rd pod
```

### Problem: Pods Not Managed by ReplicaSet

If pod labels don't match ReplicaSet selector, they won't be managed:

``` bash
# ReplicaSet selector
spec:
  selector:
    matchLabels:
      app: web

# Pod without matching label
metadata:
  labels:
    app: website  # ❌ Doesn't match!
```

**Fix:** Update pod labels or ReplicaSet selector.

---

## Quick Recap

| Concept | Explanation |
|---------|-------------|
| **ReplicaSet** | Maintains desired pod count |
| **Deployment** | Manages ReplicaSets for updates |
| **Selector** | How ReplicaSet finds its pods |
| **Template** | Pod specification to create |
| **Self-Healing** | Automatically replaces failed pods |
| **Rolling Update** | Uses two ReplicaSets (old + new) |

---

## Further Reading

- [ReplicaSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [Deployments](deployments.md) - How Deployments use ReplicaSets
- [ReplicaSet vs ReplicationController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/) - Legacy comparison

---

## What's Next?

You understand how ReplicaSets maintain pod counts. Next, learn about **[StatefulSets](statefulsets.md)** for applications that need stable identities and ordered deployment (like databases).

---

**Key Takeaway:** Use Deployments in production, not ReplicaSets directly. But understanding ReplicaSets helps you debug and understand how Kubernetes works under the hood.
