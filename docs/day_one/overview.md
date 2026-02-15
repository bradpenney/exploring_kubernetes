---
title: Getting Started with Kubernetes - Day One Guide
description: A developer-focused introduction to Kubernetes. Learn what Kubernetes is and how to deploy your first application with confidence.
---
# Day One: Getting Started with Kubernetes

!!! tip "Your First Day with Kubernetes"
    You just got access to your company's dev cluster. You have an application to deploy. You're probably thinking: *What is this? How do I use it? Will I break something?*

## Welcome to Kubernetes

This is your first time working with Kubernetes, and you're probably feeling one of two ways:

- **Curious:** "Everyone talks about Kubernetes. Time to see what it's about!"
- **Nervous:** "I just got cluster access and need to deploy my app. No idea where to start..."

**Both are valid. You're in the right place.**

This guide is for application developers who need to deploy their code to a Kubernetes cluster. Not infrastructure engineers (yet)—just developers who need to ship their applications.

## What You'll Learn

By the end of Day One, you'll know how to:

- **Understand Kubernetes** - What it is, why your company uses it, what problem it solves
- **Connect to your cluster** - Authentication, contexts, and namespaces
- **Deploy your first application** - From container image to running pods
- **Use essential commands** - The tools you'll use daily for deployment and troubleshooting
- **Understand what you just did** - What actually happens when you deploy
- **Explore safely** - Read-only vs. destructive operations, troubleshooting basics

## Your First Day: The Journey

Day One offers two paths depending on how your team works with Kubernetes:

- **kubectl Path (From Scratch)**: Write YAML manifests manually and deploy with `kubectl apply` commands
- **Helm Path (Package Manager)**: Use packaged charts and deploy with `helm install` commands

Both paths follow a similar pattern and reach the same destination—a deployed application. The difference is in the tools you use. Choose the journey that matches how your team deploys applications:

=== "kubectl Journey (From Scratch)"

    ```mermaid
    flowchart TD
        Start[You receive kubectl<br/>credentials]
        Connect[Connect to cluster<br/>kubectl config]
        Verify[Verify access<br/>kubectl get nodes]
        Check[Check your namespace<br/>kubectl config view]
        Deploy[Deploy your app<br/>kubectl apply -f app.yaml]
        Watch[Watch deployment<br/>kubectl get pods]
        Logs[Check logs<br/>kubectl logs pod-name]
        Success[App running!<br/>Confidence built]

        Start --> Connect
        Connect --> Verify
        Verify --> Check
        Check --> Deploy
        Deploy --> Watch
        Watch --> Logs
        Logs --> Success

        style Start fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Connect fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Verify fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Check fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Deploy fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Watch fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Logs fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Success fill:#2f855a,stroke:#cbd5e0,stroke-width:2px,color:#fff
    ```

    **The kubectl path:** Write YAML manifests by hand, deploy with `kubectl apply`, understand every building block from the ground up.

=== "Helm Journey (Package Manager)"

    ```mermaid
    flowchart TD
        Start[You receive kubectl<br/>credentials]
        InstallHelm[Install Helm CLI<br/>brew install helm]
        Connect[Connect to cluster<br/>helm list]
        Repo[Add chart repository<br/>helm repo add]
        Deploy[Deploy your app<br/>helm install my-app chart/]
        Watch[Watch deployment<br/>helm status my-app]
        Logs[Check logs<br/>kubectl logs pod-name]
        Success[App running!<br/>Confidence built]

        Start --> InstallHelm
        InstallHelm --> Connect
        Connect --> Repo
        Repo --> Deploy
        Deploy --> Watch
        Watch --> Logs
        Logs --> Success

        style Start fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style InstallHelm fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Connect fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Repo fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Deploy fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Watch fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Logs fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
        style Success fill:#2f855a,stroke:#cbd5e0,stroke-width:2px,color:#fff
    ```

    **The Helm path:** Use packaged charts (from CI/CD or vendors), deploy with `helm install`, customize through values files rather than raw YAML.

## Understanding Your Context

This guide is written for a specific situation—one that's incredibly common but rarely addressed directly in Kubernetes tutorials.

<div class="grid cards" markdown>

-   :material-account-check: **Your Setup**

    ---

    **You have:**

    - A containerized application (Docker image in a registry)
    - Cluster access configured (IT/ops gave you credentials)
    - Access to a development Kubernetes cluster (real cluster, not minikube)
    - A namespace you can deploy to
    - Basic understanding that containers exist
    - Either `kubectl` for manual YAML or Helm charts from your CI/CD pipeline

    **You don't have:**

    - Deep Kubernetes knowledge (that's why you're here!)
    - Infrastructure or ops responsibilities (yet)
    - A cluster to manage (you're using an existing one)
    - Command line confidence (we'll fix that)

    **You are:**

    - An application developer
    - Possibly intimidated by the terminal
    - Responsible for getting your app running in dev/staging
    - Learning Kubernetes because your company adopted it

-   :material-account-group: **The Team Reality**

    ---

    Let's be honest about what's probably happening at your company:

    - **Your manager said "we're using Kubernetes now"** but doesn't know how it works either
    - **Your teammates are figuring this out alongside you** - nobody on the team is a Kubernetes expert
    - **The platform/ops team gave you access** but can't hand-hold through every deployment
    - **Nobody expects you to become a Kubernetes expert** - just functional enough to deploy your app
    - **You're learning by necessity, not choice** - this wasn't on your career roadmap, but here you are

    **This guide gets you unblocked.** Not certified, not expert-level—just competent enough to ship code and debug when things break.

    **You're not alone.** Thousands of application developers are in the exact same situation: handed cluster credentials and told "figure it out." This guide is for all of you.

</div>

---

## Common Developer Scenarios

Why are you really here? Probably one of these situations (examples show both paths):

=== ":material-rocket-launch: Deploy My App to Dev"

    **Your situation:** QA needs to test your latest build. You have a Docker image. Now what?

    === "kubectl Path"

        **What you need to learn:**

        - How to write a deployment YAML (or modify the existing one)
        - `kubectl apply -f deployment.yaml` - push your changes
        - `kubectl get pods` - verify it's running
        - `kubectl rollout status` - watch the deployment happen

    === "Helm Path"

        **What you need to learn:**

        - How to customize your chart's `values.yaml` file
        - `helm install my-app ./chart` - deploy your application
        - `kubectl get pods` - verify it's running (same as kubectl path!)
        - `helm status my-app` - check deployment status

    **Day One gets you here.** By the end, you'll deploy confidently.

=== ":material-bug: Check Logs When Bug Reported"

    **Your situation:** QA says "it's broken in dev" or "I'm seeing weird errors." You need to investigate.

    **What you need to learn (same for both paths):**

    - `kubectl get pods` - find your pod name
    - `kubectl logs pod-name-xyz` - see what's failing
    - `kubectl logs -f pod-name-xyz` - follow logs live
    - `kubectl describe pod pod-name-xyz` - see why it crashed

    !!! info "Both Paths Use kubectl for Debugging"
        Even Helm users rely on `kubectl` commands for troubleshooting. Helm deploys resources, but you inspect and debug them with kubectl.

    **Essential Commands teaches this.** Debugging becomes routine.

=== ":material-cog: Update Environment Variable"

    **Your situation:** Need to point to a different database, change an API key, or update a config value.

    === "kubectl Path"

        **What you need to learn:**

        - ConfigMaps and Secrets (where config lives)
        - How to update them with `kubectl edit` or `kubectl apply`
        - How to restart your deployment to pick up changes
        - How to verify the new value is being used

    === "Helm Path"

        **What you need to learn:**

        - How values flow from `values.yaml` to ConfigMaps/Secrets
        - Update `values.yaml` with new configuration
        - Run `helm upgrade my-app ./chart` to apply changes
        - How to verify the new value is being used

    **Level 1: ConfigMaps and Secrets covers this.** Configuration management made clear.

=== ":material-undo: Roll Back Bad Deploy"

    **Your situation:** You deployed something and now dev is broken. You need to undo it. Fast.

    === "kubectl Path"

        **What you need to learn:**

        - `kubectl rollout undo deployment/yourapp` - instant rollback
        - `kubectl rollout history` - see what versions exist
        - How to verify you're back to the working version

    === "Helm Path"

        **What you need to learn:**

        - `helm rollback my-app` - instant rollback to previous release
        - `helm history my-app` - see all release versions
        - How to verify you're back to the working version

    **Day One and Level 2 cover this.** Rollbacks become your safety net.

=== ":material-scale-balance: Scale Up for Load Testing"

    **Your situation:** Performance team wants to load test. You need more instances running.

    === "kubectl Path"

        **What you need to learn:**

        - `kubectl scale deployment/yourapp --replicas=10` - simple scaling
        - How to verify all replicas are running
        - How to scale back down when done

    === "Helm Path"

        **What you need to learn:**

        - Update `replicas: 10` in your `values.yaml`
        - Run `helm upgrade my-app ./chart` to apply new scale
        - How to verify all replicas are running
        - Update values and upgrade again to scale back down

    **Level 2: Deployments covers scaling.** It's easier than you think.

**Sound familiar?** These are the real-world tasks you'll master. Not theoretical Kubernetes architecture—practical skills for shipping code.

---

## Which Day One Path Are You On?

Not sure which path to choose? Use these criteria to decide:

<div class="grid cards" markdown>

-   :material-console: **The kubectl Path**

    ---

    **Choose this if:**

    - You're learning Kubernetes from the ground up
    - You'll be writing your own YAML manifests
    - Your team deploys with `kubectl apply`
    - You want to understand every building block

-   :material-dharmachakra: **The Helm Path (Coming Soon)**

    ---

    **Choose this if:**

    - Your CI/CD pipeline generates Helm charts for you
    - You need to deploy vendor charts (PostgreSQL, Redis, etc.)
    - Your team standardizes on Helm for all deployments
    - You were told "just run `helm install`"

</div>

---

## The Articles

### Shared Foundation
- :material-help-circle: **[What Is Kubernetes and Why?](what_is_kubernetes.md)** - Understand the problem Kubernetes solves.

=== "kubectl Path (From Scratch)"

    <div class="grid cards" markdown>

    - :material-connection: **[Getting kubectl Access](kubectl/access.md)**
      Connect to your company's cluster and verify access.

    - :material-rocket-launch: **[Your First Deployment](kubectl/first_deploy.md)**
      Deploy a simple application from a YAML file.

    - :material-console: **[Essential kubectl Commands](kubectl/commands.md)**
      The 10 commands you'll use every single day.

    - :material-lightbulb: **[Understanding What Happened](kubectl/understanding.md)**
      Learn about Pods, ReplicaSets, and how they fit together.

    </div>

=== "Helm Path (Package Manager) 🚧"

    !!! info "Coming Soon"
        The Helm path articles are currently in development. In the meantime, the kubectl path will teach you the foundational Kubernetes concepts that apply to both approaches.

    **Planned articles:**

    <div class="grid cards" markdown>

    - :material-connection: **Getting Helm Access**

      Install Helm CLI and connect to your cluster using existing kubectl credentials.

    - :material-dharmachakra: **Your First Helm Deployment**

      Deploy your first chart from CI/CD pipeline or vendor repository.

    - :material-console: **Essential Helm Commands**

      Master `helm install`, `helm upgrade`, `helm rollback`, and release management.

    - :material-lightbulb: **Understanding What Helm Created**

      See how Helm translates values into Kubernetes resources under the hood.

    </div>

!!! success "kubectl Path Complete!"
    The kubectl path through Day One is complete! Start with **[What Is Kubernetes?](what_is_kubernetes.md)** and work through the five articles to complete your first day with Kubernetes.

    The Helm path articles are in development and will be released soon.

---

## The Philosophy

Throughout Day One, we emphasize **safety and confidence**. You're working on a shared development cluster:

!!! tip "Read-Only Commands Are Safe"

    These commands won't hurt anything—explore freely:

    - `kubectl get` - See what's running
    - `kubectl describe` - Inspect resources
    - `kubectl logs` - Read application logs
    - `kubectl explain` - Get documentation

!!! warning "Some Commands Need Care"

    These commands modify or delete resources—use with namespace awareness:

    - `kubectl delete` - Removes resources (yours, hopefully!)
    - `kubectl apply` - Creates/updates resources (stay in your namespace!)
    - `kubectl edit` - Modifies running resources

    **We'll clearly label which is which** throughout the articles.

!!! info "Your Namespace Is Your Sandbox"

    Your company likely gave you a namespace (like `dev-yourname` or `team-frontend-dev`). Think of it as your personal workspace:

    - **You can create/delete freely in YOUR namespace**
    - **Other namespaces are off-limits** (unless you're explicitly given access)
    - **Production namespaces require extra caution** (healthy respect for production!)

    We'll teach you to always check which namespace you're working in.

---

## Our Teaching Approach

Day One focuses on **practical, immediate needs**—the skills you need to deploy your application today.

<div class="grid cards" markdown>

-   :material-lightbulb-on: **We Teach**

    ---

    **The "Why"** — Why Kubernetes exists, why this command, why it matters

    **Real Scenarios** — "You need to deploy your app to dev" not abstract theory

    **Finding Answers** — How to help yourself when stuck (`kubectl explain`, `describe`, `logs`)

    **Safety Habits** — Namespace awareness, read-only vs destructive commands

</div>

---

## What's Next?

Once you're comfortable deploying applications and troubleshooting them, you're ready for **Level 1: Core Primitives**. That's where you'll dive deeper into Pods, Services, ConfigMaps, and the fundamental building blocks of Kubernetes—regardless of whether you deployed them with kubectl or Helm.

But first, let's get you deploying applications. That's what Day One is all about.

---

## Ready?

Start with **[What Is Kubernetes and Why?](what_is_kubernetes.md)** to understand what you're working with and why it matters.

Your journey from intimidated app developer to confident Kubernetes user starts here. Let's get you deploying.
