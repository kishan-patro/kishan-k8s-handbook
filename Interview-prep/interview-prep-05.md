# Kubernetes Interview Prep 05

Detailed Kubernetes interview prep covering architecture, controllers, EKS concepts, RBAC, scheduling, troubleshooting, and common manifest patterns.

---

## 1. What happens when you run `kubectl apply -f app.yaml`?

`kubectl` sends the manifest to the API server. The API server validates and stores the desired state in etcd. Relevant controllers notice the new object and start reconciling actual cluster state toward the declared configuration.

---

## 2. What is etcd and why is it important?

`etcd` is the distributed key-value store that holds Kubernetes cluster state. If etcd is slow or unhealthy, the control plane becomes unreliable because desired state, leases, and configuration all depend on it.

---

## 3. What does the scheduler do?

The scheduler selects a suitable node for a pod that does not yet have one. It considers requests, taints, tolerations, affinity rules, topology constraints, and resource availability.

---

## 4. What is the job of the kubelet?

The kubelet runs on each node and ensures the containers described in pod specs actually run. It talks to the API server, works with the container runtime, mounts volumes, and reports node and pod status.

---

## 5. What is a ReplicaSet?

A ReplicaSet ensures a specified number of identical pod replicas exist. In practice, you usually manage ReplicaSets through Deployments rather than directly.

---

## 6. Why do we use Deployments instead of creating pods directly?

Pods are ephemeral and not self-healing by themselves. A Deployment gives declarative rollout management, replica control, rollout history, and rollback capability.

---

## 7. What is the difference between `requests` and `limits`?

Requests influence scheduling by reserving expected resources. Limits cap the maximum usage a container may consume. Memory limit breaches usually lead to `OOMKilled`, while CPU limits cause throttling.

---

## 8. What is a Namespace used for?

Namespaces logically separate workloads for teams, environments, and access boundaries. They help with RBAC scoping, quotas, and organization, but do not provide complete isolation on their own.

---

## 9. What is a ConfigMap and when would you use it?

A ConfigMap stores non-sensitive configuration, such as application flags, endpoints, and settings. Use it for runtime config that should be externalized from container images.

---

## 10. How is a Secret different from a ConfigMap?

Secrets are intended for sensitive values such as passwords, certificates, and tokens. They should be protected further with encryption at rest, restricted RBAC, and ideally external secret management.

---

## 11. What is a Service in Kubernetes?

A Service gives a stable network identity to a set of pods selected by labels. It decouples clients from changing pod IPs.

### Service types

- `ClusterIP` for internal access
- `NodePort` for exposing through node ports
- `LoadBalancer` for cloud-provider integration
- `ExternalName` for DNS aliasing

---

## 12. What is Ingress?

Ingress is an API object for HTTP and HTTPS routing into the cluster. It supports host-based and path-based rules and requires an ingress controller to implement those rules.

---

## 13. What are liveness, readiness, and startup probes?

- Liveness checks whether the container should be restarted.
- Readiness checks whether the pod should receive traffic.
- Startup protects slow-booting containers from premature liveness failures.

These three should be tuned independently.

---

## 14. What is RBAC?

RBAC controls which users, groups, or service accounts can perform actions on Kubernetes resources.

### Core objects

- `Role`
- `ClusterRole`
- `RoleBinding`
- `ClusterRoleBinding`

---

## 15. What is a DaemonSet?

A DaemonSet ensures one pod runs on every eligible node. It is commonly used for node exporters, log collectors, CNI agents, and security monitoring components.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
	name: node-agent
spec:
	selector:
		matchLabels:
			app: node-agent
	template:
		metadata:
			labels:
				app: node-agent
		spec:
			containers:
				- name: agent
					image: busybox
					command: ["sh", "-c", "sleep 3600"]
```

---

## 16. What is a StatefulSet?

A StatefulSet manages stateful workloads that need stable pod identity, ordered rollout behavior, and persistent storage. It is used for databases, brokers, and clustered systems where identity matters.

---

## 17. What is the difference between StatefulSet and Deployment?

Deployments assume interchangeable replicas. StatefulSets provide stable identities like `app-0`, `app-1`, persistent volume association, and ordered operations.

---

## 18. What is a PVC and why would it stay pending?

A PersistentVolumeClaim is a request for storage. It may stay pending if no matching storage class exists, provisioning fails, access modes do not match, or zone or topology constraints prevent binding.

```bash
kubectl get pvc,pv
kubectl describe pvc <name>
kubectl get storageclass
```

---

## 19. What causes a pod to remain in `Pending`?

Common reasons include insufficient node resources, taints without tolerations, restrictive affinity, unbound PVCs, or cluster autoscaler lag.

---

## 20. What causes `CrashLoopBackOff`?

The container starts and fails repeatedly. Causes include bad commands, missing config, probe failure, dependency failure, or application crashes.

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

---

## 21. What causes `ImagePullBackOff`?

Kubernetes cannot pull the image due to a wrong image name or tag, failed registry authentication, missing image pull secret, or registry connectivity issue.

---

## 22. How do you debug a service with no endpoints?

I check whether the service selector matches pod labels and whether the pods are ready. A service may exist, but without ready matching pods it will have no endpoints.

```bash
kubectl describe svc <service>
kubectl get endpoints <service>
kubectl get pods --show-labels
```

---

## 23. What is EKS?

Amazon EKS is AWS-managed Kubernetes. AWS operates the control plane, while you manage worker nodes, networking decisions, IAM integration, addons, and workloads.

### Points worth mentioning

- Managed control plane
- IAM integration
- OIDC provider for IRSA
- AWS VPC CNI by default

---

## 24. What is IRSA in EKS?

IRSA means IAM Roles for Service Accounts. It allows a Kubernetes service account to assume a scoped AWS IAM role, so a pod can access AWS resources without relying on node-level credentials.

---

## 25. What is the AWS VPC CNI?

The AWS VPC CNI gives pods IP addresses from the VPC network. That simplifies AWS-native networking and security group patterns, but it requires careful subnet and IP planning.

---

## 26. How would you explain blue-green node group upgrades in EKS?

I create a new node group with the target AMI or Kubernetes version, move workloads gradually by cordoning and draining old nodes, validate service health, and then decommission the old group. This reduces in-place upgrade risk and gives a clear rollback path.

### Useful commands

```bash
kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```

---

## Quick manifest examples

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: web
spec:
	replicas: 3
	selector:
		matchLabels:
			app: web
	template:
		metadata:
			labels:
				app: web
		spec:
			containers:
				- name: web
					image: nginx:1.25
					ports:
						- containerPort: 80
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
	name: web
spec:
	selector:
		app: web
	ports:
		- port: 80
			targetPort: 80
```

### CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
	name: cleanup
spec:
	schedule: "0 2 * * *"
	jobTemplate:
		spec:
			template:
				spec:
					restartPolicy: OnFailure
					containers:
						- name: cleanup
							image: busybox
							command: ["sh", "-c", "echo cleanup"]
```

### RBAC example

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
	name: pod-reader
	namespace: default
rules:
	- apiGroups: [""]
		resources: ["pods"]
		verbs: ["get", "list", "watch"]
```

---

## Final guidance

- Explain each object in terms of purpose and operational behavior.
- When asked a troubleshooting question, answer with an ordered debug path.
- Tie EKS answers back to IAM, networking, and production operations.
