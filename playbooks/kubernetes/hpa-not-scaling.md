
# HPA Not Scaling (Kubernetes Production Playbook)

## Scenario
The service is under load (latency and errors rising), but **HPA does not scale** as expected.
Alternatively, HPA scales pods but **traffic does not improve** (pods stay NotReady / no endpoints).

This is common in large-scale FinTech during:
- Market open / volatility spikes
- Batch jobs / reconciliation windows
- Sudden traffic bursts + cold starts
- Misconfigured metrics or resource requests

---

## Impact
- Rising P95/P99 latency
- Increased timeouts and retries (amplifies load)
- Queue backlog growth (RabbitMQ) because consumers can’t keep up
- User journey failures: trade placement, payments, wallet balance checks
- Risk: cascading failure across dependencies (Redis/DB)

---

## Symptoms (What you see)
### In dashboards
- RPS increases, latency increases
- Error rate rises
- CPU may be low or high (depends on issue)

### In Kubernetes
- HPA replicas remain constant (no scale)
- OR HPA scales, but pods are NotReady
- OR pods are created but cannot be scheduled (Pending)

---

## Fast Triage (First 5 minutes)

### 1) Confirm HPA state
```bash
kubectl -n <ns> get hpa
kubectl -n <ns> describe hpa <hpa-name>
````

Look for:

* Current/Target metrics (e.g. 80%/50%)
* Conditions: `AbleToScale`, `ScalingActive`, `ScalingLimited`
* Errors like:

  * `failed to get cpu utilization`
  * `missing request for cpu`
  * `unable to fetch metrics`

---

### 2) Check if metrics pipeline works (metrics-server)

```bash
kubectl top nodes
kubectl -n <ns> top pods
```

If `kubectl top` fails or returns nothing:

* metrics-server is missing/broken
* HPA cannot compute utilization

---

### 3) Check if pods can be scheduled (capacity issue)

```bash
kubectl -n <ns> get pods
kubectl -n <ns> get pods | egrep -i "pending|notready|crash|error|backoff"
kubectl get nodes
kubectl top nodes
```

If pods are Pending:

```bash
kubectl -n <ns> describe pod <pod> | sed -n '1,200p'
```

Common reasons:

* Insufficient CPU/memory
* Node taints / affinity mismatch
* IP exhaustion (VPC CNI)
* PVC not bound

---

### 4) If HPA scaled but service still slow — check endpoints/readiness

```bash
kubectl -n <ns> get endpoints <service>
kubectl -n <ns> describe pod <pod> | sed -n '1,220p'
kubectl -n <ns> logs <pod> --tail=200
```

If endpoints are empty or low:

* pods are NotReady
* readiness probes failing
* scaling is "fake capacity" (pods exist, not serving)

---

## Common Root Causes (and how to confirm)

### Cause A: Missing resource requests (CPU/Memory)

**HPA based on CPU requires `resources.requests.cpu`**.

Confirm:

```bash
kubectl -n <ns> get deploy/<deploy> -o yaml | sed -n '1,220p'
```

Look for:

* `resources: requests: cpu: ...`

Mitigation:

* Add CPU requests
* Re-deploy
* Consider setting realistic requests based on observed usage

---

### Cause B: metrics-server issues (no metrics → no scaling)

Confirm:

```bash
kubectl -n kube-system get pods | grep -i metrics
kubectl -n kube-system logs deploy/metrics-server --tail=120
```

Mitigation:

* Restore metrics-server
* Ensure API aggregation works
* Validate permissions and TLS settings

---

### Cause C: Wrong scaling signal (CPU low but latency high)

Typical in I/O bound services:

* waiting on DB
* waiting on Redis
* waiting on network
  CPU stays low, but user latency skyrockets.

Confirm:

* CPU utilization low in dashboards
* DB/Redis latency high
* Application spans show dependency waiting time

Mitigation options:

* Use HPA on custom metrics (RPS, queue depth, latency SLI)
* Add backpressure (rate limiting, circuit breaker)
* Scale dependency or reduce load

---

### Cause D: Readiness/Startup delays hide capacity

HPA creates pods but:

* they take too long to become Ready
* readiness probe misconfigured
* startup probe missing

Confirm:

```bash
kubectl -n <ns> describe pod <pod> | sed -n '1,240p'
```

Look for:

* readiness probe failures
* slow image pull
* slow init containers

Mitigation:

* Add `startupProbe`
* Fix readiness probe path/port/timeouts
* Use pre-warmed images or smaller images
* Gradual rollout / warmup logic

---

### Cause E: Cluster capacity / node scaling lag (EKS + Karpenter/ASG)

Pods scale, but nodes don’t appear fast enough.

Confirm:

* many pods pending
* nodes at high utilization
* node provisioning events

Mitigation:

* Ensure cluster autoscaler / Karpenter is healthy
* Increase node pool limits
* Use mixed instance types
* Pre-scale before known peak windows (market open)

---

## Safe Mitigation Patterns (during incident)

Choose the least risky stabilization first:

1. Reduce traffic amplification

* Rate limit at gateway/ingress
* Reduce retries / add backoff+jitter
* Fail fast (circuit breaker)

2. Restore serving capacity

* Fix readiness/probes
* Add replicas manually (short-term)

```bash
kubectl -n <ns> scale deploy/<deploy> --replicas=10
```

3. Rollback risky changes

```bash
kubectl -n <ns> rollout undo deploy/<deploy>
```

4. If dependency-bound

* Protect DB/Redis first (bulkhead / queue / cache strategy)
* Prevent “scale pods to kill DB” behavior

---

## Prevention Checklist

* [ ] All workloads have realistic `requests/limits`
* [ ] metrics-server health monitored
* [ ] HPA uses correct signals (CPU is not always correct)
* [ ] Readiness + Startup probes validated under load
* [ ] Autoscaling strategy includes node scaling (Karpenter/ASG)
* [ ] Load tests include HPA behavior and cold start time
* [ ] Alerts:

  * error budget burn rate
  * HPA unable to get metrics
  * pending pods growth
  * readiness failure rate

---

## Interview Story (60 seconds)

"During a peak trading window, latency spiked but HPA didn’t scale.
We quickly checked HPA describe and saw it couldn’t compute CPU utilization because the deployment had no CPU requests.
We stabilized the system with rate limiting and a controlled manual scale, then patched resource requests and redeployed.
Later we improved our scaling strategy by adding custom metrics for dependency-bound services and validated probes with load tests."

---

## Summary

HPA not scaling is usually one of:

* No metrics (metrics-server)
* No requests (HPA can't calculate utilization)
* Wrong signal (CPU low but dependencies slow)
* Fake capacity (pods not Ready / no endpoints)
* No cluster capacity (nodes not scaling fast enough)

Stabilize first, then fix the scaling chain end-to-end:
**metrics → HPA → pods → readiness → nodes → dependencies**


