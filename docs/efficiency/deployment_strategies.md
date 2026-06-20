---
title: "Blue-Green and Canary Deployments with Labels"
description: "Shift production traffic declaratively by changing which labels a Service selects — blue-green cutovers and replica-ratio canaries, the GitOps-safe way."
---
# Routing by Label: Blue-Green and Canary

<!-- STASH (moved out of essentials/labels_selectors.md 2026-06-19): these are
     deployment *strategies*, not label primitives — they require Deployments and
     traffic-shifting, which is Efficiency/Workloads material, not Essentials Core
     Primitives. Rework into the Efficiency tier (Workloads or a deployment-
     strategies article); assumes the reader already knows Deployments. -->

This is where labels stop being bookkeeping and become a deployment strategy. Because a Service routes by selector, you can shift production traffic by changing *which labels the Service selects* — no Pod restarts, no IP juggling.

## Blue-green, declaratively

Run two versions side by side, each labelled with its colour. The Service's selector decides which one is live. To cut over, you **edit the Service manifest and re-apply** — the change is in Git, reviewable and revertable, not a one-off imperative patch.

``` yaml title="backend-svc.yaml — flip version to switch traffic" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
    version: blue   # (1)!
  ports:
  - port: 80
    targetPort: 8080
```

1. Change `blue` → `green`, then `kubectl apply -f backend-svc.yaml`. Traffic moves the instant the endpoints recompute. Rolling back is the reverse edit — and because it's a committed manifest, `git revert` *is* your rollback.

!!! tip "Why not `kubectl patch`?"
    You *could* flip the selector with an imperative `kubectl patch`, and you'll see it in blog posts. Don't make it your workflow: the live cluster would then disagree with Git, and the next sync would silently revert your cutover (or your rollback). Edit the manifest, apply the manifest — the selector change is a deliberate, auditable act.

## Canary, declaratively

Two Deployments behind one Service. The Service selects only the shared label, so it spreads traffic across *both* versions in proportion to their replica counts — 9 stable + 1 canary ≈ 10% canary traffic.

``` yaml title="canary.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-stable
spec:
  replicas: 9
  selector:
    matchLabels: { app: backend, track: stable }
  template:
    metadata:
      labels: { app: backend, track: stable, version: "1.4.2" }
    spec:
      containers: [{ name: app, image: myapp:1.4.2 }]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-canary
spec:
  replicas: 1
  selector:
    matchLabels: { app: backend, track: canary }
  template:
    metadata:
      labels: { app: backend, track: canary, version: "1.5.0" }
    spec:
      containers: [{ name: app, image: myapp:1.5.0 }]
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend   # (1)!
  ports:
  - port: 80
    targetPort: 8080
```

1. Selects on `app: backend` only — both `track: stable` and `track: canary` Pods match, so the Service load-balances across both. Adjust the ratio by changing replica counts and re-applying.

This is the bare-Kubernetes version. Production traffic-shaping (percentage-based, header-based) usually moves to an Ingress controller or service mesh — but the underlying idea is always "a selector decides who's in the pool."

## Watch out for: two workloads sharing a selector

If two Deployments use overlapping `matchLabels`, they can fight over the same Pods — each thinking it owns them, scaling against each other. Keep selectors distinct per workload; the `track`/`version` label discipline above is how you avoid it.
