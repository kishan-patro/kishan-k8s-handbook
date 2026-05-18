# Kubernetes Interview Prep 01

Focused notes for foundational Kubernetes interviews. Use this file to explain concepts clearly, connect them to production behavior, and give concrete examples.

---

## 1. What is Kubernetes?

Kubernetes is a container orchestration platform that automates deployment, scaling, service discovery, self-healing, and rollout management for containerized applications.

### Strong answer

Kubernetes gives you a declarative control plane. You define the desired state, such as how many replicas should run, what image they should use, and how they should be exposed. Controllers continuously compare desired state with actual state and reconcile drift.

### What to mention in interviews

- It is not just a scheduler; it is a full control plane.
- It separates desired state from runtime execution.
- It is useful when you need repeatable deployment and operational consistency.

---

## 2. What are the main Kubernetes components?

### Control plane

- `kube-apiserver`: Front door for all cluster operations.
- `etcd`: Key-value store for cluster state.
- `kube-scheduler`: Chooses nodes for unscheduled pods.
- `kube-controller-manager`: Runs reconciliation loops for deployments, nodes, endpoints, and more.
- `cloud-controller-manager`: Integrates with cloud provider APIs.

### Node components

- `kubelet`: Ensures pods are running on the node as defined.
- `kube-proxy`: Manages service networking rules.
- `container runtime`: Runs containers through `containerd` or another supported runtime.

### Interview shortcut

The API server is the source of truth interface, etcd stores state, controllers enforce intent, and worker nodes run the workloads.

---

## 3. What is a Pod?

A pod is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace, IP address, and storage volumes.

### Good explanation

In most cases, one pod runs one main application container. Sidecars, init containers, and helper containers may also be present if the workload requires them.

### Production nuance

Pods are ephemeral. You do not scale or manage individual pods directly for long-lived apps. You usually manage them through deployments, statefulsets, or jobs.

---

## 4. What is a Deployment?

A deployment manages stateless applications by handling replica sets, rolling updates, and rollbacks.

### Strong answer

You define the desired number of replicas and the pod template. The deployment controller creates replica sets and ensures the target number of healthy pods stays available during updates.

### When to use it

- Web applications
- APIs
- Background workers that do not require stable identity

---

## 5. What is the difference between Deployment, StatefulSet, and DaemonSet?

| Resource | Best use case | Key behavior |
|---|---|---|
| Deployment | Stateless apps | Rolling updates, interchangeable pods |
| StatefulSet | Stateful apps | Stable identity, ordered startup, stable storage |
| DaemonSet | Node-level agents | One pod per node or per matching node |

### Interview version

Use deployments for stateless services, statefulsets for databases or clustered systems, and daemonsets for agents like log collectors, node exporters, and security tools.

---

## 6. How does service discovery work?

Kubernetes services provide a stable virtual IP and DNS name for a changing set of backend pods.

### Core idea

Pods are replaceable and their IPs change. Services solve that by selecting pods via labels and providing a stable access point.

### Common service types

- `ClusterIP`: Internal-only access
- `NodePort`: Exposes the service on a port on each node
- `LoadBalancer`: Uses cloud load balancer integration
- `ExternalName`: Maps a service to an external DNS name

---

## 7. What is Ingress?

Ingress is an HTTP and HTTPS routing abstraction for exposing services externally.

### Strong answer

Ingress lets you define host-based and path-based routing rules. It requires an ingress controller such as NGINX Ingress or AWS Load Balancer Controller to actually implement those rules.

### Important distinction

Ingress is the API object. The ingress controller is the component that enforces it.

---

## 8. What are ConfigMaps and Secrets?

- `ConfigMap`: Non-sensitive configuration like flags, hostnames, or application settings
- `Secret`: Sensitive data like passwords, API keys, or certificates

### Good explanation

Both can be injected as environment variables or mounted as files. Secrets are base64-encoded, not encrypted by default, so production environments should also use envelope encryption and scoped access.

---

## 9. What is a namespace?

A namespace is a logical partition in a cluster used for organization, policy boundaries, and scoped access.

### Practical use cases

- Separate teams or environments
- Apply resource quotas and limit ranges
- Scope RBAC bindings

### Interview nuance

Namespaces do not provide hard isolation by themselves. Strong isolation still needs RBAC, network policies, admission control, and resource governance.

---

## 10. How does scaling work in Kubernetes?

### Horizontal Pod Autoscaler

Scales pod replicas based on CPU, memory, or custom metrics.

### Vertical Pod Autoscaler

Adjusts resource requests and limits based on observed usage.

### Cluster Autoscaler

Adds or removes worker nodes when pods cannot be scheduled or nodes are underutilized.

### Strong answer

Pod scaling solves workload-level demand. Node scaling solves infrastructure capacity. You usually need both working together.

---

## 11. What are requests and limits?

- `requests`: Minimum guaranteed CPU or memory used for scheduling
- `limits`: Maximum amount a container can consume

### Why they matter

- The scheduler uses requests when deciding placement.
- Exceeding a memory limit usually causes `OOMKilled`.
- Missing requests can lead to noisy-neighbor behavior.

---

## 12. What causes CrashLoopBackOff?

CrashLoopBackOff means the container keeps starting, crashing, and being restarted with backoff delay.

### Common causes

- Bad startup command
- Missing environment variable or secret
- Application boot failure
- Wrong probe configuration

### First commands to run

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

---

## 13. How would you debug a pod that is not starting?

### Practical sequence

1. Check pod status and events with `kubectl describe pod`.
2. Check current and previous logs.
3. Confirm the image, command, env vars, and mounted secrets.
4. Verify probes are not failing immediately.
5. Confirm node capacity and scheduling constraints.

### Strong interview answer

Describe a sequence, not random commands. Interviewers want to see structured debugging.

---

## 14. What is RBAC?

Role-Based Access Control defines what identities can do inside the cluster.

### Main objects

- `Role` and `ClusterRole`
- `RoleBinding` and `ClusterRoleBinding`

### Strong answer

Use least privilege. Grant namespace-scoped permissions with roles where possible, and reserve cluster roles for truly cluster-wide operations.

---

## 15. What are readiness and liveness probes?

- `Readiness probe`: Determines whether the pod should receive traffic
- `Liveness probe`: Determines whether Kubernetes should restart the container
- `Startup probe`: Protects slow-starting applications from premature restarts

### Common interview point

If readiness fails, traffic stops. If liveness fails, the container restarts. Mixing them up is a common production mistake.

---

## 16. What is the difference between rolling update and recreate?

- `RollingUpdate`: Replaces pods gradually while keeping some capacity online
- `Recreate`: Stops old pods before starting new ones

### Strong answer

Rolling updates are safer for highly available stateless services. Recreate is simpler but introduces downtime.

---

## 17. What are taints and tolerations?

Taints repel pods from nodes. Tolerations allow specific pods to be scheduled onto tainted nodes.

### Why this matters

They are useful for isolating GPU workloads, system workloads, or dedicated nodes.

---

## 18. What are affinity and anti-affinity?

- `Node affinity`: Prefer or require certain node labels
- `Pod affinity`: Schedule near certain pods
- `Pod anti-affinity`: Avoid placing matching pods together

### Good use case

Pod anti-affinity helps spread replicas across nodes or zones to reduce blast radius.

---

## 19. How do persistent volumes work?

Persistent volumes decouple storage lifecycle from pod lifecycle.

### Main objects

- `PersistentVolumeClaim`: What the application asks for
- `PersistentVolume`: The storage resource that satisfies the claim
- `StorageClass`: Dynamic provisioning policy

### Interview shortcut

The pod uses the claim, the claim binds to storage, and the storage remains available independently of the pod.

---

## 20. Closing guidance

- Explain the control loop model clearly.
- Tie every definition to real cluster behavior.
- Prefer examples from incidents, upgrades, or migrations you have handled.
- If asked a simple question, answer simply first, then add depth.
