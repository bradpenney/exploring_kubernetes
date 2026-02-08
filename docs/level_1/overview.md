# Level 1: Core Primitives

!!! tip "Building on Day One"
    You've deployed your first application. Now let's master the fundamental building blocks of Kubernetes.

## What You'll Master

Level 1 teaches the **core Kubernetes resources** you'll use in every deployment:

- **Pods** - The smallest deployable units, how containers actually run
- **Services** - Making pods accessible, service discovery, networking basics
- **ConfigMaps and Secrets** - Externalizing configuration, managing sensitive data
- **Namespaces** - Logical separation, resource organization
- **Labels and Selectors** - Organizing and targeting resources

These are the primitives everything else builds on. Master these, and everything else makes sense.

---

## The Articles

<div class="grid cards" markdown>

-   :material-cube: **[Pods Deep Dive](pods.md)** *(coming soon)*

    ---

    **The Foundation** — Multi-container pods, init containers, sidecar patterns, pod lifecycle

-   :material-lan: **[Services: Connecting to Pods](services.md)** *(coming soon)*

    ---

    **Networking Basics** — ClusterIP, NodePort, LoadBalancer, service discovery, DNS

-   :material-cog: **[ConfigMaps and Secrets](config_and_secrets.md)** *(coming soon)*

    ---

    **Configuration Management** — Environment variables, mounted files, sensitive data handling

-   :material-folder: **[Namespaces](namespaces.md)** *(coming soon)*

    ---

    **Logical Separation** — Multi-tenancy, resource quotas, working across namespaces

-   :material-tag: **[Labels and Selectors](labels_selectors.md)** *(coming soon)*

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

After Level 1, you'll move to **Level 2: Workload Management**, where you'll learn about Deployments, StatefulSets, DaemonSets, and how to manage applications at scale with rolling updates, rollbacks, and replica management.

But first, let's master these core primitives. Everything builds on these foundations.
