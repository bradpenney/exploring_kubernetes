---
title: "Meeting Kubernetes Pod Security Standards"
description: "Why a Pod is rejected for violating the restricted Pod Security Standard, what each profile requires, and how to comply — plus enforcing it on a namespace."
---
# Meeting Pod Security Standards

!!! tip "Part of [Essentials: Security](security_overview.md)"
    This article ties the section together. The settings from [Securing Your Containers](security_context.md) aren't just good practice — increasingly, your cluster *requires* them. Here's the standard that does the requiring.

You apply a Pod and the cluster bounces it:

``` text
Error from server (Forbidden): error when creating "pod.yaml":
pods "app" is forbidden: violates PodSecurity "restricted:latest":
allowPrivilegeEscalation != false, unrestricted capabilities,
runAsNonRoot != true, seccompProfile not set
```

This isn't RBAC and it isn't a typo — it's **Pod Security Admission** enforcing a **Pod Security Standard**. The cluster has decided which security settings are mandatory, and your Pod didn't meet the bar. The fix is entirely in your hands, and it's the same `securityContext` you already met earlier in this section.

## What You'll Learn

- The three Pod Security Standards — `privileged`, `baseline`, and `restricted`
- How enforcement works, and how to check what your namespace requires (read-only)
- How to read a rejection and map each violation to a fix
- A `restricted`-compliant Pod spec you can build from

## The Three Profiles

Pod Security Standards come in three escalating levels. Your platform team picks which one a namespace enforces.

<div class="grid cards" markdown>

-   :material-lock-open-variant: **privileged**

    ---

    **What it allows:** Everything — no restrictions. Reserved for trusted, infrastructure-level workloads.

    **You'll see this:** Rarely, and not for app workloads.

-   :material-lock: **baseline**

    ---

    **What it allows:** Blocks known privilege escalations (host namespaces, privileged containers) but stays lenient. A minimal safety net.

    **You'll see this:** On clusters easing into enforcement.

-   :material-lock-check: **restricted**

    ---

    **What it requires:** Heavily hardened — non-root, no privilege escalation, all capabilities dropped, a seccomp profile. The modern default for app namespaces.

    **You'll see this:** Most production clusters. Target this and you're covered everywhere.

</div>

## How Enforcement Works

Pod Security Admission is configured with **namespace labels**. There are three *modes*, and they can point at different profiles:

- `enforce` — violating Pods are **rejected** outright.
- `audit` — violations are allowed but recorded in the audit log.
- `warn` — violations are allowed but you get a **warning** on `kubectl apply`.

``` bash title="Check what your namespace requires (read-only)"
kubectl get namespace payments -o jsonpath='{.metadata.labels}'
# {"pod-security.kubernetes.io/enforce":"restricted", ...}
```

!!! note "Warnings are a gift"
    If you see a `Warning: would violate PodSecurity "restricted"` message but your Pod still ran, the namespace is in `warn`/`audit` mode — not yet `enforce`. Treat that warning as a deadline: fix it now, before enforcement flips on and your deploy starts failing.

### The Operator Side: Turning Enforcement On

Pod Security Admission is built into the API server — there's nothing to install. As the namespace owner, you opt a namespace into a profile by **labelling the namespace** (declaratively, like any other manifest):

``` yaml title="namespace-psa.yaml" linenums="1"
apiVersion: v1
kind: Namespace
metadata:
  name: payments
  labels:
    pod-security.kubernetes.io/enforce: restricted   # (1)!
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted       # (2)!
```

1. `enforce: restricted` — non-compliant Pods are rejected on admission.
2. Also setting `warn` (and/or `audit`) surfaces violations from higher-level controllers too (a Deployment whose Pod template violates the profile gets a warning at apply time, not a silent failure later).

!!! tip "Roll out enforcement in stages — don't flip straight to enforce"
    Dropping `enforce: restricted` onto a namespace full of running workloads will start rejecting their next rollout with no warning. The safe sequence: apply `warn: restricted` (and `audit`) first, watch what trips, fix or grant exceptions, **then** add `enforce`. This staged rollout is the difference between a smooth tightening and an incident — and it's exactly why the warnings above exist.

## Mapping the Rejection to Fixes

Each clause in the error maps to one `securityContext` field. Here's the rejection from the top of this article, decoded:

| Violation in the error | The fix |
| :--- | :--- |
| `runAsNonRoot != true` | `runAsNonRoot: true` |
| `allowPrivilegeEscalation != false` | `allowPrivilegeEscalation: false` |
| `unrestricted capabilities` | `capabilities.drop: [ALL]` |
| `seccompProfile not set` | `seccompProfile.type: RuntimeDefault` |

The first three you already know from [Securing Your Containers](security_context.md). The one new requirement for `restricted` is the **seccomp profile**, which filters the system calls the container may make.

## A restricted-Compliant Pod

Apply every requirement and the rejection disappears:

``` yaml title="restricted-compliant.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  securityContext:
    runAsNonRoot: true            # (1)!
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault        # (2)!
  containers:
    - name: app
      image: nginxinc/nginx-unprivileged:1.27
      ports:
        - containerPort: 8080
      securityContext:
        allowPrivilegeEscalation: false  # (3)!
        capabilities:
          drop:
            - ALL                 # (4)!
        readOnlyRootFilesystem: true  # (5)!
      volumeMounts:
        - name: tmp
          mountPath: /tmp
  volumes:
    - name: tmp
      emptyDir: {}
```

1. Non-root — required by `restricted`
2. The new one: apply the runtime's default seccomp filter
3. No privilege escalation — required
4. Drop every capability — required
5. Not strictly required by `restricted`, but excellent practice (and free here)

!!! success "Pre-flight check (read-only)"
    Don't guess whether you'll pass. Do a server-side dry run — it validates against admission *without* creating anything:

    ``` bash title="Validate without deploying"
    kubectl apply -f restricted-compliant.yaml --dry-run=server
    # pod/app created (server dry run)
    ```

    No warning, no error — you're compliant before you commit.

## Watch Out For

!!! warning "Common snags"
    **An init container slipped through.** Every container in the Pod must comply — `initContainers` and ephemeral containers included, not just your main one. A bare init container is a frequent cause of a "but I hardened it!" rejection.

    **The image insists on root.** `restricted` needs `runAsNonRoot: true`, but some images only run as root. Switch to an unprivileged image variant or set an explicit non-root `runAsUser` — see [Securing Your Containers](security_context.md) for the pattern.

    **You hardcoded a profile version.** Labels can pin a version (`restricted:v1.29`) or float (`restricted:latest`). You don't control the label, but knowing it explains why the *same* spec might start failing after a cluster upgrade tightens the profile.

## Practice Exercises

??? question "Exercise 1: Fix the rejection"
    Your Pod is rejected with:

    ``` text
    violates PodSecurity "restricted:latest": allowPrivilegeEscalation != false,
    seccompProfile not set
    ```

    Your spec already has `runAsNonRoot: true` and drops all capabilities. What two lines do you add, and where?

    ??? tip "Solution"
        Two fields are missing:

        ``` yaml
        spec:
          securityContext:
            seccompProfile:
              type: RuntimeDefault  # (1)!
          containers:
            - name: app
              securityContext:
                allowPrivilegeEscalation: false  # (2)!
        ```

        1. Add at Pod level.
        2. Add at container level.

        Re-validate with `kubectl apply --dry-run=server` before deploying. Both clauses should clear.

??? question "Exercise 2: Will it enforce or just warn?"
    A namespace has the label `pod-security.kubernetes.io/warn: restricted` but **no** `enforce` label. You apply a non-compliant Pod. What happens?

    ??? tip "Solution"
        The Pod **is created**, but you get a `Warning:` describing the `restricted` violations.

        With only a `warn` label and no `enforce` label, nothing is *blocked* — the cluster is in advisory mode. But that warning is a clear signal the team intends to enforce `restricted` eventually. Fix the Pod now so the future flip to `enforce` doesn't break your deploys.

        **What you learned:** `warn`/`audit`/`enforce` are independent — a namespace can warn about a profile long before it enforces it.

## Quick Recap

| Profile | For | Your job |
| :--- | :--- | :--- |
| `privileged` | Infra workloads | Rarely relevant |
| `baseline` | Light enforcement | Avoid host access, privileged containers |
| `restricted` | Most app namespaces | Non-root, no priv-esc, drop ALL caps, seccomp `RuntimeDefault` |

| Task | Command (read-only) |
| :--- | :--- |
| Check namespace policy | `kubectl get ns <ns> -o jsonpath='{.metadata.labels}'` |
| Validate before deploy | `kubectl apply -f pod.yaml --dry-run=server` |

## What's Next?

That completes **Essentials: Security** — you can now run hardened containers, handle credentials safely, understand your access, and pass Pod Security enforcement.

Security keeps deepening as you grow into platform work: the Efficiency tier covers securing traffic with Network Policies, and the Mastery tier covers designing RBAC, enforcing Pod Security across a cluster, and hardening at scale.

## Further Reading

### Official Documentation

- [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) - The full definition of all three profiles
- [Enforce Pod Security Standards with Namespace Labels](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/) - How the enforce/audit/warn labels work
- [Seccomp in Kubernetes](https://kubernetes.io/docs/tutorials/security/seccomp/) - What `RuntimeDefault` actually filters

### Deep Dives

- [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) - The controller doing the enforcing

### Related Articles

- [Securing Your Containers](security_context.md) - The securityContext fields these standards require
- [Essentials: Security Overview](security_overview.md) - The full Security section
