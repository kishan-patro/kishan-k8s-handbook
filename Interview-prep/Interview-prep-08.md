# Interview Prep 08: Kubernetes Troubleshooting and Operations

## 1) A Pod is running but the Service is not accessible. How do you debug this issue?
- Verify Service selector matches Pod labels.
- Check Service endpoints: `kubectl get endpoints <service-name>`.
- Confirm Pod container port matches Service `targetPort`.
- Test connectivity from inside the cluster (`kubectl exec`, `curl`, `nc`).
- Check NetworkPolicy, security groups, firewall rules, and Ingress config if external.

## 2) What is CrashLoopBackOff and how do you resolve it?
- `CrashLoopBackOff` means a container starts, crashes, and Kubernetes keeps retrying with backoff.
- Check current and previous logs:
	- `kubectl logs <pod> -c <container>`
	- `kubectl logs <pod> -c <container> --previous`
- Inspect events and container exit code with `kubectl describe pod <pod>`.
- Common fixes: correct command/args, missing env/config, failed dependency, probe misconfiguration, or insufficient resources.

## 3) What is OOMKilled and how do you prevent it in Kubernetes?
- `OOMKilled` means the container exceeded its memory limit and was terminated by the kernel.
- Prevent by setting realistic `requests` and `limits` based on profiling.
- Tune application memory usage and JVM/GC settings where applicable.
- Use Vertical Pod Autoscaler (recommendation mode first) for right-sizing.
- Monitor memory trends and set alerts before saturation.

## 4) How does Kubernetes handle Pod failures automatically?
- A Pod restart is handled by `restartPolicy` (usually `Always` in Deployments).
- Deployments/ReplicaSets recreate failed Pods to maintain desired replicas.
- If a node fails, Pods are rescheduled to healthy nodes (except some local-state constraints).
- Readiness/liveness probes help route traffic only to healthy containers.

## 5) Explain the difference between Deployment and StatefulSet with a use case.
- Deployment:
	- Best for stateless apps.
	- Pods are interchangeable and have random identities.
	- Typical use case: web API or frontend.
- StatefulSet:
	- Best for stateful apps requiring stable identity/storage.
	- Pods get stable names and per-Pod persistent volumes.
	- Typical use case: databases like PostgreSQL, Kafka, or Elasticsearch.

## 6) How do you perform rolling updates in Kubernetes?
- Update image or Pod template in Deployment:
	- `kubectl set image deployment/<name> <container>=<image>:<tag>`
	- or `kubectl apply -f deployment.yaml`
- Watch rollout progress: `kubectl rollout status deployment/<name>`.
- Configure strategy with `maxSurge` and `maxUnavailable` for controlled replacement.

## 7) How do you rollback a Kubernetes deployment?
- Check rollout history: `kubectl rollout history deployment/<name>`.
- Roll back to previous revision: `kubectl rollout undo deployment/<name>`.
- Roll back to specific revision: `kubectl rollout undo deployment/<name> --to-revision=<n>`.

## 8) Why does a Pod remain in Pending state and how do you fix it?
- Common reasons:
	- Insufficient CPU/memory.
	- PVC not bound.
	- Node selector/affinity/taints block scheduling.
	- Image pull secret or registry access issues.
- Diagnose with `kubectl describe pod <pod>` and scheduler events.
- Fix constraints/resources, then recreate or let scheduler retry.

## 9) How do you scale applications in Kubernetes?
- Manual scaling:
	- `kubectl scale deployment/<name> --replicas=<n>`
- Auto scaling:
	- Horizontal Pod Autoscaler (HPA) for Pods.
	- Cluster Autoscaler for node count.
	- Vertical Pod Autoscaler for request/limit tuning.

## 10) How does Horizontal Pod Autoscaler work?
- HPA watches metrics (CPU, memory, or custom/external metrics).
- It adjusts replica count to match target utilization.
- Formula concept:
	- desired replicas is proportional to current metric divided by target metric.
- Requires Metrics Server or Prometheus adapter for metrics pipeline.

## 11) What happens when a Kubernetes node goes down?
- Node becomes `NotReady` after missed heartbeats.
- Pods on that node are marked unavailable.
- Controller reschedules replacement Pods on healthy nodes.
- Pods using local storage may lose local data unless backed by network PV.

## 12) How do you check logs of a Pod in Kubernetes?
- Single container: `kubectl logs <pod>`
- Specific container: `kubectl logs <pod> -c <container>`
- Previous crashed instance: `kubectl logs <pod> --previous`
- Stream logs: `kubectl logs -f <pod>`

## 13) How do you expose a Kubernetes application externally?
- `ClusterIP`: internal-only service.
- `NodePort`: exposes on each node IP and fixed port range.
- `LoadBalancer`: cloud-managed external load balancer.
- `Ingress`: HTTP/HTTPS routing with host/path rules (requires Ingress Controller).

## 14) How do you manage configuration changes in Kubernetes?
- Store non-secret config in ConfigMaps; secrets in Secret objects.
- Mount as environment variables or volumes.
- Version config and use immutable patterns where possible.
- Trigger rollout after config change (annotation bump, checksum pattern, or reapply).

## 15) How do you secure access to a Kubernetes cluster?
- Use RBAC with least privilege.
- Integrate auth with OIDC/IAM and short-lived credentials.
- Restrict API server access (private endpoint, IP allow list).
- Enforce network segmentation using NetworkPolicies.
- Audit API activity and enable logging/monitoring.

## 16) How do you manage secrets in Kubernetes?
- Avoid plain text in manifests and Git.
- Use Secret objects with encryption at rest enabled.
- Prefer external secret managers (Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault).
- Rotate credentials and avoid long-lived static secrets.

## 17) How do you monitor a Kubernetes cluster in production?
- Metrics: Prometheus + Alertmanager + Grafana.
- Logs: centralized logging (EFK/ELK or Loki stack).
- Traces: OpenTelemetry + tracing backend.
- Watch SLOs: API latency, error rates, saturation, restarts, and node health.

## 18) How do you debug DNS issues inside Kubernetes?
- Validate CoreDNS pods and service in `kube-system`.
- Run DNS checks from a test Pod: `nslookup`, `dig`.
- Verify `/etc/resolv.conf` in Pod and cluster domain settings.
- Check NetworkPolicies blocking DNS (UDP/TCP 53).

## 19) What happens if a container crashes repeatedly inside a Pod?
- Kubernetes restarts it based on `restartPolicy`.
- Repeated failures lead to `CrashLoopBackOff` with increasing backoff delay.
- Pod may stay unready, causing traffic loss for that replica.
- Use logs, events, probes, and resource analysis to identify root cause.

## 20) How do you ensure zero-downtime deployments in Kubernetes?
- Use rolling updates with proper `maxSurge` and `maxUnavailable` (often `maxUnavailable: 0`).
- Configure readiness probes so traffic shifts only to healthy Pods.
- Run multiple replicas across failure domains.
- Use PodDisruptionBudgets to protect availability during voluntary disruptions.
- Use canary or blue-green strategies for safer production rollouts.