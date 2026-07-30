---
date: "2026-07-30 10:00"
title: "Kubernetes Efficiency: Runtime, Networking & Scheduling"
description: "Beyond single Pods: how the CRI actually runs your containers, how the scheduler picks a Node, and how to put a cluster app on the real internet."
---
# Efficiency

!!! tip "Building on Essentials"
    Essentials gave you the primitives and the objects you already touch as an app dev. Efficiency is where the platform underneath them stops being a black box — the runtime that actually runs your containers, the scheduler that places them, and the networking that gets real traffic to them.

Essentials treated `kubectl apply` as something that just works once you understand the objects. Efficiency asks the next question: what makes it work? kubelet doesn't run your container directly — it hands the job to a runtime through a well-defined interface. The scheduler doesn't place Pods randomly — it filters and scores every Node before picking one. And exposing a Service to the internet is a chain of independent, composable pieces, not a single Ingress rule.

This tier treats you as the platform engineer now responsible for these systems, not just a consumer of them.

## Start Here: Architecture & Scheduling

The mechanics underneath every Pod you've already deployed.

<div class="grid cards two-col" markdown>

-   :material-cog-sync: **[The CRI](container_runtime.md)**

    ---

    How kubelet actually talks to containerd and CRI-O — and the dockershim history behind why this interface exists at all.

-   :material-timer-sand: **[The Scheduler](scheduler.md)**

    ---

    Filtering and scoring: how a Pod picks a Node, and why affinity should target a *type* of Node, never one specific Node.

</div>

## Putting It on the Internet

The chain of pieces between a running Pod and a real domain name.

<div class="grid cards two-col" markdown>

-   :material-door: **[Gateway API](networking/gateway_api.md)**

    ---

    The standard front door — Traefik as the Gateway API implementation, replacing the aging Ingress model.

-   :material-certificate: **[cert-manager](networking/cert_manager.md)**

    ---

    Certificates as cluster resources — automated issuance and renewal, no more manually copied `.pem` files.

-   :material-web: **[external-dns](networking/external_dns.md)**

    ---

    Pointing your domain at the cluster automatically, kept in sync as Services and Ingresses change.

-   :material-layers-triple: **Advanced Workloads** *(coming soon)*

    ---

    StatefulSets for stable identity, DaemonSets for node-level agents, and the rollout controls that keep them healthy.

</div>

---

## What You'll Take Away

By the end of Efficiency you'll be able to:

- Trace a container from `kubectl apply` through kubelet, the CRI, and the runtime to a running process
- Explain why a Pod landed on the Node it did — and predict where the next one will go
- Stand up a real front door for a cluster app: routing, TLS, and DNS, all reconciled automatically

---

## What's Next?

Start with **[The CRI](container_runtime.md)** — the layer between the object you applied and the process actually running.

After Efficiency, the Mastery tier takes you into production operations: storage, scheduling depth, security, and observability for clusters you're responsible for.
