---
title: Your First Helm Deployment - Deploying Kubernetes Charts
description: Learn how to deploy applications using Helm charts. A simpler way to manage complex Kubernetes applications.
---
# Your First Helm Deployment

!!! tip "Part of Day One: Getting Started"
    This is the third article in the Helm Path for [Day One: Getting Started](../overview.md). Make sure you've completed [Getting Helm Access](access.md) first.

You have Helm installed. You have cluster access. Now comes the moment of truth: putting your first application onto the cluster.

In the Helm world, we don't just "deploy" files; we manage **Releases**. A release is a specific instance of a chart running in your cluster. You can have three different "releases" of the same "nginx" chart running in the same namespace, each with its own name and configuration.

This article covers the two ways you'll most likely encounter Helm in your daily work.

!!! info "What You'll Learn"
    By the end of this article, you'll be able to:

    - **Choose between the two common scenarios**: CI/CD charts vs. Vendor charts
    - **Customize a deployment** using a `values.yaml` file
    - **Install your first release** and verify it's running
    - **Update a running application** with new configuration
    - **Clean up resources** when you're finished

---

## Which Scenario Are You In?

Before we start, identify which situation matches your task today:

=== ":material-rocket-launch: Scenario 1: The CI/CD Chart"

    **Your situation:** Your company's CI/CD pipeline (Jenkins, GitLab, GitHub Actions) has built your code and packaged it into a Helm chart. You need to deploy *your* app to a dev environment.

    **What you have:** A directory containing a `Chart.yaml` and a `values.yaml` file, or a `.tgz` package.

=== ":material-store: Scenario 2: The Vendor Chart"

    **Your situation:** You need to install a standard piece of infrastructure—like a database (PostgreSQL), a cache (Redis), or a web server—using a chart provided by a vendor or the community.

    **What you have:** A chart name like `bitnami/nginx` or `prometheus-community/prometheus`.

---

## Scenario 1: Deploying a CI-Generated Chart

If your pipeline created a chart, you'll likely find it in your project directory.

### 1. Examine the values.yaml
In Helm, you **never** edit the templates (the complex Kubernetes YAML). Instead, you edit `values.yaml`. This file is the "settings menu" for your application.

```yaml title="values.yaml (example)" linenums="1"
replicaCount: 1  # (1)!

image:
  repository: my-company/my-app
  tag: "v1.0.1"  # (2)!

service:
  type: ClusterIP
  port: 80
```

1. How many copies of the app to run.
2. The specific version of your code built by the CI pipeline.

### 2. Install the Release
Navigate to the directory containing your chart and run:

⚠️ **Caution (Modifies Resources):**

```bash title="Install from local directory"
# Usage: helm install <release-name> <path-to-chart>
helm install my-app ./my-chart
```

### 3. Customize with Overrides
You don't have to edit the main `values.yaml`. You can provide your own "overrides" file for your specific environment.

```yaml title="dev-values.yaml"
replicaCount: 2
image:
  tag: "feature-branch-test"
```

```bash title="Install with custom values"
helm install my-app ./my-chart -f dev-values.yaml
```

---

## Scenario 2: Deploying a Vendor Chart

If you're installing something like Nginx or Redis, you'll fetch it from a repository.

### 1. Find the settings (Values)
Every vendor chart has different options. To see what you can customize, run:

✅ **Safe (Read-Only):**

```bash title="See available settings"
helm show values bitnami/nginx > nginx-values.yaml
```

### 2. Deploy with overrides
You usually only need to change a few things. Instead of a file, you can use the `--set` flag for quick changes.

⚠️ **Caution (Modifies Resources):**

```bash title="Install a vendor chart"
helm install my-web-server bitnami/nginx --set replicaCount=2
```

---

## Verifying the Deployment

Regardless of how you installed it, you need to make sure it worked.

### 1. Check Helm Status
Helm will tell you if the *Release* was successful.

```bash title="Check Helm release"
helm list
# NAME       NAMESPACE REVISION STATUS   CHART         APP VERSION
# my-app     default   1        deployed my-app-0.1.0  1.0.1
```

### 2. Use kubectl to Investigate
Helm creates Kubernetes resources. Use your `kubectl` skills to see them.

✅ **Safe (Read-Only):**

```bash title="See resources created by Helm"
kubectl get all
# You should see the Pods and Services created by your chart!
```

---

## Updating and Rolling Back

One of Helm's superpowers is the ability to change things safely.

### Upgrading
If you change your `values.yaml` (e.g., increase replicas to 3), run:

⚠️ **Caution (Modifies Resources):**

```bash title="Upgrade a release"
helm upgrade my-app ./my-chart -f dev-values.yaml
```

### Rolling Back
If the upgrade breaks something, you can go back to the previous version instantly.

⚠️ **Caution (Modifies Resources):**

```bash title="Rollback to version 1"
helm rollback my-app 1
```

---

## Cleaning Up

When you're done with your test, you can remove everything Helm created with one command.

🚨 **DANGER (Destructive):**

```bash title="Uninstall a release"
helm uninstall my-app
```

---

## Practice Exercise

??? question "Exercise: Deploy and Scale Nginx"
    1. Add the Bitnami repo (if you haven't: `helm repo add bitnami https://charts.bitnami.com/bitnami`).
    2. Install a release named `practice-web` using the `bitnami/nginx` chart.
    3. Scale it to 3 replicas using `helm upgrade` and the `--set replicaCount=3` flag.
    4. Verify the 3 pods are running with `kubectl get pods`.
    5. Uninstall the release.

    ??? tip "Solution"
        ```bash
        # Install
        helm install practice-web bitnami/nginx

        # Scale
        helm upgrade practice-web bitnami/nginx --set replicaCount=3

        # Verify
        kubectl get pods

        # Cleanup
        helm uninstall practice-web
        ```

---

## Quick Recap

| Action | Command | Why It Matters |
|--------|---------|----------------|
| **Deploy** | `helm install <name> <chart>` | Creates a new Release |
| **Customize** | `-f values.yaml` or `--set` | Overrides default settings |
| **Verify** | `helm list` | Confirms the release status |
| **Update** | `helm upgrade <name> <chart>` | Applies changes to a release |
| **Rollback** | `helm rollback <name> <rev>` | Reverts to a known good state |
| **Delete** | `helm uninstall <name>` | Removes all created resources |

---

## What's Next?

You've deployed your first application using Helm! You've seen how `values.yaml` controls the deployment and how Helm manages the lifecycle.

**Next:** **[Essential Helm Commands](commands.md)** - Master the 10 commands you'll use every day to manage your releases.
