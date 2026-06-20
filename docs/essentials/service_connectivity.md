---
title: "Kubernetes Service Has No Endpoints: Selector Mismatches"
description: "When a Service exists but traffic reaches nothing, it's almost always a label/selector mismatch. How to diagnose empty endpoints and fix it the durable way."
---
# Service Has No Endpoints

!!! tip "Part of [Essentials: Troubleshooting](troubleshooting_overview.md)"
    The symptom: a Service exists, but nothing answers — requests hang or get refused. The cause is almost always that the Service's **selector doesn't match any Pod's labels**, so its endpoint list is empty. Builds on [Services](services.md) and [Labels and Selectors](labels_selectors.md).

A Service doesn't hold a list of Pods — it runs a label query and routes to whatever currently matches. When that query matches nothing, `kubectl get endpoints` shows `<none>` and traffic goes nowhere. The overwhelming majority of "my Service isn't working" reports are this.

## Diagnose it

Three read-only commands tell you everything — compare what the Service *wants* against what the Pods *have*:

```bash title="Diagnose label mismatch (✅ read-only)"
kubectl describe service backend-svc  # (1)!
kubectl get pods --show-labels  # (2)!
kubectl get endpoints backend-svc  # (3)!
```

1. Check the selector the Service is using — look for the `Selector:` line.
2. Check what labels the Pods actually have.
3. The verdict: if `ENDPOINTS` shows `<none>`, the selector matches no running Pod.

## The two ways a selector misses

??? warning "The labels simply don't match"
    The Service wants `app: backend`; the Pods are labelled `app: api` (or `app: Backend`). Nothing matches. Label matching is **exact and case-sensitive** — `app: Backend` ≠ `app: backend`.

??? warning "The selector requires a label the Pods don't all have"
    A selector is an **AND**. If the Service selects `app: backend, version: "1.0"` but the Pods only carry `app: backend`, nothing matches. Extra labels on a Pod are fine; a *missing required* label is fatal.

## The fix

Align one side to the other **in the manifest**, not with a live edit — decide whether the Service's selector or the Pods' labels are "correct," change that file, and re-apply. A `kubectl edit`/`patch` would fix it until the next GitOps sync quietly reverts you.

```bash title="Apply the corrected manifest"
kubectl apply -f backend-svc.yaml
kubectl get endpoints backend-svc  # (1)!
```

1. Endpoints should now list Pod IPs — the Service is routing again.

<!-- TODO (fuller write-up): add the cases beyond label mismatch — port/targetPort
     mismatch, and a Pod whose readiness probe is failing (Ready Pods only appear in
     endpoints). Consolidated here 2026-06-19 from services.md + labels_selectors.md. -->
