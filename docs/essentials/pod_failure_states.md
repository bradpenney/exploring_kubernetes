---
title: "Diagnosing Pod Failure States in Kubernetes"
description: "What CrashLoopBackOff, ImagePullBackOff, and Pending actually mean in kubectl get pods — and the commands that take you from symptom to cause to fix."
---
# Diagnosing Pod Failure States

!!! tip "Part of [Essentials: Troubleshooting](troubleshooting_overview.md)"
    These are the statuses you'll see in the `STATUS` column of `kubectl get pods` when something's wrong. Each maps to a small set of causes and a quick way to confirm which one you're hitting. For the Pod *lifecycle phases* themselves (Pending → Running → Succeeded/Failed), see [Pods Deep Dive](pods.md).

Most Pod problems announce themselves as one of a handful of statuses. Learn what each one is actually telling you, and the fix usually follows.

## CrashLoopBackOff

`CrashLoopBackOff` is not a Pod phase — it's a container status that appears in `kubectl get pods` when a container starts, immediately crashes, and Kubernetes keeps trying to restart it.

**What it tells you:**

- The container started (image pulled correctly)
- The container crashed almost immediately (application error)
- Kubernetes is backing off between restart attempts (the "Backoff" part)

**What to do:**

```bash title="Investigate CrashLoopBackOff"
kubectl get pods  # (1)!
kubectl describe pod <pod-name>  # (2)!
kubectl logs <pod-name>  # (3)!
kubectl logs <pod-name> --previous  # (4)!
```

1. See the crash status.
2. Check events — often explains why.
3. Read the container's output before it crashed.
4. If the container is too fast to catch, read the previous run's logs.

The most common causes: missing environment variables, wrong command, misconfigured entrypoint, or a configuration file the app expected that isn't mounted.

## Pending — the Pod won't schedule

A Pod stuck in `Pending` hasn't been placed on a node yet.

**Possible causes:**

- No node has enough CPU or memory to schedule the Pod
- The container image name is wrong or the registry is inaccessible
- A required PersistentVolumeClaim doesn't exist

```bash title="Diagnose scheduling failure"
kubectl describe pod <pod-name>  # (1)!
```

1. Look at the `Events:` section at the bottom — it explains why scheduling failed.

## ImagePullBackOff

The container image couldn't be pulled.

**Possible causes:** wrong image name, wrong tag, private registry without credentials, network issue.

```bash title="Diagnose ImagePullBackOff"
kubectl describe pod <pod-name>  # (1)!
```

1. The `Events` section shows the pull error (image not found, unauthorized, etc.).

## Running, but the app doesn't respond

`kubectl get pods` shows `Running` and `1/1 Ready`, but HTTP requests fail.

**This means:** the container is alive, but your application inside it isn't listening on the expected port, or is returning errors.

```bash title="Diagnose unresponsive app"
kubectl logs <pod-name>  # (1)!
kubectl exec -it <pod-name> -- sh  # (2)!
```

1. Look for application startup errors.
2. Check if the process is running.

<!-- TODO (fuller write-up): add OOMKilled, Error/Completed, ContainerCreating;
     a symptom → cause → fix table; and the connectivity/label-mismatch path
     cross-linking Services and Labels. Stub assembled from pods.md 2026-06-19. -->
