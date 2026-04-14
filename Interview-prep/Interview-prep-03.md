# Kubernetes Interview Prep 03

DevOps Architect interview prep focused on system design, platform ownership, multi-team Kubernetes operations, and production decision-making.

---

## 1. How do you design a multi-tenant Kubernetes platform?

### Strong answer

I start by separating concerns across identity, network, compute, and deployment boundaries. Namespaces provide logical separation, but real multi-tenancy requires RBAC, network policies, resource quotas, admission controls, and workload guardrails.

### Design pillars

- Namespace-per-team or namespace-per-environment model
- Least-privilege RBAC with centralized identity integration
- Network policies to limit east-west traffic
- Resource quotas and limit ranges to control noisy-neighbor effects
- Standardized deployment patterns through Helm, Kustomize, or GitOps

### Production nuance

Soft multi-tenancy is enough for many internal platforms. Hard isolation may require separate clusters for strong security or compliance boundaries.

---

## 2. How would you structure environments across dev, stage, and prod?

### Strong answer

I separate environments by risk, not just convenience. For critical systems, production should have its own cluster or at least dedicated node groups and access boundaries.

### Recommended pattern

- Dev and test can share lower-trust infrastructure
- Stage should mirror prod as closely as practical
- Prod should have stricter change control, observability, and rollback requirements

### Interview point

Environment design is really about blast radius, access control, and confidence in release promotion.

---

## 3. How do you manage configuration across environments?

### Strong answer

I keep base manifests reusable and layer environment-specific configuration on top with overlays or values files. The goal is to avoid copy-paste YAML drift.

### Common tools

- Helm values files
- Kustomize overlays
- GitOps repositories with clear promotion paths

### Key principle

One source of truth, environment-specific overrides, and auditable change history.

---

## 4. How do you secure secrets in Kubernetes?

### Strong answer

I avoid storing long-lived secrets directly in manifests. I prefer external secret managers and short-lived credentials where possible.

### Practical controls

- External Secrets Operator or CSI driver integration
- Envelope encryption for Kubernetes secrets at rest
- IRSA or workload identity for cloud access
- Restricted RBAC and audit logging on secret access

### Strong interview point

Base64 is not security. Secret handling must include storage, access path, rotation, and auditability.

---

## 5. How do you approach cluster upgrades?

### Strong answer

I treat upgrades as a compatibility exercise, not just a version bump. I validate API deprecations, addon compatibility, node image readiness, and rollback options before touching production.

### Upgrade flow

1. Read release notes and deprecation notices.
2. Upgrade non-prod first.
3. Validate controllers, CNI, ingress, CSI, and monitoring stack.
4. Upgrade control plane where applicable.
5. Roll worker nodes gradually.
6. Watch service health and error budgets.

---

## 6. What would you standardize on a platform team?

### Strong answer

I standardize the parts that create operational consistency: base deployment templates, observability, security policies, ingress patterns, workload identity, autoscaling defaults, and incident response conventions.

### Good examples

- Mandatory probes and resource requests
- Standard labels and annotations
- Shared dashboards and alerts
- Approved base images and scanning pipeline
- GitOps-based deployment flow

---

## 7. How do you reduce YAML sprawl?

### Strong answer

I reduce sprawl by creating opinionated abstractions, not by asking every team to handcraft manifests. Teams should provide app-specific values, not rebuild platform logic.

### Practical approaches

- Shared Helm charts
- Kustomize component model
- Policy-as-code to enforce safe defaults
- Git repository conventions

---

## 8. How would you handle Terraform state failure or corruption?

### Strong answer

I first protect the current infrastructure from accidental drift. Then I verify the last good state, lock behavior, backend health, and whether the issue is state corruption, concurrent modification, or missing resources.

### Recovery approach

1. Stop concurrent applies.
2. Back up the current state file.
3. Inspect remote state backend and locks.
4. Use `terraform state pull` and targeted reconciliation.
5. Import resources if the infrastructure exists but state is incomplete.

### Interview point

The goal is controlled reconciliation, not panic-driven reapply.

---

## 9. How do you monitor platform health?

### Strong answer

I monitor both infrastructure and user impact. Cluster health alone is not enough.

### Core areas

- API server latency and error rate
- Node pressure and unschedulable pods
- Pod restart rates and failed rollouts
- Ingress and service latency
- Application SLIs such as request success rate and tail latency

### Strong interview point

Platform observability should let you answer two things quickly: what is broken, and who is affected.

---

## 10. How do you design incident response for Kubernetes?

### Strong answer

I define clear severity levels, ownership boundaries, escalation rules, and evidence collection procedures. During incidents, speed matters more than elegance.

### Operational model

- Alert routing by service ownership
- Runbooks for common failure modes
- Rollback or traffic-shift as first mitigation
- Timeline capture during the incident
- Post-incident review with action tracking

---

## 11. How do you make stateful workloads safer in Kubernetes?

### Strong answer

I only run stateful workloads in Kubernetes when the operational model is clear. Stable identity, storage class behavior, backup strategy, and failure testing are mandatory.

### Key requirements

- StatefulSet instead of Deployment
- Durable storage with tested snapshots
- Pod disruption controls
- Backup and restore validation
- Awareness of quorum and replication behavior

---

## 12. How do you choose between self-managed Kubernetes and managed services like EKS?

### Strong answer

Managed Kubernetes removes control plane burden and usually accelerates team productivity. Self-managed clusters give more control but increase operational cost and failure surface.

### Decision factors

- Team size and expertise
- Compliance and customization needs
- Upgrade burden
- Networking and IAM integration needs
- Total cost of ownership

---

## 13. How do you build a secure software supply chain for containers?

### Strong answer

I control the image lifecycle end to end: trusted base images, vulnerability scanning, signature verification, minimal runtime footprint, and admission policies.

### Practical controls

- Build from minimal bases
- Scan in CI and registry
- Sign images
- Enforce provenance or signature checks at admission
- Run as non-root where possible

---

## 14. How do you avoid platform team bottlenecks?

### Strong answer

The platform team should provide paved roads, not manual gatekeeping. Self-service templates, policy automation, and clear ownership reduce ticket-driven operations.

### Good patterns

- Golden paths for common services
- Automated policy checks in CI
- Standard bootstrap repos
- Documentation backed by working examples

---

## 15. How do you answer architect-level questions well?

### Guidance

- Start with principles and constraints.
- Explain tradeoffs, not just tooling choices.
- Show how security, reliability, and operability fit together.
- Connect design choices to team scale and incident history.

### One-line summary

Architect interviews reward clear decision-making under real operational constraints, not the longest list of tools.
