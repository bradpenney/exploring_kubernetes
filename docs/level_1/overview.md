---
title: "Level 1: Core Primitives — Kubernetes Building Blocks"
description: "Master the fundamental Kubernetes resources: Pods, Services, ConfigMaps, Namespaces, and Labels. Build on Day One with deeper understanding of how Kubernetes works."
---
# Level 1: Core Primitives

!!! tip "Building on Day One"
    You've deployed your first application. Now let's master the fundamental building blocks of Kubernetes.

## What You'll Master

Level 1 covers the **core Kubernetes resources** you'll use in every deployment — the primitives everything else builds on. Master these, and the rest makes sense.

---

<div class="grid cards" markdown>

-   :material-cube: **[Pods Deep Dive](pods.md)**

    ---

    **The Foundation** — What a Pod is, multi-container sidecars, init containers, Pod lifecycle, and essential `kubectl` debugging commands

-   :material-lan: **[Services: Stable Networking for Pods](services.md)**

    ---

    **Networking Basics** — ClusterIP, NodePort, LoadBalancer, DNS service discovery, and port-forwarding for local testing

-   :material-cog: **ConfigMaps and Secrets** *(coming soon)*

    ---

    **Configuration Management** — Environment variables, mounted files, sensitive data handling

-   :material-folder: **Namespaces** *(coming soon)*

    ---

    **Logical Separation** — Multi-tenancy, resource quotas, working across namespaces

-   :material-tag: **Labels and Selectors** *(coming soon)*

    ---

    **Organization** — Targeting workloads, filtering resources, best practices

</div>

---

## Who This Is For

**You are:**

- An application developer getting comfortable with Kubernetes
- Confident deploying applications (thanks to Day One!)
- Ready to understand HOW Kubernetes works under the hood
- Building more complex applications that need configuration, networking, and organization

**You'll learn:**

- How to structure multi-container applications
- How to make services talk to each other
- How to manage configuration separately from code
- How to organize resources logically
- How to target specific resources with selectors

---

## Learning Safely in Your Namespace

**Remember: You're still in the dev cluster, still in your namespace.**

Throughout Level 1, keep these safety principles in mind:

!!! tip "Safe Learning Environment"
    **You can't break production:**

    - Production is in a completely different cluster (or at minimum, a different namespace you don't have access to)
    - Your experiments stay in your namespace—isolated from other teams
    - Worst case: you delete your pods and redeploy (annoying, not catastrophic)

    **Exploring is encouraged:**

    - Read-only commands (`get`, `describe`, `logs`) are always safe
    - Creating and deleting resources in YOUR namespace is fine
    - Practice makes perfect—deploy, break, fix, learn

    **Double-check your namespace:**

    - Always verify which namespace you're working in (`kubectl config view --minify`)
    - Use namespace-aware commands (`kubectl get pods -n yournamespace`)
    - Set your default namespace once and forget it (covered in Namespaces article)

**The goal:** Build confidence through hands-on practice. You can't learn Kubernetes by reading—you learn by deploying, debugging, and understanding what went wrong.

---

## What's Next?

Start with the foundation of everything in Kubernetes.

**Next:** **[Pods Deep Dive](pods.md)** — What a Pod is, how containers share a network and storage, the Pod lifecycle, and the debugging commands you'll use every day.
