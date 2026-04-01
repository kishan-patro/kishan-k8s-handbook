# Kubernetes Interview Prep 04

Senior DevOps interview prep focused on debugging depth, production reasoning, and incident handling. These are the kinds of questions where the interviewer expects a structured troubleshooting path.

---

## 1. Walk me through the request flow from user to pod.

### Strong answer

A user request typically hits DNS first, then a public load balancer, then ingress or a gateway layer, then the Kubernetes service, and finally one of the matching backend pods.

### Good sequence to mention

1. Client resolves DNS.
2. Traffic reaches ALB, NLB, or ingress endpoint.
3. Ingress controller applies routing rules.
4. Service selects healthy endpoints.
5. Pod receives traffic if readiness is passing.

### Why this matters

When debugging latency or 5xx issues, I isolate which hop is failing rather than treating the system as one black box.

---

## 2. A pod keeps restarting. How do you debug it?

### Strong answer

I confirm whether the restart is due to the application crashing, a failed health probe, or resource exhaustion.

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod <pod>
```

### Common causes

- Crash on startup
- Wrong config or secret
- OOMKilled
- Readiness or liveness probe failures

---

## 3. An API suddenly becomes slow. What do you check first?

### Strong answer

I first confirm whether the slowdown is user-visible and whether it is isolated to one service, one path, or the whole platform. Then I check latency by layer: ingress, application, downstream dependencies, and node saturation.

### Useful dimensions

- Request latency percentiles
- Error rate
- CPU or memory saturation
- Connection pool exhaustion
- Slow database or cache dependency

---

## 4. How do you identify the real bottleneck in a distributed system?

### Strong answer

I avoid guessing based on the busiest component. I correlate traffic, latency, queue depth, retry behavior, and dependency timings. The bottleneck is the resource that limits end-to-end throughput or response time.

### Good examples

- API looks slow but database lock contention is the true bottleneck
- HPA scales pods, but downstream Redis or RDS saturation is the real cap
- CPU is fine, but external API timeout causes thread exhaustion

---

## 5. What monitoring blind spots have you seen in real systems?

### Strong answer

The most common blind spot is measuring infrastructure health without measuring user outcomes. A cluster can look green while customers see errors.

### Typical blind spots

- No request tracing across dependencies
- No alert on failed deployments or readiness loss
- No visibility into queue depth or retry storms
- No SLO-based alerting

---

## 6. Users report `502 Bad Gateway`. How do you triage it?

### Structured path

1. Check ingress or load balancer health.
2. Confirm service endpoints exist.
3. Check pod readiness and app logs.
4. Verify upstream timeouts and backend protocol settings.

```bash
kubectl describe ingress <name>
kubectl get svc,endpoints
kubectl get pods
```

### Strong interview point

`502` often means the proxy cannot get a valid response from the backend, so I validate the whole chain from ingress to pod.

---

## 7. Redis latency spikes during traffic increase. What could be happening?

### Strong answer

I would check whether the problem is CPU saturation, network throughput, hot keys, connection storms, or command-level misuse.

### Areas to inspect

- Cache hit ratio
- Connection count
- Slow commands
- Memory fragmentation or eviction
- Client retry behavior

### Good interview nuance

Sometimes Redis is not the root cause. Application retry floods can make Redis look unhealthy when the real issue started upstream.

---

## 8. HPA exists, but the app still fails under load. Why?

### Strong answer

Autoscaling only helps if the bottleneck is parallelizable and the metrics reflect real demand. HPA cannot fix a slow database, external dependency limit, poor requests configuration, or cold-start-heavy application.

### Common reasons

- CPU is not the right scaling signal
- Resource requests are wrong
- Downstream dependency saturates first
- Max replicas too low
- Scale-out is slower than traffic spike

---

## 9. How do you investigate a memory leak in production?

### Strong answer

I first confirm whether memory usage trends upward across restarts or over time on a stable code path. Then I correlate it with deployments, traffic shape, and feature usage.

### Practical steps

- Compare versions before and after rollout
- Review heap or process memory metrics
- Capture profiles if the runtime supports it
- Verify whether cache growth or unbounded queueing is involved

---

## 10. What does graceful failure look like in a microservice environment?

### Strong answer

Graceful failure means the system degrades predictably instead of collapsing. Timeouts, retries, circuit breaking, fallback behavior, and backpressure should protect dependencies and preserve core user journeys.

### Good examples

- Return cached data when live dependency is degraded
- Fail closed or fail open depending on risk model
- Shed non-critical traffic before critical paths are affected

---

## 11. How do you reduce MTTR?

### Strong answer

I reduce MTTR by shortening detection, diagnosis, and safe mitigation time. The fastest incident is the one where responders immediately know what changed, what is impacted, and what rollback path is available.

### High-value improvements

- Better alerts tied to ownership
- Clear runbooks
- Deployment correlation in dashboards
- Fast rollback path
- Centralized logs and traces

---

## 12. How do you prevent repeat incidents?

### Strong answer

I convert incident lessons into guardrails: tests, policy checks, safer defaults, and stronger observability. A postmortem only matters if the system changes afterward.

### Examples

- Add admission policy for missing resource requests
- Add canary step before global rollout
- Alert on dependency saturation earlier
- Fix documentation and response ownership

---

## 13. Interview guidance

- Answer in a sequence, not as a command dump.
- Separate symptom, impact, trigger, and root cause.
- Show when you would mitigate first and diagnose second.
- Be explicit about tradeoffs under pressure.
