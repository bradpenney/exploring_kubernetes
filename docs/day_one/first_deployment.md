# Your First Deployment

!!! tip "Part of Day One: Getting Started"
    This is the third article in [Day One: Getting Started](overview.md). Make sure you've completed [Getting kubectl Access](kubectl_access.md) first.

You're connected to your cluster. You have kubectl working. Now let's deploy something.

We're going to deploy a simple web application and see it run in Kubernetes. This is the moment it becomes real.

---

## What We're Deploying

A simple nginx web server. Why nginx?
- Small image (downloads fast)
- Well-known and stable
- Easy to test (just open a browser)
- Same principles apply to YOUR application later

---

## The Deployment YAML

``` yaml title="nginx-deployment.yaml" linenums="1"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-first-app
  labels:
    app: nginx
spec:
  replicas: 2  # Two copies for reliability
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

**Don't memorize this.** Just understand:
- `kind: Deployment` - We're creating a Deployment (manages Pods for us)
- `replicas: 2` - We want 2 copies running
- `image: nginx:1.21` - What container to run
- `containerPort: 80` - What port the app listens on

---

## Deploying It

### Step 1: Create the file

Create `nginx-deployment.yaml` with the content above.

### Step 2: Apply it

```bash
kubectl apply -f nginx-deployment.yaml
# deployment.apps/my-first-app created
```

**That's it.** Kubernetes is now creating your Pods.

### Step 3: Watch it come up

```bash
kubectl get pods
# NAME                            READY   STATUS              RESTARTS   AGE
# my-first-app-7c5ddbdf54-2xkqn   0/1     ContainerCreating   0          5s
# my-first-app-7c5ddbdf54-8mz4p   0/1     ContainerCreating   0          5s
```

Wait a few seconds and check again:

```bash
kubectl get pods
# NAME                            READY   STATUS    RESTARTS   AGE
# my-first-app-7c5ddbdf54-2xkqn   1/1     Running   0          20s
# my-first-app-7c5ddbdf54-8mz4p   1/1     Running   0          20s
```

**STATUS: Running** - SUCCESS! 🎉

---

## Making It Accessible

Your Pods are running, but you can't access them yet. They're isolated inside the cluster. We need a **Service**.

``` yaml title="nginx-service.yaml" linenums="1"
apiVersion: v1
kind: Service
metadata:
  name: my-first-app-svc
spec:
  type: NodePort  # Exposes externally for testing
  selector:
    app: nginx  # Matches our Deployment's labels
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Access at http://<node-ip>:30080
```

**Apply the Service:**

```bash
kubectl apply -f nginx-service.yaml
# service/my-first-app-svc created

kubectl get service my-first-app-svc
# NAME                TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
# my-first-app-svc    NodePort   10.96.123.45    <none>        80:30080/TCP   10s
```

---

## Testing It

### Option 1: Port Forward (Easiest)

```bash
kubectl port-forward service/my-first-app-svc 8080:80
# Forwarding from 127.0.0.1:8080 -> 80
```

Open browser: `http://localhost:8080`

You should see: **Welcome to nginx!**

### Option 2: From Inside the Cluster

```bash
# Create a test pod
kubectl run test --image=busybox -it --rm -- sh

# Inside the pod:
wget -O- http://my-first-app-svc
# Shows nginx HTML
```

---

## What Just Happened?

Let's trace what Kubernetes did:

1. **You ran kubectl apply** - Told K8s what you want
2. **Deployment created** - K8s created a Deployment object
3. **ReplicaSet created** - Deployment created a ReplicaSet
4. **Pods created** - ReplicaSet created 2 Pods
5. **Containers started** - Pods pulled nginx image and started containers
6. **Service created** - K8s created a load balancer pointing to your Pods

**All automatic. You just declared what you wanted.**

---

## Exploring Your Deployment

```bash
# See everything
kubectl get all

# Detailed Deployment info
kubectl describe deployment my-first-app

# Pod logs
kubectl logs my-first-app-7c5ddbdf54-2xkqn

# Execute command in pod
kubectl exec my-first-app-7c5ddbdf54-2xkqn -- nginx -v
```

---

## Scaling Up

Want more copies? Change replicas:

```bash
kubectl scale deployment my-first-app --replicas=5
# deployment.apps/my-first-app scaled

kubectl get pods
# Now you have 5 pods!
```

---

## Cleaning Up

When done testing:

```bash
kubectl delete deployment my-first-app
kubectl delete service my-first-app-svc

# Or delete from files:
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

---

## Practice Exercise

??? question "Exercise: Deploy Your Own App"
    **Goal:** Deploy nginx with 3 replicas, expose it with a Service, and access it.

    **Steps:**
    1. Create deployment YAML with 3 replicas
    2. Apply it
    3. Create service YAML
    4. Apply it
    5. Access it with port-forward
    6. Scale to 5 replicas
    7. Clean up

    ??? tip "Solution"
        [Full solution with YAML files]

---

## Quick Recap

| What You Did | Command |
|--------------|---------|
| **Created Deployment** | `kubectl apply -f deployment.yaml` |
| **Checked Pods** | `kubectl get pods` |
| **Created Service** | `kubectl apply -f service.yaml` |
| **Accessed App** | `kubectl port-forward` |
| **Scaled** | `kubectl scale deployment` |
| **Cleaned Up** | `kubectl delete` |

---

## Further Reading

- [Deployments Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Services Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)
- [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply)

---

## What's Next?

You've deployed your first application! Now learn the **[Essential kubectl Commands](essential_commands.md)** you'll use every single day.

---

**Congratulations!** You're officially running applications on Kubernetes.
