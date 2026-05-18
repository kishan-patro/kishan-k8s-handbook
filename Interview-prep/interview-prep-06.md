# Kubernetes Interview Prep 06

Focused notes for interviews covering EKS, Argo CD, deployment strategies, AWS load balancers, and Linux basics.

---

## 1. Describe Your Day-to-Day Work

Interviewers usually start with your real operating model, not tool trivia.

**How to answer:**

Describe the actual sequence of work you do during a normal day. Keep it concrete.

Example:

> I start by checking alerts and dashboards, review open PRs, work on Terraform or Kubernetes changes, validate deployments through CI/CD, and troubleshoot production issues when something regresses. I regularly work with tools like Terraform, Jenkins, Argo CD, Prometheus, and kubectl.

**What they are testing:**

- Whether you have hands-on ownership
- Whether you understand operations, not just definitions
- Whether you can explain tools in terms of outcomes

---

## 2. Kubernetes and EKS

### Common question

What is your experience with EKS, and how is it different from Kubernetes in general?

### Strong answer

EKS is AWS-managed Kubernetes. AWS operates the control plane, while you manage the worker compute layer, workload configuration, IAM integration, networking decisions, and day-2 operations.

### Key points to cover

- **Control plane:** AWS manages the API server and etcd for you.
- **Node groups:** You can use managed node groups, self-managed nodes, or Fargate depending on the workload.
- **IRSA:** IAM Roles for Service Accounts lets a pod assume a scoped AWS role without storing static credentials.
- **Networking:** EKS commonly uses the AWS VPC CNI, where pods receive IPs from the VPC address space.

### IRSA in simple terms

IRSA lets you grant AWS permissions to one Kubernetes service account instead of to an entire node. That means a specific application can access only the AWS resources it needs, such as one S3 bucket or one DynamoDB table.

### Node groups in simple terms

Node groups are pools of worker machines. You typically separate them by workload type, instance family, scaling policy, or cost profile.

### CNI in simple terms

The Container Network Interface decides how pods get network connectivity. In EKS, the AWS VPC CNI attaches pod networking directly to your VPC, which simplifies access to AWS resources but requires careful IP planning.

---

## 3. Argo CD

### Common question

How does Argo CD work, and how would you migrate it from one cluster to another?

### Strong answer

Argo CD is a GitOps controller. It continuously compares Kubernetes manifests in Git with the live cluster state and reconciles drift.

### Migration approach

1. Back up Argo CD applications, projects, repository credentials, RBAC config, and secrets from the source cluster.
2. Install Argo CD on the target cluster using the same Helm chart and values.
3. Restore the application definitions and required secrets.
4. Reconnect repositories and validate sync and health status.
5. Confirm that all applications become `Synced` and `Healthy` before decommissioning the old setup.

### What to mention if they probe deeper

- Store secrets carefully during migration
- Re-create any external DNS, ingress, or load balancer dependencies
- Validate SSO, RBAC, and repository access after restore

---

## 4. Build EKS from Scratch

### Common question

You are given a blank AWS account. How do you get to a working EKS cluster with Argo CD?

### Strong sequence

1. **Baseline the account**
   Enable audit logging, define administrative access, and prepare a remote state location if you are using Terraform.

2. **Create IAM roles and policies**
   Prepare roles for cluster management, worker nodes, and workload-level access. Enable the cluster OIDC provider for IRSA.

3. **Design the network**
   Create a VPC with public and private subnets across multiple availability zones. Public subnets typically support load balancers, while private subnets host worker nodes.

4. **Provision the cluster**
   Use Terraform or `eksctl` to create the EKS control plane.

5. **Create node groups**
   Add managed node groups with instance types sized for expected workloads.

6. **Configure cluster access**
   Update `kubeconfig` with `aws eks update-kubeconfig` and validate connectivity with `kubectl`.

7. **Install Argo CD**
   Deploy Argo CD with Helm, expose it securely, connect it to Git, and bootstrap the first application.

### Areas interviewers care about most

- Subnet design and routing
- NAT and egress planning
- IAM boundaries
- Secure cluster access
- How GitOps becomes the deployment path after cluster creation

---

## 5. Rolling Update vs Canary

### Common question

What is the difference between a rolling update and a canary deployment?

### Strong answer

- **Rolling update:** Gradually replaces old pods with new ones until the deployment is fully updated.
- **Canary:** Sends a small portion of traffic to the new version first, validates behavior, and then increases traffic gradually.

### Interview version in plain terms

Rolling updates are the Kubernetes default and work well for straightforward application upgrades. Canary deployments are safer for high-risk changes because they limit blast radius, but they usually require extra tooling such as Argo Rollouts, a service mesh, or ingress-based traffic shaping.

---

## 6. ALB vs NLB

### Common question

When should you use an Application Load Balancer versus a Network Load Balancer?

### Strong answer

| Feature | ALB | NLB |
|---|---|---|
| OSI layer | Layer 7 | Layer 4 |
| Protocol awareness | HTTP/HTTPS | TCP/UDP/TLS |
| Routing | Path, host, header-based | Connection-level forwarding |
| Static IP | No | Yes |
| Best fit | Web apps, REST APIs, ingress-style routing | High-throughput TCP services, fixed-IP needs, low-latency transport |

### Plain-English explanation

- **ALB:** Understands HTTP and can route based on hostnames and URL paths.
- **NLB:** Forwards traffic very quickly at the transport layer without interpreting application paths.

Use ALB for web applications and microservices front doors. Use NLB when you need static IPs, TLS pass-through patterns, or raw TCP/UDP behavior.

---

## 7. Linux Basics

### Common question

How do you create a user and set up SSH access correctly?

### Quick reference

```bash
sudo useradd -m -s /bin/bash devuser
sudo passwd devuser

ssh-keygen -t ed25519 -C "devuser@company.com"

sudo mkdir -p /home/devuser/.ssh
sudo cp ~/.ssh/id_ed25519.pub /home/devuser/.ssh/authorized_keys
sudo chown -R devuser:devuser /home/devuser/.ssh
sudo chmod 700 /home/devuser/.ssh
sudo chmod 600 /home/devuser/.ssh/authorized_keys
```

### Why permissions matter

SSH rejects keys when `.ssh` or `authorized_keys` is too open. The expected secure defaults are:

- `700` for the `.ssh` directory
- `600` for the `authorized_keys` file

---

## 8. Interview Tips

- Answer from real experience first, then give the definition.
- Keep architecture explanations simple before going deep.
- Name the tradeoff, not just the feature.
- If you have done something in production, say exactly what you owned.

---

## 9. One-Line Summary Answers

- **EKS:** AWS-managed Kubernetes control plane with AWS-native IAM and networking integration.
- **IRSA:** Pod-level AWS permissions through Kubernetes service accounts.
- **Argo CD:** GitOps controller that keeps cluster state aligned with Git.
- **Rolling update:** Replaces old pods gradually.
- **Canary:** Tests a new version on a small percentage of traffic first.
- **ALB:** Layer 7 load balancer for HTTP routing.
- **NLB:** Layer 4 load balancer for fast TCP/UDP forwarding.

---

Use this file as a speaking guide, not a script. In interviews, clarity beats volume.