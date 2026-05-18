# Kubernetes Incident RCA: Intermittent 503s During Checkout

## Incident Summary
A production microservice returned intermittent `503` errors.
The failure rate was low and irregular: about 1 in 50 calls to the payment service failed.

Users saw random checkout errors with no obvious pattern.

## Initial Signals
At first, the cluster looked healthy:
- `kubectl get pods` showed all Pods in `Running` state.
- No `CrashLoopBackOff`.
- Zero restarts.

The first question was whether the issue was Pod-specific or Service-wide.

## Investigation Steps
1. Verified Service endpoint registration:
	- `kubectl get endpoints <service-name>`
2. Verified Pod readiness state:
	- `kubectl get pods`
3. Compared Pod ages and noticed an anomaly:
	- One Pod age: ~4 hours.
	- Other three Pods: ~6 days.

This newer Pod passed liveness checks, but the application needed about 90 seconds to fully initialize its connection pool.

The readiness probe passed after only 10 seconds.

Kubernetes treated the Pod as ready before it could safely handle payment traffic.

## Validation
To isolate behavior, traffic was sent directly to the suspect Pod via `kubectl port-forward`.

Results:
- `/health` returned `200`.
- Payment endpoint returned `503`.

## Why Failure Rate Was ~1 in 50 (Not 1 in 4)
The low error rate was explained by connection behavior:
- `kube-proxy` load balancing is connection-oriented, not request-oriented.
- Clients commonly reuse TCP connections (keep-alive).
- Most traffic stayed pinned to existing healthy Pod connections.
- Only some new connections landed on the not-yet-ready Pod.

## Full Diagnosis Checklist
1. Checked readiness probe config in Deployment YAML (timing and endpoint semantics).
2. Watched endpoint updates during rollout:
	- `kubectl get endpoints -w`
3. Correlated `503` logs by Pod IP to identify the error-producing Pod.

## Fix Applied
1. Changed readiness probe to validate real payment path behavior, not only `/health`.
2. Increased `initialDelaySeconds` from 10 to 60.
3. Added rollout safety controls:
	- `minReadySeconds: 30`
	- RollingUpdate `maxUnavailable: 0`

After the next deployment, `503` errors stopped.

## Lessons Learned
1. Watch `kubectl get endpoints -w` during every rolling deployment.
2. Test business endpoints directly on a Pod, not only health endpoints.
3. A readiness probe is meaningful only if it verifies real readiness.
4. `minReadySeconds` helps prevent unstable Pods from being treated as available.

The cluster was healthy; the availability issue came from probe and rollout configuration.

## Command Pack: Reproduce This Debugging Workflow

Use these commands as a quick runbook when you see intermittent `503` errors from a Service.

```bash
# 0) Set context variables
NS=prod
APP_LABEL="app=payment"
SVC=payment-service
DEPLOY=payment
PORT=8080

# 1) Baseline health: pods, restarts, ages, readiness
kubectl get pods -n "$NS" -l "$APP_LABEL" -o wide
kubectl get pods -n "$NS" -l "$APP_LABEL" \
	-o custom-columns=NAME:.metadata.name,READY:.status.containerStatuses[*].ready,RESTARTS:.status.containerStatuses[*].restartCount,AGE:.metadata.creationTimestamp

# 2) Service to endpoint mapping
kubectl get svc -n "$NS" "$SVC" -o wide
kubectl get endpoints -n "$NS" "$SVC" -o wide

# 3) Watch endpoint changes during rollout
kubectl get endpoints -n "$NS" "$SVC" -w

# 4) Check rollout config and status
kubectl get deploy -n "$NS" "$DEPLOY" -o yaml | rg -n "readinessProbe|livenessProbe|initialDelaySeconds|minReadySeconds|maxUnavailable|maxSurge"
kubectl rollout status deploy/"$DEPLOY" -n "$NS"
kubectl rollout history deploy/"$DEPLOY" -n "$NS"

# 5) Find newest pod (often suspect during rollout)
NEWEST_POD=$(kubectl get pods -n "$NS" -l "$APP_LABEL" --sort-by=.metadata.creationTimestamp -o jsonpath='{.items[-1:].metadata.name}')
echo "$NEWEST_POD"

# 6) Port-forward directly to that pod and test real business path
kubectl port-forward -n "$NS" pod/"$NEWEST_POD" 18080:"$PORT"
# In another terminal:
curl -i http://127.0.0.1:18080/health
curl -i http://127.0.0.1:18080/payment

# 7) Correlate errors by pod from recent logs
for p in $(kubectl get pods -n "$NS" -l "$APP_LABEL" -o name); do
	count=$(kubectl logs -n "$NS" "$p" --since=30m | rg -c " 503 |HTTP 503|status=503" || true)
	echo "$p 503_count=$count"
done

# 8) Optional: compare readiness state vs endpoint registration quickly
kubectl get pods -n "$NS" -l "$APP_LABEL" -o wide
kubectl get endpoints -n "$NS" "$SVC" -o yaml
```

### Expected Outcome Pattern
1. A newly created Pod appears in endpoints early.
2. Health check passes, but business endpoint fails on that Pod.
3. `503` counts are concentrated on the newest Pod.

### Fast Remediation Checklist
1. Point readiness probe to a true business-readiness endpoint.
2. Increase `initialDelaySeconds` to cover warm-up time.
3. Set `minReadySeconds` to enforce stability before availability.
4. Use rolling update with `maxUnavailable: 0` for safer cutover.