
# Observability in FinTech Systems  
## Prometheus, Grafana and OpenTelemetry

---

## Why Observability matters in FinTech
In FinTech systems, failures are rarely isolated.
A single slow dependency can cause:
- Payment retries
- Trade execution delays
- Wallet inconsistencies
- Regulatory incidents

Observability allows SREs to:
- Detect issues early
- Understand blast radius
- Debug across services
- Make safe, data-driven decisions

---

## The Three Pillars of Observability

### 1) Metrics – “Is it getting worse?”
Metrics provide **fast, aggregated signals**.

Common FinTech metrics:
- Request rate (RPS)
- Error rate (4xx/5xx)
- Latency (P95/P99)
- Queue depth (RabbitMQ)
- Cache hit ratio
- DB connection usage

### Prometheus
Prometheus is the **source of truth** for metrics.

Key characteristics:
- Pull-based scraping
- Time-series storage
- Label-based dimensionality
- Alerting via Alertmanager

Example metrics:
```text
http_requests_total{service="payments",status="500"}
http_request_duration_seconds_bucket
redis_connected_clients
db_pool_active_connections
````

---

### 2) Logs – “What exactly failed?”

Logs provide **context and evidence**.

In FinTech, logs must:

* Be structured (JSON)
* Include correlation IDs
* Avoid sensitive data (PII, PAN)

Good log fields:

* trace_id / correlation_id
* user_id (hashed)
* request_id
* error_code
* dependency_name

Bad practices:

* Free-text logs only
* Missing request identifiers
* Logging secrets or card data

---

### 3) Traces – “Why did it fail?”

Traces show **request flow across services**.

They are essential for:

* Debugging latency
* Understanding retries
* Finding slow dependencies

Key concepts:

* Span: a unit of work
* Trace: a collection of spans
* Parent/child relationships

---

## OpenTelemetry (OTel)

### Why OpenTelemetry

OpenTelemetry provides:

* Vendor-neutral instrumentation
* Unified signals (metrics, logs, traces)
* Consistent context propagation

In large FinTech orgs, OTel prevents:

* Tool lock-in
* Fragmented observability
* Inconsistent tracing

---

### Typical Architecture

```
Application
  └─ OpenTelemetry SDK
      └─ OpenTelemetry Collector
          ├─ Prometheus (metrics)
          ├─ Grafana Tempo (traces)
          └─ Logging backend (Loki / Splunk)
```

---

## Grafana – The Visualization Layer

Grafana provides:

* Unified dashboards
* Correlation between signals
* Fast ad-hoc analysis

Best practices:

* Dashboards per service + per journey
* Show SLOs and error budget burn
* Link metrics → logs → traces

---

## FinTech Example – Payment Latency Spike

### What happened

* Payment API latency spiked to P99 > 2s
* Error rate increased
* HPA scaled but did not help

### Metrics view

* CPU normal
* DB latency increased
* Redis timeouts spiked

### Traces view

* Majority of time spent waiting on DB
* Multiple retries per request
* Retry logic amplified load

### Logs view

* Timeout exceptions
* Circuit breaker opening

### Outcome

* Rate limiting enabled
* Retries capped
* DB connection pool tuned
* New alerts added on latency burn rate

---

## Alerting Philosophy (SRE-grade)

Do alert on:

* Error budget burn rate
* Sustained latency degradation
* Dependency saturation

Do NOT alert on:

* Single errors
* Short spikes without impact
* Metrics without user impact

Example:

> Alert when 30% of monthly error budget is consumed in 1 hour.

---

## Common Anti-Patterns

* Dashboards without ownership
* Alerts without runbooks
* Metrics without labels strategy
* Traces without sampling control
* Logging sensitive data

---

## Interview-Ready Summary

"Observability means being able to understand a system’s internal state from its outputs.
In FinTech, we rely on metrics for detection, logs for evidence, and traces for causality.
OpenTelemetry allows us to correlate all three and debug incidents safely under pressure."

---

## Summary

* Metrics tell you **something is wrong**
* Logs tell you **what happened**
* Traces tell you **why it happened**
* OpenTelemetry ties everything together
* Observability is not tooling — it is **operational awareness**

