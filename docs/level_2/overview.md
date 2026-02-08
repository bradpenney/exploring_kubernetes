# Level 2: Workload Management

!!! tip "From Single Pods to Production Deployments"
    You understand Pods and Services. Now learn how Kubernetes manages workloads at scale with updates, rollbacks, and automatic recovery.

## What You'll Master

Level 2 teaches **workload controllers**—the Kubernetes resources that manage pods for you:

- **Deployments** - Declarative updates, rolling updates, rollbacks, scaling
- **ReplicaSets** - How Deployments work under the hood
- **StatefulSets** - Ordered deployment, stable identities for stateful applications
- **DaemonSets** - Running one pod per node (logging agents, monitoring)
- **Jobs and CronJobs** - Batch processing, scheduled tasks, cleanup jobs

This is where Kubernetes becomes an **orchestrator**, not just a container runner.

---

## The Articles

<div class="grid cards" markdown>

-   :material-sync: **[Deployments Explained](deployments.md)** *(coming soon)*

    ---

    **The Workhorse** — Rolling updates, rollback strategies, scaling, declarative management

-   :material-layers: **[ReplicaSets Under the Hood](replicasets.md)** *(coming soon)*

    ---

    **How It Works** — What Deployments manage, when to use ReplicaSets directly (rarely!)

-   :material-database: **[StatefulSets](statefulsets.md)** *(coming soon)*

    ---

    **Stateful Applications** — Ordered deployment, stable network IDs, persistent storage

-   :material-server-network: **[DaemonSets](daemonsets.md)** *(coming soon)*

    ---

    **Node-Level Services** — One pod per node, use cases, scheduling behavior

-   :material-clock: **[Jobs and CronJobs](jobs_cronjobs.md)** *(coming soon)*

    ---

    **Batch Processing** — One-time tasks, scheduled work, cleanup jobs, parallelism

</div>

---

## Who This Is For

**You are:**
- An application developer deploying real applications
- Comfortable with Pods, Services, and configuration
- Ready to manage applications that need updates, scaling, and high availability
- Looking for production deployment patterns

**You'll learn:**
- How to update applications without downtime (rolling updates)
- How to roll back when deployments go wrong (instant rollbacks)
- When to use Deployments vs StatefulSets vs DaemonSets (choosing the right tool)
- How to run scheduled and batch tasks (cron jobs, one-off tasks)
- How Kubernetes maintains desired state automatically (self-healing)

---

## Common Developer Scenarios at This Level

**Why you're learning this:**

=== ":material-update: Deploy New Version Without Downtime"

    **Your situation:** You fixed a bug, built a new image, and need to update production without users noticing.

    **What you'll learn:** Deployments handle rolling updates automatically—gradually replacing old pods with new ones. Zero downtime, automatic rollback if things break.

=== ":material-undo: Emergency Rollback"

    **Your situation:** You deployed v2.0 and it's broken. You need to roll back to v1.9 immediately.

    **What you'll learn:** `kubectl rollout undo` instantly reverts to the previous version. No panic, no long fixes—just undo and investigate later.

=== ":material-scale-balance: Scale for Peak Traffic"

    **Your situation:** Marketing is running a campaign. You expect 10x normal traffic. You need more pods.

    **What you'll learn:** `kubectl scale deployment/app --replicas=20` instantly scales up. Scale back down when done. Simple horizontal scaling.

=== ":material-database: Run a Database"

    **Your situation:** You need to run a database (PostgreSQL, MySQL) in Kubernetes. Regular Deployments won't work (pods need stable identities).

    **What you'll learn:** StatefulSets provide ordered deployment, stable network IDs, and persistent storage. Purpose-built for stateful applications.

=== ":material-clock: Scheduled Maintenance Tasks"

    **Your situation:** You need to clean up old data every night at 2 AM, or generate reports daily.

    **What you'll learn:** CronJobs run scheduled tasks automatically. Define the schedule, Kubernetes handles the rest.

---

## The Shift to Production Thinking

Level 2 marks a shift in mindset:

- **Day One / Level 1:** "How do I get this working?"
- **Level 2:** "How do I keep this working reliably?"

You're no longer just deploying—you're **managing applications in production**.

---

## What's Next?

After Level 2, you'll move to **Level 3: Networking**, where you'll dive deep into how services communicate, Ingress controllers for HTTP routing, and Network Policies for security.

Master these workload controllers first—they're the foundation of every production Kubernetes deployment.
