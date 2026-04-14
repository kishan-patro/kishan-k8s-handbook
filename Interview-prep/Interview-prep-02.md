# Kubernetes Interview Prep 02

Focused notes for troubleshooting and scenario-based interviews. This file is aimed at the questions where the interviewer wants to see operating judgment, not just definitions.

---

## 1. A pod is stuck in `Pending`. What do you check?

### Strong answer

I check scheduling events first, because `Pending` usually means the scheduler could not place the pod.

### Debug sequence

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl top nodes
```

### Common root causes

- No nodes have enough CPU or memory
- Taints are blocking placement
- Node selectors or affinity rules are too restrictive
- PVC is still pending
- Image pull secret or admission issue is blocking scheduling flow

---

## 2. A service is not reachable. How do you debug it?

### Ordered approach

1. Check whether the service exists and has endpoints.
2. Verify the selector matches the pod labels.
3. Confirm the target container is listening on the expected port.
4. Test from inside the cluster before testing external access.

### Useful commands

```bash
kubectl get svc <service>
kubectl get endpoints <service>
kubectl describe svc <service>
kubectl get pods --show-labels
```

### Strong interview point

A service without endpoints is usually a selector or readiness problem, not a networking mystery.

---

## 3. A deployment rollout is stuck. What do you do?

### Strong answer

I check rollout status, new pod health, and deployment events. A stuck rollout usually means new replicas are not becoming ready.

```bash
kubectl rollout status deployment/<name>
kubectl describe deployment <name>
kubectl get rs
kubectl get pods
```

### Common reasons

- Readiness probe failures
- Bad image or image pull failure
- Insufficient cluster capacity
- Misconfigured environment variables, config maps, or secrets

---

## 4. Containers are restarting repeatedly. How do you isolate the cause?

### Debug sequence

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Things to check

- Exit code and termination reason
- Probe failures
- OOM events
- Application startup dependencies like DB or DNS connectivity

### Strong answer

I separate platform causes from application causes. If the platform is healthy and the container exits with an app error, I move quickly into configuration or code path validation.

---

## 5. A node becomes `NotReady`. What is your response?

### Immediate priorities

1. Check whether workloads were drained or impacted.
2. Verify kubelet status and node conditions.
3. Confirm network and disk health.

### Typical checks

```bash
kubectl describe node <node>
ssh <node>
sudo systemctl status kubelet
df -h
```

### Strong interview answer

I first protect service availability, then determine whether the issue is kubelet, resource exhaustion, network partition, or underlying VM failure.

---

## 6. DNS is failing inside the cluster. How do you debug it?

### Useful approach

1. Confirm the issue from a test pod.
2. Check CoreDNS pods and logs.
3. Verify network policies are not blocking DNS traffic.
4. Confirm upstream resolver behavior if external names also fail.

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system deployment/coredns
kubectl exec -it <pod> -- nslookup kubernetes.default.svc.cluster.local
```

---

## 7. A PVC stays in `Pending`. What does that usually mean?

### Strong answer

It usually means there is no matching storage class, dynamic provisioner, or available backing volume that satisfies the claim.

### Checks

```bash
kubectl get pvc,pv
kubectl describe pvc <claim>
kubectl get storageclass
```

### Common causes

- Wrong `storageClassName`
- Provisioner not installed or failing
- Requested access mode unsupported
- Zone mismatch in cloud environments

---

## 8. How do you debug `ImagePullBackOff`?

### Main causes

- Wrong image name or tag
- Private registry auth failure
- Network egress restriction
- Registry rate limiting

### Commands

```bash
kubectl describe pod <pod>
kubectl get secret
```

### Strong answer

I start with the events because Kubernetes usually tells you whether the failure is auth, name resolution, or not found.

---

## 9. How do you handle a production incident in Kubernetes?

### Strong structure

1. Stabilize user impact first.
2. Capture symptoms, timeline, and blast radius.
3. Roll back or reroute traffic if needed.
4. Preserve logs and evidence.
5. Fix root cause and write a post-incident review.

### Interviewers want to hear

- Calm prioritization
- Bias toward mitigation before perfect diagnosis
- Evidence-based root cause analysis

---

## 10. What metrics matter most for Kubernetes troubleshooting?

### Core categories

- Pod restarts and failed scheduling
- CPU and memory saturation
- Node pressure signals
- API server latency and error rates
- Request latency, success rate, and traffic volume for applications

### Strong answer

Infrastructure metrics alone are not enough. I combine platform metrics with service-level indicators so I can tell whether users are actually affected.

---

## 11. How do you reduce risk during deployment?

### Good answer

I reduce risk with small rollouts, health gates, rollback readiness, and pre-deployment validation.

### Practical controls

- Readiness and startup probes
- Rolling updates or canaries
- Automated rollback conditions
- Config validation before apply
- Monitoring during rollout

---

## 12. What would you do if HPA is not scaling as expected?

### Debug sequence

```bash
kubectl get hpa
kubectl describe hpa <name>
kubectl top pods
```

### Common causes

- Metrics server missing or unhealthy
- Requests not defined correctly
- Custom metric pipeline broken
- Max replicas too low

### Important point

HPA decisions are relative to resource requests, so bad requests produce misleading autoscaling behavior.

---

## 13. A pod can start but the application still fails. What next?

### Strong answer

At that point I stop thinking only in Kubernetes terms and shift to dependency flow. I verify database access, DNS resolution, IAM permissions, certificates, feature flags, and upstream service health.

### Interview insight

The pod running does not mean the service is healthy. Platform success and application success are different checks.

---

## 14. How would you explain a root cause analysis?

### Simple structure

- What happened
- What users saw
- What changed
- Why defenses did not prevent it
- How it was mitigated
- What permanent fixes were added

### Strong answer

Good RCA is factual, timeline-driven, and action-oriented. It avoids blame and focuses on system conditions and prevention.

---

## 15. Closing guidance

- Walk through incidents in order, not as scattered observations.
- Differentiate symptom, trigger, and root cause.
- Mention tradeoffs and the fastest safe mitigation path.
- Show that you can operate under pressure without losing structure.
