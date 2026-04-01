# Pod

A pod is the smallest deployable unit in Kubernetes. It wraps one or more containers that are scheduled together and share certain runtime resources.

## What a pod provides

Containers inside a pod share:

- Network namespace
- IP address
- Port space
- Mounted volumes

This is why a pod is usually used for one application component and any tightly coupled helper containers.

## Single-container pod

The most common pattern is one main container per pod.

This keeps scaling, logging, health checks, and lifecycle management simpler.

## Multi-container pod

Multiple containers can run in the same pod when they must work closely together.

Common examples:

- App container plus log shipping sidecar
- App container plus proxy sidecar
- App container plus metrics exporter

## Basic pod example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
    - name: web
      image: nginx:1.25
      ports:
        - containerPort: 80
```

## Important pod fields

- `metadata`: labels, annotations, name
- `spec.containers`: application containers
- `volumes`: shared storage definitions
- `initContainers`: pre-start tasks
- `restartPolicy`: restart behavior
- `nodeSelector`, affinity, tolerations: scheduling controls

## Why pods are ephemeral

Pods are not meant to be permanent pets. They are replaceable units managed by controllers such as Deployments and StatefulSets.

## Interview explanation

A pod is a shared execution boundary for one or more containers. It gives them shared networking and storage context and is the base unit Kubernetes schedules onto nodes.
