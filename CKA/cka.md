CKA Certification Roadmap (30 Days)
Exam Overview
- Duration: 2 hours | Version: Kubernetes v1.34 | Format: Performance-based, hands-on CLI
- Domains: Storage (10%), Troubleshooting (30%), Workloads & Scheduling (15%), Cluster Architecture (25%), Services & Networking (20%)
---
Phase 1: Foundation (Days 1-5)
Day 1-2: Kubernetes Architecture & Core Concepts
- Cluster architecture (control plane, worker nodes, etcd)
- API Server, Controller Manager, Scheduler, Kubelet, Kube-proxy
- Pod lifecycle, containers, namespaces
- Setup practice cluster with minikube or kind
Day 3-4: CLI & YAML Fundamentals
- kubectl commands (get, describe, create, apply, delete, exec, logs)
- Writing YAML manifests (Deployment, Service, ConfigMap, Secret)
- Imperative vs declarative approaches
- Practice: Deploy sample app from scratch
Day 5: Kubernetes Documentation Navigation
- Bookmark official docs: kubernetes.io/docs, kubernetes.io/reference
- Practice searching docs under exam conditions
- Time yourself finding: RBAC docs, PV/PVC examples, Service types
---
Phase 2: Core Topics (Days 6-15)
Day 6-7: Workloads & Scheduling (15%)
- Deployments (create, update, rollbacks, rolling strategies)
- ReplicaSets, DaemonSets, StatefulSets
- Jobs and CronJobs
- Scheduling: nodeSelector, node affinity, taints/tolerations, pod affinity/anti-affinity
- Resource limits, requests, and quotas
- Practice: Rolling update with rollback, configure pod scheduling
Day 8-9: Services & Networking (20%)
- ClusterIP, NodePort, LoadBalancer services
- Headless services, ExternalName
- Endpoints and EndpointSlices
- Ingress controllers and Ingress resources
- NetworkPolicies (deny all, allow specific)
- CoreDNS internals
- Gateway API basics
- Practice: Expose apps via Ingress, implement network policies
Day 10-12: Cluster Architecture, Installation & Configuration (25%)
- kubeadm installation (single and multi-node clusters)
- Kubeadm init, join, reset commands
- etcd backup/restore
- HA control plane setup
- RBAC: Roles, ClusterRoles, RoleBindings, ClusterRoleBindings
- ServiceAccounts
- Helm: install, repo management, charts
- Kustomize: overlays, patches
- CRDs: custom resources, API versions
- Operators (install via Operator Lifecycle Manager)
- CNI, CSI, CRI interfaces theory
- Practice: Build cluster with kubeadm, configure RBAC, install via Helm
Day 13-15: Storage (10%)
- PersistentVolume (PV) and PersistentVolumeClaim (PVC)
- StorageClasses and dynamic provisioning
- Volume types (emptyDir, hostPath, NFS, CSI)
- Access modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany)
- Reclaim policies (Retain, Delete, Recycle)
- Practice: Configure storage, provision PVCs dynamically
---
Phase 3: Operations & Troubleshooting (Days 16-22)
Day 16-17: Troubleshooting Clusters & Nodes
- Node troubleshooting (NotReady, memory/CPU pressure)
- Control plane component health (kube-apiserver, etcd, controller-manager, scheduler)
- kubectl describe, kubectl logs for diagnostics
- crictl for container runtime inspection
- kubelet logs and status
- Practice: Fix broken node scenarios
Day 18-19: Troubleshooting Applications & Networking
- Pod crash loops, ImagePullBackOff, OOMKilled
- Container log analysis
- kubectl exec for debugging
- Service connectivity issues
- DNS resolution problems in CoreDNS
- Network policy troubleshooting
- Practice: Debug failing pods and services
Day 20-22: Monitoring & Resource Management
- Metrics Server and kubectl top
- LimitRange and ResourceQuota
- Pod disruption budgets
- Application logging
- Horizontal Pod Autoscaler (HPA)
- Practice: Configure HPA, set resource quotas
---
Phase 4: Integration & Exam Prep (Days 23-28)
Day 23-24: Practice Exam Simulation 1
- Attempt Killer.sh exam simulator (first attempt)
- Time yourself: 2 hours strict
- Review mistakes and weak areas
Day 25-26: Targeted Remediation
- Focus on domains with lowest scores
- Practice commands until memorized:
  - kubectl set image, kubectl rollout undo
  - kubectl taint, kubectl cordon, kubectl drain
  - kubectl create role, kubectl auth can-i
  - helm install/upgrade, kubectl kustomize
Day 27-28: Practice Exam Simulation 2
- Attempt Killer.sh exam simulator (second attempt)
- Practice under exam conditions
- Focus on speed and documentation navigation
---
Phase 5: Final Review (Days 29-30)
Day 29: Command Reference Review
- Group commands by domain
- Review YAML manifest templates
- Practice dry-run commands: kubectl run --dry-run=client -o yaml
Day 30: Light Review & Relaxation
- Review only critical concepts
- Ensure exam environment readiness
- Rest before exam day
---
Daily Practice Commands (All 30 Days)
# Create alias for speed
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kgs='kubectl get services'
alias kd='kubectl describe'
alias kdpl='kubectl describe pod'
alias ka='kubectl apply -f'
complete -F __start_kubectl k
# Essential shortcuts
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
Key Resources
- Docs: kubernetes.io/docs, kubernetes.io/reference/generated/kubectl
- Course: LFS258 (Kubernetes Fundamentals)
- Practice: Killer.sh simulator, KillerCoda, Play with Kubernetes
- Community: Kubernetes Slack (#certified-kubernetes-admin)
---
Tip: Allocate ~2-3 hours daily. Troubleshooting (30%) is the highest-weight domain—prioritize hands-on practice over reading. Use the official documentation during practice to simulate exam conditions.