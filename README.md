# Kubernetes Notes and Exercises

This repository is a personal Kubernetes knowledge base built around hands-on notes, interview preparation, troubleshooting runbooks, and topic-wise explanations.

It is organized as a study and reference project rather than a deployable application. The content focuses on practical Kubernetes concepts, incident response, platform operations, and ecosystem topics such as Argo CD and Helm.

## Repository Structure

### `K8s-real-time-stories/`

Scenario-driven notes and runbooks focused on practical, real-world Kubernetes incidents and operations.

Subfolders:

- `assets/` for diagrams and images used by the stories

- Incident response and troubleshooting guides
- Scenario-based Kubernetes questions
- Architecture and networking notes
- Quick command references

Notable files include:

- `01-K8s-incident-response-runbook.md`
- `02-K8s-troubleshooting-runbook.md`
- `03-K8s-scenario-based-questions.md`
- `04-K8s-why-stateful-is-hard.md`
- `05-K8s-from-pod-to-ingress.md`
- `06-K8s-production-basics.md`
- `07-K8s-how-https-tls-works.md`
- `08-K8s-cluster-architecture.md`
- `09-K8s-ebs-iops-rca.md`
- `10-K8s-commands.md`

### `Interview-prep/`

Dedicated interview revision content in a separate top-level folder for visibility.

- `Interview-prep-01.md` to `Interview-prep-08.md`

### `Kubernetes/`

Topic-based study material grouped by subject.

- `01-containers/` covers container basics, isolation, and orchestration
- `02-k8s/` covers core Kubernetes architecture, GVK/GVR, kubeconfig, and user creation
- `03-pod/` covers pod lifecycle, init containers, sidecars, YAML, and imperative vs declarative workflows

### `Argo/`

Reserved for Argo or Argo CD related notes and examples.

### `Helm/`

Reserved for Helm charts, examples, or packaging notes.

### `⚙️ 🔭 📊/`

Currently empty. This can be used for tooling, observability, or platform operations notes if needed.

## Recommended Reading Path

If you are using this repository to learn Kubernetes in a structured order, this sequence is a practical starting point:

1. Start with `Kubernetes/01-containers/`
2. Move to `Kubernetes/02-k8s/`
3. Continue with `Kubernetes/03-pod/`
4. Read the runbooks in `K8s-real-time-stories/`
5. Use `Interview-prep/` for revision and practice

## Who This Is For

This repository is useful for:

- DevOps engineers
- SREs
- Platform engineers
- Kubernetes beginners building fundamentals
- Candidates preparing for Kubernetes and cloud interviews

## What You Will Find Here

- Practical Kubernetes troubleshooting commands
- Explanations of core concepts in simple language
- Interview-oriented summaries and speaking points
- Architecture and operations notes based on real-world scenarios

## Current Status

The repository is content-focused and actively growing. Some top-level directories are placeholders for future Argo and Helm material.

## License

This project is licensed under the terms of the [LICENSE](LICENSE) file.