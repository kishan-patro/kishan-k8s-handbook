# Kubernetes Concept Story Cheat Sheet (1-Page)

Source: 16-K8s-concepts-story-guide.md

## Rapid Q and A

### 1) Why not run production directly as a single Pod?
A single Pod is ephemeral. If it dies, availability is gone until something recreates it.

### 2) Why Deployment after Pod?
Deployment keeps desired replica count, replaces failed Pods, and supports controlled rollouts.

### 3) Why Service after Deployment?
Pod IPs change. Service gives a stable virtual IP and DNS name, routing by labels.

### 4) Why Ingress after Service?
Many public Services can become expensive and hard to manage. Ingress centralizes host/path routing.

### 5) Why does Ingress still need a controller?
Ingress is only policy. An Ingress Controller enforces it (for example NGINX, Traefik, AWS LB Controller).

### 6) Why ConfigMap?
To separate non-secret config from images so the same image works across environments.

### 7) Why Secret if ConfigMap exists?
ConfigMap is not for sensitive data. Secret supports stronger access control and works with encryption at rest.

### 8) Are Kubernetes Secrets encrypted by default?
Not by default in all setups. Values are base64 encoded; enable etcd encryption at rest for real encryption.

### 9) Why HPA?
Manual scaling is slow and wasteful. HPA scales Pods based on metrics.

### 10) Why do Pods stay Pending even with HPA?
HPA scales Pods, not nodes. If the cluster has no capacity, Pods remain unschedulable.

### 11) What solves node capacity issues?
Node autoscaling (Karpenter or Cluster Autoscaler) adds and removes nodes based on demand.

### 12) Why Requests and Limits?
Requests help scheduling. Limits prevent noisy-neighbor resource starvation.

## Concept Chain (Memory Trigger)
Pod -> Deployment -> Service -> Ingress -> Ingress Controller -> ConfigMap -> Secret -> HPA -> Node Autoscaler -> Requests and Limits

## Interview 30-Second Version
Kubernetes abstractions exist to remove specific operational pain.
Deployment gives self-healing replicas, Service gives stable discovery, Ingress gives centralized entry, ConfigMap and Secret separate config from code securely, HPA scales Pods, node autoscaler scales infrastructure, and requests/limits keep workloads predictable.

## Red Flags To Mention In Interviews
1. Hardcoded environment config in container images.
2. Passwords stored in ConfigMaps.
3. HPA configured without cluster capacity strategy.
4. Workloads without resource requests and limits.
5. Assuming Ingress works without an Ingress Controller.
