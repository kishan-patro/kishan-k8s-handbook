# Kubeconfig

A kubeconfig file stores the information kubectl and other Kubernetes clients use to connect to clusters.

It usually contains:

- Cluster details
- User or credential details
- Context definitions that pair a user with a cluster and optional namespace

## Default location

By default, kubectl reads:

```text
~/.kube/config
```

You can also point to another file with the `KUBECONFIG` environment variable.

## Main sections of kubeconfig

### clusters

Defines the API server endpoint and certificate authority information.

### users

Defines authentication material such as client certificates, tokens, or exec-based authentication.

### contexts

Combines a cluster, a user, and optionally a namespace into a named profile.

### current-context

Defines which context kubectl uses by default.

## Basic commands

### View current config

```bash
kubectl config view
```

### List contexts

```bash
kubectl config get-contexts
```

### Switch context

```bash
kubectl config use-context <context-name>
```

### Set default namespace for a context

```bash
kubectl config set-context --current --namespace=<namespace>
```

## Example structure

```yaml
apiVersion: v1
kind: Config
clusters:
	- name: dev-cluster
		cluster:
			server: https://api.dev.example.com
contexts:
	- name: dev-admin
		context:
			cluster: dev-cluster
			user: dev-user
			namespace: default
current-context: dev-admin
users:
	- name: dev-user
		user:
			token: REDACTED
```

## Security considerations

- Treat kubeconfig like a credential file
- Avoid sharing configs with embedded admin tokens
- Prefer short-lived authentication methods where possible
- Keep production contexts clearly named to avoid mistakes

## Multi-cluster workflow

Teams often keep several contexts in one kubeconfig or merge multiple files through `KUBECONFIG`.

This helps when working across:

- Dev, stage, and prod clusters
- Multiple cloud providers
- Different customer or team environments

## Interview explanation

Kubeconfig is the client-side connection profile for Kubernetes. It tells kubectl which cluster to talk to, which identity to use, and which context is currently active.
