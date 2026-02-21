---
title: Exploring Kubernetes - Developer to Platform Engineer Guide
description: A practical journey through Kubernetes for developers and platform engineers. Learn core concepts, kubectl, and Helm through real-world scenarios.
---
<img src="images/exploring_kubernetes.png" alt="Exploring Kubernetes" class="img-responsive-right" width="300">

# Exploring Kubernetes

**From first deployment to production operations.**

A subsection of [BradPenney.io](https://bradpenney.io), this site teaches practical Kubernetes skills through a progressive learning journey—from application developer deploying your first app to platform engineer running production clusters.

It emphasizes real-world scenarios, production safety, and the "why" behind every resource and command.

---

## Learning Path

### Day One: Getting Started

**Everyone starts here:** Learn what Kubernetes is and why it exists, then choose your path based on how you'll deploy applications.

**Shared foundation (read these first):**

- [Day One Overview](day_one/overview.md) - Your complete Day One roadmap
- [What Is Kubernetes?](day_one/what_is_kubernetes.md) - The problem Kubernetes solves and why companies adopt it

**Then choose your deployment path:**

<div class="grid cards" markdown>

- :material-code-braces: **Path 1: From Scratch (kubectl)**

    ---

    **For:** Learning Kubernetes fundamentals by writing YAML and using `kubectl` directly

    **Your situation:** You're learning Kubernetes from the ground up, or your team deploys with raw YAML manifests

    **What you'll do:** Write Deployment and Service YAML, apply with `kubectl`, understand Pods, ReplicaSets, and Services from first principles

    **Articles:**

    - [Getting `kubectl` Access](day_one/kubectl/access.md)
    - [Your First Deployment](day_one/kubectl/first_deploy.md)
    - [Essential `kubectl` Commands](day_one/kubectl/commands.md)
    - [Understanding What Happened](day_one/kubectl/understanding.md)

- :material-package-variant: **Path 2: Using Helm**

    ---

    **For:** Working with Helm charts—either from CI/CD pipelines or vendor distributions

    **Your situation:** Your CI/CD pipeline (Jenkins, GitLab CI, GitHub Actions, etc.) generates Helm charts, or you need to deploy vendor software like Prometheus/Grafana from Bitnami charts

    **What you'll do:** Deploy and customize Helm charts with `values.yaml`, use `helm` commands, understand what Helm creates under the hood

    **Articles:**

    - [Getting Helm Access](day_one/helm/access.md)
    - Your First Helm Deployment *(coming soon)*
    - Essential Helm Commands *(coming soon)*
    - Understanding What Helm Created *(coming soon)*

</div>

!!! tip "Both Paths Converge at Level 1"
    Whether you start From Scratch or Using Helm, both paths teach you to deploy applications and understand Kubernetes. They converge at Level 1 where everyone needs the same deep knowledge of Pods, Services, and Deployments.

### Continuing Your Journey

<div class="grid cards" markdown>

- :material-cube: **Level 1 - Core Primitives** *(Coming Soon)*

    ---

    **For:** App developers getting comfortable

    **Goal:** Master the fundamental building blocks

    - Pods Deep Dive
    - Services: Connecting to Pods
    - ConfigMaps and Secrets
    - Namespaces
    - Labels and Selectors

- :material-sync: **Level 2 - Workload Management** *(Coming Soon)*

    ---

    **For:** App developers deploying real applications

    **Goal:** Manage applications at scale

    - Deployments Explained
    - ReplicaSets Under the Hood
    - StatefulSets
    - DaemonSets
    - Jobs and CronJobs

- :material-lan: **Level 3 - Networking** *(Coming Soon)*

    ---

    **For:** App developers + Platform engineers

    **Goal:** Connect services, expose applications

    - Services Deep Dive
    - Ingress Controllers
    - Network Policies
    - DNS and Service Discovery
    - Troubleshooting Networking

- :material-database: **Level 4 - Storage and State** *(Coming Soon)*

    ---

    **For:** Both personas, stateful applications

    **Goal:** Handle persistent data

    - Understanding Volumes
    - Persistent Volumes (PV)
    - Persistent Volume Claims (PVC)
    - StorageClasses
    - Running Databases on Kubernetes

- :material-shield-lock: **Level 5 - Advanced Scheduling & Security** *(Coming Soon)*

    ---

    **For:** Platform engineers

    **Goal:** Production-ready operations

    - Resource Requests and Limits
    - Taints and Tolerations
    - Node Affinity and Pod Affinity
    - RBAC
    - Security Best Practices

- :material-eye: **Level 6 - Production Operations** *(Coming Soon)*

    ---

    **For:** SREs and platform engineers

    **Goal:** Observe, maintain, scale

    - Logging Architecture
    - Monitoring and Metrics
    - Health Checks and Probes
    - Helm Package Manager
    - Operators and Custom Resources

</div>

## Philosophy

<div class="grid cards" markdown>

-   :material-shield-check: **Production Safety**

    ---

    Real-world scenarios you'll encounter with actual clusters. Learn when commands are dangerous and why. Safety-first approach throughout.

-   :material-lightbulb-on: **Purpose-Driven Learning**

    ---

    Understand the "why," not just memorize YAML. Every resource type, every command—you'll know why it exists and when to use it.

-   :material-stairs: **Progressive Complexity**

    ---

    From Day One deployment to production operations. Start as an intimidated app developer, become a confident platform engineer.

-   :material-account-group: **Dual Personas**

    ---

    Serving both application developers (Day One - Level 2) AND platform engineers (Level 3-6). The content meets you where you are.

</div>

## Your Learning Journey

```mermaid
flowchart TD
    Start[Application Developer<br/>kubectl access<br/>Intimidated by terminal]

    DayOne[Day One: First Deployment<br/>Essential Commands]
    L12[Level 1-2: Core Skills<br/>Pods, Services, Deployments]
    L34[Level 3-4: Operations<br/>Networking, Storage]
    L56[Level 5-6: Production<br/>Security, Monitoring]

    End[Platform Engineer<br/>Production clusters<br/>Confident & capable]

    Start --> DayOne
    DayOne --> L12
    L12 --> L34
    L34 --> L56
    L56 --> End

    style Start fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style DayOne fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style L12 fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style L34 fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style L56 fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style End fill:#2f855a,stroke:#cbd5e0,stroke-width:2px,color:#fff
```


=== "Starting Point: Application Developer"

    **Day One (both paths) and Level 1-2** assume you're an application developer with:

    - Access to a real development cluster (not minikube!)
    - An application to deploy (or vendor software to install)
    - Limited command-line experience
    - No Kubernetes knowledge
    - **A team where nobody knows Kubernetes either** (you're all figuring this out)
    - **A manager who said "we're using Kubernetes now"** (but can't teach you)

    **After the shared foundation, choose your path:**

    - **[From Scratch (kubectl)](day_one/kubectl/access.md)** if you're writing YAML or learning fundamentals
    - **[Using Helm](day_one/helm/access.md)** if your CI/CD pipeline generates charts or you're deploying vendor products

    **The reality:** Your company adopted Kubernetes. Your teammates are learning alongside you. Nobody expects you to become an expert—just functional enough to deploy your code and debug when things break.

    **This site gets you unblocked.** Not certified, not expert-level—just competent enough to ship code.

=== "Destination: Platform Engineer"

    **Level 3-6** shift to platform engineering concerns:

    - Networking and security (Ingress, Network Policies, DNS)
    - Storage and state management (PV, PVC, StorageClasses)
    - Resource management and scheduling (Requests, Limits, Affinity)
    - Production operations and observability (Logging, Monitoring, Helm)

    By Level 6, you'll be running production Kubernetes clusters with confidence.

## What Makes This Different

This site takes a unique approach to teaching Kubernetes:

- **Two entry points** — Learn from scratch with YAML, or start with Helm charts (both valid!)
- **Assumes real cluster access** — You already have credentials, skip the local setup
- **Starts with deployment** — Get working first, understand architecture later
- **Acknowledges the reality** — Your team is figuring this out together, you're not alone
- **Builds confidence gradually** — Read-only commands first, then move to changes
- **Progressive personas** — Start as app developer (Day One - Level 2), grow into platform engineer (Level 3-6)
- **Production safety throughout** — Namespace awareness, command labels, safety-first approach

**Start with the shared foundation** (Overview + What Is Kubernetes?), then:

- **If you're writing YAML manifests** or learning fundamentals → [Day One: From Scratch (kubectl)](day_one/kubectl/access.md)
- **If you're working with Helm charts** from CI/CD pipelines or vendors → [Day One: Using Helm](day_one/helm/access.md)

**If you're a platform engineer** preparing to run production clusters, you'll still benefit from the progressive learning path, but can move faster through early levels.

## Connect

- Main site: [bradpenney.io](https://bradpenney.io)
- Source code: [GitHub](https://github.com/bradpenney/exploring_kubernetes)
- Related: [Exploring Enterprise Linux](https://linux.bradpenney.io) - Progressive Linux learning
