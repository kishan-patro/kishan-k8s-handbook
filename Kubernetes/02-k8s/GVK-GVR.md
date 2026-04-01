# GVK and GVR in Kubernetes

GVK and GVR are foundational API concepts in Kubernetes. They describe what an object is and how it is addressed through the API.

## GVK: Group, Version, Kind

GVK identifies the type of object.

- Group: the API group, such as `apps`
- Version: the API version, such as `v1`
- Kind: the object type, such as `Deployment`

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
```

In this example:

- Group is `apps`
- Version is `v1`
- Kind is `Deployment`

## GVR: Group, Version, Resource

GVR identifies the REST resource path used by the Kubernetes API.

- Group: `apps`
- Version: `v1`
- Resource: `deployments`

Example API path:

```text
/apis/apps/v1/namespaces/default/deployments
```

## Kind vs Resource

`Kind` is the logical object type. `Resource` is the plural name used in API endpoints.

Examples:

- Kind: `Pod`, Resource: `pods`
- Kind: `Service`, Resource: `services`
- Kind: `Ingress`, Resource: `ingresses`

## Why this distinction matters

GVK is used when defining objects in YAML and when the API server deserializes them.

GVR is used when clients and controllers talk to the API dynamically through REST endpoints.

This becomes especially important for:

- Custom Resource Definitions
- Dynamic clients
- Controllers and operators
- API discovery logic

## Example with a custom resource

Suppose you define a CRD for `Widget` objects:

- Group: `example.com`
- Version: `v1`
- Kind: `Widget`
- Resource: `widgets`

The YAML uses GVK:

```yaml
apiVersion: example.com/v1
kind: Widget
```

The API path uses GVR:

```text
/apis/example.com/v1/namespaces/default/widgets
```

## Interview explanation

GVK tells Kubernetes what type of object it is dealing with. GVR tells clients which API resource path to use when interacting with that object.
