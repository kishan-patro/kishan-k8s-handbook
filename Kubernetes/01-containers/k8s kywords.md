# Kubernetes Keywords

This glossary covers the core Kubernetes terms that appear constantly in interviews and day-to-day cluster operations.

## Core terms

### Cluster

A set of control plane and worker nodes that together run Kubernetes workloads.

### Node

A machine, virtual or physical, that runs pods. Nodes contain kubelet, kube-proxy, and a container runtime.

### Pod

The smallest deployable unit in Kubernetes. A pod can run one or more closely related containers.

### Namespace

A logical partition used for organizing resources and scoping RBAC, quotas, and policies.

### Deployment

A controller for managing stateless applications through replica sets, rolling updates, and rollbacks.

### ReplicaSet

Ensures a target number of pod replicas are running.

### StatefulSet

Used for stateful applications that need stable identity and persistent storage.

### DaemonSet

Ensures a pod runs on every eligible node.

### Job

Runs a task until completion.

### CronJob

Runs jobs on a schedule.

## Networking terms

### Service

A stable network endpoint that routes traffic to a selected group of pods.

### ClusterIP

Internal-only service type.

### NodePort

Exposes a service on a port on each node.

### LoadBalancer

Creates an external cloud load balancer for a service.

### Ingress

An API object for HTTP and HTTPS routing into the cluster.

### Ingress Controller

The component that implements ingress rules.

## Configuration and security terms

### ConfigMap

Stores non-sensitive application configuration.

### Secret

Stores sensitive values such as passwords, certificates, or tokens.

### RBAC

Role-Based Access Control. Defines who can do what in the cluster.

### Role and ClusterRole

Permission definitions. Role is namespace-scoped, ClusterRole can be cluster-wide.

### RoleBinding and ClusterRoleBinding

Attach permissions to users, groups, or service accounts.

## Scheduling and runtime terms

### Requests

Minimum resources a container asks for. The scheduler uses these values.

### Limits

Maximum resources a container can consume.

### Taints and Tolerations

Taints repel pods from nodes. Tolerations allow specific pods onto those nodes.

### Affinity and Anti-affinity

Rules that influence where pods should or should not run.

### Eviction

The removal of pods from a node, often due to pressure or administrative action.

## Operational terms

### kube-apiserver

The main API endpoint for the cluster.

### etcd

The key-value store that holds cluster state.

### kube-scheduler

Chooses nodes for unscheduled pods.

### kube-controller-manager

Runs controllers that reconcile actual state toward desired state.

### kubelet

Runs on each node and ensures pods are started and healthy from the node perspective.

### kube-proxy

Handles service networking rules on nodes.
