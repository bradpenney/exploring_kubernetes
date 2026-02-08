# Level 5: Advanced Scheduling & Security

!!! tip "Production-Ready Kubernetes"
    You can deploy, network, and persist data. Now learn how to control where pods run, manage resources, and secure your cluster.

## What You'll Master

Level 5 teaches **advanced Kubernetes operations**—production readiness:

- **Resource Requests and Limits** - CPU/memory management, QoS classes, resource quotas
- **Taints and Tolerations** - Node specialization, dedicated nodes, eviction
- **Node Affinity and Pod Affinity** - Advanced scheduling, co-location, anti-affinity
- **RBAC** - Users, service accounts, roles, role bindings, least privilege
- **Security Best Practices** - Pod Security Standards, security contexts, admission control

This is **platform engineer and SRE territory**—production operations at scale.

---

## The Articles

<div class="grid cards" markdown>

-   :material-memory: **Resource Requests and Limits** *(coming soon)*

    ---

    **Resource Management** — CPU/memory requests, limits, QoS classes, resource quotas, LimitRanges

-   :material-close-circle: **Taints and Tolerations** *(coming soon)*

    ---

    **Node Specialization** — Dedicated nodes, GPU nodes, eviction, scheduling control

-   :material-magnet: **Node Affinity and Pod Affinity** *(coming soon)*

    ---

    **Advanced Scheduling** — Node selection, pod co-location, anti-affinity, topology

-   :material-shield-lock: **RBAC** *(coming soon)*

    ---

    **Access Control** — Users, service accounts, roles, role bindings, cluster roles, least privilege

-   :material-security: **Security Best Practices** *(coming soon)*

    ---

    **Hardening** — Pod Security Standards, security contexts, network policies, admission controllers

</div>

---

## Who This Is For

**You are:**
- A platform engineer managing Kubernetes infrastructure
- An SRE responsible for cluster reliability and security
- Someone preparing for production deployments
- Looking to implement security and resource management

**You'll learn:**
- How to prevent resource contention and starvation
- How to schedule pods on specific nodes (GPUs, SSDs, high-memory)
- How to implement least-privilege access control
- How to harden clusters against security threats
- How to handle multi-tenancy securely

---

## The Persona Shift: Platform Engineer

Starting at Level 5, the focus shifts **decisively to platform engineering:**

- **Not just "my app"**—now it's "all apps on this cluster"
- **Not just "does it work?"**—now it's "will it stay working under load?"
- **Not just "can I deploy?"**—now it's "who should be allowed to deploy what?"

This is where you become responsible for the **platform** that enables other developers.

---

## Production Readiness Checklist

By the end of Level 5, you'll be able to answer:

- ✅ Do we have resource limits to prevent one app starving others?
- ✅ Do we have node taints for specialized workloads?
- ✅ Do we have RBAC policies for least-privilege access?
- ✅ Do we have security contexts to prevent privileged containers?
- ✅ Do we have network policies to control pod-to-pod traffic?

These are **table stakes for production Kubernetes**.

---

## What's Next?

After Level 5, you'll move to **Level 6: Production Operations**, where you'll learn about logging, monitoring, health checks, Helm, and operators—the final pieces of running Kubernetes in production.

Level 5 is challenging. It's also where Kubernetes clusters become truly production-ready.
