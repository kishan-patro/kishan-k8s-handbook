# Init Containers

Init containers are special containers that run before the main application containers in a pod start. They are useful for setup, validation, and dependency checks.

## Key behavior

- They run in order
- Each one must complete successfully before the next starts
- The main containers do not start until all init containers finish

## Common use cases

- Waiting for a database or external service to become reachable
- Downloading configuration or bootstrap files
- Performing schema checks or environment validation
- Preparing shared volumes before the main app starts

## Why init containers are useful

They keep startup logic separate from the main application image. This makes application containers simpler and avoids mixing runtime behavior with one-time setup logic.

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: init-example
spec:
	initContainers:
		- name: wait-for-service
			image: busybox
			command: ["sh", "-c", "until nslookup myservice; do sleep 2; done"]
	containers:
		- name: app
			image: nginx
```

## Troubleshooting tips

If a pod is stuck in `Init`, check:

```bash
kubectl describe pod <pod>
kubectl logs <pod> -c <init-container-name>
```

## Interview explanation

Init containers are for one-time startup tasks that must finish before the application begins serving traffic.
