---
title: "Kubernetes Namespace Stuck in Terminating: Finalizers"
description: "Why a namespace (or Pod or PVC) hangs in Terminating, how to find the finalizer blocking deletion, and why force-removing it can orphan real cloud resources."
---
# Resources Stuck in `Terminating`

!!! tip "Part of [Essentials: Troubleshooting](troubleshooting_overview.md)"
    You ran `kubectl delete` and the object won't go away — it just sits in `Terminating`. This is almost always a **finalizer**, and it's the same story whether it's a namespace, a Pod, or a PersistentVolumeClaim.

A namespace that hangs in `Terminating` almost always means an object inside it has a **finalizer** — a hook that tells Kubernetes "run cleanup before you delete me" — and whatever should clear that finalizer isn't running (a removed CRD controller, a webhook that's down).

``` bash title="Find what's blocking deletion (✅ read-only)"
kubectl get namespace team-a-dev -o jsonpath='{.spec.finalizers}'
kubectl api-resources --verbs=list --namespaced -o name \
  | xargs -n1 kubectl get --show-kind --ignore-not-found -n team-a-dev
```

Force-removing finalizers *will* unstick it, but it orphans whatever the finalizer was meant to clean up (e.g. a cloud load balancer that now leaks). Understand what the finalizer guards before you bypass it — that's a cluster-operations decision, not a reflex.

<!-- TODO (fuller write-up): generalize to stuck Pods and PVCs; show the safe
     sequence (fix/restart the controller that owns the finalizer) vs. the
     last-resort finalizer patch, with the orphaning caveat spelled out.
     Moved out of namespaces.md 2026-06-19. -->
