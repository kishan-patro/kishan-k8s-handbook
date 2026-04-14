# Kubernetes Architecture — How the Cluster Actually Works

At a high level, a Kubernetes cluster has two main parts: the **Control Plane** and **Worker Nodes**.

Understanding their responsibilities helps you debug and operate clusters better.

## 1. Control Plane

Responsible for managing the cluster state. Runs on dedicated master node(s).

- **API Server** — Entry point for all cluster communication (`kubectl`, controllers, operators). Every component talks through it.
- **etcd** — Distributed key-value store that holds the entire cluster state. If etcd is lost and there's no backup, the cluster state is gone.
- **Scheduler** — Decides which node a new Pod should run on, based on resource requests, taints, tolerations, and affinity rules.
- **Controller Manager** — Runs controllers (ReplicaSet, Deployment, Node, Job, etc.) that continuously ensure the actual state matches the desired state.

## 2. Worker Nodes

Nodes are where containers actually run.

- **Kubelet** — Agent on every node that communicates with the API Server, pulls container images, and manages Pod lifecycle.
- **Container Runtime** — Runs containers (e.g., containerd, CRI-O). Docker was removed as a runtime since Kubernetes v1.24.
- **Kube-proxy** — Maintains network rules on each node so Services can route traffic to the correct Pods.

## 3. How Everything Works Together

Typical flow when you run `kubectl apply`:

1. `kubectl` sends the manifest to the **API Server**
2. API Server validates it and writes the desired state to **etcd**
3. **Scheduler** watches for unassigned Pods, picks a suitable node
4. **Kubelet** on that node picks up the assignment, tells the **container runtime** to start the container
5. **Kube-proxy** sets up networking so the Pod is reachable via its Service

## 4. Troubleshooting with Architecture in Mind

Understanding which component is responsible helps you narrow down issues fast:

| Symptom | Likely Component | What to Check |
|---|---|---|
| Pod stuck in `Pending` | Scheduler | Insufficient resources, taints, node affinity |
| Pod in `CrashLoopBackOff` | Kubelet / App | `kubectl logs`, `kubectl describe pod` |
| Pod can't reach other Pods | Kube-proxy / CNI | Network policies, CNI plugin health |
| `kubectl` not responding | API Server | API server logs, etcd health |
| Cluster state lost after restart | etcd | etcd backups, quorum status |

![alt text](image.png)
