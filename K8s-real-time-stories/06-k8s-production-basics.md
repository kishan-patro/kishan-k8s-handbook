# Kubernetes Production Basics Most Teams Miss

This is a practical checklist for avoiding common production outages.

## 1. Prefer Secret Volumes Over envFrom for Rotating Credentials
If secrets are injected as environment variables (`envFrom`/`valueFrom`), running containers do not see updates until restart.
If secrets are mounted as files, kubelet refreshes mounted content (eventually), so apps that reload files can pick up changes without a full rollout.

```yaml
volumes:
	- name: app-secret
		secret:
			secretName: app-secret
containers:
	- name: api
		volumeMounts:
			- name: app-secret
				mountPath: /etc/secrets
				readOnly: true
```

## 2. Understand Secret Volume Permissions (Do Not Assume Root Requirement)
Secret volumes are owned by root, but readability depends on mode bits and group settings.
`defaultMode: 0440` is useful only when your container process is in a group that can read those files (commonly via `fsGroup`).
If `fsGroup` is not set appropriately, non-root apps can lose read access.

```yaml
spec:
	securityContext:
		runAsUser: 1000
		fsGroup: 2000
	volumes:
		- name: app-secret
			secret:
				secretName: app-secret
				defaultMode: 0440
```

## 3. Create PodDisruptionBudgets Before Maintenance Windows
Without a PDB, voluntary disruptions (drain/upgrade/autoscaler evictions) can remove too many replicas at once.
With `minAvailable` or `maxUnavailable`, Kubernetes limits voluntary evictions.
PDBs do not protect against involuntary failures (for example, node crashes).

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
	name: api-pdb
spec:
	maxUnavailable: 1
	selector:
		matchLabels:
			app: api
```

## 4. Set Resource Requests and Limits Intentionally
No requests means poor scheduling signals; no limits can let noisy neighbors consume too much.
Use measured requests, then apply limits based on workload behavior.
For CPU-heavy services, overly tight CPU limits can cause throttling and latency spikes.

```yaml
resources:
	requests:
		cpu: 250m
		memory: 256Mi
	limits:
		cpu: "1"
		memory: 512Mi
```

## 5. Tune terminationGracePeriodSeconds to Real Shutdown Time
If your app needs 60s to drain but grace period is 30s, kubelet will SIGKILL before shutdown completes.
That can drop in-flight requests and corrupt stateful operations.
Combine proper grace period with `preStop` logic and readiness transitions.

```yaml
spec:
	terminationGracePeriodSeconds: 75
	containers:
		- name: api
			lifecycle:
				preStop:
					exec:
						command: ["/bin/sh", "-c", "sleep 15"]
```

## 6. Configure Readiness Probes (and Keep Liveness Conservative)
Without readiness probes, traffic can hit pods that are not ready for real requests.
Readiness gates traffic. Liveness should be slower and more tolerant to avoid restart storms under load.

```yaml
readinessProbe:
	httpGet:
		path: /ready
		port: 8080
	periodSeconds: 5
	failureThreshold: 3
livenessProbe:
	httpGet:
		path: /healthz
		port: 8080
	periodSeconds: 30
	failureThreshold: 5
```

## 7. Never Use latest for Production Image Tags
`:latest` is mutable and undermines reproducibility and rollback confidence.
Pin immutable tags (version or digest) so every replica runs the exact same build.

```yaml
containers:
	- name: api
		image: ghcr.io/acme/api:v1.9.3
		# or digest pinning:
		# image: ghcr.io/acme/api@sha256:9d7a...
```

## 8. Keep Pods Disposable; Store State Outside the Pod Filesystem
Data written to container writable layers is ephemeral across rescheduling/recreation.
For durable state, use managed stores (for example RDS/S3/Redis) or Kubernetes persistent volumes.

```yaml
volumes:
	- name: app-data
		persistentVolumeClaim:
			claimName: app-data-pvc
containers:
	- name: api
		volumeMounts:
			- name: app-data
				mountPath: /var/lib/app
```

## Quick Rule
If a pod dies and recovery is painful, your workload is not yet production-hardened.
