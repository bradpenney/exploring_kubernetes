# Pods: What Actually Runs Your Application

!!! tip "Part of Level 1: Core Primitives"
    This article is part of [Level 1: Core Primitives](overview.md). If you're new to Kubernetes, start with [Day One: Getting Started](../day_one/overview.md).

## Why You Need to Know About Pods

**Your actual problem:** You have an application (Java, Python, C++, whatever). It's containerized. You need it running in the dev cluster so QA can test it.

**What you'll encounter:**
- Running `kubectl get pods` (everyone's first command)
- Seeing `kubectl describe pod yourapp-xyz-abc123` in error messages
- Logs that say "pod is CrashLoopBackOff" (what?!)
- Someone asking "how many pods are running?"

**You need to know:** A pod is the thing your container actually runs inside. That's it. When you deploy to Kubernetes, you're creating pods. When you debug, you're inspecting pods. When you scale, you're adding more pods.

**The simple truth:** Your container doesn't run directly on Kubernetes. It runs inside a pod, which runs on Kubernetes. Understanding pods means understanding what's actually happening when you deploy.

---

## What Is a Pod, Really?

In most container environments (like Docker on your laptop), the container is the smallest thing you manage. In Kubernetes, there's a wrapper around your container called a **Pod**.

**Think of it like this:**
- Your container is your application code
- A pod is the box that holds your container
- Kubernetes manages pods (not containers directly)

A pod can hold one container (most common) or several containers that need to work together (less common, but useful).

---

## Why Not Just Manage Containers Directly?

---

## Why Not Just Containers?

Imagine you have two containers: a **Web Server** and a **Log Scraper**. The scraper *must* run on the same machine as the web server to read its logs.

If we managed them as separate containers, Kubernetes might schedule the Web Server on Node A and the Scraper on Node B. They wouldn't be able to talk to each other efficiently.

A Pod solves this by ensuring that all containers inside it are **guaranteed to be co-located** on the same physical or virtual machine.

---

## What Containers Share in a Pod

Containers inside a single Pod share three critical things:

### A. Shared Network (IP Address)

The Pod itself gets a single IP address. All containers in the Pod talk to each other using `localhost`.

- *Analogy:* Different rooms in the same house.

### B. Shared Storage (Volumes)

You can attach a disk to a Pod, and every container in that Pod can read and write to it.

- *Analogy:* A shared refrigerator in the kitchen.

### C. Shared Life Cycle

If the Pod is deleted, all containers inside it are killed. If the Pod is moved to a different server, the whole group moves together.

---

## The One-Container-Per-Pod Rule

Even though Pods *can* hold multiple containers, **90% of Pods only have one container.**

You should only put multiple containers in one Pod if they are tightly coupled (e.g., a "Sidecar" pattern where a helper container assists the main application). If two apps can run on different servers, they should be in different Pods.

---

## Pod Lifecycle

### Pods Are Temporary (Don't Get Attached)

Here's a critical concept: **Pods are temporary.** They die and get replaced. Don't get attached to them.

**Why this matters:**
- If a server fails, the pod running on it is deleted
- When you update your app, the old pod is killed and a new one starts
- Each new pod gets a new IP address (the old one is gone)
- This is by design—Kubernetes treats pods as replaceable

**The consequence:** You never connect directly to a pod by its IP address. You use a Service (covered in the next article) which automatically finds healthy pods for you.

**Translation for app developers:** When you deploy a new version of your code, Kubernetes doesn't update your running pods—it kills them and creates new ones with the new code. This is why deployments can update applications without downtime (gradually replacing old pods with new ones).

### Pod Phases

A Pod goes through these phases:

| Phase | Meaning |
|-------|---------|
| **Pending** | Pod accepted, but containers not yet created |
| **Running** | At least one container is running |
| **Succeeded** | All containers terminated successfully |
| **Failed** | At least one container failed |
| **Unknown** | Can't determine state (node communication issue) |

---

## Creating Your First Pod

### Simple Single-Container Pod

``` yaml title="nginx-pod.yaml" linenums="1"
apiVersion: v1  # (1)!
kind: Pod  # (2)!
metadata:
  name: nginx-pod  # (3)!
  labels:
    app: nginx  # (4)!
spec:
  containers:
  - name: nginx  # (5)!
    image: nginx:1.21  # (6)!
    ports:
    - containerPort: 80  # (7)!
```

1. API version for core resources
2. Type of resource (Pod)
3. Name must be unique within namespace
4. Labels for organizing and selecting pods
5. Container name (must be unique within pod)
6. Container image - always pin versions!
7. Port the container listens on

**Apply it:**

``` bash title="Create the Pod"
kubectl apply -f nginx-pod.yaml
# pod/nginx-pod created
```

**Check it:**

``` bash title="Get Pod Status"
kubectl get pods
# NAME        READY   STATUS    RESTARTS   AGE
# nginx-pod   1/1     Running   0          10s

kubectl describe pod nginx-pod
# Shows detailed information about the pod
```

---

## Multi-Container Pod (Sidecar Pattern)

Here's a pod with a main container and a sidecar log collector:

``` yaml title="multi-container-pod.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: web-with-logger
spec:
  containers:
  - name: web-server  # (1)!
    image: nginx:1.21
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs  # (2)!
      mountPath: /var/log/nginx

  - name: log-sidecar  # (3)!
    image: busybox
    command: ['sh', '-c', 'tail -f /logs/access.log']
    volumeMounts:
    - name: shared-logs  # (4)!
      mountPath: /logs

  volumes:
  - name: shared-logs  # (5)!
    emptyDir: {}
```

1. Main application container
2. Mount shared volume at nginx's log location
3. Sidecar container for log processing
4. Same volume mounted at different path
5. Shared volume - exists for pod lifetime

**Key insight:** Both containers can access `/var/log/nginx` (web-server) and `/logs` (sidecar) because they're the same volume.

---

## Working with Pods

### Essential kubectl Commands

``` bash title="Pod Operations"
# Get all pods in current namespace
kubectl get pods

# Get pods with more details
kubectl get pods -o wide
# Shows IP, NODE, NOMINATED NODE, READINESS GATES

# Describe a specific pod
kubectl describe pod nginx-pod

# View pod logs
kubectl logs nginx-pod

# For multi-container pods, specify container
kubectl logs web-with-logger -c web-server
kubectl logs web-with-logger -c log-sidecar

# Execute command in pod
kubectl exec nginx-pod -- ls /usr/share/nginx/html

# Get interactive shell
kubectl exec -it nginx-pod -- /bin/bash

# Delete pod
kubectl delete pod nginx-pod
# Or
kubectl delete -f nginx-pod.yaml
```

!!! warning "Deleting Pods"
    Deleting a pod is permanent. The pod and its data are gone. If you're using Deployments (Level 2), a new pod will automatically be created to replace it.

---

## Safety Check: What's Safe vs. What's Destructive

**As an application developer in your namespace, here's what's safe:**

=== ":material-check-circle: Safe (Read-Only)"

    These commands **only read information** - use them freely:

    | Command | What It Does | Why It's Safe |
    |---------|-------------|---------------|
    | `kubectl get pods` | List pods | Just looking—no changes |
    | `kubectl describe pod name` | Inspect pod details | Read-only—shows config, events, status |
    | `kubectl logs pod-name` | View application logs | Reading logs—doesn't modify anything |
    | `kubectl exec pod-name -- command` | Run command in pod | Depends on the command, but reading is safe |

    **Bottom line:** Exploring and investigating never breaks anything.

=== ":material-alert: Caution (Modifies Resources)"

    These commands **change things** - use with namespace awareness:

    | Command | What It Does | Why You Need Care |
    |---------|-------------|-------------------|
    | `kubectl apply -f pod.yaml` | Create/update pod | Creates resources—stay in YOUR namespace |
    | `kubectl edit pod name` | Modify running pod | Changes live configuration |
    | `kubectl delete pod name` | Remove pod | Permanent deletion—make sure it's yours |

    **Bottom line:** These work, but double-check your namespace first.

!!! tip "You Won't Break Production"
    **Remember:**
    - You're in the **dev cluster** (completely separate from production)
    - You only have access to **your namespace** (isolated from other teams)
    - Even if you delete all your pods, **they'll recreate automatically** (if using Deployments)
    - Worst case: you redeploy and learn something

    **Exploring is safe.** The read-only commands (get, describe, logs) can't hurt anything. Use them liberally to build confidence.

---

## Practice Problems

??? question "Practice Problem 1: Shared IP"
    Container A runs on port 8080. Container B runs on port 9090. They are in the same Pod. How does Container B send a message to Container A?

    ??? tip "Solution"
        It sends the message to **`localhost:8080`**.

        Because containers in a Pod share the same network namespace, they act as if they are running on the same local machine.

??? question "Practice Problem 2: Scaling"
    You have a Web Server Pod and you want to handle more traffic. Should you add a second "Web Server" container inside the *same* Pod, or should you create a *second Pod*?

    ??? tip "Solution"
        **Create a second Pod.**

        Scaling in Kubernetes is done by creating more instances of Pods (Horizontal Scaling), not by stuffing more containers into one Pod. This allows Kubernetes to spread the load across different physical servers.

        This is exactly what Deployments do (covered in Level 2).

??? question "Practice Problem 3: Pod Failure"
    A pod crashes and enters the "Failed" state. Will Kubernetes automatically restart it?

    ??? tip "Solution"
        **It depends on how the pod was created.**

        - **Standalone pod** (created directly with `kubectl apply`): **NO** - it stays failed
        - **Pod managed by Deployment/ReplicaSet**: **YES** - a new pod is created automatically

        This is why you almost never create pods directly in production—you use Deployments (Level 2).

---

## Key Takeaways

| Feature | Description |
| :--- | :--- |
| **Atomic** | The smallest unit K8s manages |
| **Co-located** | Containers always run on the same node |
| **Shared IP** | One IP address per Pod, containers use localhost |
| **Shared Storage** | Volumes can be mounted in multiple containers |
| **Ephemeral** | Pods are temporary, IPs change when recreated |
| **Sidecar** | A helper container inside a Pod |

---

## Further Reading

### Official Documentation

- [Kubernetes Docs: Pods](https://kubernetes.io/docs/concepts/workloads/pods/) - Official pod reference
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) - Command reference
- [Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) - Phases and conditions

### Deep Dives

- [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/) - Sidecar patterns
- [Multi-Container Pod Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns/) - When to use multi-container pods

### Related Articles

- [Services: Connecting to Pods](services.md) - How to access pods reliably
- **Deployments Explained** - Managing pods at scale (coming in Level 2)
- [ConfigMaps and Secrets](config_and_secrets.md) - Configuring pods

---

## What's Next?

You understand pods—the atomic unit of Kubernetes. Next, learn how to access them reliably with **[Services: Connecting to Pods](services.md)**. Services provide stable networking for ephemeral pods.

---

The Pod is the foundation of the Kubernetes object model. By grouping containers into logical "units of service," Kubernetes simplifies the complex task of managing distributed systems across a cluster of machines.
