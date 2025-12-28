# FinTech SRE Playbook

A practical, production-oriented Site Reliability Engineering (SRE) playbook focused on large-scale FinTech systems.

This repository is designed as a real-world SRE knowledge base, combining:
- Architecture thinking
- Incident response
- Kubernetes production operations
- Observability (metrics, logs, traces)
- FinTech-specific failure scenarios

## Who is this for?
- SRE / DevOps engineers
- Engineers working in FinTech, trading, payments, or high-scale systems
- Interview preparation for Senior SRE roles
- Teams building reliability culture

## Repository Structure

- docs/ – Core SRE concepts and mindset
- playbooks/ – Incident response and troubleshooting scenarios
- commands/ – Essential Linux / Kubernetes / Cloud commands
- examples/ – Alerts, dashboards, configs (Prometheus, Grafana, OTEL)
- templates/ – RCA, SLO, runbook templates


## Topics Covered
- SLA / SLO / Error Budgets (FinTech-oriented)
- Observability: Prometheus, Grafana, OpenTelemetry, Tempo
- Kubernetes in production (HPA, Karpenter, probes)
- Redis, SQL, RabbitMQ failure modes
- Incident management & postmortems
- Performance, capacity & cost awareness
- Security & DevSecOps for SREs

## Philosophy
> Reliability is not about preventing failure —  
> it is about **controlling blast radius and recovery time**.

This playbook focuses on:
- What actually breaks in production
- How to detect it fast
- How to mitigate safely
- How to prevent recurrence

---

Built as a living document — evolving with real production lessons.
