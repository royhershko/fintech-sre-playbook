
# Python for SRE Automation (Production-Oriented)

This document focuses on **how SREs use Python in real production environments** —
for automation, incident response, diagnostics, and operational safety.

No application development.
No theory.
Only **what helps during on-call and reliability work**.

---

## Why Python Matters for SREs

Python replaces:
- Manual checks
- Long bash pipelines
- Ad-hoc debugging
- Error-prone shell scripts

Python allows SREs to:
- Collect data from systems
- Analyze signals quickly
- React programmatically
- Integrate with CI/CD and monitoring tools

**Interview-ready sentence:**
> “Python allows me to observe, analyze, and react to production issues in a safe and repeatable way.”

---

## 1) One-liner execution (`python3 -c`)

```bash
python3 -c "import sys; print(sys.version)"
````

### What it does

* Executes Python code inline
* No file, no environment setup

### Why it matters in production

* Verify Python availability
* Confirm runtime version
* Detect mismatched Python environments

### Real-world scenario

A script fails on a legacy server → you quickly discover it runs Python 3.6 without required modules.

---

## 2) Virtual Environments (`venv`)

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### What it does

* Creates an isolated Python environment

### Why SREs use it

* Prevents breaking system Python
* Safe to use on bastion / prod-adjacent hosts
* Ideal for temporary diagnostics tooling

**Interview note:**

> “I always isolate dependencies to avoid impacting the host system.”

---

## 3) HTTP Health Checks (no external libraries)

```python
import urllib.request

urllib.request.urlopen(
    "http://localhost:9090/-/ready",
    timeout=2
)
```

### What it does

* Sends an HTTP request
* Fails on timeout or connection error

### Why it matters

* Lightweight health checks
* Replacement for `curl` inside scripts
* Base for synthetic monitoring

### Production example

Prometheus service is running, but `/ready` fails → the service is not actually available.

---

## 4) Validating Prometheus Metrics Endpoints

```python
data = (
    urllib.request
    .urlopen(url, timeout=2)
    .read()
    .decode()
    .splitlines()
)
```

### What it does

* Fetches raw metrics text

### Why it matters

* Validate exporters
* Inspect labels and counters
* Debug why PromQL returns empty results

**Interview tip:**

> “I always verify raw metrics before blaming queries or dashboards.”

---

## 5) Log Filtering and Pattern Matching

```python
import re
```

### What it enables

* Pattern matching
* Counting occurrences
* Grouping errors

### Why SREs prefer this over `grep`

* Structured logs (JSON)
* Extendable logic
* Statistics, not just matching

### Real incident example

Count how many `timeout` errors occurred in the last 5 minutes, not just whether they exist.

---

## 6) Running System Commands (`subprocess`)

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True
)
```

### What it does

* Executes OS commands
* Captures stdout, stderr, and exit code

### Why this is critical for SREs

* Wrap `kubectl`, `systemctl`, `df`, `journalctl`
* Build smart automation around system state

### Production use case

Incident script:

* Check service status
* Restart if needed
* Alert if restart fails

---

## 7) Exit Codes – The Core of Automation

```python
exit(result.returncode)
```

### Meaning

* `0` → success
* non-zero → failure

### Why exit codes matter

* CI/CD pipelines
* Cron jobs
* Monitoring checks
* Runbooks

**Interview-ready sentence:**

> “I always return meaningful exit codes so automation can react correctly.”

---

## 8) Why Python Over Bash (Most of the Time)

| Bash                   | Python               |
| ---------------------- | -------------------- |
| Hard to read           | Clear and structured |
| Fragile parsing        | Robust parsing       |
| Limited error handling | Exceptions           |
| Difficult debugging    | Tracebacks           |

**Golden rule:**

> “Bash is great for a single command. Python is better for a process.”

---

## SRE Interview Summary

> “As an SRE, I use Python to perform health checks, inspect metrics and logs,
> wrap system commands, and automate incident responses — in a way that is safe,
> readable, and production-friendly.”

---

## Key Takeaway

Python is not about building services.
It is about **operational leverage**:

* Faster diagnosis
* Safer actions
* Repeatable responses


