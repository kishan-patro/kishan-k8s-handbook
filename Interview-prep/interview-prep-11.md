# Interview Prep 11 - Reviewed Answer Guide

## Round 1 - Infra, Kubernetes, and Cloud Patterns (45 mins)

### 1) Design a multi-tenant EKS cluster with isolation across dev, QA, and prod, with no noisy neighbors.
Answer:
- Prefer hard environment isolation first: separate AWS accounts per env (dev/qa/prod) and separate EKS clusters for prod when risk/compliance is high.
- Inside each cluster: namespace-per-team/app, RBAC least privilege, Pod Security Admission at restricted baseline, NetworkPolicy default deny.
- Prevent noisy neighbors: per-namespace ResourceQuota + LimitRange, requests/limits mandatory, PriorityClass + preemption policy, Cluster Autoscaler with separate node groups.
- Workload isolation: taints/tolerations and dedicated node groups for critical services.
- Governance: OPA/Gatekeeper or Kyverno policies to block missing limits, privileged pods, hostPath, and wildcard RBAC.

### 2) What is your approach to managing 10+ Kustomize overlays without drift or duplication?
Answer:
- Keep one base per service and only env deltas in overlays.
- Use components for reusable concerns (logging sidecar, PDB, HPA policy, securityContext).
- Prefer JSON6902 strategic patches over copy-paste full manifests.
- Enforce conventions with CI: `kustomize build` + `kubeconform` + policy checks + diff against last applied.
- GitOps guardrails: only overlay values differ, base changes require PR templates/checklists.
- Version overlays through promotion flow (dev -> qa -> prod), not manual edits.

### 3) Explain how you would secure cross-region S3 replication and validate data integrity at scale.
Answer:
- Enable versioning on source and destination buckets.
- Use dedicated replication IAM role with least privilege and explicit bucket policy trust.
- Encrypt with SSE-KMS and multi-region keys if required; allow key usage for replication principal.
- Restrict network and access: VPC endpoints, block public access, bucket ownership controls, CloudTrail data events.
- Integrity validation: S3 Inventory + checksum fields, periodic Athena queries to compare object count/size/checksum by prefix/date.
- Alert on replication metrics (bytes pending, operations failed, replication latency).

### 4) What happens when systemd hits a failing unit in a containerized node? How would you auto-recover?
Answer:
- `systemd` marks unit failed after restart threshold (`StartLimitBurst/StartLimitIntervalSec`) and stops retrying.
- For kube nodes, this can break kubelet/container runtime and make node NotReady.
- Auto-recovery:
- Set sane `Restart=on-failure` and start-limit policies for critical units.
- Add node-level health checks (SSM/DaemonSet) to detect failed critical units.
- Trigger remediation: restart unit, drain node, recycle instance via ASG lifecycle hooks if unhealthy persists.
- Preserve diagnostics before recycle: `journalctl -u <unit>`, `dmesg`, runtime logs to centralized store.

### 5) Walk through your strategy to detect and mitigate pod-to-pod lateral movement inside a cluster.
Answer:
- Detection:
- Flow logs from CNI (Cilium/Hubble/Calico), unusual east-west connections, DNS anomaly detection.
- Runtime signals (Falco/eBPF): shell spawn in app containers, suspicious binaries, privileged escalation attempts.
- Mitigation:
- Default-deny NetworkPolicy in every namespace, allow-list only required service paths.
- Enforce mTLS identity between services (service mesh) and short-lived workload identities.
- Disable service account token auto-mount unless needed; least-privilege RBAC.
- Block host namespace sharing, privileged pods, and hostPath mounts.

### 6) How do you perform zero-downtime upgrades for a stateful workload using Helm 3?
Answer:
- Prereqs: readiness/liveness probes correct, PDB in place, anti-affinity for replicas, storage class supports online operations.
- Use rolling strategy per StatefulSet partition if needed.
- Helm flow:
- `helm upgrade --atomic --timeout <sufficient>` with canary values first.
- If schema change exists, run backward-compatible DB migration (expand/contract pattern).
- Validate with synthetic checks and SLO metrics before full rollout.
- Keep rollback path: chart version pinning + `helm history` + tested restore for data layer.

### 7) Describe a hybrid cloud routing architecture between GCP and AWS. Where do you enforce boundaries?
Answer:
- Connectivity options: Cloud Interconnect + Direct Connect (preferred) or IPSec VPN for smaller scale.
- Use hub-and-spoke with transit gateways/cloud routers and BGP route exchange.
- Segment by environment and trust zone using separate VRFs/VPCs/projects/accounts.
- Enforce boundaries at multiple layers:
- Network ACLs, security groups/firewall rules, route table scoping.
- L7 policy at ingress/egress gateways (WAF, API gateway).
- Identity boundary: workload identity federation and least-privilege IAM.
- Add centralized observability for cross-cloud path latency, packet drops, and route flap alerts.

### 8) Your Terraform state got corrupted during a backend migration. Rebuild strategy?
Answer:
- Freeze changes: stop CI applies and lock deployment pipelines.
- Recover fastest safe copy: backend version history/snapshots/object version restore.
- Validate state integrity locally: `terraform state pull` and JSON sanity checks.
- Reconcile drift:
- `terraform import` for critical resources missing from state.
- `terraform plan -refresh-only` to inspect real-vs-state.
- Split risky modules if blast radius is large.
- Apply in small batches with explicit targets only if required.
- Postmortem: enforce remote backend locking, state snapshots, migration runbook, and break-glass restore drill.

### 9) Bash one-liner: Find all running containers using more than 500MB RSS memory on a node.
Answer:
```bash
crictl stats --no-stream | awk 'NR==1{next} {gsub("MiB","",$4); if ($4+0 > 500) print $1, $2, $4 "MiB"}'
```
Notes:
- For Docker-based nodes, use `docker stats --no-stream --format` equivalent.
- RSS semantics can vary by runtime output; confirm column mapping on your node image.

## Round 2 - Real Fire, RCA, and Chaos Control (75 mins)

### 1) A new AWS ALB config caused TLS handshakes to fail intermittently. Walk through your full RCA path.
Answer:
- Scope blast radius: which hostnames, AZs, client types, percent failure.
- Compare before/after ALB listener, TLS policy, cert attachment, target group, and health check changes.
- Validate cert chain/SNI:
- `openssl s_client -connect host:443 -servername host` from multiple networks.
- Check ACM cert expiry, intermediate chain, and mismatch across listeners.
- Correlate ALB access logs + CloudWatch metrics (`TLSNegotiationErrorCount`, 4xx/5xx, target resets).
- If intermittent by AZ, inspect subnet/NACL/security groups and target registration churn.
- Mitigation: roll back listener policy/cert mapping via IaC, then reintroduce changes behind canary ALB.

### 2) Kubernetes nodes are healthy, but kubectl logs is blank for critical pods. What is happening?
Answer:
- Common causes:
- Container restarted and logs rotated out before retrieval.
- App writing to file, not stdout/stderr.
- Wrong container selected in multi-container pod.
- CRI/logging driver issue on node (`/var/log/containers` symlink/runtime log path broken).
- RBAC denies logs but command path hides details.
- Checks:
- `kubectl get pod -o wide`, `kubectl describe pod`, `kubectl logs -c <name> --previous`.
- Node runtime logs (`containerd`, `kubelet`) and disk pressure/rotation settings.
- Fix: standardize stdout logging, retention, and centralized collection (Fluent Bit/OTel).

### 3) You deployed a sidecar logging agent. Suddenly CPU throttling spikes. Diagnose and rollback.
Answer:
- Confirm throttling:
- cAdvisor/Prometheus metrics: `container_cpu_cfs_throttled_periods_total` and throttled seconds.
- Compare before/after deployment and isolate namespaces/workloads.
- Inspect sidecar limits/requests and log volume (high parse/compress overhead).
- Check noisy patterns: multiline regex backtracking, heavy enrichment, sync flush intervals too aggressive.
- Immediate mitigation:
- Roll back Helm release or disable sidecar injection via label/annotation.
- Raise cpu limits temporarily for impacted critical workloads.
- Long-term fix:
- Move heavy parsing to central pipeline, keep sidecar lightweight, sample noisy logs.

### 4) Autoscaling is not kicking in despite CPU crossing threshold. What is broken - metrics, HPA, or API server?
Answer:
- Triage chain in order:
- Metrics availability: `kubectl top pods/nodes` and metrics-server health.
- HPA object: `kubectl describe hpa` events, current vs target metric, scale limitations.
- Target workload: resource requests set? HPA CPU utilization depends on requests.
- API behavior: check `kube-controller-manager` logs and API server errors/latency.
- Constraints: maxReplicas reached, PDB, unschedulable pods due to node shortage.
- Fix root cause based on first failing hop, not by changing threshold blindly.

### 5) Prod users report 504s, but ELB health checks are green. Explain your isolation and triage process.
Answer:
- 504 indicates upstream timeout path, often beyond health-check endpoint.
- Isolate layer-by-layer:
- Client -> CDN/WAF -> ELB/ALB -> ingress -> service -> pod -> downstream (DB/cache/API).
- Compare health endpoint vs real user path (auth, DB calls, external APIs).
- Check connection pools, thread saturation, GC pauses, and per-route latency histograms.
- Inspect ingress timeouts, keepalive mismatch, idle timeout mismatch between ALB and upstream.
- Mitigate with timeout alignment, circuit breaker, and fail-open/fail-fast where appropriate.

### 6) Systemd journal logs vanish on reboot across some AMIs. What do you check in image build and boot sequence?
Answer:
- Verify persistent journald config in image:
- `/etc/systemd/journald.conf` with `Storage=persistent`.
- Ensure `/var/log/journal` exists with correct ownership/permissions.
- Confirm no cloud-init/bootstrap script wipes log dirs on startup.
- Validate disk/partition mounts for `/var` are present before journald starts.
- Check tmpfs usage or immutable image steps that reset runtime state.
- Add AMI conformance test: reboot and assert previous-boot journal is readable.

### 7) A production pod was OOMKilled, but you cannot find logs. Walk through a forensic-level debug.
Answer:
- Collect pod lifecycle evidence:
- `kubectl describe pod` (last state, restart count, OOMKilled reason).
- `kubectl logs --previous -c <container>`.
- Node-level evidence:
- `dmesg -T | grep -i -E 'oom|killed process'`.
- kubelet and runtime logs around kill timestamp.
- Analyze limits vs actual usage:
- container working set trend, memory spikes, alloc pattern, page cache behavior.
- Confirm QoS class and whether node memory pressure evicted neighboring pods.
- Prevent recurrence: right-size limits/requests, memory profiling, and add early warning alerts.

### 8) Kernel panic on a GKE node mid-deploy. How do you identify if it is infra, base image, or app-level?
Answer:
- Build timeline: deployment change window, node events, kernel panic timestamp.
- Correlate by node image and kernel version: is panic only on a specific node pool/image family?
- Check GKE release notes and known kernel/runtime regressions.
- Compare workload traits on affected nodes:
- privileged/hostNetwork/eBPF modules/custom drivers/high syscall load.
- Run controlled repro in staging with same image and workload.
- If infra/image: roll node pool to known-good image.
- If app-triggered: cordon risky workload, reduce concurrency, add safeguard limits.

## Round 3 - Leadership, Engineering Influence, and Production Principles (30 mins)

### 1) How do you design infrastructure that empowers devs without giving them footguns?
Answer:
- Provide paved roads: golden templates for service, deployment, observability, and security defaults.
- Shift-left guardrails: policy-as-code in CI, secure defaults in Helm charts/modules.
- Self-service with boundaries: devs can ship within quota/policy envelopes; privileged changes require review.
- Fast feedback: preview environments, cost visibility, and SLO dashboards per team.

### 2) What is your Linux-level checklist before approving any custom AMI to production?
Answer:
- Security baseline: CIS hardening, patched packages, minimal services, SSH restrictions.
- Runtime correctness: container runtime/kubelet versions validated and pinned.
- Logging and observability: persistent journald, node exporter/agent health.
- Reliability: reboot test, disk pressure behavior, time sync, entropy, DNS resolution.
- Supply chain: image provenance/SBOM/signature and immutable build pipeline.
- Performance smoke: CPU, memory, IO baseline and regression thresholds.

### 3) Move from centralized logging to service-mesh-based observability model. Tradeoffs?
Answer:
- Pros:
- Better service-level telemetry (latency, retries, mTLS identity) without app code changes.
- Rich traffic policy and tracing context propagation.
- Cons:
- Sidecar/control-plane overhead, higher resource cost, operational complexity.
- Mesh telemetry is not full log replacement; app/business logs still needed.
- Vendor lock and debugging complexity can increase.
- Practical approach: hybrid model, phase migration by critical services first.

### 4) Describe how you simulate production-level chaos in staging for Kubernetes.
Answer:
- Start from real failure modes seen in incident history.
- Inject failures gradually:
- pod kill, node drain, DNS latency, packet loss, dependency timeout, disk pressure.
- Define steady-state SLO hypotheses before each experiment.
- Run during business hours with rollback playbooks and on-call participation.
- Capture learnings into runbooks, alerts, and automation improvements.

### 5) How do you handle pushback from leadership when SLOs threaten velocity?
Answer:
- Reframe in business terms: error budget equals controlled risk budget.
- Show data: incident cost, customer impact, and delivery interruption from unreliability.
- Offer options, not blockers:
- continue feature work with capped risk,
- or invest in reliability to unlock sustained velocity.
- Use staged commitments with measurable milestones and review cadence.

## Quick Review Feedback

- This set is strong on real-world troubleshooting and leadership depth.
- To make your interview performance sharper, answer each scenario with this structure:
- Scope and blast radius
- Fastest safe mitigation
- Root cause proof
- Long-term prevention