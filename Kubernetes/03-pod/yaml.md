# YAML in Kubernetes

YAML is the most common format used to define Kubernetes resources. A YAML manifest describes the desired state of an object so Kubernetes can create or reconcile it.

## Basic manifest structure

Most Kubernetes manifests include these top-level fields:

- `apiVersion`: which API version to use
- `kind`: the type of resource
- `metadata`: identifying information such as name, labels, and annotations
- `spec`: the desired configuration of the resource

## Example deployment manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: web
	labels:
		app: web
spec:
	replicas: 2
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
					ports:
						- containerPort: 80
```

## Common metadata fields

### name

Human-readable object name.

### labels

Key-value tags used for selection and organization.

### annotations

Non-identifying metadata often used by tools and controllers.

## Common spec patterns

- Container image definitions
- Resource requests and limits
- Environment variables
- Volume mounts
- Probes
- Selectors and labels

## YAML best practices

- Keep indentation consistent
- Use meaningful labels
- Define requests and limits explicitly
- Separate reusable base config from environment-specific overrides
- Validate manifests before applying them

## Useful commands

```bash
kubectl apply --dry-run=client -f file.yaml
kubectl diff -f file.yaml
kubectl explain deployment.spec
```

## Common mistakes

- Wrong indentation
- Using the wrong `apiVersion`
- Label selector mismatch between services and pods
- Changing immutable fields on existing resources

## Interview explanation

YAML in Kubernetes is the declarative representation of desired state. Good manifests are readable, version-controlled, validated, and safe to promote across environments.
