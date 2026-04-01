# Kubernetes Architecture

Kubernetes follows a control-plane and worker-node architecture. The control plane stores and manages desired state, while worker nodes run application workloads.

## High-level view

The cluster has two major parts:

- Control plane: makes decisions and stores cluster state
- Worker nodes: run pods and provide runtime resources

## Control plane components

### kube-apiserver

The API server is the front door of Kubernetes. All cluster changes go through it, whether they come from kubectl, controllers, operators, or the UI.

Responsibilities:

- Validates requests
- Enforces authentication and authorization
- Applies admission control
- Persists state to etcd

### etcd

A distributed key-value store that holds the cluster's source of truth.

Stores things like:

- Pods, deployments, services, and config
- Leases and leader-election data
- Resource state and metadata

If etcd is unhealthy, the control plane becomes unstable.

### kube-scheduler

Chooses which node should run a pending pod.

It considers:

- Resource requests
- Taints and tolerations
- Affinity rules
- Topology spread constraints
- Node readiness

### kube-controller-manager

Runs control loops that watch cluster state and reconcile it toward the desired state.

Examples include:

- Deployment controller
- ReplicaSet controller
- Node controller
- Endpoint controller

### cloud-controller-manager

Integrates Kubernetes with cloud provider APIs. It handles things like load balancers, routes, and node metadata in cloud-managed environments.

## Worker node components

### kubelet

The node agent. It watches pod specs assigned to the node and ensures the required containers are running.

### container runtime

Runs containers on the node. The common runtime today is containerd.

### kube-proxy

Implements service networking rules so traffic can reach pods behind Kubernetes services.

## How a workload gets created

1. You submit a manifest with kubectl or another client.
2. The API server validates and stores it.
3. Controllers create or update dependent resources.
4. The scheduler assigns pods to nodes.
5. The kubelet starts containers on those nodes.
6. Services and networking expose the workload.

## Cluster networking basics

Kubernetes networking is built around a few assumptions:

- Every pod gets an IP address.
- Pods can usually talk to each other directly across nodes.
- Services provide stable access to changing pod backends.

The exact implementation depends on the CNI plugin, such as Calico, Cilium, or the AWS VPC CNI.

## Why the architecture matters

Understanding the architecture helps you debug issues correctly. For example:

- Scheduling issues point to scheduler, resources, or policies
- Service routing issues point to kube-proxy, endpoints, or ingress
- State persistence issues point to etcd or the API server path

## Interview explanation

Kubernetes is a declarative control system. The API server accepts desired state, etcd stores it, controllers reconcile it, the scheduler places work, and nodes run the workloads.
