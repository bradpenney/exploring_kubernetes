# StatefulSets: Stable Identity for Stateful Apps

!!! tip "Part of Level 2: Workload Management"
    This article is part of [Level 2: Workload Management](overview.md). Read [Deployments Explained](deployments.md) and [ReplicaSets Under the Hood](replicasets.md) first.

You've been using Deployments for stateless applications. But what if your app needs:

- Stable, unique network identifiers
- Stable, persistent storage
- Ordered deployment and scaling
- Ordered updates and rollbacks

**That's what StatefulSets are for.**

---

## When You Need StatefulSets

<div class="grid cards" markdown>

-   **Databases**

    ---
    PostgreSQL, MySQL, MongoDB clusters need stable identities and persistent data.

-   **Distributed Systems**

    ---
    Kafka, ZooKeeper, etcd require predictable hostnames and ordered startup.

-   **Caching Layers**

    ---
    Redis Sentinel, Memcached clusters with specific node roles.

-   **Legacy Apps**

    ---
    Applications that expect specific hostnames or require ordered initialization.

</div>

---

## StatefulSet vs Deployment

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| **Pod Names** | Random suffix (`web-7c5ddbdf54-xxxxx`) | Ordinal suffix (`web-0`, `web-1`, `web-2`) |
| **Network Identity** | Changes on restart | Stable across restarts |
| **Storage** | Shared or ephemeral | Unique persistent volume per pod |
| **Startup Order** | Parallel (all at once) | Sequential (`0` → `1` → `2`) |
| **Scaling** | Parallel | Sequential |
| **Use Case** | Stateless apps | Stateful apps with identity needs |

**Rule of thumb:** Can any pod handle any request? Use Deployment. Do pods have specific roles or data? Use StatefulSet.

---

## StatefulSet Anatomy

```yaml title="statefulset.yaml" linenums="1"
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-svc  # (1)!
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
        image: postgres:14
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:  # (2)!
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

1. Headless Service for stable network identity
2. Creates unique PVC for each pod

---

## Stable Network Identity

StatefulSets require a **Headless Service** (Service with `clusterIP: None`):

```yaml title="headless-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: postgres-svc
spec:
  clusterIP: None  # (1)!
  selector:
    app: postgres
  ports:
  - port: 5432
```

1. Makes this a headless service

**What you get:**

=== "Pod Hostnames"
    ```
    postgres-0.postgres-svc.default.svc.cluster.local
    postgres-1.postgres-svc.default.svc.cluster.local
    postgres-2.postgres-svc.default.svc.cluster.local
    ```

=== "DNS Resolution"
    ```bash
    # From another pod:
    nslookup postgres-0.postgres-svc
    # Returns: postgres-0's IP address

    nslookup postgres-svc
    # Returns: ALL pod IPs
    ```

=== "Connection Strings"
    ```bash
    # Connect to specific pod:
    psql -h postgres-0.postgres-svc -U admin

    # Connect to any pod (load balanced):
    psql -h postgres-svc -U admin
    ```

**Why it matters:** Even if `postgres-0` gets rescheduled to a different node, its hostname remains the same.

---

## Persistent Storage Per Pod

StatefulSets automatically create a **PersistentVolumeClaim** for each pod:

```mermaid
graph LR
    SS[StatefulSet: 3 replicas] --> P0[postgres-0]
    SS --> P1[postgres-1]
    SS --> P2[postgres-2]

    P0 --> PVC0[PVC: postgres-storage-postgres-0]
    P1 --> PVC1[PVC: postgres-storage-postgres-1]
    P2 --> PVC2[PVC: postgres-storage-postgres-2]

    PVC0 --> PV0[PersistentVolume: 10Gi]
    PVC1 --> PV1[PersistentVolume: 10Gi]
    PVC2 --> PV2[PersistentVolume: 10Gi]

    style SS fill:#48bb78,stroke:#2d3748,color:#fff
    style P0 fill:#2d3748,stroke:#cbd5e0,color:#fff
    style P1 fill:#2d3748,stroke:#cbd5e0,color:#fff
    style P2 fill:#2d3748,stroke:#cbd5e0,color:#fff
    style PVC0 fill:#4299e1,stroke:#2d3748,color:#fff
    style PVC1 fill:#4299e1,stroke:#2d3748,color:#fff
    style PVC2 fill:#4299e1,stroke:#2d3748,color:#fff
    style PV0 fill:#805ad5,stroke:#2d3748,color:#fff
    style PV1 fill:#805ad5,stroke:#2d3748,color:#fff
    style PV2 fill:#805ad5,stroke:#2d3748,color:#fff
```

**If a pod dies:**
- New pod gets same name (`postgres-0`)
- Reattaches to same PVC
- Data persists across restarts

---

## Ordered Operations

### Deployment Order

StatefulSets deploy pods sequentially:

```bash
kubectl apply -f statefulset.yaml

# Watch the deployment:
kubectl get pods -w
# NAME          READY   STATUS
# postgres-0    0/1     Pending
# postgres-0    1/1     Running  ← Waits until Running before next
# postgres-1    0/1     Pending
# postgres-1    1/1     Running
# postgres-2    0/1     Pending
# postgres-2    1/1     Running
```

**Why?** Some apps need `postgres-0` (primary) running before replicas start.

### Scaling Order

```bash
# Scale up: adds postgres-3, postgres-4 (in order)
kubectl scale statefulset postgres --replicas=5

# Scale down: removes postgres-4, postgres-3 (reverse order)
kubectl scale statefulset postgres --replicas=3
```

!!! warning "Scale Down Deletes Pods, Not Data"
    Scaling down deletes pods but **keeps their PVCs**. If you scale back up, the same PVCs are reattached.

### Update Strategy

```yaml
spec:
  updateStrategy:
    type: RollingUpdate  # (1)!
    rollingUpdate:
      partition: 0  # (2)!
```

1. Updates pods in reverse order (`postgres-2` → `postgres-1` → `postgres-0`)
2. Only update pods with ordinal >= partition

**Canary updates:**
```bash
# Update only postgres-2 (ordinal 2)
kubectl patch statefulset postgres -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":2}}}}'
kubectl set image statefulset/postgres postgres=postgres:15

# Test postgres-2 with new version
# If good, update all:
kubectl patch statefulset postgres -p '{"spec":{"updateStrategy":{"rollingUpdate":{"partition":0}}}}'
```

---

## Real-World Example: PostgreSQL Cluster

```yaml title="postgres-statefulset.yaml" linenums="1"
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None  # Headless service
  selector:
    app: postgres
  ports:
  - port: 5432
    name: postgres

---
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
        image: postgres:14
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_DB
          value: "myapp"
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        - name: PGDATA  # (1)!
          value: "/var/lib/postgresql/data/pgdata"
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        livenessProbe:  # (2)!
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:  # (3)!
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

1. PostgreSQL requires subdirectory for data
2. Restarts container if database is unhealthy
3. Only routes traffic when database is ready

**Deploy it:**
```bash
# Create secret
kubectl create secret generic postgres-secret \
  --from-literal=username=admin \
  --from-literal=password=supersecret

# Deploy StatefulSet
kubectl apply -f postgres-statefulset.yaml

# Watch pods come up in order
kubectl get pods -w -l app=postgres
```

---

## Working with StatefulSets

### Accessing Specific Pods

=== "Direct Pod Access"
    ```bash
    # Connect to postgres-0 (primary)
    kubectl exec -it postgres-0 -- psql -U admin -d myapp

    # Connect to postgres-1 (replica)
    kubectl exec -it postgres-1 -- psql -U admin -d myapp
    ```

=== "Via Service"
    ```yaml
    # From another pod:
    apiVersion: v1
    kind: Pod
    metadata:
      name: client
    spec:
      containers:
      - name: psql-client
        image: postgres:14
        env:
        - name: DATABASE_URL
          value: "postgres://admin:supersecret@postgres-0.postgres:5432/myapp"
        command: ['sh', '-c', 'sleep 3600']
    ```

### Viewing Storage

```bash
# List PVCs created by StatefulSet
kubectl get pvc
# NAME                              STATUS   VOLUME      CAPACITY
# postgres-storage-postgres-0       Bound    pv-001      10Gi
# postgres-storage-postgres-1       Bound    pv-002      10Gi
# postgres-storage-postgres-2       Bound    pv-003      10Gi

# Describe a PVC
kubectl describe pvc postgres-storage-postgres-0
```

### Manual Intervention

```bash
# Delete a specific pod (will be recreated)
kubectl delete pod postgres-1
# Pod recreates with same name and PVC

# Force delete stuck pod
kubectl delete pod postgres-0 --grace-period=0 --force
```

!!! danger "Forced Deletion Risks"
    Force-deleting StatefulSet pods can cause split-brain in databases. Only use as last resort.

---

## StatefulSet Gotchas

### 1. PVCs Don't Auto-Delete

When you delete a StatefulSet, PVCs remain:

```bash
kubectl delete statefulset postgres
# Pods deleted, but PVCs remain!

kubectl get pvc
# Still there!
```

**Why?** To prevent accidental data loss. You must manually delete PVCs:

```bash
kubectl delete pvc postgres-storage-postgres-0
kubectl delete pvc postgres-storage-postgres-1
kubectl delete pvc postgres-storage-postgres-2
```

### 2. Scaling Down Keeps PVCs

```bash
# Scale from 3 to 1
kubectl scale statefulset postgres --replicas=1
# Deletes postgres-2 and postgres-1 pods
# But keeps their PVCs!

# Scale back up to 3
kubectl scale statefulset postgres --replicas=3
# postgres-1 and postgres-2 recreate with OLD data
```

### 3. Parallel Pod Management

By default, StatefulSets manage pods sequentially. For faster operations:

```yaml
spec:
  podManagementPolicy: Parallel  # (1)!
```

1. Starts/stops all pods at once (like Deployments)

**Trade-off:** Faster, but breaks ordering guarantees needed by some apps.

---

## When NOT to Use StatefulSets

❌ **Don't use StatefulSets if:**
- Your app is stateless
- Pods don't need unique identities
- You don't need persistent storage per pod
- Ordering doesn't matter

✅ **Use Deployments instead** for most apps.

**Example of misuse:**
```yaml
# DON'T DO THIS
kind: StatefulSet
metadata:
  name: web
spec:
  replicas: 10
  # No volumeClaimTemplates
  # No need for stable identity
  # Just use a Deployment!
```

---

## Troubleshooting StatefulSets

### Problem: Pod Stuck in Pending

```bash
kubectl describe pod postgres-0
# Events:
#   Warning  FailedScheduling  No nodes available (insufficient CPU)
```

**Common causes:**
- No available PersistentVolumes
- Insufficient node resources
- Pod waiting for previous pod to be Ready

### Problem: Pod Won't Start

```bash
kubectl describe statefulset postgres
# Events:
#   Warning  FailedCreate  pods "postgres-0" already exists
```

**Solution:** Delete the stuck pod:
```bash
kubectl delete pod postgres-0
```

### Problem: Data Loss After Restart

```bash
# Check if PVC is bound
kubectl get pvc postgres-storage-postgres-0
# STATUS: Bound ✅ or Pending ❌

# Check mount inside pod
kubectl exec postgres-0 -- df -h | grep postgresql
```

**If unbound:** Your cluster might not have a StorageClass or available PVs.

---

## Quick Recap

| Concept | Explanation |
|---------|-------------|
| **StatefulSet** | Manages stateful apps with stable identities |
| **Headless Service** | Provides stable DNS names for pods |
| **Ordinal Index** | Pods get predictable names (`app-0`, `app-1`) |
| **VolumeClaimTemplate** | Creates unique PVC per pod |
| **Ordered Deployment** | Pods start sequentially |
| **Ordered Scaling** | Scale up/down maintains order |
| **PVC Retention** | PVCs survive pod deletion |

---

## Practice Exercises

??? question "Exercise 1: Deploy a StatefulSet"
    Deploy a 3-replica Redis StatefulSet with persistent storage.

    **Requirements:**
    - Headless Service named `redis`
    - 3 Redis pods with 1Gi storage each
    - Verify stable hostnames work

    ??? tip "Solution"
        ```yaml title="redis-statefulset.yaml"
        ---
        apiVersion: v1
        kind: Service
        metadata:
          name: redis
        spec:
          clusterIP: None
          selector:
            app: redis
          ports:
          - port: 6379

        ---
        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          name: redis
        spec:
          serviceName: redis
          replicas: 3
          selector:
            matchLabels:
              app: redis
          template:
            metadata:
              labels:
                app: redis
            spec:
              containers:
              - name: redis
                image: redis:7
                ports:
                - containerPort: 6379
                volumeMounts:
                - name: redis-storage
                  mountPath: /data
          volumeClaimTemplates:
          - metadata:
              name: redis-storage
            spec:
              accessModes: ["ReadWriteOnce"]
              resources:
                requests:
                  storage: 1Gi
        ```

        ```bash
        kubectl apply -f redis-statefulset.yaml

        # Watch pods come up
        kubectl get pods -w -l app=redis

        # Test DNS resolution
        kubectl run -it --rm debug --image=busybox --restart=Never -- \
          nslookup redis-0.redis
        ```

??? question "Exercise 2: Scale and Verify Persistence"
    Scale the Redis StatefulSet from 3 to 1, then back to 3. Verify data persists.

    ??? tip "Solution"
        ```bash
        # Write data to redis-1
        kubectl exec redis-1 -- redis-cli SET mykey "hello world"

        # Scale down
        kubectl scale statefulset redis --replicas=1

        # Check PVCs still exist
        kubectl get pvc
        # redis-storage-redis-1 should still be there

        # Scale back up
        kubectl scale statefulset redis --replicas=3

        # Verify data persisted
        kubectl exec redis-1 -- redis-cli GET mykey
        # Output: "hello world"
        ```

---

## Further Reading

### Official Documentation
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [StatefulSet Basics Tutorial](https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/)
- [Headless Services](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services)

### Deep Dives
- [Running PostgreSQL on Kubernetes](https://kubernetes.io/docs/tutorials/stateful-application/postgres/)
- [Deploying Cassandra with StatefulSets](https://kubernetes.io/docs/tutorials/stateful-application/cassandra/)
- [StatefulSet Pod Management Policies](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/#pod-management-policies)

### Related Articles
- [Deployments](deployments.md) - For stateless applications
- **Persistent Volumes** - Storage deep dive (coming in Level 4)
- **Init Containers** - For StatefulSet initialization (coming in Level 3)

---

## What's Next?

You understand StatefulSets for apps with stable identities. Next, learn about **[DaemonSets](daemonsets.md)** for running a pod on every node (monitoring agents, log collectors, etc.).

---

**Key Takeaway:** Use StatefulSets only when you need stable network identity, persistent storage per pod, or ordered operations. For everything else, use Deployments.
