# 🧩 Simple Diagram: Crash Loop from EBS IOPS Throttling

```
   ┌──────────────────────────────┐
   │ 1. Disk IOPS Limit (EBS)     │
   └─────────────┬────────────────┘
                 │
                 ▼
   ┌──────────────────────────────┐
   │ 2. Disk Stalls, Processes    │
   │    Freeze                    │
   └─────────────┬────────────────┘
                 │
                 ▼
   ┌──────────────────────────────┐
   │ 3. Health Probes Timeout     │
   └─────────────┬────────────────┘
                 │
                 ▼
   ┌──────────────────────────────┐
   │ 4. Kubernetes Kills Pods     │
   │    (SIGTERM)                 │
   └─────────────┬────────────────┘
                 │
                 ▼
   ┌──────────────────────────────┐
   │ 5. Pod Restart = More Disk   │
   │    I/O (logs, mounts, etc.)  │
   └─────────────┬────────────────┘
                 │
                 ▼
   ┌──────────────────────────────┐
   │ 6. More Disk Pressure        │
   │    (> IOPS limit)            │
   └─────────────┬────────────────┘
                 │
                 ▼
           (LOOP REPEATS)
```

**Summary:**
Low IOPS → Disk freezes → Probes fail → Pods restart → Restarts generate more disk I/O → More throttling → Loop continues.

---

# 📄 Root Cause Analysis (RCA)

---

## Root Cause Summary

**Issue:**
Node `i-abcd` experienced repeated pod crashes and health-check failures due to **EBS gp3 IOPS throttling** on its ephemeral storage volume `vol-abcd` (`/dev/xvdb`).

**Trigger Condition:**
The EBS volume repeatedly hit its **3,000 IOPS gp3 baseline limit**, causing severe disk I/O starvation.

**Observed Evidence:**
- `EBSVolumeIOPSExceeded` alerts triggered multiple times (21m, 31m, 11m, 2m ago)
- Repeated disk stalls caused processes to stop responding to Kubernetes health probes

**Self-Reinforcing Failure Loop:**
1. Disk IOPS exceeded → disk stalls
2. Processes freeze → readiness/liveness probes timeout
3. Kubelet kills pods (exit code 143)
4. Pod restarts cause additional disk I/O (overlayfs mounts, container logs, containerd metadata writes)
5. Increased I/O worsens IOPS throttling
6. Loop repeats → **437 restarts across 16 pods**

**Why Only This Node?**
This node had a significantly higher density of infrastructure workloads.
The sibling node (`i-abcd`) had only 5 pods with 0 restarts and did not exceed its disk IOPS budget.

**Secondary Impact:**
- `rancher-webhook` repeatedly crashed due to disk stalls
- `cattle-cluster-agent` then failed to start due to missing webhook endpoints, blocking several dependent system workloads

---

## Remediation Summary

**Immediate Actions:**
- Cordon the node to prevent new scheduling:
  ```sh
  kubectl cordon i-abcd
  ```

**Long-Term Fix:**
Increase the ephemeral EBS gp3 performance in the NodeClass:

```yaml
ephemeralStorage:
  iops: 6000        # Increased from 3000
  throughput: 250   # Increased from 125 MB/s
  size: 100G
```

This prevents future IOPS saturation for nodes hosting infrastructure-heavy workloads.

---
