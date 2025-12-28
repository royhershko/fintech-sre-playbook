# Redis Connection Storm (FinTech Production Playbook)

## Scenario
A sudden spike in traffic or a bug causes many pods to open new Redis connections rapidly.
Redis becomes overloaded, latency increases, timeouts spread across services, and the incident cascades.

This is common in FinTech during:
- Market open / volatility spikes
- Campaigns / user bursts
- Autoscaling events (many new pods start together)
- Misconfigured connection pooling

---

## Impact
- Increased P95/P99 latency
- Timeouts and retries amplify load (retry storm)
- Queue backlogs (RabbitMQ) due to slower processing
- User-facing failures: payments, trading, wallet balance updates
- Potential risk: stale cache used as “truth” by some services

---

## Symptoms (What you see)
### In Grafana / Prometheus
- Redis CPU high OR Redis latency up
- `connected_clients` spikes
- `rejected_connections` increases
- `evicted_keys` might appear (if memory pressure)
- Application error rate rises (5xx / timeouts)

### In logs
- `RedisTimeoutException`
- `Connection reset by peer`
- `Too many connections`
- Increased retry logs / circuit breaker open

### In Kubernetes
- Pods restart (OOM, CrashLoopBackOff due to retry loops)
- HPA scales up, making it worse (more pods → more connections)

---

## Immediate Triage (First 5–10 minutes)
### 1) Confirm Redis is the bottleneck
- Check Redis latency and client connections
- Correlate with application timeouts and error rate
- Check if DB is also impacted (cache miss wave can hit DB)

### 2) Stop the bleeding (reduce amplification)
Choose safe mitigations in this order:
- **Rate limit** / reduce traffic at ingress / API gateway
- **Disable aggressive retries** if possible (feature flag / config)
- **Enable circuit breaker** / fail fast instead of piling up
- **Scale Redis** (if managed service supports it quickly)
- **Temporarily scale DOWN noisy clients** (the service causing storm)

### 3) Stabilize Kubernetes behavior
- Consider temporarily pausing HPA for the worst offender
- Avoid scaling to infinite pods that open more Redis connections

---

## Commands (Kubernetes / Infra)
### Identify top offenders (pods/services)
```bash
kubectl get pods -A | grep -i <service-name>
kubectl top pods -A | sort -k3 -nr | head
kubectl logs -n <ns> <pod> --tail=200 | grep -iE "redis|timeout|connection"
