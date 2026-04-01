# Containers 101

Containers package an application with its runtime dependencies so it can run consistently across environments. They are a standard way to build, ship, and run software.

## What is a container?

A container is an isolated process running on a host OS with its own filesystem, network view, and resource controls. It is created from a container image.

An image is a packaged blueprint. A container is a running instance of that image.

## Why containers became popular

Containers help solve the classic problem of software behaving differently across laptops, test systems, and production servers.

Benefits include:

- Consistent runtime environment
- Fast startup compared to virtual machines
- Better resource efficiency
- Easier packaging and distribution
- Clear dependency management

## Container vs virtual machine

| Topic | Container | Virtual machine |
|---|---|---|
| Isolation model | Shares host kernel | Has its own guest OS |
| Startup time | Usually seconds or less | Usually slower |
| Resource overhead | Lower | Higher |
| Portability | High | Good, but heavier |
| Security boundary | Weaker than a VM | Stronger isolation |

## Container lifecycle

The typical lifecycle is:

1. Build an image from a Dockerfile or similar build definition
2. Push the image to a registry
3. Pull the image onto a runtime host
4. Start the container process
5. Stop, restart, or replace the container as needed

In Kubernetes, containers are usually managed inside pods rather than directly.

## Key container concepts

### Image

An immutable template that contains the application and its dependencies.

### Registry

A location where images are stored and distributed, such as Docker Hub, ECR, or GCR.

### Runtime

The component that actually runs containers, such as containerd.

### Layered filesystem

Images are built from layers, which makes builds and pulls more efficient.

## Common use cases

- Microservices
- CI/CD build agents
- Batch jobs
- Local development environments
- Portable tooling and automation tasks

## What containers do not solve by themselves

Containers do not automatically provide clustering, scaling, networking policy, or high availability. That is where orchestration platforms like Kubernetes come in.

## Interview explanation

Containers package applications into portable, isolated runtime units. They are lighter than virtual machines because they share the host kernel, but that also means they need orchestration and security hardening in production.
