# Kubernetes Concepts & Commands Reference

> Quick reference for Kubernetes concepts, terminology, and `kubectl` commands.

---

## 1. Kubernetes Basics

| Term | Description |
|------|-------------|
| **Pod** | Smallest deployable unit — holds one or more containers. |
| **Node** | Worker machine in Kubernetes. |
| **Cluster** | Set of nodes that run containerized applications. |
| **kubectl** | Command-line tool for interacting with a Kubernetes cluster. |
| **kubelet** | Agent running on each node; communicates with the API server. |
| **API Server** | Front-end to the cluster's control plane. |
| **etcd** | Consistent, highly-available key-value store backing all cluster data. |
| **Control Plane** | Collection of processes that control Kubernetes nodes. |
| **Namespace** | Virtual cluster backed by the same physical cluster — used for isolation. |

---

## 2. Workloads & Controllers

| Resource | Description |
|----------|-------------|
| **Deployment** | Manages a replicated stateless application. |
| **ReplicaSet** | Ensures a specified number of pod replicas are running at all times. |
| **StatefulSet** | Manages pods with ordering and uniqueness guarantees — for stateful apps. |
| **DaemonSet** | Ensures all (or some) nodes run a copy of a Pod. |
| **Job** | Creates one or more pods and ensures a specified number terminate successfully. |
| **CronJob** | Manages time-based Jobs — runs at a specific time or periodic intervals. |
| **HPA** | Automatically scales pod count based on observed CPU or memory usage. |

---

## 3. Services & Networking

| Term | Description |
|------|-------------|
| **Service** | Exposes an application running in Pods as a stable network endpoint. |
| **Ingress** | Manages external HTTP/HTTPS access to services within a cluster. |
| **Network Policy** | Defines how Pods communicate with each other. |
| **Service Discovery** | Connects to services dynamically based on a logical name (via CoreDNS). |
| **Load Balancer** | Service type that distributes traffic across multiple Pods via a cloud LB. |

---

## 4. Configuration & Storage

| Term | Description |
|------|-------------|
| **ConfigMap** | Stores non-sensitive config data separately from container images. |
| **Secret** | Stores sensitive data — passwords, OAuth tokens, SSH keys (base64-encoded). |
| **Environment Variables** | Used within Pods for configuration and service discovery. |
| **Volume** | Persistent storage attached to a Pod. |
| **PersistentVolumeClaim (PVC)** | User's request for storage from a PersistentVolume. |
| **PersistentVolume (PV)** | Cluster-level storage independent of Pod lifecycle. |
| **StorageClass** | Describes different "classes" of storage offered by administrators. |

---

## 5. Monitoring, Logging & Debugging

| Tool | Description |
|------|-------------|
| **kube-state-metrics** | Listens to the K8s API and generates metrics about object state. |
| **Prometheus** | Open-source monitoring and alerting toolkit. |
| **Grafana** | Analytics and monitoring dashboards — commonly paired with Prometheus. |
| **ELK Stack** | Elasticsearch + Logstash + Kibana — used for centralized logging. |
| **Kubernetes Dashboard** | General-purpose web UI for cluster management. |
| **kubectl debug** | Tool for debugging running or crashed pods. |

---

## 6. Security & Authorization

| Term | Description |
|------|-------------|
| **RBAC** | Role-Based Access Control — controls who can do what in the cluster. |
| **Pod Security Admission** | Enforces security constraints on Pods (replaces deprecated PodSecurityPolicy). |
| **ServiceAccount** | Identity attached to Pods for interacting with the Kubernetes API. |
| **TLS** | Transport Layer Security — used for encrypted communication in the cluster. |

---

## 7. Autoscaling

| Type | Description |
|------|-------------|
| **HPA** (Horizontal Pod Autoscaler) | Scales pod replicas based on CPU, memory, or custom metrics. |
| **VPA** (Vertical Pod Autoscaler) | Auto-adjusts CPU/memory requests for containers based on usage. |
| **Cluster Autoscaler** | Adds or removes nodes based on pending pods or underutilization. |

---

## 8. Cluster Maintenance

- **Node Maintenance**: Drain a node before taking it down — `kubectl drain`.
- **Cluster Upgrades**: Upgrade control plane first, then worker nodes one by one.
- **Backup & Disaster Recovery**: Automate `etcdctl snapshot save` to S3/cloud storage.

---

## 9. Extending Kubernetes

| Term | Description |
|------|-------------|
| **Custom Resource (CRD)** | Extends the Kubernetes API with your own resource types. |
| **Custom Controller** | Automates handling of Custom Resources. |
| **Operator** | Application-specific controller that manages complex stateful apps. |
| **API Server Extension** | Adds custom endpoints to the Kubernetes API via aggregation. |

---

## 10. Advanced Features

| Feature | Description |
|---------|-------------|
| **Service Mesh (Istio/Linkerd)** | Manages microservice traffic, security, and observability transparently. |
| **Pod Priority & Preemption** | Higher-priority pods can evict lower-priority ones when resources are tight. |
| **Taints & Tolerations** | Nodes repel pods unless the pod explicitly tolerates the taint. |
| **Node Affinity** | Controls pod placement based on node labels. |
| **Pod Disruption Budget (PDB)** | Limits voluntary disruptions to ensure minimum pod availability. |

---

## 11. Cloud-Managed Kubernetes

| Provider | Service |
|----------|---------|
| AWS | Amazon EKS |
| Google Cloud | Google Kubernetes Engine (GKE) |
| Azure | Azure Kubernetes Service (AKS) |

---

## 12. CI/CD in Kubernetes

- **Jenkins**: Popular CI/CD automation server.
- **GitLab CI**: Built-in CI/CD in GitLab — builds and tests on every push.

---

## 13. Helm — Kubernetes Package Manager

| Term | Description |
|------|-------------|
| **Helm Chart** | Package of pre-configured Kubernetes resources. |
| **Helm Repository** | Collection of charts hosted for distribution. |
| **Release** | A deployed instance of a chart. |

---

## 14. Kubernetes Networking Deep Dive

| Term | Description |
|------|-------------|
| **CNI** | Container Network Interface — standard for networking plugins. |
| **Flannel** | Simple overlay network provider. |
| **Calico** | BGP-based networking with NetworkPolicy support. |
| **Cilium** | eBPF-based CNI with advanced observability and security. |
| **kube-proxy** | Maintains network rules on each node for Service routing. |

---

## 15. Development Tools

| Tool | Description |
|------|-------------|
| **Minikube** | Runs a single-node K8s cluster locally for development/testing. |
| **Skaffold** | Continuous development tool for Kubernetes — auto-builds and deploys. |
| **Kompose** | Converts Docker Compose files to Kubernetes manifests. |
| **Kubeadm** | Tool to bootstrap a production-grade Kubernetes cluster. |

---

## kubectl Commands Reference

### Cluster Information
```bash
kubectl cluster-info                        # Display cluster info
kubectl version                             # Display client and server version
kubectl get componentstatuses               # Check control plane health
```

### Nodes
```bash
kubectl get nodes                           # List all nodes
kubectl get nodes -o wide                   # List nodes with extra detail
kubectl describe node <node-name>           # Show details of a specific node
kubectl cordon <node-name>                  # Mark node unschedulable
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data  # Drain node
kubectl uncordon <node-name>                # Mark node schedulable again
```

### Pods
```bash
kubectl get pods                            # List pods in current namespace
kubectl get pods -A                         # List pods across all namespaces
kubectl get pods -o wide                    # Include node and IP info
kubectl run <name> --image=<image>          # Create a pod
kubectl describe pod <pod-name>             # Show pod details and events
kubectl logs <pod-name>                     # Fetch pod logs
kubectl logs <pod-name> --previous          # Logs from the last crashed container
kubectl logs <pod-name> -c <container>      # Logs from a specific container
kubectl logs -f deployment/<name> --since=1h  # Tail logs for last 1 hour
kubectl delete pod <pod-name>               # Delete a pod
kubectl exec -it <pod-name> -- /bin/sh      # Shell into a running pod
kubectl port-forward <pod-name> <local>:<pod-port>  # Forward a port locally
```

### Deployments
```bash
kubectl create deployment <name> --image=<image>           # Create deployment
kubectl get deployments                                     # List deployments
kubectl describe deployment <name>                          # Show deployment details
kubectl scale deployment <name> --replicas=<num>            # Scale replicas
kubectl rollout status deployment/<name>                    # Check rollout status
kubectl rollout history deployment/<name>                   # View rollout history
kubectl rollout undo deployment/<name>                      # Rollback to previous version
kubectl set image deployment/<name> <container>=<image>     # Update container image
```

### Services
```bash
kubectl get services                                        # List all services
kubectl get svc -o wide                                     # Services with detail
kubectl describe service <name>                             # Service details
kubectl expose deployment <name> --type=LoadBalancer --port=8080  # Expose as service
kubectl get endpoints <name>                                # Check service endpoints
```

### Namespaces & Contexts
```bash
kubectl get namespaces                      # List all namespaces
kubectl create namespace <name>             # Create a namespace
kubectl config get-contexts                 # Show all contexts
kubectl config current-context             # Show current context
kubectl config use-context <name>           # Switch context
kubectl get pods -n <namespace>             # Get pods in a specific namespace
```

### ConfigMaps & Secrets
```bash
kubectl get configmaps                                      # List ConfigMaps
kubectl create configmap <name> --from-file=<path>          # Create from file
kubectl create configmap <name> --from-literal=key=value    # Create from literal
kubectl get secrets                                         # List secrets
kubectl create secret generic <name> --from-literal=key=val # Create secret
kubectl describe secret <name>                              # Describe secret
```

### Storage
```bash
kubectl get pv                              # List PersistentVolumes
kubectl get pvc                             # List PersistentVolumeClaims
kubectl get storageclass                    # List storage classes
```

### Apply & Delete
```bash
kubectl apply -f <filename.yaml>            # Apply configuration from file
kubectl apply -f <directory/>               # Apply all manifests in directory
kubectl delete -f <filename.yaml>           # Delete resources from file
kubectl delete pod <name> --force           # Force delete a pod
```

### Resource Usage
```bash
kubectl top nodes                           # CPU/memory usage per node
kubectl top pods -A                         # CPU/memory usage per pod
kubectl top pods -A --sort-by=cpu           # Sort by CPU
kubectl top pods -A --sort-by=memory        # Sort by memory
```

### Autoscaling
```bash
kubectl autoscale deployment <name> \
  --min=<min> --max=<max> --cpu-percent=<pct>   # HPA based on CPU
kubectl get hpa                                  # List HPAs
kubectl get hpa --watch                          # Watch HPA scaling
```

### RBAC
```bash
kubectl get roles                           # List roles in current namespace
kubectl get rolebindings                    # List role bindings
kubectl get clusterroles                    # List cluster roles
kubectl get clusterrolebindings             # List cluster role bindings
kubectl auth can-i list pods --as=<user> -n <ns>  # Check user permissions
```

### Network Policies
```bash
kubectl get networkpolicies -n <namespace>  # List network policies
kubectl describe networkpolicy <name>       # Describe a network policy
```

### Jobs & CronJobs
```bash
kubectl get jobs                            # List jobs
kubectl get cronjobs                        # List cron jobs
kubectl logs job/<job-name>                 # Fetch logs from a job
kubectl create job <name> --from=cronjob/<cj-name>  # Trigger a cronjob manually
```

### Misc & Advanced
```bash
kubectl get all -n <namespace>              # List all resources in namespace
kubectl get all -A                          # List all resources cluster-wide
kubectl explain <resource>                  # Show API docs for a resource type
kubectl get events -n <namespace>           # Show events in a namespace
kubectl get events --sort-by=.lastTimestamp # Sort events by time
kubectl api-resources                       # List all available resource types
```

### Helm
```bash
helm list                                   # List installed releases
helm list -A                                # All namespaces
helm repo add <name> <url>                  # Add a chart repository
helm repo update                            # Update chart repos
helm search repo <keyword>                  # Search for a chart
helm install <release> <chart>              # Install a chart
helm install <release> <chart> -f values.yaml  # Install with custom values
helm upgrade <release> <chart>              # Upgrade a release
helm rollback <release> <revision>          # Rollback to a revision
helm uninstall <release>                    # Uninstall a release
helm template <chart>                       # Render chart templates locally
```

### kubectl Plugins (krew)
```bash
kubectl krew search                         # Search available plugins
kubectl krew install <plugin-name>          # Install a plugin
kubectl krew list                           # List installed plugins
```
