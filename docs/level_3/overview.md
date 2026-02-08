# Level 3: Networking

!!! tip "Connecting Services, Exposing Applications"
    You can deploy and manage workloads. Now learn how to connect them together and expose them to users.

## What You'll Master

Level 3 teaches **Kubernetes networking**—how services communicate and how traffic flows:

- **Services Deep Dive** - Service types, endpoints, kube-proxy, service mesh preview
- **Ingress Controllers** - HTTP routing, path-based routing, TLS termination
- **Network Policies** - Pod-to-pod communication control, security, isolation
- **DNS and Service Discovery** - How services find each other, DNS patterns
- **Troubleshooting Networking** - Common issues, debugging tools, packet inspection

This is where **application developers and platform engineers meet**—both need to understand networking.

---

## The Articles

<div class="grid cards" markdown>

-   :material-lan-connect: **[Services Deep Dive](services_deep_dive.md)** *(coming soon)*

    ---

    **Beyond Basics** — Service types, endpoints, kube-proxy modes, headless services

-   :material-web: **[Ingress Controllers](ingress.md)** *(coming soon)*

    ---

    **HTTP Routing** — Path-based routing, host-based routing, TLS, annotations

-   :material-security-network: **[Network Policies](network_policies.md)** *(coming soon)*

    ---

    **Traffic Control** — Pod-to-pod firewalls, security, zero-trust networking

-   :material-dns: **[DNS and Service Discovery](dns_service_discovery.md)** *(coming soon)*

    ---

    **Finding Services** — How DNS works in K8s, service names, FQDN, troubleshooting

-   :material-bug: **[Troubleshooting Networking](troubleshooting_networking.md)** *(coming soon)*

    ---

    **Debugging** — Connection failures, DNS issues, network policy problems, tools

</div>

---

## Who This Is For

**You are:**
- An application developer needing to connect microservices
- A platform engineer managing cluster networking
- Someone debugging "why can't Service A talk to Service B?"
- Looking to expose applications to external users

**You'll learn:**
- How service discovery actually works (not magic!)
- When to use ClusterIP vs NodePort vs LoadBalancer vs Ingress
- How to control which pods can talk to each other
- How to debug networking issues systematically
- How to expose applications securely

---

## The Persona Shift

Starting at Level 3, articles serve **both personas:**

- **Application Developers:** Focus on services, ingress, service-to-service communication
- **Platform Engineers:** Focus on network policies, troubleshooting, cluster-wide networking

You might be either—or both. These articles accommodate both perspectives.

---

## What's Next?

After Level 3, you'll move to **Level 4: Storage and State**, where you'll learn about persistent volumes, storage classes, and running stateful applications like databases.

Networking is often the trickiest part of Kubernetes. Take your time with this level—it's worth mastering.
