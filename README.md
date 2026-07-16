# Exploring Kubernetes

**From first deployment to production operations.**

A progressive learning site for Kubernetes—from application developer deploying your first app to platform engineer running production clusters. Built with [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/).

🌐 **Live site:** [k8s.bradpenney.io](https://k8s.bradpenney.io)

---

## Who This Is For

### Starting Point: Application Developer

**Day One and Level 1-2** are designed for application developers who:

- Just received `kubectl` access to their company's dev cluster
- Need to deploy an application but don't know Kubernetes yet
- May be intimidated by the command line
- Are learning because their company adopted K8s (not by choice!)
- Have a team where nobody knows Kubernetes either

**The reality:** Your company adopted Kubernetes. Your teammates are learning alongside you. This site gets you unblocked—not certified, just functional enough to deploy your code and debug when things break.

### Destination: Platform Engineer

**Level 3-6** shift to platform engineering concerns:

- Networking and security (Ingress, Network Policies, DNS)
- Storage and state management (PV, PVC, StorageClasses)
- Resource management and scheduling (Requests, Limits, Affinity)
- Production operations and observability (Logging, Monitoring, Helm)

By Level 6, you'll be running production Kubernetes clusters with confidence.

---

## Learning Path

```
Application Developer                           Platform Engineer
(kubectl access, intimidated)                   (production clusters, confident)
         │                                               │
         ▼                                               ▼
┌─────────────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│   Day One       │──▶│ Level 1-2│──▶│ Level 3-4│──▶│ Level 5-6│
│ First Deployment│   │Core Skills   │Operations│   │Production│
└─────────────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Day One - Getting Started

**For:** Application developers with kubectl access to a real cluster

**Goal:** Deploy your first application successfully and understand what you're doing

**Articles:**
- What Is Kubernetes? ✅
- Getting kubectl Access ✅
- Your First Deployment ✅
- Essential kubectl Commands ✅
- Understanding What Happened ✅

### Level 1 - Core Primitives

**For:** App developers getting comfortable

**Topics:** Pods ✅, Services ✅, ConfigMaps *(coming soon)*, Secrets *(coming soon)*, Namespaces *(coming soon)*, Labels & Selectors *(coming soon)*

### Level 2 - Workload Management *(Coming Soon)*

**For:** App developers deploying real applications

**Topics:** Deployments, ReplicaSets, StatefulSets, DaemonSets, Jobs & CronJobs

### Level 3 - Networking *(Coming Soon)*

**For:** App developers + Platform engineers

**Topics:** Services Deep Dive, Ingress, Network Policies, DNS, Troubleshooting

### Level 4 - Storage and State *(Coming Soon)*

**For:** Both personas, stateful applications

**Topics:** Volumes, Persistent Volumes, StorageClasses, Databases on K8s

### Level 5 - Advanced Scheduling & Security *(Coming Soon)*

**For:** Platform engineers

**Topics:** Resource Requests/Limits, Taints & Tolerations, Affinity, RBAC, Security

### Level 6 - Production Operations *(Coming Soon)*

**For:** SREs and platform engineers

**Topics:** Logging, Monitoring, Health Checks, Helm, Operators

---

## Philosophy

### 🛡️ Production Safety

Real-world scenarios you'll encounter with actual clusters. Learn when commands are dangerous and why. Safety-first approach throughout.

### 💡 Purpose-Driven Learning

Understand the "why," not just memorize YAML. Every resource type, every command—you'll know why it exists and when to use it.

### 📈 Progressive Complexity

From Day One deployment to production operations. Start as an intimidated app developer, become a confident platform engineer.

### 👥 Dual Personas

Serving both application developers (Day One - Level 2) AND platform engineers (Level 3-6). The content meets you where you are.

---

## What Makes This Different

This site takes a unique approach to teaching Kubernetes:

- **Assumes real cluster access** — You already have kubectl credentials, skip the local setup
- **Starts with deployment** — Get working first, understand architecture later
- **Acknowledges the reality** — Your team is figuring this out together, you're not alone
- **Builds confidence gradually** — Read-only commands first, then move to changes
- **Progressive personas** — Start as app developer, grow into platform engineer
- **Production safety throughout** — Namespace awareness, command labels, safety-first approach

---

## Tech Stack

- **[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)** - Documentation site generator
- **Python 3.12+** - Runtime
- **Poetry** - Dependency management
- **Markdown** - Content format with extensions for tabs, admonitions, diagrams

---

## Local Development

### Prerequisites

- Python 3.12+
- [Poetry](https://python-poetry.org/) for dependency management

### Setup

```bash
# Install dependencies
poetry install

# Serve locally (http://localhost:8000)
poetry run mkdocs serve

# Build static site
poetry run mkdocs build --strict
```

The `--strict` flag validates all internal links and fails the build on errors.

---

## Project Structure

```
exploring_kubernetes/
├── docs/
│   ├── day_one/          # Getting started articles
│   ├── level_1/          # Core primitives
│   ├── level_2/          # Workload management
│   ├── level_3/          # Networking
│   ├── level_4/          # Storage
│   ├── level_5/          # Advanced scheduling & security
│   ├── level_6/          # Production operations
│   ├── images/           # Site images and diagrams
│   └── index.md          # Homepage
├── mkdocs.yaml           # Site configuration
├── pyproject.toml        # Poetry dependencies
└── README.md             # This file
```

---

## Where This Fits in the bradpenney.io Network

Kubernetes sits above containers and below GitOps in the stack — it orchestrates what one site
teaches and gets driven by the pipeline another one automates:

- **[containers.bradpenney.io](https://containers.bradpenney.io)** - what a Pod's containers
  actually are, and what `kubelet` is talking to through the CRI. This site orchestrates
  containers; it doesn't teach what one is — that site does.
- **[linux.bradpenney.io](https://linux.bradpenney.io)** - every node is a Linux machine, and
  the namespaces/cgroups a container runtime constructs are Linux kernel mechanisms explained
  there in depth.
- **[gitops.bradpenney.io](https://gitops.bradpenney.io)** - how production actually deploys to
  a cluster: Flux and OCI artifacts, not `kubectl apply` run by hand.
- [Exploring Computer Science](https://cs.bradpenney.io) - CS theory for working engineers
- [Exploring Python](https://python.bradpenney.io) - Python from basics to advanced, including
  Kubernetes automation and client libraries

The short version: containers explains the unit Kubernetes schedules, Linux explains the node
underneath it, and GitOps explains how a change actually gets there in production.

---

## Connect

- **Main site:** [bradpenney.io](https://bradpenney.io)
- **Live documentation:** [k8s.bradpenney.io](https://k8s.bradpenney.io)
- **Source code:** [github.com/bradpenney/exploring_kubernetes](https://github.com/bradpenney/exploring_kubernetes)

---

## License

Content is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). You're free to share and adapt the material with attribution.

---

**Status:** 🚧 Under active development. Day One articles being published progressively.
