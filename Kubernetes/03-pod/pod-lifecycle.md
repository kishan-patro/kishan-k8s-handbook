# Pod Lifecycle

Pods are ephemeral objects. They are created, scheduled, started, monitored, and eventually terminated or replaced by controllers. Understanding the lifecycle helps with debugging and rollout design.

## Main pod phases

### Pending

The pod has been accepted by Kubernetes but is not fully running yet.

Common reasons:

- Waiting for scheduling
- Pulling images
- Waiting for volumes
- Running init containers

### Running

The pod has been scheduled to a node and at least one container is running or starting.

### Succeeded

All containers terminated successfully and will not be restarted. Common with jobs.

### Failed

All containers have terminated and at least one failed.

### Unknown

The cluster cannot determine the pod state, often because the node is unreachable.

## Pod conditions

Pod phases are high level, but conditions give more detail.

Common conditions include:

- `PodScheduled`
- `Initialized`
- `ContainersReady`
- `Ready`

## Lifecycle flow

1. Pod object is created
2. Scheduler assigns it to a node
3. Kubelet pulls images and starts init containers
4. Main containers start
5. Readiness determines if traffic is allowed
6. Pod may be restarted, evicted, completed, or terminated

## Termination behavior

When a pod is deleted:

1. Kubernetes sends `SIGTERM`
2. Grace period begins
3. PreStop hooks, if defined, run
4. Containers are force-killed if they do not exit in time

## Why pods get replaced

Pods are not repaired in place very often. Controllers usually replace them when:

- A node fails
- A deployment is updated
- A pod becomes unhealthy
- Scaling changes occur

## Useful commands

```bash
kubectl get pod <pod> -o wide
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

## Interview explanation

The pod lifecycle is not just about status labels. It is the full path from scheduling through startup, readiness, execution, and termination.
