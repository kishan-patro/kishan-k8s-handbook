# Kubernetes Interview Prep 07

100-question reference covering fundamentals, networking, scaling, security, storage, advanced concepts, and real-world scenarios. Aimed at 2–5 years experience level.

---

## I. Kubernetes Fundamentals and Architecture

### 1. What is Kubernetes and why is it used?

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It solves the challenges of managing large, distributed applications across multiple servers by providing:

- Automated scheduling
- Self-healing (restarts failed containers, reschedules on failed nodes)
- Rolling updates and rollbacks
- Service discovery and load balancing
- Horizontal and vertical scaling

---

### 2. Explain the core components of Kubernetes architecture.

**Control Plane:**
- **kube-apiserver** — Front-end of the control plane. All cluster communication goes through it.
- **etcd** — Distributed key-value store. Stores all cluster state, config, and metadata.
- **kube-scheduler** — Assigns newly created Pods to nodes based on resource requirements and constraints.
- **kube-controller-manager** — Runs controller loops (Node, Replication, Endpoints, ServiceAccount controllers).

**Worker Nodes:**
- **kubelet** — Agent on each node. Ensures containers are running as described in PodSpecs.
- **kube-proxy** — Maintains network rules on nodes. Enables service routing to Pods.
- **Container Runtime** — Runs containers. Examples: containerd, CRI-O.

---

### 3. What is a Pod? Why is it the smallest deployable unit?

A Pod is the smallest deployable unit in Kubernetes — it wraps one or more containers. Containers in a Pod share:

- The same network namespace and IP address (communicate via `localhost`)
- The same storage volumes

This tight coupling makes the Pod the atomic unit of scheduling and scaling.

---

### 4. What is the relationship between Kubernetes and Docker?

- **Docker** packages and runs applications in isolated containers.
- **Kubernetes** orchestrates those containers across a cluster — scheduling, healing, scaling, and networking.

> "Docker builds the container. Kubernetes manages the cluster."

Kubernetes works with any OCI-compliant container runtime (containerd, CRI-O), not just Docker.

---

### 5. Explain Namespaces. When would you use them?

Namespaces are virtual clusters within a physical cluster. Use them for:

- **Resource isolation** — Separate dev, staging, prod on the same cluster.
- **Access control** — Apply RBAC policies per namespace.
- **Resource quotas** — Cap CPU/memory usage per team or environment.

Default namespaces: `default`, `kube-system`, `kube-public`, `kube-node-lease`.

---

### 6. What are Labels and Selectors?

- **Labels** — Key-value pairs attached to objects (Pods, Services, Deployments) for identification and grouping.
- **Selectors** — Filter objects by their labels. Services use selectors to route traffic to Pods; Deployments use them to manage their Pods.

---

### 7. What is a Service? Why is it needed?

A Service provides a stable IP and DNS name for a group of Pods. Pods are ephemeral — their IPs change on restart. A Service abstracts over this with:

- A fixed ClusterIP (internal)
- A DNS name: `<service>.<namespace>.svc.cluster.local`
- Load balancing across matching Pods via label selectors

---

### 8. Differentiate between ClusterIP, NodePort, and LoadBalancer.

| Type | Access | Use case |
|------|--------|----------|
| **ClusterIP** | Internal only | Service-to-service communication |
| **NodePort** | `NodeIP:NodePort` (30000–32767) | Testing, non-prod external access |
| **LoadBalancer** | Cloud load balancer provisioned automatically | Production external traffic |
| **ExternalName** | Maps to an external DNS name | Abstracting third-party services |

---

### 9. What is a Deployment?

A Deployment manages the lifecycle of Pods and ReplicaSets. It provides:

- Declarative desired state (e.g., "run 3 replicas of this image")
- Rolling updates — gradually replaces old pods with new
- Rollback — `kubectl rollout undo deployment/<name>`
- Self-healing — restarts pods that fail

---

### 10. What is the difference between a ReplicaSet and a Deployment?

- **ReplicaSet** — Ensures a specified number of identical Pod replicas are running. Low-level controller.
- **Deployment** — Manages ReplicaSets. Adds rolling updates, rollback, and versioning on top. Always prefer Deployments for stateless apps.

---

### 11. What is a StatefulSet and when would you use it?

StatefulSet manages stateful applications with:

- **Stable, unique network identifiers** — Each pod gets a predictable hostname (`pod-0`, `pod-1`).
- **Stable, persistent storage** — Each pod gets its own PVC via `volumeClaimTemplates`.
- **Ordered deployment and scaling** — Pods start and stop in order.

Use for: MySQL, PostgreSQL, Kafka, Zookeeper, Elasticsearch, Redis (clustered).

---

### 12. What is a DaemonSet and when would you use it?

A DaemonSet ensures one pod runs on every node (or a subset). Use for:

- Log collectors: Fluentd, Fluent Bit
- Monitoring agents: Prometheus Node Exporter
- Network plugins: Calico, Cilium
- Security agents

When a new node joins, the DaemonSet pod is automatically scheduled on it.

---

### 13. Explain ConfigMaps and Secrets. What's the difference?

| | ConfigMap | Secret |
|---|-----------|--------|
| **Data type** | Non-sensitive config (env vars, config files) | Sensitive data (passwords, tokens, TLS certs) |
| **Encoding** | Plain text | base64-encoded |
| **Encryption** | No | Optional (requires encryption at rest) |

Both can be injected as env vars or mounted as files. Use Sealed Secrets or Vault for production secret management.

---

### 14. What is Ingress? How does it differ from a LoadBalancer Service?

- **Ingress** — Layer 7 (HTTP/HTTPS). Routes traffic by hostname or path. Requires an Ingress Controller (NGINX, Traefik, AWS ALB).
- **LoadBalancer Service** — Layer 4 (TCP/UDP). Provisions a cloud load balancer per service. Simpler but more expensive.

Ingress: one controller handles all external HTTP traffic. LoadBalancer: one cloud LB per service.

---

### 15. How does Kubernetes handle storage orchestration?

- **PersistentVolume (PV)** — Cluster-level storage provisioned by admin or dynamically.
- **PersistentVolumeClaim (PVC)** — User's request for storage (size + access mode). Kubernetes binds it to a matching PV.
- **StorageClass** — Defines storage tiers (e.g., "fast SSD", "standard"). Enables dynamic provisioning.

---

## II. Networking

### 16. Explain the Kubernetes networking model.

Kubernetes enforces a flat network model:
- All Pods can communicate with all other Pods without NAT.
- All Nodes can communicate with all Pods without NAT.
- A Pod's IP is the same from inside and outside.

Implemented by a CNI plugin (Calico, Flannel, Cilium).

---

### 17. How does Pod-to-Pod communication work on the same node?

The CNI creates a virtual Ethernet pair per Pod — one end in the Pod's network namespace, the other on a bridge on the node. Pods on the same node communicate via this bridge.

---

### 18. How does Pod-to-Pod communication work across different nodes?

The CNI plugin encapsulates Pod traffic (VXLAN, IPIP, BGP) and routes it between nodes. The overlay or underlay network handles inter-node routing transparently.

---

### 19. What are Network Policies? When would you use them?

Network Policies define rules for which Pods can communicate with each other. Use them to:

- Isolate frontend, backend, and database tiers.
- Restrict access to sensitive services.
- Implement micro-segmentation (default-deny, then whitelist).

Requires a CNI that enforces them (Calico, Cilium). Without a supporting CNI, policies exist but are ignored.

---

### 20. How does Kubernetes handle DNS for services and pods?

Kubernetes runs **CoreDNS** as the internal DNS server.

- **Service**: `<service>.<namespace>.svc.cluster.local`
- **Pod**: `<pod-ip-dashes>.<namespace>.pod.cluster.local`
- Same namespace: use short name — `my-service`
- Cross-namespace: use FQDN — `my-service.backend.svc.cluster.local`

---

## III. Deployment and Scaling

### 21. Describe the process of a rolling update.

Kubernetes creates new Pods with the updated image, waits for them to pass readiness checks, then gradually terminates old Pods. Controlled by `maxSurge` (extra pods during update) and `maxUnavailable` (max pods down at once).

---

### 22. How do you perform a rollback of a Deployment?

```bash
kubectl rollout undo deployment/<name>
kubectl rollout undo deployment/<name> --to-revision=<number>
kubectl rollout history deployment/<name>  # view revision history
```

---

### 23. What is HPA? How does it work?

HPA (Horizontal Pod Autoscaler) automatically scales pod replicas based on CPU, memory, or custom metrics. It runs a control loop every 15 seconds and uses the formula:

```
desired replicas = ceil(current replicas × current metric / target metric)
```

Requires `metrics-server`. For event-driven scaling (Kafka, SQS), use **KEDA**.

---

### 24. What is VPA?

VPA (Vertical Pod Autoscaler) automatically adjusts CPU and memory **requests** and **limits** for containers based on historical usage. Helps right-size pods. Note: VPA may reschedule pods to apply changes.

---

### 25. What is Cluster Autoscaler?

Cluster Autoscaler adds nodes when pods can't be scheduled due to insufficient resources, and removes underutilized nodes. Works with cloud provider node groups (ASGs in AWS).

> HPA scales pods. Cluster Autoscaler scales nodes. You need both for a truly elastic cluster.

---

### 26. How would you troubleshoot a failed Deployment?

```bash
kubectl get deployments
kubectl describe deployment <name>        # events, replica status
kubectl get replicasets
kubectl get pods -l app=<label>
kubectl describe pod <pod-name>           # events, volumes, container state
kubectl logs <pod-name>
kubectl logs <pod-name> --previous        # logs from crashed container
kubectl exec -it <pod-name> -- /bin/sh
```

If no pods are scheduling: check kube-scheduler logs. If pods are stuck Pending: check kubelet logs on the node.

---

### 27. Explain Liveness and Readiness Probes.

| Probe | Failure action | Purpose |
|-------|---------------|---------|
| **Liveness** | Container is restarted | Detects deadlocked or hung containers |
| **Readiness** | Pod removed from Service endpoints | Prevents traffic to unready pods |
| **Startup** | Container is restarted | Gives slow-starting apps time to initialize |

Always set `initialDelaySeconds` longer than your app's startup time to avoid premature kills.

---

### 28. What is a CronJob? Give a use case.

A CronJob creates Jobs on a repeating schedule (cron syntax).

**Use cases:** Daily database backups, weekly report generation, log cleanup, cache warming.

```yaml
schedule: "0 2 * * *"   # run at 2am every day
```

---

## IV. Security and Access Control

### 29. Explain RBAC in Kubernetes.

RBAC controls **who** can do **what** to **which resources**.

| Object | Scope |
|--------|-------|
| `Role` | Namespace-scoped permissions |
| `ClusterRole` | Cluster-wide permissions |
| `RoleBinding` | Binds a Role to a subject within a namespace |
| `ClusterRoleBinding` | Binds a ClusterRole to a subject cluster-wide |

Subjects: Users, Groups, ServiceAccounts.

---

### 30. What are Service Accounts? How are they used?

A ServiceAccount provides an identity for processes running in Pods. Used to:

- Allow Pods to call the Kubernetes API.
- Authenticate with cloud providers (e.g., IRSA on EKS).
- Control which cluster resources a Pod can access via RBAC.

A default ServiceAccount is created in every namespace.

---

### 31. How do you secure access to the Kubernetes API server?

- **Authentication** — Client certificates, tokens, OIDC, cloud provider auth.
- **Authorization (RBAC)** — Granular permissions.
- **Admission Controllers** — Enforce policies before objects are persisted.
- **Network restrictions** — Private endpoint + IP allowlist.
- **Encryption in transit** — TLS for all communication.
- **Audit logging** — Log all API server activity.

---

### 32. What are Pod Security Standards (PSS)?

PSS define security best practices for Pods, replacing the deprecated PodSecurityPolicy:

| Level | Description |
|-------|-------------|
| **Privileged** | Unrestricted. For highly trusted workloads. |
| **Baseline** | Prevents known privilege escalations. |
| **Restricted** | Hardened. Enforces best practices (no root, read-only root FS, etc.). |

Enforced via the `PodSecurity` admission controller by labelling namespaces.

---

## V. Monitoring and Logging

### 33. How do you monitor a Kubernetes cluster?

- **Metrics**: Prometheus (scraping) + Grafana (dashboards + alerts).
- **Logging**: Fluent Bit/Fluentd → Elasticsearch/Loki. Visualize with Kibana or Grafana.
- **Tracing**: OpenTelemetry → Jaeger or Tempo.
- **Quick checks**: `kubectl top nodes`, `kubectl top pods`.
- **Cloud-native**: CloudWatch Container Insights (AWS), Google Cloud Monitoring, Azure Monitor.

Deploy `kube-prometheus-stack` Helm chart on day one — it installs Prometheus, Alertmanager, Grafana, and all dashboards together.

---

### 34. How do you collect logs from applications?

- Applications should write to **stdout/stderr** — the container runtime captures these.
- Deploy **Fluent Bit as a DaemonSet** to collect logs from each node and forward to Elasticsearch or Loki.
- For apps that write to files: use a **sidecar container** to tail the log file from a shared volume.

---

### 35. What is the role of Prometheus in Kubernetes monitoring?

Prometheus scrapes metrics from targets (nodes, pods, K8s components) at configured intervals and stores them in a time-series database. It drives alerts via Alertmanager and feeds dashboards in Grafana.

---

## VI. Advanced Topics and Troubleshooting

### 36. When would you use a CRD and Operator?

**Scenario**: Managing a distributed database like Cassandra.

- **CRD** defines the desired state: number of nodes, version, backup schedule.
- **Operator** (Custom Controller) watches for CRD changes and automates day-2 operations: provisioning nodes, handling upgrades, taking backups, recovering from failures.

Operators encode application-specific operational knowledge into code.

---

### 37. Explain Taints and Tolerations.

- **Taint** — Applied to a node: "don't schedule pods here unless they explicitly allow it."
- **Toleration** — Applied to a pod: "I can run on nodes with this taint."

Taint effects:
- `NoSchedule` — Won't schedule new pods.
- `PreferNoSchedule` — Tries to avoid scheduling.
- `NoExecute` — Evicts already-running pods too.

**Use cases:** Dedicated GPU nodes, isolating prod from dev, node maintenance.

---

### 38. What are Node Affinity and Anti-Affinity?

- **Node Affinity** — Schedule pods on nodes with specific labels (e.g., SSD nodes, specific AZ).
  - `requiredDuringScheduling...` — Hard requirement.
  - `preferredDuringScheduling...` — Soft preference.
- **Pod Anti-Affinity** — Spread pods across nodes/zones so they don't all land on the same node.

Always add `podAntiAffinity` to production Deployments with multiple replicas.

---

### 39. How would you debug a Pod stuck in Pending?

```bash
kubectl describe pod <pod-name>
```

Look for:
- **Insufficient CPU/Memory** — Cluster is full. Check node capacity.
- **No matching node** — nodeSelector, taints, or affinity rules don't match any node.
- **PVC not bound** — PersistentVolumeClaim waiting for a PersistentVolume.
- **CNI issues** — Network plugin not functioning correctly on nodes.

```bash
kubectl get events --field-selector involvedObject.name=<pod-name>
kubectl top nodes
```

---

### 40. How would you troubleshoot an application unreachable from outside?

1. Check Service type — is it `NodePort` or `LoadBalancer`? `ClusterIP` is internal only.
2. `kubectl describe service <name>` — check Endpoints. Empty = label mismatch.
3. `kubectl get pods -l <selector>` — are pods Running and Ready?
4. Check Ingress — `kubectl describe ingress <name>`. Verify rules, backend service names, events.
5. Check Ingress Controller pods are healthy. Check its logs.
6. Verify firewall/security groups allow traffic to the NodePort or LB IP.
7. Verify DNS records point to the correct Ingress Controller IP.
8. Check kube-proxy is running on all nodes.

---

### 41. What is a Helm chart? Why is it useful?

Helm is the package manager for Kubernetes. A chart bundles all K8s resources for an application.

Benefits:
- **Packaging** — Version and distribute your entire app as one unit.
- **Templating** — Parameterize YAML for different environments via `values.yaml`.
- **Lifecycle management** — `helm upgrade`, `helm rollback`, `helm list`.
- **Reuse** — Community charts for Prometheus, Redis, Grafana, etc.

---

### 42. How does Kubernetes handle self-healing?

- **Liveness probes** restart unhealthy containers.
- **ReplicaSets/Deployments** maintain the desired pod count.
- **Controller manager** reschedules pods from failed nodes to healthy ones.
- **Rollback** — Failed deployments can be reverted automatically or manually.

---

### 43. What is a Pod Disruption Budget (PDB)?

A PDB limits voluntary disruptions by specifying the minimum number of pods that must remain available during operations like `kubectl drain` or cluster upgrades.

```yaml
spec:
  minAvailable: 3     # or use maxUnavailable: 1
  selector:
    matchLabels:
      app: my-app
```

Use for stateful apps and mission-critical services that can't tolerate a majority going down simultaneously.

---

### 44. How does `kubectl exec` work?

`kubectl exec` establishes a secure bidirectional stream (similar to SSH) between your local machine and a container in a pod. It enables you to run commands or open an interactive shell inside a running container without needing direct node access.

---

### 45. What is GitOps and how does it relate to Kubernetes?

GitOps uses Git as the single source of truth for cluster state. All Kubernetes manifests and Helm charts are stored in Git. An agent (Argo CD, Flux) continuously watches the repo and reconciles the cluster to match the desired state.

Benefits: Auditable deployments, fast disaster recovery, consistent environments, easier collaboration.

---

## VII. Real-World Scenarios

### 46. Pods are constantly restarting. How do you investigate?

```bash
kubectl get pods                  # check RESTARTS column
kubectl describe pod <name>       # look for OOMKilled, CrashLoopBackOff
kubectl logs <name>               # current logs
kubectl logs <name> --previous    # logs from the crashed container
```

Common causes: OOMKilled (increase memory limit), liveness probe too aggressive (increase `initialDelaySeconds`), application bug.

---

### 47. Deploy a new version with zero downtime.

Use `RollingUpdate` strategy with:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

Add a **readiness probe** so new pods only receive traffic when ready. Add a **PreStop sleep** so the LB has time to deregister the pod before shutdown.

---

### 48. Pods are stuck in `ImagePullBackOff`. What's wrong?

Common causes:
- Incorrect image name or tag in the manifest.
- Private registry with no `imagePullSecrets` configured.
- Node can't reach the registry (firewall, DNS, proxy issue).
- Image was deleted from the registry.

```bash
kubectl describe pod <name>   # check Events for specific error
```

---

### 49. You need to scale down a StatefulSet database. What do you consider?

- StatefulSets scale down in **reverse ordinal order** (`db-2` → `db-1` → `db-0`).
- Ensure graceful shutdown — database must flush writes and complete transactions.
- Check for data sharding/rebalancing requirements before removing a node.
- Scaling down does **not** delete PVCs automatically — data is retained.
- Don't scale below the minimum for quorum (e.g., Zookeeper requires 3 for a majority).

---

### 50. Ensure a critical app always has at least 3 pods, even during node maintenance.

Apply a Pod Disruption Budget:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: critical-app-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: critical-app
```

This prevents `kubectl drain` from proceeding if it would drop below 3 available pods.

---

### 51. High CPU utilization on a worker node. How do you investigate?

```bash
kubectl top nodes
kubectl describe node <name>
kubectl top pods -A --sort-by=cpu
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- top
```

Resolution options:
- Scale the Deployment horizontally.
- Tighten CPU limits or add requests.
- Profile the application for bottlenecks.
- Let Cluster Autoscaler add nodes if the whole cluster is saturated.

---

### 52. App needs to access a database in a different namespace. How do you do it securely?

1. Access via FQDN: `<db-service>.<db-namespace>.svc.cluster.local`
2. Apply a **NetworkPolicy** in the database namespace that allows ingress from the app's namespace:

```yaml
ingress:
- from:
  - namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: app-namespace
    podSelector:
      matchLabels:
        app: my-app
```

3. Ensure the app's ServiceAccount has no more RBAC permissions than needed.

---

## VIII. CNCF Ecosystem

### 53. What is the CNCF?

The Cloud Native Computing Foundation (CNCF) hosts open-source cloud-native projects including Kubernetes, Prometheus, Helm, Envoy, Jaeger, and Flux. Its goal is to make cloud-native computing ubiquitous.

---

### 54. Briefly explain key CNCF projects.

| Project | Purpose |
|---------|---------|
| **Prometheus** | Metrics collection and alerting |
| **Grafana** | Dashboards and visualization |
| **Helm** | Kubernetes package manager |
| **Istio** | Service mesh — traffic management, mTLS, observability |
| **Envoy** | High-performance proxy used as sidecar in service meshes |
| **Cilium** | eBPF-based CNI — networking, security, observability |
| **Argo CD** | GitOps continuous delivery for Kubernetes |
| **Flux** | GitOps toolkit for Kubernetes |

---

### 55. What CI/CD pipelines have you worked with for Kubernetes?

Common patterns: Build image → push to registry → deploy via Helm or `kubectl apply` → verify rollout status. Tools: GitHub Actions, GitLab CI, Jenkins, Argo CD, Flux.

---

### 56. What challenges have you faced with Kubernetes in production?

Common real answers:
- Networking complexity — CNI issues, NetworkPolicy debugging.
- Resource management — OOMKilled pods, right-sizing requests/limits.
- Stateful application management — storage, data integrity.
- Secret management — avoiding plaintext secrets in Git.
- Cluster upgrades — minimizing disruption.
- Debugging distributed failures — logs spread across many pods.

---

## IX. Deeper Dive into Kubernetes Concepts

### 57. Explain the lifecycle of a Pod.

| Phase | Meaning |
|-------|---------|
| **Pending** | Accepted by K8s, but containers not yet created (e.g., image still pulling). |
| **Running** | At least one container is running. |
| **Succeeded** | All containers terminated successfully. |
| **Failed** | All containers terminated, at least one with failure (non-zero exit or OOM). |
| **Unknown** | Pod state could not be determined (node communication lost). |

---

### 58. What are Init Containers and when are they useful?

Init containers run to completion **before** the main containers start. Use for:

- Waiting for a dependent service to be ready.
- Running database migrations.
- Cloning a Git repo or downloading config.
- Setting file permissions on shared volumes.

Each init container must exit `0` before the next starts.

---

### 59. Explain resource requests and limits.

| | Requests | Limits |
|---|----------|--------|
| **Purpose** | Minimum guaranteed resources. Used by scheduler. | Maximum allowed. Enforced at runtime. |
| **CPU exceed** | N/A | Container is throttled. |
| **Memory exceed** | N/A | Container is OOMKilled. |

Always set both. Without requests, the scheduler can't make good placement decisions. Without limits, one pod can starve the entire node.

---

### 60. What is `ReadWriteMany` PVC access mode?

`ReadWriteMany` (RWX) allows a volume to be mounted read-write by **multiple nodes simultaneously**. Use cases: shared file storage across replicas.

Supported by: NFS, CephFS, GlusterFS, AWS EFS, Azure Files, Google Filestore.

Note: Standard cloud block storage (AWS EBS, Azure Disk, GCP PD) only supports `ReadWriteOnce` (single node).

---

### 61. How do you do application-level load balancing without cloud load balancers?

- **ClusterIP Service** — Basic round-robin across healthy pods.
- **Ingress** — Layer 7 routing by host/path via an Ingress Controller.
- **Service Mesh (Istio/Linkerd)** — Advanced: weighted routing, canary splits, A/B testing, circuit breaking.

---

### 62. Difference between `emptyDir` and `hostPath` volumes?

| | `emptyDir` | `hostPath` |
|---|-----------|-----------|
| **Lifetime** | Tied to the Pod. Deleted when Pod is removed. | Persists on the node. Survives Pod restarts. |
| **Portability** | Portable — not tied to a node. | Tied to a specific node — not portable. |
| **Use case** | Scratch space, inter-container sharing, caching. | Accessing node-level data, monitoring agents, log files. |

`hostPath` is generally discouraged for application data. Use PVCs for persistent storage.

---

### 63. A Service is not routing traffic to any pods. How do you troubleshoot?

```bash
kubectl describe service <name>       # check Endpoints field
kubectl get pods -l <service-selector> # pods with correct labels?
```

1. Empty Endpoints → label mismatch between Service selector and Pod labels.
2. Verify `targetPort` in Service matches `containerPort` in Pod spec.
3. `kubectl exec` into a pod and `curl` the service ClusterIP directly.
4. Check kube-proxy logs on the node.

---

### 64. What is a headless Service?

A headless Service has `clusterIP: None`. Instead of load balancing, DNS returns the individual Pod IP addresses directly.

**Use cases:**
- StatefulSets — allows direct pod-to-pod discovery by stable hostname (e.g., `db-0.postgres.default.svc.cluster.local`).
- Applications that need to control their own load balancing (e.g., Cassandra peer discovery).

---

### 65. Explain `tolerations` and `nodeSelector` in Pod scheduling.

- **nodeSelector** — Hard constraint. Pod only schedules on nodes with matching labels. Simple but inflexible.
- **tolerations** — Works with taints. Allows a Pod to be scheduled on a tainted node. Does not force it there.

For more flexibility: use `nodeAffinity` (supports `In`, `NotIn`, `Exists` operators and soft preferences).

---

## X. Scenario-Based Questions

### 66. Application requires a specific kernel module. How do you deploy it in Kubernetes?

Best approach: Use **dedicated nodes** with the module pre-loaded. Schedule pods to those nodes using `nodeSelector` or `nodeAffinity`.

- Use a **DaemonSet** with an init container to load the module on all relevant nodes if it can be done programmatically.
- Avoid using `hostPath` volume hacks — they compromise portability and security.
- If the dependency can't be removed, consider whether the workload belongs outside Kubernetes.

---

### 67. Moving from monolith to microservices on Kubernetes. Key considerations?

- **Containerize** — Optimized Dockerfiles, minimal base images.
- **Decompose** — Define clear service boundaries and API contracts.
- **Configuration** — ConfigMaps and Secrets per service.
- **State** — Identify stateful components; use StatefulSets or external managed databases.
- **Networking** — NetworkPolicies for tier isolation.
- **Observability** — Centralized logging, distributed tracing, per-service metrics.
- **CI/CD** — GitOps with Helm or Kustomize.
- **Resilience** — Liveness/readiness probes, retries, circuit breakers (service mesh).
- **Security** — RBAC per service account, image scanning, PSS.

---

### 68. Restrict network access to a database pod. Only specific app pods may connect.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-app-only
  namespace: database
spec:
  podSelector:
    matchLabels:
      app: database
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: app-namespace
      podSelector:
        matchLabels:
          app: my-app
  policyTypes:
  - Ingress
```

Requires a CNI that enforces NetworkPolicies (Calico, Cilium).

---

### 69. Handle config differences between dev, staging, and prod.

Options (from simple to advanced):

1. **ConfigMaps/Secrets** — One set per environment, loaded by pods via `envFrom`.
2. **Helm values files** — `values-dev.yaml`, `values-prod.yaml`. Deploy with `helm install -f values-prod.yaml`.
3. **Kustomize** — Base manifests + environment overlays. No templating needed.
4. **External secret management** — AWS Secrets Manager, Vault via External Secrets Operator.

---

### 70. A pod can't write to its mounted volume. How do you diagnose?

```bash
kubectl describe pod <name>                         # check mount errors
kubectl get pvc <name>                              # is it Bound?
kubectl describe pvc <name>                         # check StorageClass, provisioner
kubectl exec -it <name> -- ls -ld /path/to/mount    # check dir permissions
```

Common causes:
- PVC not Bound (no matching PV or provisioner failure).
- StorageClass provisioner not working.
- File permissions — container runs as UID that doesn't own the directory.
- Fix with `securityContext.fsGroup` or `runAsUser` in the Pod spec.
- SELinux/AppArmor on the host blocking access.

---

## XI. Advanced Kubernetes Concepts

### 71. What is an Admission Controller?

Admission controllers intercept API server requests **after authentication and authorization, before the object is persisted** in etcd.

- **Mutating** — Can modify the object (e.g., inject sidecars, add labels, set defaults).
- **Validating** — Can allow or deny the request, but cannot modify it.

Examples: `LimitRanger`, `ResourceQuota`, `PodSecurity`, `MutatingAdmissionWebhook`, `ValidatingAdmissionWebhook`.

---

### 72. How do you manage certificates and TLS in Kubernetes?

- **Kubernetes Secret** (`type: kubernetes.io/tls`) — Store cert and key.
- **Ingress** — Reference the TLS Secret for SSL termination at the Ingress Controller.
- **cert-manager** — Automates certificate issuance and renewal from Let's Encrypt or internal CAs.
- **Istio/Linkerd** — Manages mTLS between services automatically (no app-level cert handling needed).

---

### 73. Explain Kubernetes Operators in detail.

An Operator is a Custom Controller that packages operational knowledge about a complex stateful application. It watches CRDs and automates day-2 operations:

- Deployment and upgrades
- Backup and restore
- Scaling with data integrity
- Failure recovery and rebalancing
- Applying application-specific security patches

Examples: Prometheus Operator, MongoDB Atlas Operator, Strimzi (Kafka), PostgreSQL Operator.

---

### 74. What is kubeconfig?

`kubeconfig` is a YAML file (default: `~/.kube/config`) that stores cluster endpoints, credentials, and contexts. `kubectl` uses it to authenticate with clusters.

```bash
kubectl config get-contexts          # list contexts
kubectl config use-context <name>    # switch cluster
kubectl config current-context       # show active context
```

---

### 75. How does Kubernetes handle garbage collection?

- **Completed Pods/Jobs** — Deleted after TTL or by the Job controller.
- **Orphaned objects** — Objects whose owner reference is gone (e.g., old ReplicaSets) are deleted by the garbage collector.
- **Unused images and containers** — kubelet periodically evicts old images based on disk pressure thresholds.

---

### 76. What is a Multi-Cluster Kubernetes setup and why would you need it?

Running multiple independent K8s clusters for:

- **HA/DR** — Spread across regions or cloud providers.
- **Latency** — Deploy closer to users in different regions.
- **Data sovereignty** — Meet regulatory data residency requirements.
- **Isolation** — Strong blast radius separation for sensitive workloads.
- **Hybrid cloud** — On-premise + cloud.

Tools: Argo CD (multi-cluster GitOps), Cluster API, Rancher, Google Anthos.

---

### 77. What is etcd? Key characteristics?

etcd is the distributed key-value store that backs all Kubernetes cluster state.

- **Consistency** — Uses the Raft consensus algorithm.
- **High Availability** — 3 or 5 nodes for HA (tolerates 1 or 2 failures respectively).
- **Watch API** — Kubernetes components subscribe to changes in etcd to react to state updates.
- **Critical** — If etcd is lost without a backup, all cluster state is gone. Back it up regularly.

---

### 78. How does the Kubernetes Scheduler work?

The scheduler assigns new unscheduled Pods to nodes in two phases:

1. **Filtering** — Eliminate nodes that don't meet the Pod's requirements (resources, taints, node selectors, affinity, PV availability, port conflicts).
2. **Scoring** — Rank remaining nodes by preference (resource balance, affinity preferences, spread).

The Pod is assigned to the highest-scoring node.

---

## XII. Best Practices

### 79. Best practices for writing Dockerfiles for Kubernetes?

- Use minimal base images (`alpine`, `distroless`).
- Use **multi-stage builds** — separate build from runtime.
- Don't run as root — add `USER` instruction.
- Use `.dockerignore` to exclude unnecessary files.
- Place frequently changing instructions last (cache optimization).
- Scan images for vulnerabilities in CI (Trivy, Grype).
- Pin base image versions — avoid `latest`.

---

### 80. How do you secure Kubernetes at the image level?

- **Image scanning** — Trivy, Clair, Anchore in CI. Block CRITICAL CVEs from merging.
- **Trusted registries** — Use private registries. Block public registry pulls in production.
- **Image signing** — Cosign + policy enforcement (Kyverno, OPA Gatekeeper).
- **Least privilege** — Run as non-root, read-only root filesystem where possible.

---

### 81. Significance of `port` vs `targetPort` in a Service?

- `port` — The port the Service exposes within the cluster (what clients connect to).
- `targetPort` — The port on the actual Pod the traffic is forwarded to (what your app listens on).

Decouples the Service interface from the container implementation. You can change the app's listening port without updating clients.

---

### 82. How do you effectively manage secrets in Kubernetes?

- Never commit secrets to Git — not even base64-encoded.
- Use **External Secrets Operator** or **Secrets Store CSI Driver** to sync secrets from Vault, AWS Secrets Manager, Azure Key Vault.
- Enable **encryption at rest** for etcd.
- Restrict access with RBAC — only the pods that need a Secret should be able to read it.
- Rotate secrets regularly.
- Apply PSS to restrict how Pods can mount and access Secrets.

---

### 83. What is kube-proxy and what are its modes?

kube-proxy maintains network rules on each node to implement the Service abstraction.

| Mode | Description |
|------|-------------|
| **iptables** (default) | Uses iptables rules for NAT and load balancing. Scalable. |
| **ipvs** | Uses Linux IPVS. Better performance for high-traffic clusters. |
| **userspace** | Deprecated. Slowest — proxies in userspace. |

---

### 84. What are immutable fields in Kubernetes API objects?

Some fields cannot be changed after an object is created (e.g., `selector` on a Service, `volumeMounts` on a Pod). To change them, you must delete and recreate the object. This maintains consistency and prevents undefined behavior from in-place changes to structural fields.

---

### 85. What is `kubectl drain`?

`kubectl drain` gracefully evicts all pods from a node, respecting Pod Disruption Budgets. Marks the node as unschedulable first. Used before node maintenance (OS patching, hardware replacement).

```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>    # re-enable scheduling after maintenance
```

---

### 86. How would you troubleshoot network latency between pods?

1. Confirm which pods and services are affected — is it isolated or widespread?
2. Check CNI plugin health and logs.
3. Check node network interface stats and CPU utilization.
4. Use `iperf3` between pods to measure throughput.
5. Use `kubectl debug` or `kubectl exec` with `tcpdump` to inspect traffic.
6. Check DNS resolution time — `time nslookup <service>`.
7. If using a service mesh — check proxy logs and telemetry.
8. Rule out the application itself (slow queries, excessive logging).

---

### 87. What are PreStop and PostStart hooks?

| Hook | Timing | Blocking? | Use case |
|------|--------|-----------|----------|
| **PostStart** | Immediately after container starts | No | Register with service registry, initial setup |
| **PreStop** | Immediately before container terminates | Yes | Graceful shutdown, drain connections, flush logs |

PreStop is used with `sleep` to give load balancers time to deregister the pod before it stops accepting connections.

---

### 88. Resource limits with multiple containers in a Pod.

Requests and limits are defined **per container**. The scheduler sums all container requests to find a node with enough aggregate resources. Limits are enforced independently per container — if one container is OOMKilled, others in the pod continue running.

---

### 89. What are mutating and validating webhooks?

- **MutatingAdmissionWebhook** — Modifies objects before they're stored. Example: inject a sidecar, add a default label, set a default resource limit.
- **ValidatingAdmissionWebhook** — Validates and allows/denies requests without modifying them. Example: enforce naming conventions, require resource limits, block privileged containers.

Both are triggered as part of the admission control pipeline before the object is written to etcd.

---

### 90. What are `volumeClaimTemplates` in StatefulSets?

`volumeClaimTemplates` automatically create a unique PVC for each StatefulSet pod replica. The PVC name follows the pattern `<template-name>-<pod-name>` and is retained even if the pod is deleted. This gives each replica its own dedicated, persistent storage.

---

### 91. Difference between a Job and a CronJob?

- **Job** — Runs pods once until a specified number complete successfully. One-off batch task.
- **CronJob** — Creates Jobs on a repeating time-based schedule (cron syntax). Recurring tasks.

---

### 92. How do you ensure high availability of the control plane?

- **Multiple API servers** behind a load balancer.
- **etcd cluster** — 3 or 5 members using Raft consensus.
- **kube-controller-manager and kube-scheduler** — Run as multiple instances; active–standby via leader election.
- **Regular etcd backups** — Test restore procedures.
- Or use a **managed service** (EKS, GKE, AKS) where the cloud provider handles control plane HA.

---

### 93. What is kubeadm?

`kubeadm` bootstraps a minimum viable Kubernetes cluster. It handles:
- Generating TLS certificates.
- Setting up control plane components.
- Configuring kubeconfig.
- Joining worker nodes.

Not a full lifecycle manager — use Cluster API or managed services for production at scale.

---

### 94. ServiceAccount + ClusterRoleBinding use case.

**Scenario:** A monitoring pod needs to list all nodes cluster-wide.

```yaml
# 1. ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: node-monitor
  namespace: monitoring

# 2. ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]

# 3. ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
- kind: ServiceAccount
  name: node-monitor
  namespace: monitoring
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

---

### 95. What is `kubectl diff`?

`kubectl diff` shows what would change if you applied a manifest, without actually applying it. Useful for reviewing changes before `kubectl apply` in CI pipelines or production.

```bash
kubectl diff -f deployment.yaml
```

---

### 96. How do you secure a Kubernetes cluster at the network level (beyond NetworkPolicies)?

- **Cloud security groups** — Restrict node/control plane ingress/egress at the VPC level.
- **Private API server endpoint** — No public access to the API server.
- **VPC peering / VPN** — Secure cross-cluster and on-premise connectivity.
- **Egress controls** — NetworkPolicies + firewall rules for outbound pod traffic.
- **mTLS via service mesh** — Encrypted, identity-verified service-to-service communication.
- **IDS/IPS** — Detect and block malicious traffic at the network edge (Falco for runtime, Cilium for network).

---

### 97. What is the Container Runtime Interface (CRI)?

CRI is a gRPC plugin interface that allows the kubelet to use different container runtimes (containerd, CRI-O) without being tightly coupled to any one implementation. This abstraction enables Kubernetes to be runtime-agnostic.

---

### 98. How do you implement blue/green deployments in Kubernetes?

1. Two Deployments: `app-blue` (current) and `app-green` (new version).
2. One Service with selector pointing to `app-blue`.
3. To cut over: update the Service selector to point to `app-green`.
4. To rollback: revert the selector to `app-blue`.

**Pros:** Instant cutover, instant rollback.  
**Cons:** Doubles resource consumption during the switch.

---

### 99. How do you implement canary deployments in Kubernetes?

1. Two Deployments: `app-stable` (most replicas) and `app-canary` (few replicas).
2. Use an Ingress controller or service mesh to split traffic — e.g., 95% to stable, 5% to canary.
3. Monitor error rates, latency, and business metrics for canary.
4. Gradually shift traffic to canary if healthy. Remove stable when done.
5. Rollback: send 0% traffic to canary.

Tools: Argo Rollouts, Flagger (automates canary analysis and promotion).

---

### 100. What is `kubectl debug`?

`kubectl debug` creates an ephemeral debug container in an existing Pod or creates a copy of a Pod with debugging tools injected.

**Use cases:**
- Attach a shell to a running pod that has no shell (`distroless` images).
- Debug a `CrashLoopBackOff` container — debug copy starts with a different command.
- Run `tcpdump`, `strace`, or `curl` inside a pod without modifying its image.
- Debug a node directly by creating a privileged pod on it.

```bash
kubectl debug -it <pod> --image=busybox --target=<container>
kubectl debug node/<node-name> -it --image=ubuntu
```
