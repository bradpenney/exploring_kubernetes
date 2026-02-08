# Understanding Volumes

!!! tip "Part of Level 4: Storage and State"
    This is the first article in [Level 4: Storage and State](overview.md). Start here before diving into Persistent Volumes.

Your container writes log files, uploads user images, or generates reports. Then the pod restarts, and everything is gone. Container filesystems are **ephemeral**—they vanish when containers restart.

Volumes solve this: they let containers persist data beyond container restarts, share data between containers in a pod, and mount configuration files from ConfigMaps or Secrets.

---

## What You'll Learn

- What volumes are and why you need them
- Different volume types and their use cases
- How to mount volumes in pods
- EmptyDir, HostPath, ConfigMap, and Secret volumes
- When to use ephemeral vs persistent volumes

---

## The Problem: Ephemeral Containers

```mermaid
graph LR
    A[Container Starts] --> B[Write to /data]
    B --> C[Container Crashes]
    C --> D[Container Restarts]
    D --> E[/data is EMPTY]

    style A fill:#4a5568,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style C fill:#e53e3e,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style D fill:#48bb78,stroke:#cbd5e0,stroke-width:2px,color:#fff
    style E fill:#e53e3e,stroke:#cbd5e0,stroke-width:2px,color:#fff
```

**Without volumes:** Data written to container filesystem is lost on restart.

**With volumes:** Data persists across container restarts (and sometimes pod restarts, depending on volume type).

---

## Volume Types Overview

<div class="grid cards" markdown>

-   :material-folder-outline: **emptyDir**

    ---

    **Why it matters:** Share data between containers in the same pod.

    **Lifecycle:** Exists as long as pod exists, deleted when pod is deleted

    **Use cases:**
    - Scratch space for temporary data
    - Sharing files between containers in a pod
    - Cache that can be rebuilt

    **Example:**
    ``` yaml
    volumes:
    - name: cache
      emptyDir: {}
    ```

-   :material-harddisk: **hostPath**

    ---

    **Why it matters:** Access files on the node (host machine) filesystem.

    **Lifecycle:** Data persists on node even after pod deletion

    **Use cases:**
    - Accessing Docker socket (`/var/run/docker.sock`)
    - Debugging (accessing node logs)
    - **Rarely used in production** (tight node coupling)

    **Example:**
    ``` yaml
    volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
    ```

-   :material-file-document: **configMap**

    ---

    **Why it matters:** Mount configuration files into containers.

    **Lifecycle:** Backed by ConfigMap object, updates propagate to pods

    **Use cases:**
    - Application config files
    - Environment-specific settings
    - Non-sensitive configuration

    **Example:**
    ``` yaml
    volumes:
    - name: config
      configMap:
        name: app-config
    ```

-   :material-key-variant: **secret**

    ---

    **Why it matters:** Mount sensitive data (passwords, tokens) into containers.

    **Lifecycle:** Backed by Secret object, base64-encoded in etcd

    **Use cases:**
    - Database passwords
    - API keys
    - TLS certificates

    **Example:**
    ``` yaml
    volumes:
    - name: secrets
      secret:
        secretName: app-secrets
    ```

-   :material-database: **persistentVolumeClaim**

    ---

    **Why it matters:** Request persistent storage that survives pod deletion.

    **Lifecycle:** Data persists even after pod is deleted

    **Use cases:**
    - Databases
    - User uploads
    - Any data that must survive pod restarts

    **Example:**
    ``` yaml
    volumes:
    - name: data
      persistentVolumeClaim:
        claimName: postgres-pvc
    ```

</div>

---

## Mounting Volumes in Pods

=== "emptyDir: Temporary Storage"

    **Scenario:** Two containers sharing a scratch space.

    ``` yaml title="emptydir-pod.yaml" linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: shared-pod
    spec:
      containers:
      - name: writer
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
        - while true; do
            echo "$(date): Hello" >> /data/output.txt;
            sleep 5;
          done
        volumeMounts:
        - name: shared-data  # (1)!
          mountPath: /data   # (2)!

      - name: reader
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
        - tail -f /data/output.txt
        volumeMounts:
        - name: shared-data  # (3)!
          mountPath: /data

      volumes:
      - name: shared-data    # (4)!
        emptyDir: {}         # (5)!
    ```

    1. Volume mount name—must match volume definition
    2. Path inside container where volume is mounted
    3. Both containers mount the same volume
    4. Volume definition at pod level
    5. emptyDir creates temporary storage for the pod

    **Key insight:** Both containers can read/write to `/data`—it's the same filesystem.

=== "hostPath: Node Filesystem"

    **Scenario:** Accessing Docker socket from inside a pod (for container management tools).

    ``` yaml title="hostpath-pod.yaml" linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: docker-client
    spec:
      containers:
      - name: docker
        image: docker:latest
        command: ["/bin/sh", "-c", "docker ps"]  # (1)!
        volumeMounts:
        - name: docker-sock
          mountPath: /var/run/docker.sock  # (2)!

      volumes:
      - name: docker-sock
        hostPath:
          path: /var/run/docker.sock  # (3)!
          type: Socket
    ```

    1. Run `docker ps` from inside container—talks to node's Docker daemon
    2. Mount Docker socket inside container
    3. Path on the node (host machine)

    !!! warning "Avoid hostPath in Production"
        hostPath ties your pod to a specific node and poses security risks. Use it only for debugging or node-level tools.

=== "configMap: Configuration Files"

    **Scenario:** Mount an nginx config file from a ConfigMap.

    **ConfigMap:**
    ``` yaml title="nginx-configmap.yaml" linenums="1"
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: nginx-config
    data:
      nginx.conf: |
        server {
          listen 80;
          location / {
            return 200 'Hello from ConfigMap!';
          }
        }
    ```

    **Pod:**
    ``` yaml title="nginx-pod-with-config.yaml" linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        volumeMounts:
        - name: config  # (1)!
          mountPath: /etc/nginx/conf.d  # (2)!

      volumes:
      - name: config
        configMap:
          name: nginx-config  # (3)!
    ```

    1. Mount ConfigMap as a volume
    2. Config file appears at `/etc/nginx/conf.d/nginx.conf` inside container
    3. References the ConfigMap created above

    **Key insight:** ConfigMap data becomes files in the container—no rebuilding images for config changes.

=== "secret: Sensitive Data"

    **Scenario:** Mount database credentials as files.

    **Secret:**
    ``` bash title="Create secret from files"
    echo -n 'admin' > username.txt
    echo -n 'supersecret' > password.txt
    kubectl create secret generic db-credentials \
      --from-file=username=username.txt \
      --from-file=password=password.txt
    ```

    **Pod:**
    ``` yaml title="pod-with-secret-volume.yaml" linenums="1"
    apiVersion: v1
    kind: Pod
    metadata:
      name: app
    spec:
      containers:
      - name: app
        image: busybox
        command: ["/bin/sh", "-c"]
        args:
        - cat /secrets/username;
          cat /secrets/password;
          sleep 3600
        volumeMounts:
        - name: db-creds  # (1)!
          mountPath: /secrets  # (2)!
          readOnly: true  # (3)!

      volumes:
      - name: db-creds
        secret:
          secretName: db-credentials  # (4)!
    ```

    1. Mount secret as volume
    2. Files appear at `/secrets/username` and `/secrets/password`
    3. Secrets should always be read-only
    4. References the Secret object

    **Key insight:** Secrets become files—your app reads them like any other file.

---

## Volume vs VolumeMount

Understanding the difference is critical:

| Concept | Where Defined | Purpose |
|---------|--------------|---------|
| **Volume** | Pod spec (`spec.volumes`) | **Declares** the volume exists and its type |
| **VolumeMount** | Container spec (`spec.containers[].volumeMounts`) | **Mounts** the volume into container at a specific path |

**Think of it like:**
- **Volume:** "This pod has a USB drive named 'data'"
- **VolumeMount:** "Mount that USB drive at `/mnt/data` in this container"

---

## EmptyDir Options

emptyDir can be customized:

``` yaml title="emptydir-with-sizelimit.yaml" linenums="1"
volumes:
- name: cache
  emptyDir:
    medium: Memory  # (1)!
    sizeLimit: 1Gi  # (2)!
```

1. Store in RAM (tmpfs) instead of disk—faster but volatile
2. Limit size to 1GB (pod will be evicted if exceeded)

**Use `medium: Memory` for:**
- Temporary caches
- Build artifacts
- Data that must be fast and doesn't need disk persistence

---

## When to Use Each Volume Type

| Volume Type | Use Case | Persistence | Best For |
|------------|----------|-------------|----------|
| **emptyDir** | Temporary scratch space | Pod lifetime | Sharing data between containers, caches |
| **hostPath** | Node filesystem access | Node lifetime | Debugging, node-level tools |
| **configMap** | Configuration files | Kubernetes object | App config, feature flags |
| **secret** | Sensitive data | Kubernetes object | Passwords, API keys, certs |
| **persistentVolumeClaim** | Durable storage | Survives pod deletion | Databases, user uploads |

---

## Practice Exercises

??? question "Exercise 1: Share Data with emptyDir"
    Create a pod with two containers sharing data via emptyDir.

    **Goal:** Understand how containers in a pod can share a filesystem.

    ??? tip "Solution"
        ``` yaml title="shared-emptydir.yaml" linenums="1"
        apiVersion: v1
        kind: Pod
        metadata:
          name: shared-volume
        spec:
          containers:
          - name: producer
            image: busybox
            command: ["/bin/sh", "-c"]
            args:
            - while true; do
                date >> /shared/log.txt;
                sleep 2;
              done
            volumeMounts:
            - name: shared
              mountPath: /shared

          - name: consumer
            image: busybox
            command: ["/bin/sh", "-c"]
            args:
            - tail -f /shared/log.txt
            volumeMounts:
            - name: shared
              mountPath: /shared

          volumes:
          - name: shared
            emptyDir: {}
        ```

        ``` bash title="Deploy and watch logs"
        kubectl apply -f shared-emptydir.yaml

        # Watch consumer logs (should see producer's output)
        kubectl logs -f shared-volume -c consumer
        ```

        **What you learned:** emptyDir lets containers in the same pod share data.

??? question "Exercise 2: Mount ConfigMap as Volume"
    Create a ConfigMap and mount it as a file in a pod.

    **Goal:** Learn how to externalize configuration using ConfigMaps.

    ??? tip "Solution"
        ``` yaml title="configmap-and-pod.yaml" linenums="1"
        apiVersion: v1
        kind: ConfigMap
        metadata:
          name: app-config
        data:
          app.properties: |
            database.host=postgres
            database.port=5432
            log.level=INFO
        ---
        apiVersion: v1
        kind: Pod
        metadata:
          name: app
        spec:
          containers:
          - name: app
            image: busybox
            command: ["/bin/sh", "-c"]
            args:
            - cat /config/app.properties && sleep 3600
            volumeMounts:
            - name: config
              mountPath: /config

          volumes:
          - name: config
            configMap:
              name: app-config
        ```

        ``` bash title="Deploy and verify"
        kubectl apply -f configmap-and-pod.yaml

        # Check logs—should show config file content
        kubectl logs app

        # Verify file exists in container
        kubectl exec app -- cat /config/app.properties
        ```

        **What you learned:** ConfigMaps become files in containers—change ConfigMap, and pods get updated config.

??? question "Exercise 3: Use Secret as Volume"
    Create a secret and mount it as files in a pod.

    **Goal:** Understand how to securely inject credentials into applications.

    ??? tip "Solution"
        ``` bash title="Create secret"
        kubectl create secret generic api-keys \
          --from-literal=api-key=abc123 \
          --from-literal=api-secret=xyz789
        ```

        ``` yaml title="pod-with-secret.yaml" linenums="1"
        apiVersion: v1
        kind: Pod
        metadata:
          name: api-client
        spec:
          containers:
          - name: client
            image: busybox
            command: ["/bin/sh", "-c"]
            args:
            - echo "API Key: $(cat /secrets/api-key)";
              echo "API Secret: $(cat /secrets/api-secret)";
              sleep 3600
            volumeMounts:
            - name: secrets
              mountPath: /secrets
              readOnly: true

          volumes:
          - name: secrets
            secret:
              secretName: api-keys
        ```

        ``` bash title="Deploy and verify"
        kubectl apply -f pod-with-secret.yaml

        # Check logs—should show secret values
        kubectl logs api-client

        # Secrets appear as files
        kubectl exec api-client -- ls /secrets
        ```

        **What you learned:** Secrets are mounted as read-only files—never hardcode credentials in images.

---

## Quick Reference

### Common Volume Types

| Volume Type | YAML Snippet |
|------------|-------------|
| **emptyDir** | `emptyDir: {}` |
| **emptyDir (RAM)** | `emptyDir: {medium: Memory, sizeLimit: 1Gi}` |
| **hostPath** | `hostPath: {path: /host/path, type: Directory}` |
| **configMap** | `configMap: {name: my-config}` |
| **secret** | `secret: {secretName: my-secret}` |
| **persistentVolumeClaim** | `persistentVolumeClaim: {claimName: my-pvc}` |

---

## Quick Recap

- **Volumes solve ephemeral storage:** Container filesystems are wiped on restart
- **emptyDir:** Temporary storage, shared between containers in a pod
- **hostPath:** Access node filesystem (rarely used, security risk)
- **configMap:** Mount configuration files without rebuilding images
- **secret:** Securely inject credentials as files
- **Volume vs VolumeMount:** Declare volume at pod level, mount it in container spec
- **PersistentVolumeClaims:** For durable storage that survives pod deletion (covered next)

---

## Further Reading

### Official Documentation
- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/) - Official volumes reference
- [Volume Types](https://kubernetes.io/docs/concepts/storage/volumes/#volume-types) - Complete list of volume types
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/) - Using ConfigMaps
- [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) - Using Secrets

### Deep Dives
- [Storage in Kubernetes](https://kubernetes.io/docs/concepts/storage/) - Storage concepts overview
- [Projected Volumes](https://kubernetes.io/docs/concepts/storage/projected-volumes/) - Advanced volume mounting

### Related Articles
- [ConfigMaps and Secrets](../level_1/config_and_secrets.md) - Managing configuration
- [Persistent Volumes](persistent_volumes.md) - Durable storage (next article)

---

## What's Next?

You understand ephemeral volumes. Now learn how to request **durable storage** that survives pod deletion: **[Persistent Volumes (PV)](persistent_volumes.md)**.
