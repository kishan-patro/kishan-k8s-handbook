# K8s Essential Commands Reference

A small, dependable set of Kubernetes commands handles the majority of work across incidents, deploy cycles, and day-to-day cluster operations.

---

## Cluster & Nodes

| Command | Purpose |
|---|---|
| `kubectl get nodes -o wide` | Node health, IPs, OS, and runtime |
| `kubectl describe node <node>` | Taints, conditions, resource pressure |
| `kubectl top nodes` | Live CPU/memory usage per node |
| `kubectl cordon <node>` | Mark node unschedulable |
| `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data` | Evict pods before maintenance |
| `kubectl get componentstatuses` | Control-plane component health |

---

## Pods & Workloads

| Command | Purpose |
|---|---|
| `kubectl get pods -A -o wide` | All pods across all namespaces with node placement |
| `kubectl describe pod <pod> -n <ns>` | Events, conditions, lifecycle issues |
| `kubectl logs <pod> -n <ns> -f` | Stream live logs |
| `kubectl logs <pod> -n <ns> -c <container>` | Logs from a specific container in a multi-container pod |
| `kubectl exec -it <pod> -n <ns> -- sh` | Open shell in container |
| `kubectl rollout restart deploy/<name> -n <ns>` | Rolling restart without downtime |
| `kubectl rollout status deploy/<name> -n <ns>` | Watch rollout progress |
| `kubectl rollout undo deploy/<name> -n <ns>` | Rollback to previous revision |
| `kubectl top pods -A --sort-by=memory` | Resource stress sorted by memory |

---

## CrashLoop / OOMKilled / ImagePullBackOff

| Command | Purpose |
|---|---|
| `kubectl logs <pod> -n <ns> --previous` | Logs from the previous (crashed) container |
| `kubectl get events -n <ns> --sort-by=.lastTimestamp` | Ordered warning/error signals |
| `kubectl describe pod <pod> -n <ns> \| grep -A5 "Last State"` | Exit code and OOM details |
| `kubectl get pod <pod> -n <ns> -o yaml` | Full pod spec and status |
| `kubectl get pod <pod> -n <ns> -o jsonpath='{..imageID}'` | Verify resolved image digest |

> **Exit code cheat sheet:** `1` = app error, `137` = OOMKilled, `139` = segfault, `143` = graceful SIGTERM

---

## Services & Networking

| Command | Purpose |
|---|---|
| `kubectl get svc,endpoints -A` | Services and their backing endpoints |
| `kubectl describe svc <svc> -n <ns>` | Port/selector mismatches |
| `kubectl exec <pod> -n <ns> -- curl -s http://<svc>:<port>` | In-cluster connectivity test |
| `kubectl run tmp --rm -it --image=busybox --restart=Never -- nslookup <svc>` | DNS resolution check |
| `kubectl get networkpolicy -A` | Inspect traffic restrictions |

---

## Ingress & Gateway

| Command | Purpose |
|---|---|
| `kubectl get ingress -A` | All ingress routing rules |
| `kubectl describe ingress <name> -n <ns>` | Backend service / TLS failures |
| `kubectl exec <pod> -n <ns> -- curl -I http://<backend>:<port>` | Backend reachability from inside cluster |
| `kubectl get gateways,httproutes -A` | Gateway API resources |

---

## Storage

| Command | Purpose |
|---|---|
| `kubectl get pvc,pv -A` | Volume lifecycle overview |
| `kubectl describe pvc <name> -n <ns>` | Stuck, Pending, or unbound claims |
| `kubectl get storageclass` | Provisioner details and default class |
| `kubectl get volumeattachment` | CSI attach/detach state |
| `kubectl logs <pod> -n <ns> \| grep -i mount` | Mount errors in pod logs |

---

## ConfigMaps & Secrets

| Command | Purpose |
|---|---|
| `kubectl get cm,secret -A` | Full config inventory |
| `kubectl describe cm <name> -n <ns>` | Detect misconfiguration |
| `kubectl get secret <name> -n <ns> -o jsonpath='{.data}' \| base64 -d` | Decode secret value |
| `kubectl exec <pod> -n <ns> -- ls /etc/<mount-path>` | Validate mounted config files |

---

## RBAC

| Command | Purpose |
|---|---|
| `kubectl auth can-i <verb> <resource> --as=<user> -n <ns>` | Permission check for a user |
| `kubectl auth can-i --list --as=<user> -n <ns>` | Full permission list for a user |
| `kubectl get rolebinding,clusterrolebinding -A` | Who has access to what |
| `kubectl describe clusterrole <name>` | Role capabilities |

---

## Cluster Discovery

| Command | Purpose |
|---|---|
| `kubectl api-resources` | All available API groups and kinds |
| `kubectl get all -A` | Broad cluster sweep |
| `kubectl get deploy,sts,ds -A` | Controller view |
| `kubectl get hpa,vpa -A` | Autoscaling resources |
| `kubectl get crd` | All custom resource definitions |

---

## kubectl Plugins (Power-Ups)

Install via [krew](https://krew.sigs.k8s.io/):

| Plugin | Purpose |
|---|---|
| `stern` | Tail logs across multiple pods simultaneously |
| `kubectl tree` | Visualise workload ownership hierarchy |
| `kubectl neat` | Strip noise from YAML output |
| `kubectl access-matrix` | Full RBAC visibility matrix |
| `kubectl node-shell` | Open a privileged shell on a node |
| `kubectx` | Switch between clusters |
| `kubens` | Switch between namespaces |

---

> Kubernetes rarely needs complex tooling for day-to-day work. Knowing this command set deeply will resolve the vast majority of incidents.
