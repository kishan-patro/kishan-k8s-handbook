# Interview Prep 10 - Scenario Q&A (Kubernetes)

## 1. How does DNS resolution work inside a Pod?
**Short answer:** Pod -> `/etc/resolv.conf` -> CoreDNS Service IP -> CoreDNS resolves cluster name -> returns A/AAAA record.

**Flow:**
1. Application inside the Pod asks for `svc-name.namespace.svc.cluster.local`.
2. Query follows nameserver from Pod `/etc/resolv.conf` (usually kube-dns/CoreDNS ClusterIP).
3. CoreDNS checks Kubernetes API-backed records for Services/Endpoints.
4. For ClusterIP Service: returns virtual IP.
5. For headless Service: returns Pod IPs.
6. Client connects, kube-proxy/IPVS/iptables routes traffic.

**First checks when service is not reachable by name:**
1. `kubectl exec -it <pod> -- cat /etc/resolv.conf`
2. `kubectl get svc -n <ns>` and confirm exact service name/namespace
3. `kubectl get endpoints <svc> -n <ns>` (empty endpoints is common root cause)
4. `kubectl -n kube-system get pods -l k8s-app=kube-dns` (or CoreDNS label)
5. `kubectl exec -it <pod> -- nslookup <svc>.<ns>.svc.cluster.local`

## 2. Controller Manager’s role during a Deployment (reconciliation view)
**What actually happens:**
1. You apply/update Deployment spec in API server (stored in etcd).
2. Deployment controller notices spec drift (observed state != desired state).
3. It creates/updates ReplicaSet(s) with revision labels.
4. ReplicaSet controller reconciles Pod count for each RS.
5. During rolling update:
	- scales new RS up using `maxSurge`
	- scales old RS down using `maxUnavailable`
	- keeps reconciling until desired replicas available.
6. If probes fail or progress deadline breaches, rollout is marked failed.

**Key point:** kubectl only writes intent and reads status; controllers execute convergence loops continuously.

## 3. What if a node with local storage is autoscaled down?
**Risk:** local node storage (`emptyDir`, node disk, hostPath, local PV tied to node) is not portable. Node termination can mean permanent data loss.

**Typical failure modes:**
1. Stateful workload accidentally using local path -> data disappears on reschedule.
2. Cluster Autoscaler evicts node -> pod recreated elsewhere with empty data.
3. App boots “clean” and silently corrupts state consistency.

**How to prevent:**
1. Use network-attached persistent volumes (EBS/GCE PD/Ceph/etc.) for durable data.
2. For StatefulSets, use `volumeClaimTemplates` and proper StorageClass.
3. Mark critical pods with PDB and safe eviction settings.
4. Restrict autoscaler from draining stateful nodes without safeguards.
5. Run periodic backup + restore drills.

## 4. Post-deploy latency spike for 30% users, no errors/logs/alerts: 3-step triage
1. **Slice blast radius by dimension**
	- break down by AZ/region/node/pod/version/path/client ASN.
	- check if slow users map to a specific zone, ingress shard, or node pool.

2. **Compare old vs new path quickly**
	- canary rollback one segment or route small traffic to previous version.
	- diff deployment spec: resources, probes, sidecars, DNS, connection pool, timeouts.
	- watch p95/p99 and saturation (`CPU throttling`, `conntrack`, queue depth).

3. **Validate network and dependency latency**
	- ingress -> service -> pod hop timing.
	- DNS lookup time, TLS handshake time, upstream dependency timings.
	- if 30% aligns with one AZ/nodegroup, cordon/drain suspect nodes and rebalance.

## 5. Runtime security enforcement in Kubernetes
**PSP is deprecated.** Use layered controls:

1. **Pod Security Admission** at namespace level (`restricted` where possible).
2. **Policy as code** with OPA Gatekeeper or Kyverno:
	- block privileged pods
	- require non-root, read-only root FS
	- enforce allowed registries and signed images
3. **Kernel/runtime controls:** AppArmor/Seccomp/SELinux profiles.
4. **Least privilege:** strict RBAC + scoped service accounts + no default SA token mount unless needed.
5. **Supply chain:** image scanning + SBOM + signature verification (Cosign).
6. **Detection:** Falco/Tetragon/eBPF runtime alerts + audit logs to SIEM.

## 6. HPA vs VPA vs Karpenter - when NOT to use each
### HPA - do not use when
1. Workload is bottlenecked by external dependency latency, not pod resources.
2. Metrics are noisy/missing and scale decisions will flap.
3. App has long cold-start and cannot tolerate oscillation.

### VPA - do not use when
1. You need strict low-disruption workloads (VPA may evict/restart pods in Auto/Recreate mode).
2. Combined carelessly with HPA on same resource signal (can conflict).
3. Stateful/latency-critical services where restarts are expensive.

### Karpenter - do not use when
1. Workload scheduling is stable and existing node groups already right-sized.
2. Organization cannot manage spot interruption and consolidation behavior.
3. Compliance requires tightly static, pre-approved node shapes only.

**Bonus: simulate HPA in staging**
1. Mirror production requests with replay tools or synthetic load.
2. Publish same custom/external metrics pipeline used in prod.
3. Run controlled step-load tests and observe desired vs current replicas, stabilization windows, and scale-down behavior.
4. Test with and without CPU throttling limits to avoid false confidence.

## 7. Example outage story (war-room style answer)
**Incident:** Intermittent 503s and latency spike after rollout.

**Timeline summary:**
1. Alert triggered on p95 latency and ingress 5xx.
2. Error rate not uniform; concentrated in one AZ.
3. Found readiness probe passing but upstream dependency timeouts increasing.
4. New deployment changed connection pool + lowered timeout.
5. Under AZ-local traffic burst, pods saturated connections and returned slow/503.

**Actions:**
1. Rolled back timeout change.
2. Raised max connections and tuned keepalive.
3. Added per-AZ SLO dashboards and dependency timeout alerts.
4. Added rollout guardrail: automated canary abort on p95 regression.

**Postmortem outputs:**
1. Root cause documented with contributing factors.
2. Corrective actions assigned with owners and due dates.
3. Game day added to reproduce dependency saturation scenario.