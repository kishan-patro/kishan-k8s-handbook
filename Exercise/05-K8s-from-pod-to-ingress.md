# From Pod to Ingress: Why Each K8s Concept Exists

Most people start learning Kubernetes the wrong way.

They see Kubernetes as a list of concepts. Pods. Deployments. Services. Ingress. They memorize them without understanding why they exist.

>> You start with a Pod.

- A pod runs your container. Simple. Clean. Done.
- Until it crashes.
- Nobody restarts it. It is just gone.

In production, that is not acceptable.

>> So you use a Deployment.

- A Deployment watches your pods.
- One dies, and it creates another.
- You want 3 running, it keeps 3 running.
- You want to scale to 10; one command does it.

Pods were too fragile for production.

Deployments fixed that.

>> But now you have a new problem.

- Every pod gets a new IP when it restarts.
- You have 3 pods running your app.
- Another service needs to talk to them.
- Which IP do you use? They keep changing.
- You cannot hardcode them.
- You cannot track them at scale.

>> So you use a Service.

- A Service gives your app one stable IP address.
- It finds your pods using labels, not IPs.
- Pods die and come back with new IPs.
- The Service does not care.
- It always finds them.
- It also load balances.
- Traffic coming in gets distributed across all healthy pods automatically.

Pods had unstable IPs. Services fixed that.

>> But your app still needs to be accessible from the internet.

- So you use a LoadBalancer Service.
- This creates a real cloud load balancer.
- AWS ALB. Azure LB. GCP LB.
- Your app gets a public endpoint.
- Works perfectly. Until you have 10 services.
- Now you have 10 load balancers.
- Each one costs money every single month.
- Your cloud bill does not care that 6 of them handle almost no traffic.

LoadBalancer Services solved external access. But one per service does not scale.

>> So you use Ingress.

- One load balancer. All your services behind it.
- Ingress routes traffic based on rules.
- Request comes in for /api, goes to the API service.
- Request comes in for /dashboard, goes to the frontend service.
- One entry point. Smart routing. One cloud load balancer on your bill.

But Ingress is just a set of rules. Something has to execute those rules.

>> So you use an Ingress Controller.

- Nginx. Traefik. AWS Load Balancer Controller.
- These are the actual engines that read your Ingress rules and make the routing happen.
- Ingress without a controller is just a config file nobody reads.

To summarize it:

> Pod ran your app but had no resilience.
> Deployment gave it resilience.
> Service gave it a stable address and load balancing.
> LoadBalancer Service gave it external access.
> Ingress replaced 10 load balancers with one.
> Ingress Controller made the rules actually work.

Each concept exists because the previous one was not enough.
