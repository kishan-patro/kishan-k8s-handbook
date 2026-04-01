# K8s Incident Response Flow

> **Golden Rule — Run this first, every time:**
> ```bash
> kubectl get events --sort-by=.lastTimestamp -n <namespace>
> ```
> Events expire in **60 minutes**. The evidence disappears while you're still Googling.

---

## The Wrong Way (First 8 Minutes)

Most engineers burn the first 8 minutes doing this:

1. Check logs → nothing obvious
2. Restart the pod → problem comes back
3. Panic-Google "CrashLoopBackOff fix" → down a rabbit hole

**The incident isn't what kills you. The wasted first 8 minutes are.**

---

## Incident Response Flowchart

---

## Pod State Diagnostic Runbook

### Pod Pending

**What it means:** The scheduler cannot place the pod on any node.

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl get nodes -o wide
kubectl describe node <node-name>
```

**Check in order:**
1. Resource quotas — `kubectl describe resourcequota -n <namespace>`
2. Node pressure (CPU, memory, disk) — look for `Taints` and `Conditions` in node describe
3. Node selectors / affinity rules that no node satisfies
4. PVC not bound (if pod mounts a volume)

---

### CrashLoopBackOff

**What it means:** The container starts, crashes, and Kubernetes keeps restarting it with exponential backoff.

```bash
# The previous container holds the crash reason
kubectl logs <pod-name> -n <namespace> --previous

# Current logs (if any output before crash)
kubectl logs <pod-name> -n <namespace>

# Full pod details — check exit codes and last state
kubectl describe pod <pod-name> -n <namespace>
```

**Common causes:**
- Application misconfiguration (bad env vars, missing config/secret mounts)
- Missing dependencies (DB not ready, service unreachable)
- Entrypoint/command error
- Exit code `1` → app error; exit code `137` → OOMKilled; exit code `139` → segfault

---

### OOMKilled

**What it means:** The container exceeded its memory limit and was killed by the kernel.

```bash
kubectl describe pod <pod-name> -n <namespace>
# Look for: Last State: Terminated  Reason: OOMKilled
```

> **Do NOT raise the memory limit immediately.** First determine if it is a memory leak.

**Check in order:**
1. Review app metrics — is memory growing unbounded over time? → likely a leak
2. Review heap dumps or profiling data if available
3. If it is a one-time spike (e.g., startup), raising the limit is appropriate
4. Set both `requests` and `limits`; use VPA for right-sizing in the long term

---

### ImagePullBackOff / ErrImagePull

**What it means:** Kubernetes cannot pull the container image.

```bash
kubectl describe pod <pod-name> -n <namespace>
# Look for: Failed to pull image / unauthorized / not found
```

**Check in order (80% of cases are one of these two):**
1. **Wrong image tag** — verify the tag exists in the registry
2. **Dead or missing registry secret** — `kubectl get secret -n <namespace>`, confirm `imagePullSecrets` is set in the pod spec
3. Registry is unreachable from the node — check network/firewall rules last

---

### Evicted

**What it means:** The pod was forcibly removed by the kubelet due to resource pressure. The pod itself did not fail.

```bash
kubectl describe node <node-name>
# Look for: Conditions — MemoryPressure / DiskPressure / PIDPressure

kubectl get events --sort-by=.lastTimestamp -n <namespace> | grep -i evict
```

**Check in order:**
1. **Disk pressure** — most common; check `/var/lib/docker` or `/var/lib/containerd` usage
2. Memory pressure — check node allocatable vs requested
3. Set `PodDisruptionBudget` to control eviction impact on critical workloads
4. Use `priorityClassName` to protect high-priority pods from eviction

---

## Quick Reference Command Cheatsheet

| Scenario | Command |
|---|---|
| First look at events | `kubectl get events --sort-by=.lastTimestamp -n <ns>` |
| Pod status overview | `kubectl get pods -n <ns> -o wide` |
| Previous container logs | `kubectl logs <pod> -n <ns> --previous` |
| Pod details & conditions | `kubectl describe pod <pod> -n <ns>` |
| Node pressure & taints | `kubectl describe node <node>` |
| Resource quota usage | `kubectl describe resourcequota -n <ns>` |
| Top resource consumers | `kubectl top pods -n <ns> --sort-by=memory` |

---

## Resolution Time Benchmark

Using this sequence, average incident resolution drops from **~40 minutes → under 7 minutes**.

The key: **start with events, match the pod state to the runbook, follow the ordered checklist.**