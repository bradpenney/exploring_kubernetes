# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Repository Overview

**Exploring Kubernetes** teaches Kubernetes for application developers and platform engineers through a progressive learning journey—from first deployment to production operations. It follows the same editorial standards and progressive structure as [Exploring Enterprise Linux](https://github.com/bradpenney/exploring_linux).

## Current Status: Editorial Quality Review & Restructuring

**CRITICAL**: The site is undergoing restructuring to implement a progressive learning path (Day One → Level 6) matching the Exploring Linux site's approach.

- Most navigation will be commented out in `mkdocs.yaml` pending editorial review
- Articles will be reviewed and enhanced with practice exercises, further reading, cross-links
- All content must be vetted before uncommenting in navigation
- **Goal**: Transform from topic reference into progressive skill-building journey

## Important Preferences

**Git Operations**: The user handles all git operations (commits, pushes, etc.) themselves. Do not commit or push changes.

**MkDocs Operations**: The user handles running `mkdocs serve` and `mkdocs build` themselves. Do not run these commands.

## Critical Persona Insight

**IMPORTANT**: This site has a unique audience assumption that differs from typical Kubernetes tutorials:

### Day One - Level 2: Application Developer Persona

**Who they are:**
- Application developer (not infrastructure/ops initially)
- Works at a company that uses Kubernetes
- Has kubectl access to a **real, running enterprise DEV cluster**
- Needs to deploy their application
- **Intimidated by the command line** (common for app developers!)
- Linux beginner at most—limited terminal comfort

**What they have:**
- kubectl installed and configured (IT/ops did this)
- Access to a development Kubernetes cluster (OpenShift, EKS, AKS, GKE, or on-prem)
- A containerized application that needs to be deployed
- Possibly zero Kubernetes knowledge

**What they DON'T have:**
- Their own cluster to manage (no minikube/kind/infrastructure focus)
- Deep Linux knowledge
- Ops/admin responsibilities (yet)

**Teaching approach for Day One - Level 2:**
- Assume real cluster access, NOT local cluster setup
- Focus on `kubectl` commands they'll use daily
- Emphasize deploying applications, not infrastructure
- Reassure them they won't break production by exploring
- Provide clear "read-only" vs "destructive" command labels
- Use their actual use case: "You need to deploy your app to dev"

### Level 3+: Platform Engineer Persona

Starting at Level 3, the persona can shift to include platform engineers and DevOps roles who need to:
- Understand cluster architecture deeply
- Manage networking, storage, security
- Run Kubernetes in production
- Handle ops concerns (logging, monitoring, scaling)

**If local cluster needed (Level 3+):**
- Recommend **k0s** (lightweight, production-like)
- NOT minikube or kind (too abstracted from real Kubernetes)

## Project Structure

- `docs/` - Markdown content organized by progressive learning levels
  - `day_one/` - Getting started (absolute beginner with cluster access) 🚧 Being created
  - `level_1/` - Core Primitives (Pods, Services, ConfigMaps, Namespaces) 🚧 Pending restructure
  - `level_2/` - Workload Management (Deployments, StatefulSets, DaemonSets) 🚧 Pending restructure
  - `level_3/` - Networking (Services deep dive, Ingress, Network Policies) 🚧 Pending restructure
  - `level_4/` - Storage and State (PV, PVC, StorageClass, StatefulSets) 🚧 Pending restructure
  - `level_5/` - Advanced Scheduling & Security (RBAC, taints, affinity, resources) 🚧 Pending restructure
  - `level_6/` - Production Operations (Logging, monitoring, Helm, operators) 🚧 Pending restructure
- `mkdocs.yaml` - Site configuration and navigation (will be commented during restructure)
- `pyproject.toml` - Poetry dependencies

**Old structure (being retired):**
- `core_concepts/` → Will be redistributed across Day One and Level 1
- `workloads/` → Will become Level 2
- `networking/` → Will become Level 3
- `storage/` → Will become Level 4
- `observability/` → Will be part of Level 6

## Learning Journey Structure

### Day One: Getting Started (NEW)
**Persona:** Application developer, cluster access, intimidated by CLI

**Goal:** Get from "What is Kubernetes?" to successfully deploying an application

Articles:
1. **What Is Kubernetes and Why?** - The container orchestration problem, why companies adopt it
2. **Getting kubectl Access** - Connecting to your company's cluster (kubeconfig, contexts, namespaces)
3. **Your First Deployment** - Deploy a simple application (nginx or hello-world app)
4. **Essential kubectl Commands** - The 10 commands you'll use daily (get, describe, logs, exec, delete, apply)
5. **Understanding What Happened** - What actually runs when you deploy (Pods, ReplicaSets, Services basics)

### Level 1: Core Primitives
**Persona:** App developer getting comfortable

**Goal:** Master the fundamental Kubernetes building blocks

Articles:
1. **Pods Deep Dive** - Multi-container pods, init containers, lifecycle, pod patterns
2. **Services: Connecting to Pods** - ClusterIP, NodePort, LoadBalancer, service discovery
3. **ConfigMaps and Secrets** - Externalizing configuration, environment variables, mounted configs
4. **Namespaces** - Logical separation, resource quotas, working across namespaces
5. **Labels and Selectors** - Organizing resources, targeting workloads, best practices

### Level 2: Workload Management
**Persona:** App developer deploying real applications

**Goal:** Deploy and manage applications at scale with confidence

Articles:
1. **Deployments Explained** - Declarative updates, rolling updates, rollbacks, scaling
2. **ReplicaSets Under the Hood** - How Deployments work, when you'd use ReplicaSets directly
3. **StatefulSets** - Ordered deployment, stable network IDs, when to use vs Deployments
4. **DaemonSets** - One pod per node, use cases (logging agents, monitoring, networking)
5. **Jobs and CronJobs** - Batch processing, scheduled tasks, cleanup jobs

### Level 3: Networking
**Persona:** App developer + platform engineer

**Goal:** Connect services, expose applications, control traffic

Articles:
1. **Services Deep Dive** - Service types, endpoints, kube-proxy, service mesh preview
2. **Ingress Controllers** - HTTP routing, path-based routing, TLS termination, annotations
3. **Network Policies** - Pod-to-pod communication control, security, isolation
4. **DNS and Service Discovery** - How services find each other, DNS patterns, troubleshooting
5. **Troubleshooting Networking** - Common issues, debugging tools, packet inspection

### Level 4: Storage and State
**Persona:** App developer running stateful apps + platform engineer

**Goal:** Handle persistent data and stateful applications

Articles:
1. **Understanding Volumes** - Volume types (emptyDir, hostPath, etc.), when to use each
2. **Persistent Volumes (PV)** - Cluster storage resources, lifecycle, reclaim policies
3. **Persistent Volume Claims (PVC)** - Requesting storage, binding, access modes
4. **StorageClasses** - Dynamic provisioning, storage parameters, cloud provider integration
5. **Running Databases on Kubernetes** - StatefulSets with storage, backup strategies, when NOT to use K8s

### Level 5: Advanced Scheduling & Security
**Persona:** Platform engineer

**Goal:** Control pod placement and secure the cluster

Articles:
1. **Resource Requests and Limits** - CPU/memory management, QoS classes, resource quotas
2. **Taints and Tolerations** - Node specialization, dedicated nodes, eviction
3. **Node Affinity and Pod Affinity** - Advanced scheduling, co-location, anti-affinity
4. **RBAC** - Users, service accounts, roles, role bindings, least privilege
5. **Security Best Practices** - Pod Security Standards, security contexts, network policies, admission controllers

### Level 6: Production Operations
**Persona:** Platform engineer / SRE

**Goal:** Observe, maintain, and scale production clusters

Articles:
1. **Logging Architecture** - Container logs, cluster logs, log aggregation (Fluentd, ELK, Loki)
2. **Monitoring and Metrics** - Metrics server, Prometheus, Grafana, alerting
3. **Health Checks and Probes** - Liveness, readiness, startup probes, graceful shutdown
4. **Helm Package Manager** - Charts, releases, repositories, templating, best practices
5. **Operators and Custom Resources** - CRDs, operator pattern, writing operators, ecosystem

## Common Commands

```bash
# Install dependencies
poetry install

# Serve locally (http://localhost:8000)
poetry run mkdocs serve

# Build static site (ALWAYS use --strict for link validation)
poetry run mkdocs build --strict
```

**Link Validation:** The project uses `mkdocs-htmlproofer-plugin` to validate all internal links. Always build with `--strict` flag to catch broken links.

## Content Guidelines

### Tone and Style

Articles must balance **playfulness with professionalism** and be **technically accurate** while remaining **accessible**. The goal: empower application developers AND platform engineers through progressive skill-building.

**Core Principles:**

- **Empathetic openings**: Acknowledge reader's context (new to K8s, intimidated by breaking things, need to deploy now)
- **The Analogy**: Use the "Control Room" / "Factory" analogy where appropriate to ground complex concepts
- **Professional yet engaging**: Use wit in parentheticals and asides, not emoji spam (limit to 1-3 per article, used strategically)
- **Safety-first approach**: Label destructive commands clearly, emphasize read-only exploration, explain consequences
- **Mentorship voice**: Author as experienced guide, not authority lecturing; "Let's deploy this together..." not "You must..."
- **Practical focus**: Real cluster scenarios, actual kubectl commands, hands-on YAML manifests
- **Purpose-driven**: Always explain the "why" - why this resource type, why it matters, why it's the right tool
- **Structured learning**: Build from simple to complex; scaffold concepts; clear progression between levels

**Required Sections (following Exploring Linux standard):**

1. Opening paragraph(s) - empathetic hook with real-world relevance
2. "What You'll Learn" or "What We're Doing" - set expectations
3. Core content with kubectl commands and YAML manifests
4. Safety warnings (where appropriate) - destructive operations, namespace considerations
5. Common pitfalls/"Watch Out For" section
6. Practice exercises with nested solutions (`??? question` with `??? tip "Solution"`)
7. "Quick Recap" or "Key Takeaways" table
8. "What's Next" - progression to related topics
9. Further Reading - kubectl docs, Kubernetes docs, CNCF resources, related articles

**Examples of Good Tone:**

- "You just got kubectl access to your company's dev cluster. Now what?" (empathetic acknowledgment)
- "Don't panic—`kubectl get pods` won't break anything." (reassuring, conversational)
- "Think of a Deployment like a factory manager: you tell it 'I want 3 copies of this app,' and it makes it happen." (accessible analogy)
- "**You won't break the dev cluster by exploring.**" (confidence-building assertion)

**Avoid:**

- Excessive emojis (🎯🚀✨🔥 scattered everywhere)
- Over-the-top phrases like "amazing!", "incredible!", "mind-blowing!"
- Condescending language or talking down to readers
- Assuming prior Kubernetes or Linux knowledge without explanation
- Jokey closings that undermine safety message
- Infrastructure focus in Day One/Level 1-2 (save for Level 3+)

### Kubernetes-Specific Writing Guidelines

**Command Names in Prose:**

ALWAYS wrap command names in backticks when mentioned in regular text:

- ✅ Correct: "Use `kubectl` to manage your cluster"
- ✅ Correct: "The `kubectl apply`, `kubectl get`, and `kubectl describe` commands"
- ❌ Wrong: "Use kubectl to manage your cluster"
- ❌ Wrong: "The kubectl apply, kubectl get, and kubectl describe commands"

**Exceptions:**
- Command names inside code blocks (already formatted)
- Command names in mermaid diagrams (doesn't render backticks)

**Common commands to wrap:** `kubectl`, `helm`, `minikube`, `kind`, `k9s`, `kubectx`, `kubens`, `docker`, `podman`, service names like `nginx`, `mysql`

**kubectl Commands:**

Always format commands with clear context and expected output:

```markdown
``` bash title="Get Running Pods"
kubectl get pods
# NAME                     READY   STATUS    RESTARTS   AGE
# nginx-7c5ddbdf54-x8f9p   1/1     Running   0          2m
```
```

**YAML Manifests:**

Always use titles, line numbers, and inline annotations:

```markdown
``` yaml title="deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment  # (1)!
metadata:
  name: nginx-deployment  # (2)!
spec:
  replicas: 3  # (3)!
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21  # (4)!
        ports:
        - containerPort: 80
```

1. Deployment manages a ReplicaSet, which manages Pods
2. Name must be unique within the namespace
3. Kubernetes will maintain exactly 3 running pods
4. Always pin versions in production—avoid `:latest`
```

**Command Safety Labels:**

Use these patterns consistently:

- ✅ **Safe (Read-Only):** `kubectl get`, `kubectl describe`, `kubectl logs`, `kubectl explain`
- ⚠️ **Caution (Modifies Resources):** `kubectl apply`, `kubectl edit`, `kubectl scale`, `kubectl set`
- 🚨 **DANGER (Destructive):** `kubectl delete`, `kubectl delete --all`, `kubectl drain`

**Namespace Awareness:**

Always remind users about namespace context:

```markdown
!!! warning "Namespace Context"
    These commands operate in your current namespace. Check with:
    ```bash
    kubectl config view --minify | grep namespace
    ```
    To specify a namespace: `kubectl get pods -n dev`
```

### Article Layout and Visual Structure

**Following Exploring Linux standards** (established 2026-02-06):

Articles must not be simple command references. They must teach skills with context and serve multiple audiences through varied layout.

#### Visual Elements Required

1. **Mermaid Diagrams** (when workflow/architecture exists)
   - Place at article top to show the big picture
   - Use for: workflows, resource relationships, request flows, architecture
   - **Follow Kubernetes slate color scheme:**
     - Standard Node (Slate 800): `fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff`
     - Highlighted Node (Slate 700): `fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff`
     - Darker Node (Slate 900): `fill:#1a202c,stroke:#cbd5e0,stroke-width:2px,color:#fff`
     - Accent Node (Green): `fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff`

2. **Card Grids** (for categorization)
   - Use `<div class="grid cards" markdown>` for grouping related concepts
   - Each card should explain "Why it matters" BEFORE showing commands/YAML
   - Format:
     ```markdown
     <div class="grid cards" markdown>

     -   :material-icon: **Card Title**

         ---

         **Why it matters:** Context and relevance

         ``` bash title="Command"
         kubectl get pods
         ```

         **Key insight:** What to look for

     </div>
     ```

3. **Content Tabs** (for scenarios, alternatives, or complexity levels)
   - Use `=== "Tab Name"` for different approaches or audiences
   - Examples:
     - "Beginner Approach" vs "Production Approach"
     - "ClusterIP" vs "NodePort" vs "LoadBalancer"
     - "YAML" vs "kubectl commands" vs "Helm"

#### Layout Patterns by Article Type

**Command/Resource Articles (like Pods, Services, Deployments):**

1. Opening - Empathetic hook, real-world scenario
2. Mermaid Diagram - Show resource relationships
3. Card Grid - Different resource types or approaches with "why it matters"
4. Scenario Tabs - Different use cases or configurations
5. Quick Reference - Common kubectl commands
6. Safety Warnings - What NOT to do
7. Practice Exercises - Hands-on reinforcement
8. Quick Recap - Bullet summary
9. Further Reading - Organized by category
10. What's Next - Progression path

**Hands-On Deployment Articles (like Your First Deployment):**

1. Opening - Where you are, what you'll accomplish
2. Prerequisites - What you need (kubectl access, namespace)
3. Step-by-step - Commands with explanations, YAML with annotations
4. Verification - How to check it worked
5. Exploration - Investigating what was created
6. Troubleshooting - Common issues (collapsible)
7. Practice Exercises
8. Further Reading
9. What's Next

**Architecture/Concept Articles (like Cluster Architecture):**

1. Opening - Real-world relevance
2. Conceptual explanation - The theory
3. Mermaid Diagram - Visual representation
4. Component Tabs - Detailed explanations
5. Workflow - How components interact
6. Practical examples - Seeing it in action
7. Practice Exercises
8. Further Reading
9. What's Next

#### Context Before Commands

**NEVER start with command syntax.** Always provide context first:

- ❌ Bad: "Run `kubectl apply -f deployment.yaml` to deploy"
- ✅ Good: "You need to tell Kubernetes what you want running. The `kubectl apply` command reads your YAML file and creates the resources..."

**In cards: "Why it matters" must come before commands/YAML**
**In scenarios: Goal and context must come before commands**
**In sections: Real-world relevance must precede technical details**

#### Serve Multiple Audiences

Every substantial article should provide:

1. **Visual learners** → Mermaid diagrams, card layouts
2. **Scenario-based learners** → Tabs for different use cases
3. **Reference seekers** → Quick reference tables/checklists
4. **Deep divers** → Practice exercises, comprehensive further reading
5. **Command-line intimidated** → Clear explanations, reassurance, read-only commands first

#### Further Reading Organization

**REQUIRED**: Organize Further Reading into categories:

```markdown
## Further Reading

### Official Documentation
- [Kubernetes Docs: Pods](url) - Official reference
- [kubectl Cheat Sheet](url) - Command reference

### Deep Dives
- [Article Title](url) - What you'll learn
- [CNCF Blog Post](url) - What it covers

### Related Articles
- [Next Topic](../level_2/deployments.md) - Where to go next
- [Related Topic](../level_1/services.md) - Connected concept
```

**Include when relevant:**
- Official Kubernetes documentation (kubernetes.io/docs)
- kubectl reference documentation
- CNCF (Cloud Native Computing Foundation) resources
- Vendor-specific docs (if discussing managed K8s)
- Related articles within this site

#### Pre-Publication Link Audit

**REQUIRED** before marking an article as complete:

1. **Check for broken links**
   - Grep for all `[.*](.*\.md)` links
   - Verify all linked articles are published
   - Replace links to unpublished articles with "coming soon" text or link to overview

2. **Add cross-links between published articles**
   - Each level should link forward and backward in the learning path
   - Related concepts should cross-reference (Pods ↔ Services, Deployments ↔ ReplicaSets)

3. **Add external authoritative links**
   - Official Kubernetes documentation
   - kubectl reference
   - CNCF resources

4. **Verify "Part of [Series]" callouts**
   - Each article in a level should have a callout linking to the level overview
   - Format: `!!! tip "Part of Level 1: Core Primitives"` with link to overview

### Practice Problems and Exercises

Every hands-on article should include practice exercises with **nested solutions**:

**Pattern (solutions nested inside questions):**

```markdown
## Practice Exercises

??? question "Exercise 1: Deploy Your First Pod"
    Create a Pod running nginx and verify it's running.

    **Goal:** Successfully deploy and verify a pod.

    **Hint:** Use `kubectl run` or create a YAML file.

    ??? tip "Solution"
        Using kubectl run:

        ``` bash title="Create Pod with kubectl run"
        kubectl run nginx-pod --image=nginx:1.21
        ```

        Verify it's running:

        ``` bash title="Check Pod Status"
        kubectl get pods
        # NAME         READY   STATUS    RESTARTS   AGE
        # nginx-pod    1/1     Running   0          10s

        kubectl describe pod nginx-pod
        ```

        **What you learned:** The quickest way to create a pod is `kubectl run`, but YAML gives you more control (covered next).
```

**Progression:**

- Day One: Simple, single-command exercises (get, describe, logs)
- Level 1: YAML creation, applying manifests
- Level 2-3: Multi-resource deployments, troubleshooting scenarios
- Level 4-6: Production patterns, complex configurations, operational tasks

## Quality Standards Checklist

Before uncommenting an article in `mkdocs.yaml`:

**✅ Content Quality:**

- [ ] Opening hooks with real-world relevance (app developer persona for Day One/Level 1-2)
- [ ] Clear learning objectives
- [ ] kubectl commands with actual output shown
- [ ] YAML manifests with line numbers and inline annotations
- [ ] Safety considerations addressed (read-only vs destructive)
- [ ] Common pitfalls section
- [ ] Practice exercises with nested solutions (`??? tip "Solution"` inside question)
- [ ] Key takeaways or quick recap
- [ ] What's Next progression
- [ ] Further Reading organized into categories (Official Docs, Deep Dives, Related Articles)

**✅ Technical Accuracy:**

- [ ] Commands tested against real cluster (or validated syntax)
- [ ] YAML manifests are valid and follow best practices
- [ ] Safety warnings for destructive operations
- [ ] Namespace awareness mentioned where relevant
- [ ] Version-aware (which Kubernetes versions, API versions)

**✅ Tone and Style:**

- [ ] Matches Exploring Linux professionalism
- [ ] Empathetic and accessible, not condescending
- [ ] Safety-conscious throughout
- [ ] Mentorship voice, not authority lecturing
- [ ] Emoji usage limited (1-3 max, strategic)
- [ ] No over-the-top marketing language
- [ ] Appropriate for persona (app developer vs platform engineer)

**✅ Layout and Structure:**

- [ ] Article is NOT a simple command list—teaches skill with context
- [ ] Uses visual elements appropriately (mermaid diagrams, card grids, tabs)
- [ ] Mermaid diagram at top (if workflow/architecture exists)
- [ ] Card grids explain "Why it matters" before commands/YAML
- [ ] Scenario tabs for different use cases (when applicable)
- [ ] Quick reference table/checklist for experienced users
- [ ] Serves multiple learning styles (visual, scenario-based, reference, deep-dive)
- [ ] Context provided BEFORE commands/YAML (never starts with syntax)

**✅ Formatting:**

- [ ] All code blocks have `title=` attribute
- [ ] YAML manifests use `linenums="1"` and inline annotations
- [ ] Admonitions used appropriately (tip, warning)
- [ ] Proper markdown list formatting (blank lines)
- [ ] Mermaid diagrams follow slate color scheme
- [ ] Internal links use relative paths
- [ ] External links to authoritative sources only (kubernetes.io, CNCF)
- [ ] **Command names wrapped in backticks in prose** (e.g., "Use `kubectl apply`" not "Use kubectl apply") - excludes code blocks and mermaid diagrams

**✅ Integration and Links:**

- [ ] Pre-publication link audit completed
- [ ] No links to unpublished articles
- [ ] Cross-links added between all published articles in level
- [ ] "Part of [Level]" callout with link to overview
- [ ] Referenced in "What's Next" from previous article
- [ ] Links to related articles across levels
- [ ] External links to official Kubernetes documentation included

**✅ Kubernetes-Specific:**

- [ ] Commands include namespace context where relevant
- [ ] Safety labels for commands (read-only vs destructive)
- [ ] YAML manifests follow best practices (no :latest tags in examples, resources defined, etc.)
- [ ] Appropriate for persona (app developer vs platform engineer)
- [ ] Assumes real cluster access (not minikube/kind for Day One - Level 2)

## Migration and Content Development

The site follows a **progressive levels strategy** matching Exploring Linux:

**Current Phase: Restructuring**
- Create Day One overview and articles
- Redistribute existing articles into appropriate levels
- Add missing components (exercises, further reading, cross-links)
- Comment out all navigation except reviewed articles
- Review and publish level-by-level

**Content must be vetted level-by-level** before uncommenting in navigation. Focus on quality over quantity.

**Breadth-first approach:**
1. Create structure for all levels (overview articles)
2. Outline key articles per level
3. Review and enhance Day One thoroughly
4. Move through levels systematically

## Key Differences from Exploring Linux

| Aspect | Linux Site | Kubernetes Site |
|--------|-----------|----------------|
| **Initial Persona** | New Linux user (SSH or local) | App developer with cluster access |
| **Environment** | Real server or VM | Real enterprise DEV cluster |
| **Setup Focus** | Getting access, connecting | Already have kubectl access |
| **Commands** | bash, shell commands | kubectl, YAML manifests |
| **Safety Concerns** | rm -rf, sudo commands | kubectl delete, namespace awareness |
| **Code Examples** | Bash with output comments | YAML with line numbers + kubectl with output |
| **Analogies** | Office/filesystem navigation | Factory/Control Room orchestration |
| **Later Levels** | System administration | Platform engineering/SRE |

## Final Notes

This site teaches **practical Kubernetes skills with production safety in mind**. Every article should empower readers to work confidently with Kubernetes while respecting the power and complexity of orchestration.

When in doubt, err on the side of:
- More context rather than less
- Simpler examples rather than complex
- Read-only commands before destructive ones
- More safety warnings rather than fewer
- Explaining the "why" before the "how"

The goal is to create Kubernetes users who are both **capable and cautious**—developers who can ship code safely and platform engineers who can run clusters reliably.
