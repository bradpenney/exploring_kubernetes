---
title: "Kubernetes Security Essentials: Hardening, Secrets, RBAC"
description: "The security every Kubernetes practitioner owns: securityContext hardening, secret hygiene, RBAC from both sides, and Pod Security Standards enforcement."
---
# Essentials: Security

!!! tip "Part of Essentials"
    This section sits alongside [Core Primitives](overview.md). It assumes you're comfortable with [Pods](pods.md), [ConfigMaps and Secrets](config_and_secrets.md), and [Namespaces](namespaces.md) — security here is mostly *fields and objects you already know*, used deliberately.

Plenty of Kubernetes security is genuinely someone else's job — admission webhooks, CNI-level policy, the audit pipeline, cluster CA rotation. But a sharp, practical slice belongs to **everyone who touches the cluster**, and it's the slice that actually stops the easy attacks: which user a container runs as, how credentials are consumed, who's allowed to do what, and the policy that decides whether a Pod is even admitted.

This section covers that slice from **both sides** — because in practice the same work spans both:

- **Shipping the workload** — setting the `securityContext`, consuming Secrets without leaking them, reading `Forbidden` errors and living within the guardrails.
- **Operating the cluster** — authoring the Role and ServiceAccount, labelling a namespace to enforce a Pod Security Standard, deciding a tenant's blast radius.

!!! note "Where Essentials stops and Mastery begins"
    Essentials gives you the mechanism *and* the basic operator moves: a single Role/RoleBinding, a dedicated ServiceAccount, turning on Pod Security enforcement for a namespace. **Designing RBAC at cluster scale** — aggregated ClusterRoles, least-privilege across dozens of teams, admission control, supply-chain and runtime security — is platform-engineering depth that lives in the Mastery tier. You'll know the levers here; Mastery is about pulling them safely at scale.

## What You'll Learn

<div class="grid cards" markdown>

-   :material-shield-account: **[Securing Your Containers](security_context.md)**

    ---

    The `securityContext` fields that carry the most weight — non-root user, read-only root filesystem, dropped Linux capabilities, no privilege escalation. The highest security-per-line-of-YAML in Kubernetes.

-   :material-key-variant: **[Handling Secrets Safely](handling_secrets.md)**

    ---

    Secret *hygiene* — the three places credentials leak (images, Git, plaintext env), how to keep them out, and what to do the moment one leaks. Builds on the Secret mechanics from Core Primitives.

-   :material-account-lock: **[Understanding Your Access](understanding_access.md)**

    ---

    RBAC from both sides: reading a `Forbidden` and checking yourself with `kubectl auth can-i`, *and* authoring the Role, RoleBinding, and ServiceAccount that grant least-privilege access in the first place.

-   :material-check-decagram: **[Meeting Pod Security Standards](pod_security_standards.md)**

    ---

    What `baseline` and `restricted` enforce, why a Pod gets rejected, how to comply — and how the operator side turns enforcement on for a namespace with a single label.

</div>

## How to Approach It

This is about the *mechanism*, not a checklist to memorize — you should finish each article understanding why the setting exists, not just which line to copy. No formal security background is assumed, but you should be comfortable in a Pod spec and a namespace.

The thread running through every article is **blast radius**: if this workload, credential, or grant is compromised, what exactly can an attacker reach? Keep asking that and the right configuration usually follows.

## What's Next?

Start with **[Securing Your Containers](security_context.md)** — `securityContext` is where the most security lands for the least effort, and the Pod Security Standards article later in this section will *require* most of it anyway.

Security then deepens as you move up: the Efficiency tier secures traffic between workloads (Network Policies), and the Mastery tier covers RBAC design, cluster-wide Pod Security enforcement, and hardening at scale.
