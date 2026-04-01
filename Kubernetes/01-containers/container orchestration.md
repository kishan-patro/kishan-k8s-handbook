# Container Orchestration

Container orchestration is the automation layer that manages how containers are deployed, scheduled, connected, scaled, updated, and recovered across multiple machines.

## Why orchestration is needed

Running one container on one server is simple. Running hundreds of containers across many servers is not. Without orchestration, teams end up doing manual work for placement, restarts, networking, scaling, and upgrades.

Typical problems orchestration solves:

- Deciding where workloads should run
- Restarting failed containers automatically
- Replacing unhealthy instances
- Exposing applications over stable network endpoints
- Rolling out new versions safely
- Scaling applications based on demand
- Managing configuration and secrets consistently

## Core capabilities

An orchestration platform usually provides these building blocks:

- Scheduling: places workloads on suitable machines
- Self-healing: restarts failed workloads and reschedules them if a node fails
- Service discovery: gives clients a stable way to reach changing backends
- Load balancing: distributes traffic across healthy instances
- Declarative management: you define desired state and the platform reconciles toward it
- Scaling: increases or decreases workload replicas automatically or manually
- Storage integration: attaches persistent storage when workloads need state

## Common orchestration tools

### Kubernetes

The most widely adopted orchestration platform. It provides a full control plane with API-driven desired state, controllers, scheduling, service discovery, autoscaling, and extensibility.

### Docker Swarm

Simpler than Kubernetes and easier to start with, but less feature-rich and less common in large production environments today.

### Nomad

A lightweight scheduler from HashiCorp that can run containers and non-container workloads. Often chosen when teams want simpler operations or already use the HashiCorp stack heavily.

## Why Kubernetes became dominant

Kubernetes became the standard because it combines strong scheduling, a declarative model, a large ecosystem, and extensibility through custom resources and controllers.

## Real-world example

Suppose an API needs 6 replicas, external access, zero-downtime updates, and automatic recovery if a node fails.

With orchestration:

- the scheduler spreads replicas across nodes,
- a service gives them a stable endpoint,
- health checks decide which replicas receive traffic,
- a deployment rolls out new versions gradually,
- controllers recreate failed pods automatically.

## Interview explanation

Container orchestration is not only about starting containers. It is about operating containerized applications reliably at scale.
