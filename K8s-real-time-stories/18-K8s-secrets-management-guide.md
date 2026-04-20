# Kubernetes Secrets: The Real Story (And What to Use in Production)

Kubernetes `Secret` is often misunderstood. It is not encrypted by default in a way that makes it magically safe.

Base64 is encoding, not encryption. If a user can read the Secret through the Kubernetes API, they can decode it immediately.

## Where the risk starts

1. Anyone with broad `kubectl` read access can fetch and decode sensitive values.
2. If etcd encryption at rest is not configured, Secret data is stored unencrypted in etcd.
3. Manual Secret creation usually leads to stale credentials, unclear ownership, and no rotation discipline.

## Step 1: Minimum hardening inside Kubernetes

Enable etcd encryption at rest using `EncryptionConfiguration` on the API server.

What it solves:
1. Protects Secret values in etcd storage.

What it does not solve:
1. Runtime access controls.
2. Rotation workflow.
3. Auditable lifecycle and ownership.

## Step 2: Move secret lifecycle to a dedicated system

Use Vault (or a cloud secret manager) for dynamic credentials, leases, and automatic rotation.

Why teams adopt it:
1. Secrets can expire automatically (TTL).
2. Rotation is policy-driven, not calendar-driven.
3. Access events are auditable.

## Step 3: Standardize pod-level secret delivery

If teams integrate differently, security posture drifts. Two common Kubernetes-native delivery patterns are:

### Option A: Vault Agent Injector

Best when applications can read secrets from files.

1. Add pod annotations.
2. A webhook injects a sidecar.
3. Sidecar authenticates with pod service account and writes secret material to mounted files.

Strength:
1. Strong Vault-native pattern with minimal app changes for file-based apps.

Tradeoff:
1. Legacy applications expecting environment variables need adaptation.

### Option B: Secrets Store CSI Driver

Best when you want provider-agnostic mounting from Vault, AWS Secrets Manager, or Azure Key Vault.

1. Secrets are mounted into the pod as a volume.
2. Secret updates in the external backend can be reflected without full workflow rewrites.

Strength:
1. Works well for file-based consumption and multi-provider environments.

Tradeoff:
1. Operational overhead grows with many `SecretProviderClass` objects.

## Step 4: Reduce drift with External Secrets Operator (ESO)

When the platform reaches dozens of services, manual sync becomes fragile.

With ESO:
1. Teams define `ExternalSecret` resources.
2. ESO reconciles from external secret backends into native Kubernetes `Secret` objects.
3. `refreshInterval` keeps data synchronized.

Strength:
1. Removes most manual `kubectl create secret` workflows.
2. Reduces path mismatch incidents and stale values.

Tradeoff:
1. You still need strong RBAC and policy boundaries for who can read the generated Kubernetes `Secret`.

## Quick decision guide

1. Need baseline hardening only: enable etcd encryption at rest.
2. Need dynamic credentials and strong audit: use Vault or cloud secret manager.
3. App can read files and you want Vault-native injection: use Vault Agent Injector.
4. Need provider-neutral file mounts in pods: use Secrets Store CSI Driver.
5. Need scalable synchronization to native Kubernetes Secrets: use ESO.

## Production checklist

1. Enable etcd encryption at rest.
2. Enforce least-privilege RBAC on Secret reads.
3. Disable broad list/watch permissions for service accounts unless required.
4. Adopt automated rotation with ownership metadata.
5. Alert on expired/unused credentials and failed reconciliations.
6. Test secret path changes in non-prod before rotation windows.

The right design is not "one perfect tool." It is choosing the smallest set of tools that solves your current failure mode without creating a bigger operational burden.