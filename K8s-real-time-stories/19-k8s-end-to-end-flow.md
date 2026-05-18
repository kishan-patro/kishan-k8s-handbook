# How Kubernetes Works: End-to-End Flow

## Overview

Kubernetes follows a **declarative, reconciliation-based model**. You describe the desired state, and Kubernetes continuously works to match it. Here's what happens from cluster setup to a running application.

---

## Step-by-Step Flow

### 1. Cluster Setup
You provision a Kubernetes cluster consisting of:
- **Control Plane** — API Server, etcd, Scheduler, Controller Manager, Cloud Controller Manager
- **Worker Nodes** — Kubelet, kube-proxy, container runtime (e.g., containerd)

### 2. Define Your Application
You write YAML manifests describing your workload:
- `Deployment` — manages Pod replicas and rolling updates
- `Service` — exposes Pods via a stable network endpoint
- `ConfigMap` / `Secret` — inject configuration and sensitive data

### 3. Submit via kubectl
```bash
kubectl apply -f deployment.yaml
```
The request is sent over HTTPS to the **API Server** — the single control point for all cluster operations.

### 4. API Server Processes the Request
The API Server:
1. **Authenticates** the request (certificates, tokens, OIDC)
2. **Authorizes** it via RBAC policies
3. **Validates** the object spec (admission controllers, webhooks)
4. Determines the operation: create, update, or delete

### 5. State Persisted in etcd
The validated object spec is written to **etcd** — the cluster's distributed key-value store and source of truth.

### 6. Controller Manager Reacts
The appropriate controller watches etcd via the API Server for changes:
- A **Deployment controller** creates or updates a `ReplicaSet`
- The **ReplicaSet controller** ensures the desired number of Pods exist

### 7. Scheduler Assigns Pods to Nodes
For each unscheduled Pod, the **Scheduler**:
1. Filters nodes that meet resource requests, taints/tolerations, and affinity rules
2. Scores remaining candidates
3. Binds the Pod to the best-fit node by writing a `nodeName` to the Pod spec

### 8. Kubelet Starts the Container
The **Kubelet** on the selected node:
1. Detects the Pod bound to its node via the API Server watch
2. Calls the **container runtime** (via CRI) to pull the image and create the container
3. Mounts volumes, injects environment variables and secrets

### 9. CNI Plugin Assigns Networking
The **CNI plugin** (e.g., Calico, Flannel, Cilium):
- Assigns the Pod a unique IP address
- Sets up routing so the Pod can communicate across the cluster network

### 10. kube-proxy Configures Service Routing
**kube-proxy** watches for Service and Endpoint changes and programs:
- `iptables` or `ipvs` rules to load-balance traffic from a Service ClusterIP to healthy Pod IPs

### 11. Kubelet Reports Status
The Kubelet continuously reports back to the API Server:
- Container readiness and liveness probe results
- Pod phase (`Pending`, `Running`, `Succeeded`, `Failed`)
- Resource usage metrics (consumed by the Metrics Server / HPA)

### 12. Controllers Maintain Desired State
If a Pod crashes, is evicted, or a node goes down:
- The ReplicaSet controller detects the count is below desired
- It creates a replacement Pod and the cycle repeats from Step 7

### 13. The Reconciliation Loop Never Stops
Kubernetes **continuously watches** actual state vs. desired state and reconciles any drift. This loop is the core design principle of the system.

---

## Summary Diagram

```
kubectl apply
      │
      ▼
 API Server ──► Authentication ──► Authorization ──► Admission
      │
      ▼
    etcd  (persists desired state)
      │
      ▼
 Controller Manager  (creates ReplicaSet / Pod objects)
      │
      ▼
   Scheduler  (binds Pod to Node)
      │
      ▼
   Kubelet  (pulls image, starts container via CRI)
      │
      ├──► CNI Plugin  (assigns Pod IP, sets up routing)
      │
      └──► kube-proxy  (programs Service iptables/ipvs rules)
      │
      ▼
  Pod Running  ◄──── Kubelet health checks & status reporting
      │
      ▼
Controller watches ──► recreates Pod if it crashes (reconciliation)
```

---

## Key Components Quick Reference

| Component | Role |
|---|---|
| **API Server** | Central control plane entry point; all reads/writes go through it |
| **etcd** | Distributed key-value store; cluster source of truth |
| **Controller Manager** | Runs built-in controllers (Deployment, ReplicaSet, Node, etc.) |
| **Scheduler** | Assigns unscheduled Pods to nodes |
| **Kubelet** | Node agent; manages Pod lifecycle on its node |
| **kube-proxy** | Maintains Service networking rules on each node |
| **Container Runtime** | Runs containers (containerd, CRI-O) via the CRI interface |
| **CNI Plugin** | Provides Pod networking and IP assignment |
