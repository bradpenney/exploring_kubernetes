# Level 4: Storage and State

!!! tip "Handling Persistent Data"
    Your applications need to remember things. Learn how Kubernetes handles storage, persistence, and stateful applications.

## What You'll Master

Level 4 teaches **Kubernetes storage**—how to persist data beyond pod restarts:

- **Volumes** - Volume types (emptyDir, hostPath, etc.), when to use each
- **Persistent Volumes (PV)** - Cluster storage resources, lifecycle, reclaim policies
- **Persistent Volume Claims (PVC)** - Requesting storage, binding, access modes
- **StorageClasses** - Dynamic provisioning, storage parameters, cloud integration
- **Running Databases on Kubernetes** - StatefulSets with storage, when NOT to use K8s

This is where **stateless meets stateful**—the hardest part of cloud-native architecture.

---

## The Articles

<div class="grid cards" markdown>

-   :material-folder-open: **[Understanding Volumes](volumes.md)** *(coming soon)*

    ---

    **Volume Types** — emptyDir, hostPath, configMap, secret, and more—when to use each

-   :material-harddisk: **[Persistent Volumes (PV)](persistent_volumes.md)** *(coming soon)*

    ---

    **Cluster Storage** — Lifecycle, reclaim policies, access modes, manual provisioning

-   :material-file-document: **[Persistent Volume Claims (PVC)](persistent_volume_claims.md)** *(coming soon)*

    ---

    **Requesting Storage** — Binding, storage requests, access modes, selector matching

-   :material-database-cog: **StorageClasses** *(coming soon)*

    ---

    **Dynamic Provisioning** — Cloud provider integration, storage parameters, default classes

-   :material-database: **Running Databases on Kubernetes** *(coming soon)*

    ---

    **Stateful Apps** — StatefulSets with storage, backup strategies, when NOT to use K8s

</div>

---

## Who This Is For

**You are:**
- An application developer running stateful applications (databases, file uploads, caches)
- A platform engineer managing cluster storage
- Someone who's lost data when a pod restarted (ouch!)
- Looking to understand when Kubernetes is and isn't appropriate for data

**You'll learn:**
- How to persist data beyond pod lifecycle
- When to use ephemeral vs persistent storage
- How dynamic provisioning works with cloud providers
- How to run databases on Kubernetes (and when NOT to!)
- Storage best practices and common pitfalls

---

## The Hard Truth About Storage

Storage in Kubernetes is **complex**. There are multiple layers:

1. **Volumes** (ephemeral, attached to pods)
2. **Persistent Volumes** (cluster-managed, independent of pods)
3. **Persistent Volume Claims** (developer requests)
4. **StorageClasses** (dynamic provisioning)
5. **Cloud Provider Integration** (EBS, Azure Disk, GCE PD, etc.)

We'll break it down piece by piece. Don't try to understand it all at once.

---

## When NOT to Use Kubernetes for Storage

**Important reality check:** Not everything should run on Kubernetes.

- **Managed databases** (RDS, Cloud SQL, Azure Database) are often simpler
- **Object storage** (S3, GCS, Azure Blob) is better for large files
- **Legacy databases** with complex replication might be happier on VMs

We'll discuss when Kubernetes storage makes sense and when it doesn't.

---

## What's Next?

After Level 4, you'll move to **Level 5: Advanced Scheduling & Security**, where you'll learn about resource management, node affinity, RBAC, and security best practices.

Storage is challenging—but it's also where Kubernetes truly enables stateful, production-ready applications.
