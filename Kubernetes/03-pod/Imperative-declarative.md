# Imperative vs Declarative

Kubernetes can be managed in two main ways: imperative commands and declarative manifests. Understanding the difference is important because production operations usually favor the declarative model.

## Imperative approach

Imperative management means telling Kubernetes exactly what action to perform right now.

Examples:

```bash
kubectl run nginx --image=nginx
kubectl scale deployment web --replicas=5
kubectl delete pod test-pod
```

### Characteristics

- Fast for ad hoc tasks
- Useful for debugging and quick experiments
- Often harder to audit and reproduce later

## Declarative approach

Declarative management means defining the desired state in YAML and applying it.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: web
spec:
	replicas: 3
	selector:
		matchLabels:
			app: web
	template:
		metadata:
			labels:
				app: web
		spec:
			containers:
				- name: web
					image: nginx:1.25
```

Then apply it:

```bash
kubectl apply -f deployment.yaml
```

### Characteristics

- Easier to version-control
- Easier to review and audit
- Better for repeatable environments
- Fits GitOps and automation workflows

## Which one is better?

Both are useful, but for different reasons.

- Use imperative commands for quick inspection, testing, and emergency actions
- Use declarative configs for long-lived workloads and team-managed infrastructure

## Interview explanation

Imperative means action-driven management. Declarative means state-driven management. Kubernetes is designed around the declarative model because controllers reconcile actual state toward the desired state.
