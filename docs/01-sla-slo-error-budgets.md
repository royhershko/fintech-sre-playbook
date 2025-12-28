# SLA, SLO and Error Budgets – FinTech Perspective

## Why this matters in FinTech
In large-scale FinTech systems (payments, trading, wallets), reliability is not optional.
Failures directly impact:
- Money movement
- Regulatory exposure
- Customer trust
- Market risk

SREs use **SLOs and Error Budgets** to balance reliability and delivery speed in a controlled way.

---

## Key Definitions

### SLA – Service Level Agreement
A **contractual commitment** to customers.

Example:
- "99.9% monthly availability"

Important:
- Defined by business and legal teams
- Violating an SLA usually has **financial or regulatory consequences**

SREs **do not design systems to SLA** — they design to SLO.

---

### SLO – Service Level Objective
An **internal reliability target** used by engineering.

Example:
- 99.95% successful payment processing per 30 days
- P99 latency < 300ms for trade execution API

SLOs:
- Are measurable
- Are owned by engineering
- Drive technical decisions

---

### SLI – Service Level Indicator
The **actual measurement**.

Examples:
- Success ratio of `/payments/execute`
- P99 latency of `/orders/create`
- Error rate of wallet balance updates

---

## Error Budget – The Control Mechanism

**Error Budget = 1 - SLO**

Example:
- SLO: 99.95% success rate
- Error Budget: 0.05%

This budget represents how much failure is **allowed**.

---

## Why Error Budgets are powerful
Error Budgets allow:
- Faster feature delivery when reliability is good
- Forced stability work when reliability degrades

They prevent:
- Endless firefighting
- Emotional decision making
- Over-engineering

---

## Real FinTech Example – Payments Service

### Service
`payment-execution-service`

### SLO
- 99.95% of payment requests succeed over 30 days

### Error Budget
- 0.05% allowed failure rate

### Incident Scenario
- DB connection pool exhaustion during peak hours
- Error rate spikes for 20 minutes
- 30% of monthly error budget consumed

### Resulting Decisions
- Feature deployments frozen
- Focus shifted to:
  - Connection pool tuning
  - Load shedding
  - Backpressure mechanisms
- SLO reviewed and alerts tightened

---

## Alerting Based on Error Budget
SRE alerts should **not trigger on single errors**.

Better approach:
- Alert when **error budget burn rate** is high

Example:
- 50% of error budget consumed in 1 hour
- Immediate escalation
- Rollback or traffic limiting

---

## Common Mistakes
- Setting SLO = 100%
- Alerting on raw error counts
- Using SLAs as engineering targets
- Ignoring business context

---

## SRE Interview Tip
If asked:
> "How do you balance reliability and feature velocity?"

Strong answer:
> "We define SLOs per critical user journey, track error budget burn, and use it as a gating mechanism for releases and incident prioritization."

---

## Summary
- SLA is a business promise
- SLO is an engineering target
- Error Budget is the decision-making tool
- In FinTech, SLOs must reflect **real money flow**

Reliability is not about perfection —  
it is about **predictability and controlled failure**.
