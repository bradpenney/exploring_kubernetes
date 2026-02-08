# ConfigMaps and Secrets: Configuring Your Application

!!! tip "Part of Level 1: Core Primitives"
    This article is part of [Level 1: Core Primitives](overview.md). Make sure you understand [Pods](pods.md) and [Services](services.md) first.

## Why You Need to Know About ConfigMaps and Secrets

**Your actual problem:** Your application needs different configuration in dev vs. staging vs. production. Different database URLs, different API keys, different feature flags. You need to change these without rebuilding your container image.

**What you'll encounter:**
- Someone says "just update the ConfigMap" (what ConfigMap? where?)
- Environment variables that come from nowhere (`DATABASE_URL` is set but not in your Dockerfile)
- Secrets mounted as files (`/etc/secrets/db-password` contains the actual password)
- Needing to change a config value and not knowing where it lives

**You need to know:** ConfigMaps and Secrets are where Kubernetes stores configuration. ConfigMaps for non-sensitive data (URLs, feature flags). Secrets for sensitive data (passwords, API keys, certificates).

**Why this matters for developers:**
- You don't hardcode configuration in your image (one image, many environments)
- You can update config without redeploying code
- You keep secrets out of version control
- Different teams can manage different configs (ops handles secrets, devs handle app config)

---

## The Configuration Problem

Your application needs configuration: API keys, database URLs, feature flags, environment-specific settings.

**Bad approach: Bake it into the container image**
- Same image can't run in dev and prod (different configs)
- Changing config requires rebuilding the entire image
- Secrets in images = security nightmare (visible to anyone with image access)
- Version control contains sensitive data

**Good approach: External configuration**
- Same image runs everywhere (dev, staging, prod)
- Change config without touching code
- Secrets managed separately from code
- Configuration is versioned and auditable

**Kubernetes solution:** ConfigMaps (non-sensitive) and Secrets (sensitive data).

---

## ConfigMaps: Non-Sensitive Configuration

ConfigMaps store configuration data as key-value pairs. Think of them as `.env` files for Kubernetes.

### Creating ConfigMaps

=== "From Literal Values"

    ``` bash title="Create ConfigMap from Command Line"
    kubectl create configmap app-config \
      --from-literal=DATABASE_URL=postgres://db:5432/myapp \
      --from-literal=LOG_LEVEL=info \
      --from-literal=FEATURE_FLAG_NEW_UI=true
    ```

=== "From File"

    ``` bash title="Create ConfigMap from File"
    # From a properties file
    kubectl create configmap app-config --from-file=app.properties

    # From environment file
    kubectl create configmap app-config --from-env-file=.env
    ```

=== "From YAML"

    ``` yaml title="configmap.yaml" linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: app-config
    data:  # (1)!
      DATABASE_URL: "postgres://db:5432/myapp"
      LOG_LEVEL: "info"
      FEATURE_FLAG_NEW_UI: "true"
      app.properties: |  # (2)!
        server.port=8080
        server.timeout=30s
    ```

    1. Key-value pairs stored as strings
    2. Multi-line values use `|` or `>`

    ``` bash title="Apply ConfigMap"
    kubectl apply -f configmap.yaml
    ```

---

## Using ConfigMaps in Pods

### As Environment Variables

``` yaml title="pod-with-configmap-env.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DATABASE_URL  # (1)!
      valueFrom:
        configMapKeyRef:
          name: app-config  # (2)!
          key: DATABASE_URL  # (3)!
    envFrom:  # (4)!
    - configMapRef:
        name: app-config
```

1. Environment variable name in container
2. ConfigMap name
3. Key within ConfigMap
4. Load ALL keys from ConfigMap as env vars

**Inside the container:**
```bash
echo $DATABASE_URL
# postgres://db:5432/myapp
```

### As Mounted Files

``` yaml title="pod-with-configmap-volume.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config-volume  # (1)!
      mountPath: /etc/config  # (2)!
      readOnly: true
  volumes:
  - name: config-volume
    configMap:
      name: app-config  # (3)!
```

1. Volume name (arbitrary)
2. Where to mount in container
3. ConfigMap to mount

**Inside the container:**
```bash
ls /etc/config/
# DATABASE_URL  LOG_LEVEL  FEATURE_FLAG_NEW_UI  app.properties

cat /etc/config/DATABASE_URL
# postgres://db:5432/myapp

cat /etc/config/app.properties
# server.port=8080
# server.timeout=30s
```

---

## Secrets: Sensitive Data

Secrets work like ConfigMaps but for sensitive data. They're base64-encoded (NOT encrypted!) and have extra access controls.

!!! warning "Secrets Are Not Encrypted by Default"
    Secrets are base64-encoded, which is NOT encryption. Anyone with cluster access can decode them. For true encryption at rest, enable encryption in etcd.

### Creating Secrets

=== "From Literal Values"

    ``` bash title="Create Secret from Command Line"
    kubectl create secret generic db-credentials \
      --from-literal=username=admin \
      --from-literal=password=super-secret-password
    ```

=== "From Files"

    ``` bash title="Create Secret from Files"
    # Create files
    echo -n 'admin' > username.txt
    echo -n 'super-secret-password' > password.txt

    # Create secret
    kubectl create secret generic db-credentials \
      --from-file=username=username.txt \
      --from-file=password=password.txt

    # Clean up files (important!)
    rm username.txt password.txt
    ```

=== "From YAML"

    ``` yaml title="secret.yaml" linenums="1"
    apiVersion: v1
    kind: Secret
    metadata:
      name: db-credentials
    type: Opaque  # (1)!
    data:  # (2)!
      username: YWRtaW4=  # (3)!
      password: c3VwZXItc2VjcmV0LXBhc3N3b3Jk
    ```

    1. Generic secret type
    2. Values must be base64-encoded
    3. `echo -n 'admin' | base64` = `YWRtaW4=`

    !!! tip "StringData for Plain Text"
        Use `stringData` instead of `data` to avoid base64 encoding:

        ```yaml
        stringData:
          username: admin
          password: super-secret-password
        ```

        Kubernetes will encode it for you.

---

## Using Secrets in Pods

### As Environment Variables

``` yaml title="pod-with-secret-env.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_USERNAME  # (1)!
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

1. Environment variable names (can differ from secret keys)

!!! warning "Secrets in Environment Variables"
    Environment variables can be viewed by anyone with pod access:
    ```bash
    kubectl exec pod-name -- env | grep PASSWORD
    ```

    Mounted files are more secure (requires shell access).

### As Mounted Files

``` yaml title="pod-with-secret-volume.yaml" linenums="1"
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true  # (1)!
  volumes:
  - name: secret-volume
    secret:
      secretName: db-credentials
      defaultMode: 0400  # (2)!
```

1. Secrets should always be read-only
2. File permissions (read-only for owner)

**Inside the container:**
```bash
cat /etc/secrets/username
# admin

cat /etc/secrets/password
# super-secret-password
```

---

## Secret Types

| Type | Use Case | Fields |
|------|----------|--------|
| `Opaque` | Generic secrets | Arbitrary key-value pairs |
| `kubernetes.io/service-account-token` | Service account tokens | Automatic |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials | `.dockerconfigjson` |
| `kubernetes.io/tls` | TLS certificates | `tls.crt`, `tls.key` |
| `kubernetes.io/ssh-auth` | SSH keys | `ssh-privatekey` |
| `kubernetes.io/basic-auth` | Basic auth | `username`, `password` |

### Docker Registry Secret Example

``` bash title="Create Docker Registry Secret"
kubectl create secret docker-registry my-registry \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=me@example.com
```

**Use in Pod:**
``` yaml
spec:
  imagePullSecrets:
  - name: my-registry
  containers:
  - name: app
    image: registry.example.com/myapp:1.0
```

---

## Working with ConfigMaps and Secrets

``` bash title="Essential Commands"
# List ConfigMaps
kubectl get configmaps
kubectl get cm  # Short form

# List Secrets
kubectl get secrets

# View ConfigMap content
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# View Secret (base64-encoded)
kubectl get secret db-credentials -o yaml

# Decode Secret
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d

# Edit ConfigMap
kubectl edit configmap app-config

# Delete
kubectl delete configmap app-config
kubectl delete secret db-credentials
```

!!! warning "ConfigMap/Secret Updates"
    When you update a ConfigMap or Secret:

    - **Environment variables:** NOT updated (need pod restart)
    - **Mounted files:** Updated automatically (may take ~60s)

    To force update: `kubectl rollout restart deployment/myapp`

---

## Best Practices

### ConfigMaps

✅ **Do:**
- Store non-sensitive configuration
- Use for feature flags, URLs, settings
- Version control your ConfigMap YAML
- Use descriptive names

❌ **Don't:**
- Store passwords or API keys (use Secrets)
- Make ConfigMaps huge (< 1MB recommended)
- Hard-code values in images

### Secrets

✅ **Do:**
- Use for passwords, API keys, certificates
- Mount as files when possible (not env vars)
- Set `readOnly: true` on mounts
- Limit access with RBAC (Level 5)
- Consider external secret management (Vault, AWS Secrets Manager)

❌ **Don't:**
- Commit Secrets to git
- Share Secret YAML files in Slack/email
- Use Secrets for large files
- Assume Secrets are encrypted (they're only base64-encoded!)

---

## Practice Exercises

??? question "Exercise 1: Application Configuration"
    Create a ConfigMap with database settings and use it in a pod.

    **Requirements:**
    - ConfigMap named `db-config` with keys: `host`, `port`, `database`
    - Pod that loads these as environment variables
    - Verify inside pod

    ??? tip "Solution"
        ``` yaml title="configmap.yaml"
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: db-config
        data:
          host: "postgres.default.svc.cluster.local"
          port: "5432"
          database: "myapp"
        ```

        ``` yaml title="pod.yaml"
        apiVersion: v1
        kind: Pod
        metadata:
          name: app
        spec:
          containers:
          - name: app
            image: busybox
            command: ['sh', '-c', 'env | grep DB_; sleep 3600']
            env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: host
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: port
            - name: DB_NAME
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: database
        ```

        ``` bash
        kubectl apply -f configmap.yaml
        kubectl apply -f pod.yaml
        kubectl logs app
        # Should show DB_ environment variables
        ```

??? question "Exercise 2: Sensitive Credentials"
    Create a Secret for database credentials and mount it as files.

    **Goal:** Credentials should be in `/etc/db/` directory inside pod.

    ??? tip "Solution"
        ``` bash
        # Create secret
        kubectl create secret generic db-creds \
          --from-literal=username=postgres \
          --from-literal=password=mysecretpassword
        ```

        ``` yaml title="pod-with-secret.yaml"
        apiVersion: v1
        kind: Pod
        metadata:
          name: app-with-secret
        spec:
          containers:
          - name: app
            image: busybox
            command: ['sh', '-c', 'cat /etc/db/username; cat /etc/db/password; sleep 3600']
            volumeMounts:
            - name: db-creds-volume
              mountPath: /etc/db
              readOnly: true
          volumes:
          - name: db-creds-volume
            secret:
              secretName: db-creds
        ```

        ``` bash
        kubectl apply -f pod-with-secret.yaml
        kubectl logs app-with-secret
        # Should show username and password
        ```

---

## Quick Recap

| Resource | Purpose | Use For | Stored As |
|----------|---------|---------|-----------|
| **ConfigMap** | Non-sensitive config | URLs, feature flags, settings | Plain text |
| **Secret** | Sensitive data | Passwords, API keys, certs | Base64-encoded |

**Usage patterns:**
- Environment variables (simple, but visible in `env`)
- Mounted files (more secure, auto-updates)

---

## Further Reading

### Official Documentation
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Configure Pods Using ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/)

### Deep Dives
- [Kubernetes Secrets Good Practices](https://kubernetes.io/docs/concepts/security/secrets-good-practices/)
- [External Secrets Operator](https://external-secrets.io/) - Integrate with Vault, AWS Secrets Manager
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) - Encrypt secrets in git

### Related Articles
- [Pods: What Actually Runs](pods.md) - How to use ConfigMaps/Secrets in pods
- [Namespaces](namespaces.md) - ConfigMaps/Secrets are namespace-scoped
- **RBAC** - Controlling secret access (coming in Level 5)

---

## What's Next?

You can now externalize configuration and manage secrets. Next, learn about **[Namespaces](namespaces.md)** to organize and isolate your resources.

---

**Remember:** Never commit secrets to git. Use ConfigMaps for configuration, Secrets for sensitive data, and external secret managers for production.
