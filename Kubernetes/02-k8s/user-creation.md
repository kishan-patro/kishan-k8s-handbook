# User Creation in Kubernetes

Kubernetes does not have a built-in user database for normal end users. User creation is usually handled outside the cluster through certificates, identity providers, or cloud IAM integration. Kubernetes then authorizes those identities through RBAC.

## Authentication vs authorization

These are separate concerns:

- Authentication answers who the user is
- Authorization answers what the user is allowed to do

Kubernetes commonly authenticates users through:

- Client certificates
- OIDC providers
- Cloud IAM integration, such as EKS with AWS IAM
- External authenticators

## RBAC basics

RBAC uses four main objects:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

Roles define permissions. Bindings attach those permissions to users, groups, or service accounts.

## Example: certificate-based user access

One traditional way to create a Kubernetes user is to issue a client certificate signed by the cluster CA.

High-level flow:

1. Generate a private key
2. Create a certificate signing request
3. Sign it with the cluster CA
4. Add the credentials to kubeconfig
5. Bind RBAC permissions to that identity

## Example Role and RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
	namespace: dev
	name: pod-reader
rules:
	- apiGroups: [""]
		resources: ["pods"]
		verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
	name: pod-reader-binding
	namespace: dev
subjects:
	- kind: User
		name: alice
		apiGroup: rbac.authorization.k8s.io
roleRef:
	kind: Role
	name: pod-reader
	apiGroup: rbac.authorization.k8s.io
```

## Example: checking permissions

```bash
kubectl auth can-i get pods -n dev --as=alice
```

## Best practices

- Prefer group-based access over per-user bindings where possible
- Follow least privilege
- Separate read-only and admin access clearly
- Use an external identity provider in production
- Audit who has cluster-admin and why

## Common mistake

Creating users is often confused with creating service accounts. Service accounts are identities for workloads inside the cluster, not for human users.

## Interview explanation

Kubernetes usually authenticates users through external identity systems or client certificates, then uses RBAC to decide what those users can do in the cluster.
