# Kubernetes Interview Q&A — 3–5 Years Experience

> Simple English | Bullet Points | YAML & Commands Included
> 30 Questions covering everything from basics to real-world troubleshooting

---

## Q01. What is Kubernetes and why is it used?

- Kubernetes (K8s) is an open-source tool that manages your containerized apps across a cluster of machines.
- You stop worrying about 'which server runs what' — Kubernetes handles that automatically.
- It keeps your apps running: if a container crashes, K8s restarts it automatically (self-healing).
- It scales your app up or down based on traffic — no manual work needed.
- It does rolling updates so your app stays live during deployments — zero downtime.
- Used heavily in microservices setups where you have 10s or 100s of small services to manage.

> **Key Insight:** Think of Kubernetes as an operating system for your cluster — it schedules, heals, and scales your apps so you don't have to.

---

## Q02. What are the core components of Kubernetes architecture?

- Kubernetes has two layers: the Control Plane (brain) and Worker Nodes (muscle).
- Control Plane components:
  - **API Server**: the front door — every `kubectl` command hits this first.
  - **etcd**: the cluster's database — stores all config and state.
  - **Scheduler**: decides which node a new pod should run on.
  - **Controller Manager**: constantly checks if actual state matches desired state.
- Worker Node components:
  - **kubelet**: the agent on each node — receives instructions from the API server.
  - **kube-proxy**: handles networking rules so services can talk to pods.
  - **Container Runtime**: actually runs containers (containerd, CRI-O, Docker).

**Example:**
```bash
# Check control plane health
kubectl get componentstatuses

# See all nodes and their roles
kubectl get nodes -o wide
```

> **Key Insight:** If the control plane goes down, existing pods keep running — but no new scheduling, healing, or scaling will happen until it recovers.

---

## Q03. What is a Pod in Kubernetes?

- A Pod is the smallest unit you can deploy in Kubernetes — not a container, but a wrapper around one or more containers.
- All containers in a Pod share the same IP address and can talk to each other via `localhost`.
- They also share the same storage volumes.
- Pods are ephemeral — they can die and be replaced anytime. Never rely on a Pod's IP directly.
- Most of the time you'll have one container per Pod. Multi-container pods are used for sidecars (logging, proxies).

**Example:**
```yaml
# Simple pod definition
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx:1.25
    ports:
    - containerPort: 80
```

> **Key Insight:** Pods are disposable. Always use a Deployment or StatefulSet to manage them — never create bare pods in production.

---

## Q04. What is a Deployment and how does it work?

- A Deployment tells Kubernetes: "I want 3 replicas of this app running at all times."
- Kubernetes creates a ReplicaSet under the hood to maintain that count.
- When you update the image or config, it does a rolling update — kills old pods gradually, brings up new ones.
- If the new version has issues, you can rollback with one command.
- Deployments are the standard way to run stateless apps in production.

**Example:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:v2
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

```bash
# Rollback if something goes wrong
kubectl rollout undo deployment/my-app

# Check rollout status
kubectl rollout status deployment/my-app
```

> **Key Insight:** Always set resource requests and limits in your Deployment. Without them, a noisy pod can starve other pods on the same node.

---

## Q05. What is a Service in Kubernetes?

- Pods get a new IP every time they restart — a Service gives you a stable IP and DNS name that never changes.
- It acts as a load balancer — traffic sent to the Service is spread across all matching Pods.
- Services find Pods using label selectors — if your Pod has label `app: my-app`, the Service routes to it.
- Even if you scale from 3 to 10 pods, the Service automatically routes to all of them.

**Example:**
```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-svc
spec:
  selector:
    app: my-app  # matches pod labels
  ports:
  - port: 80
    targetPort: 8080
  type: ClusterIP
```

```bash
# Check if service has endpoints (pods)
kubectl get endpoints my-app-svc
```

> **Key Insight:** If `kubectl get endpoints` shows `<none>`, your Service selector doesn't match your pod labels — that's the #1 cause of "service not working" issues.

---

## Q06. What are the different types of Services — ClusterIP, NodePort, LoadBalancer?

- **ClusterIP** (default): only accessible inside the cluster. Use this for internal service-to-service communication.
- **NodePort**: opens a port (30000–32767) on every node. External traffic can reach it via `NodeIP:NodePort`. Good for testing, not ideal for prod.
- **LoadBalancer**: creates a cloud load balancer (AWS ELB, GCP LB) automatically. The right choice for production external traffic.
- **ExternalName**: maps a Service to an external DNS name. Used to abstract third-party services.

**Example:**
```yaml
# LoadBalancer service for production
apiVersion: v1
kind: Service
metadata:
  name: my-app-lb
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - port: 443
    targetPort: 8080
```

```bash
# Check which external IP was assigned
kubectl get svc my-app-lb
```

> **Key Insight:** In production, use LoadBalancer or Ingress — never expose NodePort directly. NodePort is fine for local clusters and quick testing.

---

## Q07. What is a Namespace and why is it used?

- A Namespace is like a virtual cluster inside your real cluster — it isolates resources.
- Different teams or environments (dev, staging, prod) can share the same cluster without stepping on each other.
- You can set resource quotas per namespace — so the dev team can't consume all the cluster CPU.
- RBAC can be scoped per namespace — dev team only has access to their own namespace.
- Default namespaces: `default`, `kube-system` (system pods), `kube-public`, `kube-node-lease`.

**Example:**
```bash
# Create a namespace
kubectl create namespace dev-team

# Run commands in a specific namespace
kubectl get pods -n dev-team
```

```yaml
# Set resource quota for a namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

> **Key Insight:** Never run everything in the `default` namespace in production. Use namespaces to enforce isolation and apply resource limits per team.

---

## Q08. What is the difference between Deployment and StatefulSet?

- **Deployment**: for stateless apps — all pods are identical and interchangeable. Order doesn't matter.
- **StatefulSet**: for stateful apps like databases — each pod has a fixed name (`pod-0`, `pod-1`) and its own persistent storage.
- StatefulSet pods start and stop in order: `pod-0` starts first, `pod-2` terminates last.
- Each StatefulSet pod gets its own PersistentVolumeClaim that stays even if the pod restarts.
- Use StatefulSet for: MySQL, PostgreSQL, Redis, Elasticsearch, Kafka, Zookeeper.

**Example:**
```yaml
# StatefulSet for a database
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

> **Key Insight:** StatefulSets give pods identity + stable storage. If you just need multiple replicas of a web app, always use a Deployment.

---

## Q09. What is a DaemonSet and when would you use it?

- A DaemonSet ensures exactly one pod runs on every node — automatically, even when new nodes are added.
- It's used for cluster-wide infrastructure tools that every node needs.
- Common use cases: log collectors (Fluentd, Fluent Bit), monitoring agents (Node Exporter), network plugins (Calico, Weave), security agents.
- When a node is removed, its DaemonSet pod is garbage collected automatically.
- You can restrict DaemonSet to specific nodes using `nodeSelector` or affinity.

**Example:**
```yaml
# DaemonSet for log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.16
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

> **Key Insight:** DaemonSets are the right tool whenever you need something running on every node — not a Deployment with replicas equal to node count.

---

## Q10. What is Ingress and how does it work?

- Ingress is a Kubernetes resource that manages external HTTP/HTTPS access to your services.
- It works like a reverse proxy / API gateway at the cluster edge.
- You need an Ingress Controller (NGINX, Traefik, AWS ALB) for it to actually work — Ingress alone does nothing.
- Supports host-based routing: `api.myapp.com` → api-service, `app.myapp.com` → frontend-service.
- Supports path-based routing: `myapp.com/api` → api-service, `myapp.com/` → frontend.
- Handles SSL termination — attach a TLS secret and Ingress decrypts HTTPS for you.

**Example:**
```yaml
# Ingress with host + path routing + TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

> **Key Insight:** Ingress replaces the need for a LoadBalancer per service — one Ingress Controller handles all external traffic. Saves significant cloud cost.

---

## Q11. What are ConfigMaps and Secrets?

- **ConfigMap**: stores non-sensitive config data as key-value pairs — like env vars, config files, feature flags.
- **Secret**: stores sensitive data like passwords, API keys, TLS certs — encoded in base64 (not encrypted by default).
- Both can be injected into pods as environment variables or mounted as files.
- Separating config from code means you don't rebuild the image just to change a config value.
- For real security, use Sealed Secrets or HashiCorp Vault — base64 is not encryption.

**Example:**
```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info

---
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:  # k8s encodes to base64 automatically
  DB_PASSWORD: mysecretpass
```

```yaml
# Use in a pod
envFrom:
- configMapRef:
    name: app-config
- secretRef:
    name: db-secret
```

> **Key Insight:** Secrets are only base64-encoded, not encrypted. Enable Encryption at Rest in the API server and use external secret managers in production.

---

## Q12. What are liveness and readiness probes?

- **Liveness probe**: checks if the container is still alive. If it fails, Kubernetes kills and restarts the container.
- **Readiness probe**: checks if the container is ready to receive traffic. If it fails, the pod is removed from the Service endpoints.
- A pod can be alive (liveness passing) but not ready (readiness failing) — useful during app startup or warm-up.
- Both support 3 types: HTTP GET, TCP socket check, or exec (run a command inside the container).
- Misconfigured probes = most common cause of unnecessary pod restarts or traffic being sent to broken pods.

**Example:**
```yaml
# Probes configuration in a pod spec
containers:
- name: app
  image: my-app:v1
  livenessProbe:
    httpGet:
      path: /healthz
      port: 8080
    initialDelaySeconds: 15  # wait before first check
    periodSeconds: 10         # check every 10s
    failureThreshold: 3       # restart after 3 failures
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
    failureThreshold: 2
```

> **Key Insight:** Always add an `initialDelaySeconds` that's longer than your app's startup time — otherwise the liveness probe kills the pod before it's even ready.

---

## Q13. What is Horizontal Pod Autoscaler (HPA)?

- HPA automatically increases or decreases the number of pod replicas based on a metric — usually CPU or memory.
- It runs a control loop every 15 seconds, checks the metrics, and adjusts replicas if needed.
- Formula: `desired replicas = ceil(current replicas × current value / target value)`.
- It requires `metrics-server` to be installed in the cluster — HPA queries it for CPU/memory data.
- For advanced use cases (scale on queue depth, HTTP requests), use KEDA instead of basic HPA.
- Scale-down has a 5-minute stabilization window by default to prevent flapping.

**Example:**
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

```bash
# Check HPA status
kubectl get hpa my-app-hpa
```

> **Key Insight:** HPA needs resource requests set on pods to work. Without requests, the metrics-server can't calculate utilization percentage.

---

## Q14. What is RBAC in Kubernetes?

- RBAC (Role-Based Access Control) controls who can do what in your cluster.
- You define a Role (what actions are allowed on what resources), then bind it to a user or service account.
- Four key objects: `Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding`.
- Role + RoleBinding = namespace-scoped permissions.
- ClusterRole + ClusterRoleBinding = cluster-wide permissions.
- Principle of least privilege: give users/apps only the permissions they actually need — nothing more.

**Example:**
```yaml
# Role: read-only access to pods in 'dev' namespace
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]

---
# Bind the role to a user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
# Check what permissions a user has
kubectl auth can-i list pods --as=john -n dev
```

> **Key Insight:** Always audit RBAC in production. Over-permissive service accounts are a common attack vector — use `kubectl auth can-i` to verify permissions.

---

## Q15. What is the difference between Role and ClusterRole?

- **Role**: permissions scoped to a specific namespace only. A Role in namespace `dev` has no effect in `prod`.
- **ClusterRole**: permissions apply cluster-wide — across all namespaces, or for cluster-scoped resources.
- Cluster-scoped resources (nodes, PersistentVolumes, namespaces) can only be managed via ClusterRole.
- You can bind a ClusterRole with a RoleBinding — this limits its effect to one namespace. Useful for reusing common roles.
- ClusterRoleBinding gives full cluster-wide access — use sparingly.

**Example:**
```yaml
# ClusterRole: read nodes cluster-wide
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]

---
# ClusterRoleBinding: apply cluster-wide
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-global
subjects:
- kind: ServiceAccount
  name: monitoring-agent
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

> **Key Insight:** Prefer Role over ClusterRole whenever possible. Grant cluster-wide access only when the use case truly requires it.

---

## Q16. How does Kubernetes networking work internally?

- Every pod gets a unique IP — pods can talk to each other directly without NAT.
- This flat network model is implemented by CNI plugins — Kubernetes doesn't handle networking itself.
- `kube-proxy` runs on every node and manages iptables/IPVS rules so Service IPs route to the right pods.
- Pod-to-pod on same node: traffic goes through the virtual bridge (`cbr0` or similar).
- Pod-to-pod across nodes: traffic is routed via the CNI plugin (VXLAN overlay, BGP peering, etc.).
- Pod-to-Service: kube-proxy rewrites the destination IP from Service IP to a pod IP.

**Example:**
```bash
# Check pod IP addresses
kubectl get pods -o wide

# Check which CNI is installed
ls /etc/cni/net.d/

# Check kube-proxy mode
kubectl get configmap kube-proxy -n kube-system -o yaml | grep mode

# Test pod-to-pod connectivity
kubectl exec -it pod-a -- curl http://<pod-b-ip>:8080
```

> **Key Insight:** Kubernetes networking relies entirely on the CNI plugin you install. Calico and Cilium are the most production-ready — Calico for simplicity, Cilium for eBPF-based performance.

---

## Q17. What is CNI and how does it function?

- CNI (Container Network Interface) is a standard spec that defines how networking plugins should configure pod networking.
- When a pod is created, kubelet calls the CNI plugin to: assign an IP, create a network interface in the pod, and set up routing.
- When the pod is deleted, the CNI plugin cleans up — releases the IP, removes routes.
- Popular CNI plugins: Calico (BGP routing, NetworkPolicy support), Flannel (simple VXLAN), Cilium (eBPF, best observability), Weave.
- CNI also enforces NetworkPolicies — controls which pods can talk to each other.

**Example:**
```bash
# Install Calico CNI (common in production)
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

```yaml
# NetworkPolicy: only allow frontend to talk to backend
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

> **Key Insight:** Without a CNI plugin that supports NetworkPolicies (like Calico or Cilium), your NetworkPolicy objects exist but are completely ignored.

---

## Q18. How does Kubernetes handle service discovery?

- Kubernetes uses CoreDNS for automatic DNS-based service discovery — no manual config needed.
- Every Service gets a DNS name: `<service>.<namespace>.svc.cluster.local`
- Same namespace: you can use just the service name — `my-service`.
- Cross namespace: you must use the full name — `my-service.backend.svc.cluster.local`.
- Kubernetes also injects env vars into pods for each service: `MY_SERVICE_HOST` and `MY_SERVICE_PORT`.
- DNS is the preferred way — env vars are legacy and can cause ordering issues.

**Example:**
```bash
# DNS names for a service called 'api-svc' in 'backend' namespace:
# From same namespace:   api-svc
# From other namespace:  api-svc.backend
# Full FQDN (always works): api-svc.backend.svc.cluster.local

# Test DNS resolution from inside a pod
kubectl exec -it my-pod -- nslookup api-svc.backend.svc.cluster.local

# Check CoreDNS is running
kubectl get pods -n kube-system -l k8s-app=kube-dns

# Debug DNS issues
kubectl exec -it my-pod -- cat /etc/resolv.conf
```

> **Key Insight:** Always use FQDNs for cross-namespace calls in production. Short names only resolve in the same namespace — a very common production bug.

---

## Q19. What is etcd and how do you manage its backup and restore?

- etcd is the cluster's brain — it stores everything: pod specs, secrets, configs, node info, RBAC rules.
- If etcd data is lost without a backup, your entire cluster state is gone. This is critical to back up.
- It's a distributed key-value store — usually runs as 3 or 5 nodes for HA (tolerates 1 or 2 node failures).
- Backup: use `etcdctl snapshot save` — takes a point-in-time snapshot of the entire cluster state.
- Restore: stop the API server, restore the snapshot, update the etcd data dir path, restart services.
- Automate backups via a CronJob — snapshot to S3 or cloud storage every hour.

**Example:**
```bash
# Take a backup (run on etcd node or with certs)
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.crt \
  --cert=/etc/etcd/server.crt \
  --key=/etc/etcd/server.key

# Verify the backup
ETCDCTL_API=3 etcdctl snapshot status /backup/etcd-$(date +%F).db

# Restore from backup
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-2024-01-01.db \
  --data-dir=/var/lib/etcd-restore
```

> **Key Insight:** Test your etcd restore process before you need it. A backup you've never tested is a backup you can't trust.

---

## Q20. What are taints and tolerations?

- **Taint**: a marker on a node that says "do not schedule pods here unless they explicitly allow it."
- **Toleration**: added to a pod spec to say "I can handle this taint — schedule me here."
- Use cases: dedicated GPU nodes for ML workloads, reserved nodes for system pods, isolate prod from dev.
- Taint effects: `NoSchedule` (won't schedule), `PreferNoSchedule` (tries to avoid), `NoExecute` (evicts existing pods too).

**Example:**
```bash
# Taint a node (no pods unless they tolerate this)
kubectl taint nodes gpu-node gpu=true:NoSchedule

# Remove a taint
kubectl taint nodes gpu-node gpu=true:NoSchedule-
```

```yaml
# Pod with toleration to run on GPU node
apiVersion: v1
kind: Pod
metadata:
  name: ml-job
spec:
  tolerations:
  - key: "gpu"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: training
    image: tensorflow/tensorflow:latest-gpu
```

> **Key Insight:** Taints push pods away; tolerations let specific pods through. Use `NoExecute` to evict already-running pods from a node you want to drain.

---

## Q21. What are affinity and anti-affinity rules?

- **Node Affinity**: schedule a pod only on nodes with specific labels (e.g., only on SSD nodes, or `us-east-1a` zone).
- **Pod Affinity**: co-locate pods on the same node/zone — useful when services need low latency between them.
- **Pod Anti-Affinity**: spread pods across nodes/zones — so all replicas don't die if one node goes down.
- **Required** (hard rule): pod won't schedule if no matching node exists.
- **Preferred** (soft rule): Kubernetes tries to match, but schedules anyway if it can't.

**Example:**
```yaml
# Anti-affinity: spread replicas across nodes
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: my-app
        topologyKey: kubernetes.io/hostname
    # Node affinity: only schedule on SSD nodes
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
```

> **Key Insight:** Always add Pod Anti-Affinity to production Deployments with multiple replicas — otherwise all pods might land on the same node and go down together.

---

## Q22. How do you secure a Kubernetes cluster?

- Enable RBAC and give users/service accounts only the permissions they need.
- Use NetworkPolicies to restrict pod-to-pod traffic — default deny all, then whitelist what's needed.
- Scan container images in CI with Trivy or Grype — block CRITICAL CVEs from reaching prod.
- Never store secrets in git — use Sealed Secrets, Vault, or AWS Secrets Manager.
- Enable PodSecurity Admission to block privileged containers and host path mounts.
- Restrict API server access — use private endpoint + IP allowlist, not public.
- Enable audit logging on the API server — log who did what and when.
- Keep Kubernetes version up to date — patch CVEs regularly.
- Use mTLS between services via Istio or Linkerd to encrypt service-to-service traffic.

**Example:**
```bash
# Check for privileged pods (security risk)
kubectl get pods -A -o json | jq '.items[] | select(.spec.containers[].securityContext.privileged==true) | .metadata.name'

# Enable audit logging in API server (kube-apiserver flags):
# --audit-log-path=/var/log/kubernetes/audit.log
# --audit-policy-file=/etc/kubernetes/audit-policy.yaml

# Run CIS security benchmark
docker run --rm --pid=host --userns=host --cap-add audit_control \
  aquasec/kube-bench:latest --benchmark eks-1.2
```

> **Key Insight:** Run `kube-bench` against your cluster periodically. It checks your setup against the CIS Kubernetes Benchmark and flags security gaps.

---

## Q23. What is Helm and how does it simplify deployments?

- Helm is the package manager for Kubernetes — think of it like `apt` or `npm` but for K8s apps.
- A Helm Chart is a collection of YAML templates for an entire application (Deployment, Service, Ingress, ConfigMap, etc.).
- You can deploy a complex app with one command: `helm install my-app ./my-chart`
- Values files (`values.yaml`) let you customize the same chart for different environments — no duplicating YAML.
- Helm tracks releases — you can rollback to a previous version with one command.
- Public charts available on Artifact Hub — use community charts for tools like Prometheus, Grafana, Redis.

**Example:**
```bash
# Install an app using Helm
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-redis bitnami/redis -f prod-values.yaml

# Override values at install time
helm install my-app ./my-chart \
  --set replicaCount=3 \
  --set image.tag=v2.1.0 \
  --namespace production

# Upgrade and rollback
helm upgrade my-app ./my-chart --set image.tag=v2.2.0
helm rollback my-app 1  # roll back to revision 1

# List all releases
helm list -A
```

> **Key Insight:** Use Helm in production but always pin chart versions in your CI. A `helm upgrade` without a version pin can pull breaking changes from upstream.

---

## Q24. A Pod is stuck in CrashLoopBackOff — how do you troubleshoot it?

- CrashLoopBackOff means the container keeps starting, crashing, and Kubernetes is backing off before restarting it.
- **Step 1**: Check events — `kubectl describe pod` — look at the Events section at the bottom.
- **Step 2**: Check the last crash logs — `kubectl logs --previous` (logs from the crashed container).
- **Step 3**: Check if it's OOMKilled — describe pod will show `OOMKilled` under Last State. If yes, increase memory limit.
- **Step 4**: Check liveness probe — if the probe fires before the app is ready, it kills the pod. Increase `initialDelaySeconds`.
- **Step 5**: Check the container command/entrypoint — run it manually with `kubectl run` to test.
- **Step 6**: Exec into a running instance to inspect — `kubectl exec -it <pod> -- /bin/sh`

**Example:**
```bash
# Step-by-step debugging
kubectl describe pod my-pod -n prod

# Get logs from the previous crashed container
kubectl logs my-pod --previous

# If multi-container pod, specify container
kubectl logs my-pod -c my-container --previous

# Quickly test the container image manually
kubectl run debug --image=my-app:v2 --restart=Never \
  --command -- /bin/sh -c 'sleep 3600'

# Exec into the running debug pod
kubectl exec -it debug -- /bin/sh
```

> **Key Insight:** The `--previous` flag on `kubectl logs` is the fastest way to see why the container crashed. Always check this first before anything else.

---

## Q25. Your application is not accessible via Service — what steps will you take?

- **Step 1**: Check if the Service exists and has the right selector — `kubectl get svc my-svc -o yaml`
- **Step 2**: Check endpoints — `kubectl get endpoints my-svc` — if it shows `<none>`, the selector doesn't match any pod.
- **Step 3**: Check pod labels — `kubectl get pods --show-labels` — compare with Service selector.
- **Step 4**: Check pod readiness — `kubectl get pods` — if pods show `0/1 READY`, readiness probe is failing.
- **Step 5**: Test from inside the cluster — `kubectl exec` into another pod and curl the service directly.
- **Step 6**: Check NetworkPolicies — one might be blocking traffic to the pod.
- **Step 7**: Check Ingress config if external access is broken — verify host, path, and backend service name.

**Example:**
```bash
# Full diagnostic flow
kubectl get svc my-svc -o wide
kubectl get endpoints my-svc
kubectl get pods -l app=my-app --show-labels

# Test connectivity from inside the cluster
kubectl run curl-test --image=curlimages/curl --restart=Never \
  --rm -it -- curl http://my-svc.default.svc.cluster.local

# Check NetworkPolicy blocking traffic
kubectl get networkpolicies -n default

# Check Ingress rules
kubectl describe ingress my-ingress
```

> **Key Insight:** Empty endpoints (`kubectl get endpoints`) = label mismatch 99% of the time. Always check this before digging into networking.

---

## Q26. How would you perform zero-downtime deployment in Kubernetes?

- Use `RollingUpdate` strategy in your Deployment — this is the default and replaces pods gradually.
- Set `maxSurge: 1` (one extra pod during update) and `maxUnavailable: 0` (no pods go down before new ones are ready).
- Configure readiness probes — new pods only receive traffic after they pass the readiness check.
- Use `PreStop` hook with a sleep — gives the load balancer time to deregister the pod before it shuts down.
- Set `terminationGracePeriodSeconds` long enough for in-flight requests to complete.
- For high-risk changes: use canary or blue-green deployments via Argo Rollouts.

**Example:**
```yaml
# Zero-downtime deployment config
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: app
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 5"]
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
```

```bash
# Monitor rollout progress
kubectl rollout status deployment/my-app
```

> **Key Insight:** `maxUnavailable: 0` + readiness probe = no traffic hits a pod that isn't ready. These two together give you true zero-downtime deploys.

---

## Q27. A node is not ready — how do you investigate?

- **Step 1**: `kubectl describe node` — check the `Conditions` section. Look for `MemoryPressure`, `DiskPressure`, `PIDPressure`, `NetworkUnavailable`.
- **Step 2**: Check kubelet logs on the node — `journalctl -u kubelet -n 100` — this shows why kubelet lost connection.
- **Step 3**: Check if the container runtime is running — `systemctl status containerd`
- **Step 4**: Check disk space — `df -h` — full disk is a very common reason for NotReady.
- **Step 5**: Check network connectivity from the node to the control plane API server.
- **Step 6**: If node is truly dead, drain it and let pods reschedule — `kubectl drain --ignore-daemonsets`

**Example:**
```bash
# Check node status and conditions
kubectl describe node worker-node-1
kubectl get node worker-node-1 -o yaml | grep -A5 conditions

# SSH to node and check kubelet
systemctl status kubelet
journalctl -u kubelet -n 50 --no-pager

# Check disk and memory on the node
df -h
free -m

# Drain the node safely before maintenance
kubectl drain worker-node-1 \
  --ignore-daemonsets \
  --delete-emptydir-data

# Mark it unschedulable without draining
kubectl cordon worker-node-1
```

> **Key Insight:** Full disk is the #1 cause of NotReady nodes in production. Set up disk usage alerts at 70% and 85% — don't wait until 100%.

---

## Q28. How do you handle high CPU/memory usage in a cluster?

- **Step 1**: Find which pods are consuming the most — `kubectl top pods -A --sort-by=cpu`
- **Step 2**: Check if resource limits are set — unlimited pods can consume everything on the node.
- **Step 3**: Set or tighten resource requests and limits on heavy pods.
- **Step 4**: Enable HPA to automatically scale out when CPU spikes.
- **Step 5**: Enable VPA (Vertical Pod Autoscaler) to auto-tune requests/limits based on historical usage.
- **Step 6**: Enable Cluster Autoscaler — if all nodes are full, it provisions new nodes automatically.
- **Step 7**: Long-term: profile and optimize the app — sometimes a code change is better than throwing more nodes at it.

**Example:**
```bash
# Find top CPU/memory consuming pods
kubectl top pods -A --sort-by=cpu
kubectl top nodes

# Check pods without resource limits (risk!)
kubectl get pods -A -o json | jq '.items[] |
  select(.spec.containers[].resources.limits == null) |
  .metadata.name'
```

```yaml
# VPA to auto-tune resource requests
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: Auto
```

> **Key Insight:** Pods without resource limits can OOMKill or CPU-starve your entire node. Always set limits — use `LimitRange` to enforce defaults at namespace level.

---

## Q29. How do you scale applications automatically in Kubernetes?

- **HPA** (Horizontal Pod Autoscaler): scales pod replicas based on CPU, memory, or custom metrics. Best for stateless apps.
- **VPA** (Vertical Pod Autoscaler): adjusts CPU/memory requests automatically based on actual usage. Best for right-sizing.
- **Cluster Autoscaler**: adds or removes nodes when pods are pending or nodes are underutilized.
- **KEDA** (Kubernetes Event-Driven Autoscaler): scales based on external events — Kafka queue depth, SQS messages, HTTP request rate, cron schedule.
- Don't use HPA and VPA together on the same resource — they conflict. Use HPA for replicas, VPA for resource sizes.

**Example:**
```bash
# HPA on CPU
kubectl autoscale deployment my-app --cpu-percent=70 --min=2 --max=20
```

```yaml
# KEDA: scale on Kafka consumer lag
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: kafka-scaler
spec:
  scaleTargetRef:
    name: my-consumer
  minReplicaCount: 1
  maxReplicaCount: 30
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka:9092
      consumerGroup: my-group
      topic: my-topic
      lagThreshold: "100"
```

```bash
# Watch HPA in action
kubectl get hpa --watch
```

> **Key Insight:** HPA scales pods; Cluster Autoscaler scales nodes. You need both working together for a truly elastic cluster. Set up CA before you hit a scaling emergency.

---

## Q30. How do you monitor and log Kubernetes workloads effectively?

- **Metrics**: deploy Prometheus to scrape metrics + Grafana for dashboards. Use pre-built K8s dashboards (`kube-state-metrics`, `node-exporter`).
- **Alerting**: configure Alertmanager — PagerDuty for P1, Slack for P2/P3.
- **Logging**: deploy Fluent Bit as a DaemonSet — it collects logs from every node and ships to Elasticsearch or Loki.
- Use Grafana Loki + Promtail for a lightweight, cost-effective logging stack.
- **Tracing**: add OpenTelemetry SDK to your services, ship traces to Jaeger or Tempo.
- **Unified**: use `kube-prometheus-stack` Helm chart — installs Prometheus, Alertmanager, Grafana, and all dashboards in one go.

**Example:**
```bash
# Install full monitoring stack with Helm
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

# Fluent Bit DaemonSet for log shipping
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit \
  --namespace logging --create-namespace

# Quick log check
kubectl logs -f deployment/my-app -n prod
kubectl logs -f deployment/my-app -n prod --since=1h

# Check resource usage
kubectl top pods -n prod
kubectl top nodes
```

> **Key Insight:** Install the `kube-prometheus-stack` Helm chart on day one of every new cluster. You want monitoring before you need it — not after an incident.
