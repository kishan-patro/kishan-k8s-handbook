# Container Isolation

Containers look lightweight compared to virtual machines, but they still need strong isolation boundaries so one application does not interfere with another. Container isolation is mainly built on Linux kernel features.

## What isolation means

Isolation means a process inside one container should not freely see, control, or consume resources that belong to other workloads on the same host.

This includes:

- Process isolation
- Filesystem isolation
- Network isolation
- User and permission isolation
- CPU and memory control

## Linux namespaces

Namespaces isolate what a process can see.

### PID namespace

Gives a container its own process tree, so processes inside the container do not see the host's full process list.

### Network namespace

Provides separate network interfaces, routing tables, and ports. This is why containers can have isolated network stacks.

### Mount namespace

Separates filesystem mount points so containers can have their own root filesystem view.

### UTS namespace

Isolates hostname and domain name settings.

### IPC namespace

Separates shared memory and inter-process communication resources.

### User namespace

Maps container users to different host users. This can reduce privilege risk when configured correctly.

## Cgroups

Control groups manage how much CPU, memory, disk I/O, and other resources a container can use.

Without cgroups, one container could consume most of the host's resources and starve other workloads.

Examples:

- CPU shares and quotas
- Memory limits
- I/O throttling
- Process count limits

## Filesystem isolation

Containers usually use layered filesystems. Each container gets its own writable layer on top of shared image layers. This helps make containers reproducible and lightweight.

## Security boundaries

Containers are isolated, but they share the host kernel. That means they are not equivalent to virtual machines from a security perspective.

Important security controls include:

- Dropping Linux capabilities
- Running as non-root
- Seccomp profiles
- AppArmor or SELinux
- Read-only root filesystems where possible
- Avoiding privileged containers

## Why this matters in Kubernetes

Kubernetes schedules many pods onto the same node. Isolation features reduce blast radius if one container is noisy, misconfigured, or compromised.

## Interview explanation

Containers rely on namespaces for visibility isolation and cgroups for resource isolation. Security hardening layers make that isolation safer in real production systems.
