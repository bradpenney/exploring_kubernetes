# Essential kubectl Commands

!!! tip "Part of Day One: Getting Started"
    This is the fourth article in [Day One: Getting Started](overview.md). If you haven't deployed anything yet, do [Your First Deployment](first_deployment.md) first.

After deploying a few applications, you'll use the same 10-15 kubectl commands over and over. Let's master them.

---

## The Daily Commands

These are the commands you'll run dozens of times per day:

### 1. Get (Viewing Resources)

```bash
# List pods
kubectl get pods

# List deployments
kubectl get deployments
kubectl get deploy  # Short form

# List services
kubectl get services
kubectl get svc  # Short form

# List everything
kubectl get all

# More details
kubectl get pods -o wide
# Shows IP, NODE, etc.
```

✅ **Safe:** Read-only, can't break anything

### 2. Describe (Detailed Info)

```bash
# Detailed pod information
kubectl describe pod <pod-name>

# Shows:
# - Events (what happened)
# - Status (is it running?)
# - Container info
# - Errors (if any)
```

✅ **Safe:** Read-only

### 3. Logs (Application Output)

```bash
# View logs
kubectl logs <pod-name>

# Follow logs (like tail -f)
kubectl logs -f <pod-name>

# Last 50 lines
kubectl logs --tail=50 <pod-name>

# Multi-container pod: specify container
kubectl logs <pod-name> -c <container-name>
```

✅ **Safe:** Read-only

### 4. Exec (Run Commands in Pod)

```bash
# Execute single command
kubectl exec <pod-name> -- ls /app

# Interactive shell
kubectl exec -it <pod-name> -- /bin/bash
# or
kubectl exec -it <pod-name> -- sh
```

⚠️ **Caution:** Can modify pod filesystem (but pod is ephemeral anyway)

---

## Deployment Commands

### 5. Apply (Create/Update Resources)

```bash
# Create from file
kubectl apply -f deployment.yaml

# Create from directory
kubectl apply -f ./configs/

# Create from URL
kubectl apply -f https://example.com/app.yaml
```

⚠️ **Caution:** Creates or modifies resources in your namespace

### 6. Delete (Remove Resources)

```bash
# Delete by file
kubectl delete -f deployment.yaml

# Delete by name
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>

# Delete everything with label
kubectl delete all -l app=myapp
```

🚨 **DANGER:** Permanently deletes resources

### 7. Scale (Adjust Replicas)

```bash
# Scale deployment
kubectl scale deployment <name> --replicas=5

# Scale down to zero
kubectl scale deployment <name> --replicas=0
```

⚠️ **Caution:** Changes running instances

---

## Troubleshooting Commands

### 8. Port Forward (Local Access)

```bash
# Forward pod port to localhost
kubectl port-forward pod/<pod-name> 8080:80

# Forward service port
kubectl port-forward service/<svc-name> 8080:80
```

✅ **Safe:** Only affects your local machine

### 9. Rollout (Manage Deployments)

```bash
# Check rollout status
kubectl rollout status deployment/<name>

# Rollout history
kubectl rollout history deployment/<name>

# Undo last rollout
kubectl rollout undo deployment/<name>

# Restart deployment (recreate all pods)
kubectl rollout restart deployment/<name>
```

⚠️ **Caution:** Affects running application

### 10. Explain (Documentation)

```bash
# Documentation for any resource
kubectl explain pods
kubectl explain deployments

# Drill down
kubectl explain pods.spec
kubectl explain pods.spec.containers
```

✅ **Safe:** Just shows documentation

---

## Useful Flags

### Namespace

```bash
# Specific namespace
kubectl get pods -n other-namespace

# All namespaces
kubectl get pods --all-namespaces
kubectl get pods -A  # Short form
```

### Output Formats

```bash
# YAML
kubectl get pod <pod-name> -o yaml

# JSON
kubectl get pod <pod-name> -o json

# Custom columns
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

### Labels

```bash
# Filter by label
kubectl get pods -l app=nginx

# Show labels
kubectl get pods --show-labels
```

### Watch

```bash
# Watch for changes
kubectl get pods --watch
kubectl get pods -w  # Short form
```

---

## Command Cheat Sheet

| Task | Command |
|------|---------|
| **List pods** | `kubectl get pods` |
| **Pod details** | `kubectl describe pod <name>` |
| **Pod logs** | `kubectl logs <name>` |
| **Shell in pod** | `kubectl exec -it <name> -- sh` |
| **Create resources** | `kubectl apply -f file.yaml` |
| **Delete resources** | `kubectl delete -f file.yaml` |
| **Scale deployment** | `kubectl scale deployment <name> --replicas=N` |
| **Port forward** | `kubectl port-forward pod/<name> 8080:80` |
| **Rollback** | `kubectl rollout undo deployment/<name>` |
| **Get docs** | `kubectl explain <resource>` |

---

## Safety Labels

Remember these labels for every command:

- ✅ **Safe (Read-Only):** get, describe, logs, explain
- ⚠️ **Caution (Modifies):** apply, scale, exec, rollout
- 🚨 **DANGER (Destructive):** delete

When in doubt, use `describe` first to understand what you're working with.

---

## Common Workflows

### Debugging a Broken Pod

```bash
# 1. List pods
kubectl get pods
# Find the pod with error

# 2. Check events
kubectl describe pod <name>
# Look at Events section

# 3. Check logs
kubectl logs <name>

# 4. If needed, shell into pod
kubectl exec -it <name> -- sh
```

### Deploying an Update

```bash
# 1. Edit your YAML
# 2. Apply it
kubectl apply -f deployment.yaml

# 3. Watch rollout
kubectl rollout status deployment/<name>

# 4. If broken, rollback
kubectl rollout undo deployment/<name>
```

### Checking Application Health

```bash
# Check pods
kubectl get pods

# Check service
kubectl get svc

# Check deployment
kubectl get deployment

# Test connectivity
kubectl port-forward service/<name> 8080:80
# Open http://localhost:8080
```

---

## Practice Exercises

[Include practice exercises with solutions]

---

## Quick Recap

**Top 10 commands you'll use daily:**
1. `kubectl get pods`
2. `kubectl describe pod <name>`
3. `kubectl logs <name>`
4. `kubectl exec -it <name> -- sh`
5. `kubectl apply -f <file>`
6. `kubectl delete <resource> <name>`
7. `kubectl scale deployment <name> --replicas=N`
8. `kubectl port-forward service/<name> 8080:80`
9. `kubectl rollout undo deployment/<name>`
10. `kubectl get all`

---

## Further Reading

- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [kubectl Command Reference](https://kubernetes.io/docs/reference/kubectl/)
- [kubectl Book](https://kubectl.docs.kubernetes.io/)

---

## What's Next?

You have the essential commands down. Now understand what actually happens when you deploy: **[Understanding What Happened](understanding_deployment.md)**.

---

**Pro Tip:** Create shell aliases for common commands:
```bash
alias k=kubectl
alias kgp='kubectl get pods'
alias kgd='kubectl get deployments'
```
