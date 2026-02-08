# Getting kubectl Access

!!! tip "Part of Day One: Getting Started"
    This is the second article in [Day One: Getting Started](overview.md). Make sure you've read [What Is Kubernetes?](what_is_kubernetes.md) first.

Your company's platform team gave you:
- A kubeconfig file (might be called `config` or `kubeconfig.yaml`)
- A namespace (like `dev-yourteam` or `yourname-dev`)
- Instructions that probably said "install kubectl and you're good to go"

Let's make sure you can actually connect.

---

## What You're Connecting To

Your company has a Kubernetes cluster—a group of servers running Kubernetes. You're not managing the cluster; you're just deploying your applications to it.

**Think of it like:**
- The cluster is a shared server
- Your namespace is your home directory
- kubectl is your SSH client

You have access to YOUR namespace, not the whole cluster.

---

## Installing kubectl

kubectl (pronounced "kube-control" or "kube-cuttle") is the command-line tool for talking to Kubernetes.

=== "macOS"

    **Using Homebrew:**
    ```bash
    brew install kubectl
    ```

    **Verify installation:**
    ```bash
    kubectl version --client
    # Should show: Client Version: v1.28.x or similar
    ```

=== "Linux"

    **Download and install:**
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```

    **Verify installation:**
    ```bash
    kubectl version --client
    ```

=== "Windows"

    **Using Chocolatey:**
    ```powershell
    choco install kubernetes-cli
    ```

    **Or download directly:**
    - Download from [Kubernetes releases](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/)
    - Add to PATH

    **Verify installation:**
    ```powershell
    kubectl version --client
    ```

---

## Setting Up Your kubeconfig

kubectl needs to know:
1. Where the cluster is (API server address)
2. Who you are (credentials)
3. Which namespace to use

All of this lives in a kubeconfig file.

### Your Platform Team Gave You a File

You probably received a file called `config` or `my-cluster.yaml`. This file contains everything kubectl needs.

**Put it in the right place:**

```bash
# Create kubectl config directory
mkdir -p ~/.kube

# Copy the file you received
cp /path/to/received-config ~/.kube/config

# Secure it (important!)
chmod 600 ~/.kube/config
```

!!! warning "Security"
    Your kubeconfig contains credentials. Treat it like a password:
    - Don't commit it to git
    - Don't share it in Slack
    - Don't paste it in public forums

---

## Understanding Contexts

Your kubeconfig might have multiple **contexts**. A context is a combination of:
- Cluster (which K8s cluster)
- User (your credentials)
- Namespace (your workspace)

**View available contexts:**

```bash
kubectl config get-contexts
# CURRENT   NAME           CLUSTER        AUTHINFO       NAMESPACE
# *         dev-cluster    prod-cluster   john-dev       team-frontend-dev
#           prod-cluster   prod-cluster   john-prod      team-frontend-prod
```

The `*` shows your current context.

**Switch contexts:**

```bash
# Switch to dev
kubectl config use-context dev-cluster
# Switched to context "dev-cluster"
```

---

## Verifying Your Connection

### Step 1: Check cluster connection

```bash
kubectl cluster-info
# Should show: Kubernetes control plane is running at https://...
```

### Step 2: Check your namespace

```bash
kubectl config view --minify | grep namespace
# Should show your assigned namespace
```

If empty, your platform team should tell you what namespace to use.

**Set your namespace:**

```bash
kubectl config set-context --current --namespace=your-namespace-here
```

### Step 3: Try listing pods

```bash
kubectl get pods
# Might show "No resources found" - that's OK! It means you're connected.
```

If you see an error like "Unauthorized" or "Forbidden", contact your platform team.

---

## Common Connection Problems

??? question "Error: The connection to the server was refused"
    **Problem:** kubectl can't reach the cluster.

    **Possible causes:**
    - VPN required but not connected
    - Wrong cluster address in kubeconfig
    - Firewall blocking connection

    **Try:**
    1. Connect to VPN if your company requires it
    2. Ask platform team if cluster is accessible from your network
    3. Verify cluster address: `kubectl config view`

??? question "Error: Forbidden / Unauthorized"
    **Problem:** Authentication failed.

    **Possible causes:**
    - Credentials expired
    - Wrong user in kubeconfig
    - Not granted access yet

    **Try:**
    1. Ask platform team if your access is active
    2. Check if credentials have expiration
    3. Verify user name: `kubectl config view`

??? question "Error: No resources found in namespace"
    **Problem:** Actually not a problem! This means you're connected but haven't deployed anything yet.

    **This is success.** Move to the next article.

---

## Understanding Your Access

### What You Can Do

In YOUR namespace, you can typically:
- ✅ Create pods, deployments, services
- ✅ View logs
- ✅ Delete resources you created
- ✅ Scale deployments

### What You Can't Do

- ❌ Access other teams' namespaces
- ❌ Modify cluster-wide resources
- ❌ Access production (unless explicitly granted)
- ❌ Create namespaces

**This is good!** It means you can experiment safely without breaking anything outside your namespace.

---

## The kubectl Cheat Sheet

Essential commands you'll use constantly:

```bash
# See what's running
kubectl get pods
kubectl get deployments
kubectl get services

# Get detailed info
kubectl describe pod <pod-name>

# View logs
kubectl logs <pod-name>

# Execute commands in pod
kubectl exec -it <pod-name> -- /bin/bash

# Apply configuration
kubectl apply -f my-app.yaml

# Delete resources
kubectl delete pod <pod-name>
```

We'll cover all of these in detail in [Essential kubectl Commands](essential_commands.md).

---

## Practice Exercise

??? question "Exercise: Verify Your Setup"
    Complete these steps to verify you're ready:

    1. Install kubectl
    2. Configure your kubeconfig
    3. Run `kubectl cluster-info` successfully
    4. Run `kubectl get pods` (even if it says "No resources found")
    5. Check your current namespace

    ??? tip "Solution"
        ```bash
        # 1. Check kubectl is installed
        kubectl version --client

        # 2. Verify kubeconfig exists
        ls ~/.kube/config

        # 3. Test cluster connection
        kubectl cluster-info

        # 4. Test namespace access
        kubectl get pods

        # 5. See current context and namespace
        kubectl config get-contexts
        kubectl config view --minify | grep namespace
        ```

        **Success looks like:**
        - `kubectl cluster-info` shows cluster address
        - `kubectl get pods` returns (even if empty)
        - No "Unauthorized" or "Forbidden" errors

---

## Quick Recap

| Task | Command |
|------|---------|
| **Install kubectl** | `brew install kubectl` (macOS) |
| **Verify connection** | `kubectl cluster-info` |
| **List pods** | `kubectl get pods` |
| **Switch context** | `kubectl config use-context <name>` |
| **Set namespace** | `kubectl config set-context --current --namespace=<ns>` |

---

## Further Reading

### Official Documentation
- [Install kubectl](https://kubernetes.io/docs/tasks/tools/) - Installation for all platforms
- [Configure kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#verify-kubectl-configuration) - kubeconfig setup
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) - Command reference

### Deep Dives
- [Understanding kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) - Configuration file format
- [kubectl Context and Configuration](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-context-and-configuration) - Managing contexts

---

## What's Next?

You're connected! kubectl is working, and you can access your namespace. Now let's deploy your first application: **[Your First Deployment](first_deployment.md)**.

---

**Don't worry if kubectl feels foreign.** After deploying a few applications, these commands will become second nature.
