---
title: "Kubernetes Troubleshooting Essentials"
description: "The systematic way to diagnose Kubernetes workloads — reading Pod failure states, mapping symptoms to causes, and going from a broken Pod to a fix."
---
# Essentials: Troubleshooting

!!! tip "Part of Essentials"
    This section sits alongside [Core Primitives](overview.md). It assumes you're comfortable with [Pods](pods.md), [Services](services.md), [ConfigMaps and Secrets](config_and_secrets.md), and [Labels and Selectors](labels_selectors.md) — troubleshooting here is mostly *those objects, read when something has gone wrong*.

When a deploy fails, the symptom shows up in one place — a Pod status, an empty endpoint list — but the cause is often somewhere else: a Service selector, a missing ConfigMap, a bad image, a resource limit. This section is the systematic way to go from symptom to cause to fix, rather than guessing.

## What You'll Learn

<div class="grid cards" markdown>

-   :material-alert-circle: **[Diagnosing Pod Failure States](pod_failure_states.md)**

    ---

    What `CrashLoopBackOff`, `ImagePullBackOff`, and `Pending` actually mean, and the `kubectl` commands that confirm which cause you're hitting.

-   :material-delete-clock: **[Resources Stuck in `Terminating`](stuck_terminating.md)**

    ---

    Why a namespace (or Pod, or PVC) won't delete, how to find the finalizer blocking it, and why force-removing it can orphan real resources.

-   :material-lan-disconnect: **[Service Has No Endpoints](service_connectivity.md)**

    ---

    A Service exists but traffic reaches nothing — diagnosing the label/selector mismatch behind empty endpoints, and fixing it in the manifest.

</div>

!!! info "More on the way"
    This section is being built out. Still planned: the core diagnostic method — `describe` → events → `logs` → `get -o wide` — as its own article.

## What's Next?

Start with **[Diagnosing Pod Failure States](pod_failure_states.md)** — the statuses you'll meet first and most often.
