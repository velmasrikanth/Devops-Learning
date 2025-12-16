# Kubernetes Core Concepts "Deep Dive"

This guide is designed to be **Interview Ready**. For every concept, we cover **What** it is, **Why** we use it, **Differences** from similar objects, and the **YAML** configuration.

---

# 1. Workloads (The "What")

## 1.1 Pods
**What:** The smallest deployable unit in Kubernetes. It represents a single instance of a running process.
**Why:** Example: "Why not run containers directly?" - Because Pods provide a wrapper that allows K8s to manage networking (shared IP), storage (shared volumes), and lifecycle.
**Interview Tip:** Never create Pods manually in production. They are ephemeral. If they die, they stay dead. Use a Controller (Deployment/DaemonSet) instead.

### Pod Lifecycle & Probes
| Probe | Purpose | Real World Use Case |
| :--- | :--- | :--- |
| **Liveness** | Restart if dead. | App enters a "deadlock" state where it's running but can't process requests. Restart fixes it. |
| **Readiness** | Cut traffic if busy/loading. | App is starting up and loading 5GB of data into memory. It takes 30s. Don't send traffic until done. |
| **Startup** | Wait for slow starts. | Legacy Java app takes 5 minutes to start. Liveness probe would kill it before it finishes. Startup probe pauses Liveness. |

#### YAML: Pod with Probes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-demo
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
```

## 1.2 Deployments
**What:** Manages a set of replicated Pods.
**Why:** To ensure High Availability. If a Node dies, the Deployment creates new Pods on other Nodes. It also handles **Updates** (Zero Downtime).

### Deployment Strategies (Interview Question)
**Q: Difference between RollingUpdate and Recreate?**
*   **RollingUpdate (Default):** Updates Pods incrementally (e.g., 1 by 1).
    *   *Result:* **Zero Downtime**.
    *   *Trade-off:* You have v1 and v2 running at the same time for a moment.
*   **Recreate:** Kills ALL old Pods, then creates ALL new Pods.
    *   *Result:* **Downtime** exists.
    *   *Trade-off:* Good if your DB schema changed and can't handle mixed versions.

#### YAML: Deployment (Rolling Update)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
```

## 1.3 StatefulSet
**What:** Manages stateful applications (Databases, Redis, Kafka).
**Why:** Unlike Deployments, Pods in a StatefulSet are NOT interchangeable.
**Difference from Deployment:**
*   **Deployment:** Pod names are random (`web-xc91k`). Order doesn't matter.
*   **StatefulSet:** Pod names are stable (`web-0`, `web-1`). Order matters (Start 0 -> then 1). Network ID is stable.

#### YAML: StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
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
        image: registry.k8s.io/nginx-slim:0.8
```

## 1.4 DaemonSet
**What:** Ensures that **ONE** copy of a Pod runs on **EVERY** Node in the cluster.
**Why:** Perfect for cluster-wide background processes.
**Use Cases:**
*   Log Collectors (Fluentd, Filebeat).
*   Monitoring Agents (Prometheus Node Exporter).

#### YAML: DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluentd:v1
```

## 1.5 Jobs & CronJobs
**What:** Pods that run **to completion** (they finish and stop).
**Difference from Pods:** Standard Pods are designed to run *forever* (Web servers). Jobs are designed to *finish* (Backups).

#### YAML: CronJob (Scheduled)
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"  # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args: [/bin/sh, -c, date; echo Hello]
          restartPolicy: OnFailure
```

---

# 2. Configuration (The "Settings")

## 2.1 ConfigMaps vs Secrets (Interview Question)
**Q: When to use ConfigMap vs Secret?**
*   **ConfigMap:** For **non-sensitive** configuration (UI theme colors, Hostnames, Port numbers). Stored in plain text.
*   **Secret:** For **sensitive** data (Passwords, API Keys, SSH Certificates). Stored in Base64 encoding.

## 2.2 ConfigMaps

#### YAML: ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-config
data:
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"
```

## 2.3 Secrets

#### YAML: Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  # echo -n "admin" | base64
  username: WRtaW4=
  # echo -n "secret123" | base64
  password: c2VjcmV0MTIz
```

---

# 3. Networking (The "Connectivity")

## 3.1 Services Types (Interview Question)
**Q: Explain ClusterIP vs NodePort vs LoadBalancer.**

1.  **ClusterIP (Default):**
    *   **Visibility:** Internal Only.
    *   **Use Case:** Backend services, Database.
2.  **NodePort:**
    *   **Visibility:** External (via Node IP + Port like 30007).
    *   **Use Case:** Quick debugging, or if you don't have a Cloud LoadBalancer.
3.  **LoadBalancer:**
    *   **Visibility:** External (Static Public IP).
    *   **Use Case:** Production HTTP/HTTPS traffic.

#### YAML: Service (NodePort)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service-np
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007 # Range: 30000-32767
```

## 3.2 Ingress
**What:** An intelligent HTTP Router (Layer 7).
**Why use Ingress instead of LoadBalancer Service?**
*   **Cost:** `LoadBalancer` Service creates 1 Cloud LB per service ($$$).
*   **Ingress:** Creates 1 Cloud LB that routes traffic to 50 different services based on URL path (`/api`, `/web`, `/auth`). It saves money and IP addresses.

#### YAML: Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  rules:
  - host: "foo.bar.com"
    http:
      paths:
      - path: /bar
        pathType: Prefix
        backend:
          service:
            name: service1
            port:
              number: 80
```

## 3.3 Network Policies
**What:** A Firewall for your Pods.
**Why:** By default, **ALL Pods can talk to ALL Pods**. This is a security risk. NetworkPolicy locks this down.

#### YAML: NetworkPolicy
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: access-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: "true"
```

---

# 4. Storage (The "Persistence")

## 4.1 PV vs PVC vs StorageClass (Interview Question)
*   **PersistentVolume (PV):** The actual hard drive (EBS volume, NFS export). Low-level resource.
*   **PersistentVolumeClaim (PVC):** A "ticket" asking for storage ("I need 10GB"). Developers create this.
*   **StorageClass:** The "Provisioner". It automates the creation of PVs.
    *   *Static:* Admin manually creates 10 PVs. User claims one.
    *   *Dynamic:* User creates PVC -> StorageClass talks to AWS -> AWS creates disk -> PV created automatically.

#### YAML: StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
```

#### YAML: PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: standard
```
