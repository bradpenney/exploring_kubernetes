---
title: "Kubernetes Architecture: Control Plane and Nodes"
description: "A working mental model of a Kubernetes cluster — the control plane that decides, the nodes that run your Pods, and the reconcile loop that ties them together."
---
# How a Kubernetes Cluster Is Built

!!! tip "Part of Essentials"
    This is the foundation the rest of Essentials leans on. Before you dig into [Pods](pods.md) and [Services](services.md), it helps to know *what* you're actually talking to when you run `kubectl apply` — what decides where your Pod runs, and what runs it.

You've been reading "Kubernetes does X" and "the cluster reconciles your manifest." This article puts names to that machinery — just enough to make the rest of Essentials precise. The full depth (etcd internals, highly-available control planes, cluster networking) lives in the Mastery tier; here we want a working mental model, not an operator's manual.

## Two Halves: the Brain and the Muscle

A cluster is two kinds of machine working together:

- **The control plane** — the brain. It makes the decisions: what should run, where, and whether reality matches what you asked for. You talk to it; you don't run your own workloads on it.
- **The nodes** — the muscle. These are the machines that actually run your Pods.

<!-- TODO: mermaid diagram — control plane box (API server, scheduler, controllers,
     etcd) talking to N node boxes (kubelet, kube-proxy, container runtime). Slate colors. -->

## The Control Plane: What Decides

<!-- TODO (light, one short paragraph each — no etcd internals):
     - API server — the front door; kubectl and every component talk to it
     - Scheduler — picks which node a new Pod lands on
     - Controllers (controller manager) — the reconcile loops that drive actual → desired
     - etcd — the cluster's source of truth (the saved desired state) -->

## The Nodes: What Runs Your Pods

<!-- TODO:
     - kubelet — runs the containers on the node and reports their status back
     - kube-proxy — programs Service routing on the node (cross-link services.md
       "What's actually doing the work")
     - container runtime — the thing that actually starts containers -->

## The Reconcile Loop: Desired vs. Actual

<!-- TODO: the single most important idea. You declare desired state; controllers
     continuously compare it to actual state and act to close the gap. This is WHY
     `kubectl apply` is idempotent and WHY a deleted Pod comes back under a Deployment.
     Cross-link pods.md "Creating Pods" (the declarative model) and the cattle-not-pets section. -->

## What's Next?

<!-- TODO: point forward to Core Primitives (Pods) now that the machine has names. -->

<!-- STUB (2026-06-19): light Essentials intro so "control plane" is a known term
     before the primitives use it. Fuller depth lives in mastery/cluster_architecture.md.
     Once published, reintroduce "control plane" (with a link here) in pods.md and
     services.md where it was softened to "Kubernetes" / "the cluster". -->
