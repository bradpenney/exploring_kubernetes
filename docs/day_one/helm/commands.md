---
title: Essential Helm Commands - Managing Kubernetes Releases
description: Learn the key Helm commands for installing, upgrading, and managing application releases in your cluster.
---
# Essential Helm Commands

!!! tip "Part of Day One: Getting Started"
    This is the fourth article in the Helm Path for [Day One: Getting Started](../overview.md). If you haven't deployed anything yet, do [Your First Helm Deployment](first_deploy.md) first.

You've successfully deployed a Helm chart. You've seen that `helm install` feels like a magic button that creates multiple Kubernetes resources at once. But as a developer, you need more than just an "install" button—you need to investigate, update, and fix things when they go wrong.

The good news is that you only need a handful of Helm commands to be productive. This guide organizes the essentials by task and shows you when to use **Helm** and when to switch back to **kubectl**.

!!! info "What You'll Learn"
    By the end of this article, you'll know:

    - The daily commands for **viewing and investigating** releases
    - The **deployment commands** for managing the lifecycle of your app
    - How to **troubleshoot** using Helm-specific tools
    - **When to use Helm vs. kubectl** (the most common developer question!)

---

## The Daily Toolbox (Read-Only)

These commands are safe to run anytime. They help you understand what is currently "managed" by Helm in your cluster.

=== "helm list"

    ### List (See Your Releases)

    The `helm list` command shows you all the applications (releases) currently running in your namespace.

    ```bash title="List all releases in current namespace"
    helm list
    # NAME    NAMESPACE REVISION STATUS   CHART         APP VERSION
    # my-app  default   2        deployed my-app-0.1.0  1.0.1
    ```

    **Pro tip:** Use `-A` to see releases in **all** namespaces (if you have permission).

    ✅ **Safe:** Read-only

=== "helm status"

    ### Status (Check Health)

    The `helm status` command shows you the "NOTES" provided by the chart author and a summary of the resources created.

    ```bash title="Check release status"
    helm status my-app
    # NAME: my-app
    # STATUS: deployed
    # REVISION: 2
    # NOTES:
    # 1. Get the application URL by running...
    ```

    ✅ **Safe:** Read-only

=== "helm get values"

    ### Get Values (Check Settings)

    Want to know exactly what configuration you applied to a release? This command shows your overrides.

    ```bash title="See applied overrides"
    helm get values my-app
    # USER-SUPPLIED VALUES:
    # replicaCount: 3
    ```

    **Pro tip:** Use `--all` to see the *entire* configuration, including the defaults you didn't change.

    ✅ **Safe:** Read-only

---

## Deployment & Management

These commands modify your cluster. Always verify your current context and namespace before running them.

=== "helm upgrade"

    ### Upgrade (Apply Changes)

    Whenever you change your `values.yaml` or want to deploy a newer version of a chart, use `upgrade`.

    ⚠️ **Caution (Modifies Resources):**

    ```bash title="Update a release"
    helm upgrade my-app bitnami/nginx -f my-values.yaml
    ```

    **Pro tip:** Use `--install` to tell Helm "Install this if it doesn't exist, otherwise upgrade it." This is the standard command used in many CI/CD pipelines.

=== "helm rollback"

    ### Rollback (Undo Errors)

    If an upgrade causes your Pods to crash, you can revert to any previous "Revision" instantly.

    ⚠️ **Caution (Modifies Resources):**

    ```bash title="Roll back to revision 1"
    helm rollback my-app 1
    ```

=== "helm uninstall"

    ### Uninstall (Clean Up)

    This removes the release and **all** associated Kubernetes resources (Pods, Services, etc.).

    🚨 **DANGER (Destructive):**

    ```bash title="Delete a release"
    helm uninstall my-app
    ```

---

## Troubleshooting & Investigation

When things don't look right, these commands help you look "under the hood."

### 1. View History
Every time you run an `upgrade` or `rollback`, Helm creates a new revision.

```bash title="See release history"
helm history my-app
# REVISION    UPDATED    STATUS      CHART         DESCRIPTION
# 1           [date]     superseded  my-app-0.1.0  Install complete
# 2           [date]     deployed    my-app-0.1.0  Upgrade complete
```

### 2. View Generated YAML
Sometimes you want to see exactly what YAML Helm is sending to Kubernetes.

```bash title="See the generated manifests"
helm get manifest my-app
# (Outputs the full Kubernetes YAML for the entire release)
```

---

## The Golden Rule: Helm vs. kubectl

This is the most important concept for a Day One user. 

| If you want to... | Use this tool | Command Example |
|-------------------|---------------|-----------------|
| **Change configuration** (replicas, env vars, ports) | **Helm** | `helm upgrade ...` |
| **Undo a bad deploy** | **Helm** | `helm rollback ...` |
| **See app logs** | **kubectl** | `kubectl logs <pod-name>` |
| **Debug a crashing pod** | **kubectl** | `kubectl describe pod <pod-name>` |
| **Test a service locally** | **kubectl** | `kubectl port-forward ...` |

**The workflow:**
1.  **Helm** deploys the application.
2.  **kubectl** investigates the results.

---

## Practice Exercises

??? question "Exercise 1: Audit a Release"
    Find a release in your namespace (deploy one if needed). List its history, then find out exactly what `replicaCount` value is currently applied to it.

    ??? tip "Solution"
        ```bash
        helm list
        helm history <release-name>
        helm get values <release-name>
        ```

??? question "Exercise 2: Dry Run"
    Before running an upgrade, you can "preview" what Helm will do without actually changing anything. Try to find the flag for this using `helm upgrade --help`.

    ??? tip "Solution"
        The flags are `--dry-run` and `--debug`.
        ```bash
        helm upgrade my-app bitnami/nginx --set replicaCount=5 --dry-run --debug
        ```
        This will print the generated YAML to your terminal but **won't** touch the cluster.

---

## Quick Recap

| Command | Purpose |
|---------|---------|
| `helm list` | See what apps are running |
| `helm status` | Check release notes and health |
| `helm upgrade --install` | The "Do Everything" deploy command |
| `helm rollback` | Revert to a previous revision |
| `helm uninstall` | Delete everything |
| `helm get manifest` | See the "raw" Kubernetes YAML |

---

## What's Next?

You've mastered the commands. Now it's time to understand **what** is actually happening when you run them.

**Next:** **[Understanding What Helm Created](understanding.md)** - See how Helm translates your `values.yaml` into the Pods and Services you see in `kubectl`.
