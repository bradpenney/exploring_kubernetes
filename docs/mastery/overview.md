---
title: "Kubernetes Mastery: Production Operations & Security"
description: "Run production Kubernetes with confidence: storage and state, resource scheduling, RBAC and security, plus logging, monitoring, probes, Helm, and operators."
---
# Mastery: Production Kubernetes

!!! tip "From user to operator"
    You can deploy and connect applications. The **Mastery** tier is about *running clusters* — the storage, scheduling, security, and operational depth that keep production healthy.

## What You'll Master

This is platform-engineer and SRE territory. The persona shifts decisively: you're now responsible for the cluster, not just your app on it.

---

<div class="grid cards" markdown>

-   :material-server-network: **[Cluster Architecture](cluster_architecture.md)**

    ---

    How a cluster actually works under the hood — the Control Plane (API server, etcd, scheduler, controller manager) and the Worker Nodes that run your workloads. The mental model everything else rests on.

-   :material-database: **[Storage and State](storage_overview.md)**

    ---

    Persisting data beyond Pod restarts — Volumes, Persistent Volumes and Claims, StorageClasses for dynamic provisioning, and the realities of running stateful workloads.

-   :material-shield-lock: **[Scheduling & Security](security_overview.md)**

    ---

    Production readiness — resource requests and limits, taints/tolerations and affinity for placement, RBAC for least-privilege access, and security best practices for hardening clusters.

-   :material-eye: **[Production Operations](operations_overview.md)**

    ---

    Keeping it running at scale — logging architecture, monitoring with Prometheus and Grafana, health probes, Helm for packaging, and the operator pattern for automating operations.

</div>

---

## Who This Is For

**You are:**

- A platform engineer or SRE responsible for production clusters
- Making architectural decisions, not just following deployment instructions
- Accountable for reliability, security, and cost at scale

If you need a local cluster to practice these topics, use **k0s** (lightweight, production-like) rather than minikube or kind.

---

## What's Next?

Start with **[Cluster Architecture](cluster_architecture.md)** if you want the foundational mental model, or jump straight to **[Storage and State](storage_overview.md)** to begin the hands-on production track.
