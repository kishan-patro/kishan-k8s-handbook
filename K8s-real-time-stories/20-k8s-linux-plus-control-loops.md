# Kubernetes Is Linux Plus Control Loops

Kubernetes is easiest to understand when you map it to Linux primitives.

- A container is a Linux process isolated by namespaces and controlled by cgroups.
- A pod is a scheduling and networking boundary around one or more container processes.
- A volume is a Linux mount presented to a container.
- Service traffic and NetworkPolicy behavior are implemented by kube-proxy and CNI (iptables, IPVS, or eBPF underneath).
- CPU and memory limits are enforced through cgroups.

That is the foundation.

Kubernetes then adds a control-plane model on top:

- YAML is a desired-state declaration.
- Controllers are reconciliation loops.
- Operators are domain-specific controllers encoded in software.

Learn Linux deeply, then learn reconciliation. Kubernetes stops feeling magical and starts feeling predictable.

## Real-Time Stories

### 1) The Pod Was Healthy, But Users Still Saw 503

What happened:
- The app process started quickly, so the container looked alive.
- The app needed 20 to 30 seconds to warm cache and database connections.
- Readiness probe was too shallow and passed before the app was actually ready.

Linux and control-loop lens:
- The process existed, so basic liveness looked fine.
- Endpoint controller kept adding and removing Pod IPs based on readiness.
- Service routed traffic to endpoints that were technically up but not truly ready.

Fix that worked:
- Made readiness probe check a real dependency path.
- Increased initial delay and tuned failure thresholds.
- Result: endpoint flapping stopped and 503 rate dropped to near zero.

### 2) CPU Limit Looked Safe, Latency Exploded

What happened:
- Team set aggressive CPU limits to protect noisy neighbors.
- Under burst load, API response time increased from milliseconds to seconds.

Linux and control-loop lens:
- cgroups enforced quota and throttled the container.
- Scheduler had placed many busy Pods on the same nodes.
- HPA observed utilization, but scale-out lagged behind request bursts.

Fix that worked:
- Raised CPU limits for latency-sensitive workloads.
- Set realistic CPU requests to improve placement.
- Tuned HPA targets and stabilization windows.
- Result: throttling reduced and p95 latency stabilized.

### 3) NetworkPolicy Passed Review, Traffic Still Broke

What happened:
- A deny-by-default policy was applied in production.
- Cross-namespace calls to auth and metrics services started timing out.

Linux and control-loop lens:
- Policy intent was high level, but enforcement happened in CNI data plane.
- iptables or eBPF rules blocked flows that were not explicitly allowed.
- DNS and kube-system dependencies were accidentally excluded.

Fix that worked:
- Added explicit allow rules for DNS, namespace selectors, and required ports.
- Validated policy with staged rollout and synthetic checks.
- Result: intended isolation remained, but critical service paths recovered.

### 4) PVC Bound, Application Still Could Not Start

What happened:
- PersistentVolumeClaim showed Bound, but app logs showed permission denied.
- Restarting Pods did not help.

Linux and control-loop lens:
- Storage was attached and mounted, so control-plane status looked correct.
- Linux file ownership and mount permissions inside the container were wrong.
- Reconciliation could not fix app-level UID and GID mismatch by itself.

Fix that worked:
- Set securityContext with fsGroup and runAsUser aligned to image user.
- Added initContainer to correct directory ownership where needed.
- Result: startup succeeded without manual node-side intervention.

## Interview Close

A strong answer in interviews is simple:
- Start with Linux primitive (process, cgroup, mount, packet filter).
- Add Kubernetes controller behavior (what loop is reconciling what state).
- End with one concrete debugging command and one corrective action.