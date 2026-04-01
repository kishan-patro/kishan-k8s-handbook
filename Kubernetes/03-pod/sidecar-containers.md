# Sidecar Containers

A sidecar container is a helper container that runs in the same pod as the main application container and extends its behavior without changing the main application image.

## Why sidecars are used

Sidecars are useful when the helper logic should stay close to the app and share the same lifecycle context.

Common examples:

- Log forwarding
- TLS proxying
- Metrics exporting
- Configuration reloading
- Service mesh data plane proxies

## Benefits

- Keeps the main app image simpler
- Encapsulates support functions cleanly
- Allows shared volumes and local communication over localhost

## Tradeoffs

- More resource usage per pod
- More complex startup and shutdown behavior
- Harder troubleshooting if containers depend on each other badly

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: sidecar-example
spec:
	containers:
		- name: app
			image: nginx
			volumeMounts:
				- name: shared-logs
					mountPath: /var/log/app
		- name: log-shipper
			image: busybox
			command: ["sh", "-c", "tail -F /var/log/app/app.log"]
			volumeMounts:
				- name: shared-logs
					mountPath: /var/log/app
	volumes:
		- name: shared-logs
			emptyDir: {}
```

## Sidecar vs init container

- Sidecar runs alongside the app during normal operation
- Init container finishes before the app starts

## Interview explanation

A sidecar is a companion container in the same pod that adds operational capability, such as proxying, log shipping, or config watching, without changing the main application container.
