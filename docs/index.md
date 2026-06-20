---
date: "2025-07-13 07:55"
title: Exploring Kubernetes - Developer to Platform Engineer Guide
description: A practical journey through Kubernetes for developers and platform engineers. Learn core concepts, kubectl, and Helm through real-world scenarios.
---
<img src="images/exploring_kubernetes.png" alt="Exploring Kubernetes" class="img-responsive-right" width="300">

# Exploring Kubernetes

**From first deployment to production operations.**

Your manager said "we're using Kubernetes now," or you saw it on a job description, or your team adopted it and you're expected to figure it out. This site takes you from "what even is this?" through deploying your first application to running production clusters — at whatever level you need.

A subsection of [BradPenney.io](https://bradpenney.io), written from years of production Kubernetes experience.

---

## 🏥 Day One: Getting Started

**Everyone starts here.** Two articles set the foundation, then you choose the deployment path that matches how your team works:

- [Day One Overview](day_one/overview.md) — your roadmap and what to expect
- [What Is Kubernetes?](day_one/what_is_kubernetes.md) — the problem it solves and why companies adopt it

<div class="grid cards" markdown>

-   :material-code-braces: **Path 1: From Scratch (`kubectl`)**

    ---

    You're learning the fundamentals, or your team deploys with raw YAML. Write Deployment and Service manifests, apply them with `kubectl`, and understand Pods and Services from first principles.

    [:octicons-arrow-right-24: Start the kubectl path](day_one/kubectl/access.md)

-   :material-package-variant: **Path 2: Using Helm**

    ---

    Your pipeline generates Helm charts, or you're installing vendor software (Prometheus, Grafana, nginx) from community charts. Deploy and customize with `values.yaml`, and see what Helm creates underneath.

    [:octicons-arrow-right-24: Start the Helm path](day_one/helm/access.md)

</div>

!!! tip "Both paths converge at Essentials"
    Whether you start with raw `kubectl` or with Helm, you land in the same place: running Pods, stable Services, and the confidence to deploy and debug.

---

## 📦 Essentials

Not how to *use* each primitive, but what it *is* — why Kubernetes works this way, and what it means for the cluster you share with everyone else.

- [Essentials Overview](essentials/overview.md) — what this tier covers and how to approach it

<div class="grid cards" markdown>

-   :material-cube: **Core Primitives**

    ---

    The atomic objects everything else is built from — Pods, Services, configuration, isolation, and the labels that wire it all together.

    [:octicons-arrow-right-24: Pods](essentials/pods.md) · [Services](essentials/services.md) · [ConfigMaps & Secrets](essentials/config_and_secrets.md) · [Namespaces](essentials/namespaces.md) · [Labels & Selectors](essentials/labels_selectors.md)

-   :material-layers-triple: **Workloads & Security** *(coming soon)*

    ---

    The workloads every app dev already uses — Deployments, ReplicaSets, Jobs — and the security you own: `securityContext`, secret hygiene, RBAC, and Pod Security Standards.

</div>

---

## ⚡ Efficiency *(Coming soon)*

Running real workloads at scale — the platform-specific pieces beyond the basics.

<div class="grid cards" markdown>

-   :material-cog-sync: **Advanced Workloads** *(coming soon)*

    ---

    Beyond Deployments: StatefulSets for stable identity and ordered startup, DaemonSets for node-level agents, and the rollout and scaling controls that keep real applications healthy.

-   :material-lan: **Networking** *(coming soon)*

    ---

    Connecting and controlling traffic across the cluster — Ingress controllers and TLS, Network Policies, DNS and service discovery, and how to troubleshoot networking when it breaks.

</div>

---

## 🎯 Mastery *(Coming soon)*

Production Kubernetes — depth, scale, and the operational reality of running clusters others depend on.

<div class="grid cards" markdown>

-   :material-harddisk: **Storage & State** *(coming soon)*

    ---

    Persistent data done right — Volumes, PersistentVolumes and Claims, StorageClasses and dynamic provisioning, and when *not* to run stateful systems on Kubernetes.

-   :material-shield-key: **Scheduling & Security** *(coming soon)*

    ---

    Controlling placement and locking things down — resource requests and limits, taints, tolerations and affinity, and RBAC designed across many teams.

-   :material-chart-line: **Production Operations** *(coming soon)*

    ---

    Keeping clusters healthy at scale — logging and monitoring, health probes, Helm at scale, and the operator pattern with custom resources.

</div>
