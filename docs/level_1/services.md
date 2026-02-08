# Services: How Your Apps Talk to Each Other

!!! tip "Part of Level 1: Core Primitives"
    This article is part of [Level 1: Core Primitives](overview.md). Make sure you've read [Pods: What Actually Runs](pods.md) first.

## Why You Need to Know About Services

**Your actual problem:** You have a frontend app that needs to talk to a backend API. Or your app needs to connect to a database. Or you need to expose your application so QA can test it.

**What you'll encounter:**
- YAML files with `kind: Service` in them (what is this?)
- Environment variables like `DATABASE_HOST: postgres-service:5432` (where did that name come from?)
- URLs like `http://backend-svc.default.svc.cluster.local` (that's not a normal URL!)
- Someone saying "create a service to expose your pods" (expose to what?)

**You need to know:** A Service is how pods find each other in Kubernetes. It's a stable name and IP address that points to your pods, even as they die and get replaced.

**The problem Services solve:** Pods are temporary and their IP addresses change constantly. You can't hardcode `10.42.0.1` in your frontend because that IP will be different tomorrow. Services give you a permanent name like `backend-api` that always works.

---

## The Networking Problem

In the [Pods article](pods.md), we learned that pods are temporary—they die and get replaced frequently. Every time a new pod starts, it gets a **new IP address**.

**Here's the problem:**
- Your frontend pod needs to talk to your backend pods
- Backend pods restart (deployments, crashes, updates)
- Every restart means new IP addresses
- Your frontend can't keep updating IP addresses manually

**The solution:** A Service provides a **single, permanent IP address** and a **permanent DNS name** for a group of pods. The pods can die and be replaced 1,000 times—the Service's IP and name never change.

---

## The Stable Entry Point

A Service is a Kubernetes object that provides a **single, permanent IP address** and a **permanent DNS name** for a group of Pods.

- The Pods can die and be replaced 1,000 times.
- The Service's IP never changes.
- The Service acts as a **Load Balancer**, automatically spreading traffic across all the healthy Pods in that group.

**Think of it like a phone number for a company:**
- Employees (pods) come and go
- The phone number (service) stays the same
- Calls are routed to available employees automatically

---

## How Services Find Pods: Labels and Selectors

How does a Service know which Pods belong to it? It uses **Labels**.

1. We give our Pods a label: `app: backend`.
2. We tell our Service to look for a **Selector**: `app: backend`.

The Service constantly scans the cluster for any Pod matching that label. If a Pod dies, the Service removes it from the list. If a new Pod is born, the Service adds it.

``` mermaid
graph LR
    Svc[Service<br/>selector: app=backend] -.->|watches| P1[Pod<br/>label: app=backend]
    Svc -.->|watches| P2[Pod<br/>label: app=backend]
    Svc -.->|watches| P3[Pod<br/>label: app=backend]
    P4[Pod<br/>label: app=frontend] -.->|ignored| Svc

    style Svc fill:#2d3748,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style P1 fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style P2 fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style P3 fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style P4 fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
```

---

## Service Types

Where do you want the traffic to come from?

### A. ClusterIP (The Default)

The Service is only visible **inside the cluster**.

- **Use Case:** Your Backend database. You don't want the internet to see it, only your Frontend.
- **IP:** Gets a cluster-internal IP (e.g., `10.96.0.1`)
- **Access:** Only from within the cluster

### B. NodePort

Exposes the Service on a specific port on **every node's IP**.

- **Use Case:** Testing, or small clusters where you don't have a cloud load balancer
- **Port Range:** 30000-32767
- **Access:** `<NodeIP>:<NodePort>` from outside cluster

### C. LoadBalancer

Integrates with your cloud provider (AWS, Azure, Google) to create a **Real External Load Balancer**.

- **Use Case:** Exposing your website to the public internet
- **Cloud:** Requires cloud provider support
- **Cost:** Usually incurs cloud provider costs

---

## Creating Services

### ClusterIP Service (Internal)

``` yaml title="backend-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: backend-svc  # (1)!
spec:
  type: ClusterIP  # (2)!
  selector:
    app: backend  # (3)!
  ports:
  - protocol: TCP
    port: 80  # (4)!
    targetPort: 8080  # (5)!
```

1. Service name - used for DNS (e.g., `http://backend-svc`)
2. ClusterIP is the default type
3. Selects pods with label `app: backend`
4. Port the service listens on
5. Port the pods are listening on

**Apply it:**

``` bash title="Create the Service"
kubectl apply -f backend-service.yaml
# service/backend-svc created

kubectl get services
# NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# backend-svc    ClusterIP   10.96.45.123    <none>        80/TCP    5s
```

**Test it from another pod:**

``` bash title="Test Service Connectivity"
kubectl run test-pod --image=busybox -it --rm -- sh
# Inside the pod:
wget -O- http://backend-svc
# This will connect to one of the backend pods
```

---

### NodePort Service (External Access)

``` yaml title="frontend-nodeport.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  type: NodePort  # (1)!
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080  # (2)!
```

1. NodePort type exposes on all node IPs
2. Optional - if omitted, K8s assigns a random port (30000-32767)

**Access it:**

``` bash title="Get Node IPs and Access Service"
kubectl get nodes -o wide
# Get any node's EXTERNAL-IP

# Access service at:
curl http://<NODE-IP>:30080
```

---

### LoadBalancer Service (Cloud)

``` yaml title="web-loadbalancer.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: LoadBalancer  # (1)!
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

1. LoadBalancer provisions external load balancer (requires cloud provider)

**Check status:**

``` bash title="Get LoadBalancer IP"
kubectl get service web-svc
# NAME      TYPE           CLUSTER-IP     EXTERNAL-IP       PORT(S)        AGE
# web-svc   LoadBalancer   10.96.10.123   34.123.45.67      80:31234/TCP   2m

# EXTERNAL-IP is the public IP address
# Access at: http://34.123.45.67
```

!!! warning "Cloud Provider Required"
    LoadBalancer services only work on cloud providers (GKE, EKS, AKS) or with on-prem load balancer solutions like MetalLB.

---

## Service Discovery with DNS

Kubernetes has a built-in DNS service. When you create a Service named `backend-svc` in namespace `default`, it's automatically registered as:

- **Short name:** `backend-svc` (within same namespace)
- **Fully qualified:** `backend-svc.default.svc.cluster.local`

**From your application code:**

```python
# Python example
import requests
response = requests.get('http://backend-svc')  # Just use the service name!
```

```javascript
// Node.js example
const response = await fetch('http://backend-svc');
```

**No IP addresses, no configuration—just the service name.**

---

## Working with Services

### Essential kubectl Commands

``` bash title="Service Operations"
# Get all services
kubectl get services
# Short form:
kubectl get svc

# Get detailed information
kubectl describe service backend-svc

# Get service endpoints (actual pod IPs)
kubectl get endpoints backend-svc

# Delete service
kubectl delete service backend-svc
# Or
kubectl delete -f backend-service.yaml
```

### Debugging Service Connectivity

``` bash title="Test Service Resolution"
# Create a test pod
kubectl run test --image=busybox -it --rm -- sh

# Inside the pod, test DNS:
nslookup backend-svc
# Should return the service's cluster IP

# Test connectivity:
wget -O- http://backend-svc
# Should get response from backend pod
```

---

## Practice Problems

??? question "Practice Problem 1: DNS Discovery"
    Your backend service is named `auth-svc`. How does your frontend code send a request to it?

    A. Use the IP address `10.42.0.5`.
    B. Use the hostname `http://auth-svc`.
    C. Use the environment variable `K8S_BACKEND_IP`.

    ??? tip "Solution"
        **B. Use the hostname `http://auth-svc`.**

        Kubernetes has a built-in DNS service. Any Service you create is automatically registered by its name. This allows your code to stay simple: just talk to the service name, and Kubernetes handles the routing to the current Pod IPs.

??? question "Practice Problem 2: Selector Match"
    A Service has a selector `app: web`. You have two Pods:

    - Pod A: `labels: {app: web, tier: frontend}`
    - Pod B: `labels: {app: db, tier: database}`

    Which Pod will receive traffic from the Service?

    ??? tip "Solution"
        **Pod A.**

        A Service matches any Pod that contains the labels listed in its selector. Since Pod A has `app: web`, it matches. Pod B does not have that label, so it is ignored.

??? question "Practice Problem 3: Service Type Selection"
    You need to expose a web application to the internet. Your cluster is running on GKE (Google Kubernetes Engine). Which service type should you use?

    ??? tip "Solution"
        **LoadBalancer**

        On a cloud provider like GKE, LoadBalancer type will automatically provision a Google Cloud Load Balancer with a public IP address. This is the recommended way to expose services to the internet on cloud platforms.

        NodePort would technically work but requires knowing node IPs and managing firewall rules manually.

---

## Key Takeaways

| Feature | Description |
| :--- | :--- |
| **Stable IP** | Provides a permanent address for ephemeral pods |
| **Load Balancing** | Spreads traffic across multiple pods |
| **Labels/Selectors** | The logic that connects Services to Pods |
| **DNS** | Provides human-readable names for services |
| **ClusterIP** | Internal-only access (default) |
| **NodePort** | External access via node IPs |
| **LoadBalancer** | External access via cloud load balancer |

---

## Further Reading

### Official Documentation

- [Kubernetes Docs: Services](https://kubernetes.io/docs/concepts/services-networking/service/) - Complete service reference
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) - How DNS works
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) - Detailed service type comparison

### Deep Dives

- [A Deep Dive into Kubernetes Services](https://kubernetes.io/blog/2018/07/09/ipvs-based-in-cluster-load-balancing-deep-dive/) - How kube-proxy works
- [Kubernetes Networking Guide](https://kubernetes.io/docs/concepts/cluster-administration/networking/) - Networking fundamentals

### Related Articles

- [Pods: The Atomic Unit](pods.md) - Understanding what services connect to
- **Ingress Controllers** - HTTP-based routing (coming in Level 3)
- **Network Policies** - Controlling pod-to-pod traffic (coming in Level 3)

---

## What's Next?

You understand how Services provide stable networking for Pods. Next, learn how to configure those Pods with **[ConfigMaps and Secrets](config_and_secrets.md)** to manage configuration and sensitive data.

---

Services are the "glue" of the Kubernetes network. They transform a chaotic group of disappearing Pods into a reliable, reachable system that can scale and self-heal without ever breaking the connection to the outside world.
