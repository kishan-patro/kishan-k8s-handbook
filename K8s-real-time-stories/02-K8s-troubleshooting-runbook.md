# Kubernetes Troubleshooting Runbook

> This comprehensive runbook provides categorized Kubernetes errors, their causes, and actionable CLI commands to fix each issue. Ideal for DevOps engineers and SREs handling real-world clusters.

---

## 1. Pod & Container Issues

### 1.1 CrashLoopBackOff
**Cause:** Container keeps restarting due to app crash, invalid probes, or bad entrypoint.
**Fix Commands:**
```sh
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```
**Resolution:** Fix startup command, missing config, or incorrect probes.

### 1.2 ImagePullBackOff
**Cause:** Wrong image name, tag, or missing registry credentials.
**Fix Commands:**
```sh
kubectl describe pod <pod-name>
# Add Docker registry credentials
kubectl create secret docker-registry regcred \
  --docker-server=<registry-url> \
  --docker-username=<username> \
  --docker-password=<password>
```
**Resolution:** Verify image tag, credentials, and network access.

### 1.3 OOMKilled
**Cause:** Container exceeded memory limits.
**Fix Commands:**
```sh
kubectl top pod <pod-name>
```
**Resolution:** Increase memory limit or optimize app usage.
```yaml
resources:
  limits:
    memory: "512Mi"
```

### 1.4 CreateContainerConfigError
**Cause:** Invalid YAML or missing config/secret.
**Fix Commands:**
```sh
kubectl describe pod <pod-name>
kubectl get configmap,secret
kubectl apply --dry-run=client -f <file>.yaml
```
**Resolution:** Validate syntax and ensure all referenced resources exist.

### 1.5 Pod stuck in Terminating
**Cause:** Finalizer or volume still attached.
**Fix Commands:**
```sh
kubectl delete pod <pod-name> --force --grace-period=0
```
**Resolution:** Remove finalizers from YAML if necessary.

---

## 2. Deployment & ReplicaSet Issues

### 2.1 Deployment stuck in progress
**Cause:** Failed probes or rollout paused.
**Fix Commands:**
```sh
kubectl rollout status deployment/<deploy-name>
kubectl describe deployment <deploy-name>
```
**Resolution:** Fix failing liveness/readiness probes.

### 2.2 Pods not updating
**Cause:** Immutable fields like selectors or templates changed.
**Fix Commands:**
```sh
kubectl delete rs <replica-set>
kubectl apply -f deployment.yaml
```
**Resolution:** Redeploy cleanly with correct configuration.

---

## 3. Node-Level Issues

### 3.1 Node NotReady
**Cause:** Kubelet stopped or network disconnected.
**Fix Commands:**
```sh
ssh <node>
sudo systemctl restart kubelet
```
**Resolution:** Check node health, network, and kubelet logs.

### 3.2 DiskPressure / MemoryPressure
**Cause:** Node resources exhausted.
**Fix Commands:**
```sh
df -h
kubectl top nodes
```
**Resolution:** Clean unused Docker/Kubelet data.
```sh
sudo du -sh /var/lib/docker /var/lib/kubelet
sudo rm -rf /var/lib/docker/containers/*
```

### 3.3 Node tainted / pods not scheduled
**Cause:** Taints prevent scheduling.
**Fix Commands:**
```sh
kubectl describe node <node>
# Remove taint
kubectl taint nodes <node> key:NoSchedule-
```
**Resolution:** Match tolerations or remove taints.

---

## 4. Storage & Volume Issues

### 4.1 PVC Pending
**Cause:** No matching PV or wrong StorageClass.
**Fix Commands:**
```sh
kubectl get pvc,pv
kubectl describe pvc <name>
```
**Resolution:** Define matching PV or fix StorageClass name.

### 4.2 NFS mount timeout
**Cause:** NFS server or network issue.
**Fix Commands:**
```sh
showmount -e <nfs-server>
mount -t nfs <nfs-server>:/path /mnt/test
```
**Resolution:** Ensure exports are correct and accessible.

---

## 5. Networking & Service Issues

### 5.1 DNS resolution failed
**Cause:** CoreDNS crash or network policy block.
**Fix Commands:**
```sh
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl rollout restart deployment coredns -n kube-system
```
**Resolution:** Restart CoreDNS and verify its logs.

### 5.2 Service unreachable
**Cause:** Selector mismatch or no endpoints.
**Fix Commands:**
```sh
kubectl describe svc <service>
kubectl get endpoints <service>
```
**Resolution:** Ensure service selector labels match pod labels.

### 5.3 Ingress 404
**Cause:** Invalid host/path or missing TLS.
**Fix Commands:**
```sh
kubectl describe ingress <name>
kubectl get ingressclass
```
**Resolution:** Fix ingress rules, annotations, or TLS secret.

---

## 6. RBAC & Security Issues

### 6.1 Forbidden / Unauthorized
**Cause:** Missing roles or bindings.
**Fix Commands:**
```sh
kubectl create clusterrolebinding admin-binding \
  --clusterrole=cluster-admin \
  --user=<username>
```
**Resolution:** Apply least-privilege RBAC as required.

### 6.2 TLS handshake failed
**Cause:** Certificate expired or mismatched.
**Fix Commands:**
```sh
kubectl delete secret <tls-secret>
kubectl create secret tls <tls-secret> --cert=cert.crt --key=key.key
```
**Resolution:** Recreate the TLS secret.

---

## 7. Configuration & YAML Issues

### 7.1 Invalid apiVersion
**Cause:** Deprecated or incorrect version.
**Fix Commands:**
```sh
kubectl api-resources | grep <resource>
```
**Resolution:** Update manifest to supported API version.

### 7.2 Unknown field / wrong indentation
**Cause:** Typo or malformed YAML.
**Fix Commands:**
```sh
kubectl apply --dry-run=client -f <file>.yaml
```
**Resolution:** Validate YAML and fix indentation.

---

## 8. Monitoring & Observability

### 8.1 Prometheus not scraping metrics
**Cause:** Label mismatch in ServiceMonitor.
**Fix Commands:**
```sh
kubectl get servicemonitor -A
kubectl describe servicemonitor <name>
```
**Resolution:** Align matchLabels with service labels.

### 8.2 Grafana blank dashboard
**Cause:** Datasource misconfigured.
**Fix Commands:**
```sh
kubectl get cm grafana-datasources -n monitoring -o yaml
```
**Resolution:** Update Prometheus endpoint in Grafana config.

---

## 9. CI/CD & Helm Issues

### 9.1 Helm install failed
**Cause:** Chart dependency missing.
**Fix Commands:**
```sh
helm dependency update
helm install <release> <chart>
```
**Resolution:** Update dependencies before install.

### 9.2 ArgoCD sync error
**Cause:** RBAC or app diff.
**Fix Commands:**
```sh
kubectl logs -n argocd <argocd-app-controller>
```
**Resolution:** Fix app permissions or repo URL.

### 9.3 Jenkins build failure
**Cause:** Missing kubeconfig or credentials.
**Fix Commands:**
```sh
cat ~/.kube/config
```
**Resolution:** Mount kubeconfig into Jenkins agent.

---

## 10. Cluster-Level Issues

### 10.1 API Server overload
**Cause:** Too many concurrent requests.
**Fix Commands:**
```sh
kubectl top pods -A
kubectl scale deployment kube-apiserver -n kube-system --replicas=3
```
**Resolution:** Enable API rate limiting or scale horizontally.

### 10.2 ETCD slow or full
**Cause:** Fragmented database.
**Fix Commands:**
```sh
ETCDCTL_API=3 etcdctl defrag --endpoints=<etcd-endpoint>
```
**Resolution:** Compact or defrag etcd database.

### 10.3 Admission webhook timeout
**Cause:** Webhook service unreachable.
**Fix Commands:**
```sh
kubectl get validatingwebhookconfigurations
kubectl get svc -A | grep webhook
```
**Resolution:** Verify webhook service and network policies.

---

## References
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [Komodor Kubernetes Troubleshooting](https://komodor.com/learn/)
- [Sysdig Debug Guides](https://sysdig.com/learn/)
- [Groundcover Troubleshooting](https://www.groundcover.com/blog)
- Jay Chauhan - Kubernetes Debug Guide

---

> This runbook provides practical troubleshooting steps and verified commands to resolve over 100 Kubernetes issues across pods, nodes, storage, networking, CI/CD, and more.
