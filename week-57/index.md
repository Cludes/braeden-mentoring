---
layout: page
title: "Monitoring and Alerting with Prometheus and Grafana"
week_number: "57"
release_date: "2027-05-18"
phase_label: "Phase 11"
phase_name: "Advanced DevOps"
phase_color: "#06b6d4"
permalink: /week-57/
---

> Week 57. Observability: knowing what your systems are doing.
> 
> You cannot secure what you cannot see. Prometheus collects metrics from services. Grafana visualises them. Together they give you dashboards, alerts, and visibility into what is normal -- which is essential for detecting what is abnormal.
> 
> You will run both locally using Docker Compose.

---

# Monitoring and Observability

**Pillar:** DevOps
**Level:** Intermediate
**Read time:** 20 minutes
**Prerequisite:** `beginner/01_ci-cd-fundamentals.md`

## The Three Pillars of Observability

Observability means being able to understand what your system is doing from the outside, using its outputs. The three signal types:

| Pillar | What it tells you | Example tool |
|--------|------------------|-------------|
| **Metrics** | Aggregate numbers over time (counts, rates, gauges) | Prometheus + Grafana |
| **Logs** | Discrete events with context | Loki, ELK, CloudWatch |
| **Traces** | End-to-end request journey across services | Jaeger, Zipkin, OTEL |

Monitoring is about knowing when something is wrong. Observability is about being able to figure out why.

---

## Metrics with Prometheus

Prometheus scrapes HTTP endpoints that expose metrics in its format, stores them as time series, and evaluates alert rules.

### Metric types

| Type | Definition | Example |
|------|-----------|---------|
| Counter | Only increases | `http_requests_total` |
| Gauge | Can go up or down | `memory_usage_bytes` |
| Histogram | Distribution of values in buckets | `http_request_duration_seconds` |
| Summary | Like histogram, calculates quantiles client-side | `rpc_duration_seconds` |

### Exposing metrics in Python

```python
from prometheus_client import Counter, Histogram, start_http_server
import time

REQUEST_COUNT = Counter("http_requests_total", "Total HTTP requests", ["method", "endpoint", "status"])
REQUEST_LATENCY = Histogram("http_request_duration_seconds", "Request latency", ["endpoint"])

@app.route("/")
def index():
    start = time.time()
    # ... your handler ...
    REQUEST_COUNT.labels(method="GET", endpoint="/", status="200").inc()
    REQUEST_LATENCY.labels(endpoint="/").observe(time.time() - start)
    return "OK"

start_http_server(8001)  # Prometheus scrapes this
```

### PromQL queries

```
# Request rate over last 5 minutes
rate(http_requests_total[5m])

# 95th percentile latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Error rate
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

---

## Alerting

### Good alert properties

- **Actionable**: every alert requires a human action. Alerts that nobody acts on become noise.
- **Accurate**: few false positives. If alerts fire when nothing is wrong, people stop caring.
- **Contextual**: the alert links to a runbook explaining what to do.

### Alert fatigue

When there are too many alerts or too many false positives, engineers start ignoring them. This is how real incidents get missed.

Rule: if an alert fires and the response is always "nothing to do", delete or lower it.

### Common alert rules

```yaml
# prometheus_rules.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate above 5%"
          runbook: "https://wiki/runbooks/high-error-rate"

      - alert: HighLatency
        expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "99th percentile latency above 2s"
```

---

## Structured Logging

Plain text logs are hard to query. Structured (JSON) logs are queryable.

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        return json.dumps({
            "ts": self.formatTime(record),
            "level": record.levelname,
            "msg": record.getMessage(),
            "logger": record.name,
        })

# Usage
logger.info("request processed", extra={"user_id": 42, "duration_ms": 120})
```

**What to always log:**
- Request ID (correlates all logs for one request)
- User or session ID (for incident investigation)
- Duration (for latency debugging)
- Status code / error code

**What to never log:**
- Passwords or tokens
- Full credit card numbers
- PII beyond what is legally required

---

## Grafana Dashboards

Grafana visualises Prometheus data. A good dashboard answers one question per panel:

| Panel type | Use for |
|-----------|---------|
| Time series | Request rate, error rate, latency over time |
| Gauge | Current CPU, memory, disk |
| Bar chart | Error counts by endpoint |
| Table | Slowest endpoints, top error sources |
| Stat | Single key number (uptime, total requests today) |

### Dashboard layout pattern

```
Row 1: Traffic (requests/sec, active connections)
Row 2: Errors (error rate %, error count by type)
Row 3: Latency (p50, p95, p99)
Row 4: Resources (CPU, memory, disk)
Row 5: Business metrics (signups/min, checkouts/min)
```

---

## The SLO Framework

**SLI (Service Level Indicator):** A metric that measures service behaviour. Example: request success rate.

**SLO (Service Level Objective):** The target value. Example: 99.9% success rate over 30 days.

**SLA (Service Level Agreement):** A contract with consequences for missing the SLO. Example: refund if uptime drops below 99.9%.

**Error budget:** The allowed downtime before you breach the SLO. At 99.9%, you have 43 minutes of downtime per month. Burn it on planned maintenance or features that require risky deploys.

## Checklist

- [ ] Can you explain the difference between a metric, a log, and a trace
- [ ] Write a PromQL query that calculates the error rate over 5 minutes
- [ ] Set up Prometheus + Grafana locally using `activities/devops/intermediate/01_monitoring-basics.md`
- [ ] Define an SLI and SLO for a service you work on

## Next Steps

- Lab: `activities/devops/intermediate/01_monitoring-basics.md`
- Guide: `advanced/01_kubernetes.md`
- Quiz: `learning/quizzes/04_devops.md`

---

## Activity: Set Up Prometheus and Grafana

1. Write a docker-compose.yml that runs Prometheus and Grafana together
1. Configure Prometheus to scrape its own metrics endpoint (localhost:9090/metrics)
1. Open Grafana (localhost:3000, default admin/admin), connect it to Prometheus as a data source
1. Build a dashboard with 3 panels: CPU usage, memory usage, and HTTP request rate
1. Set up an alert rule: fire an alert if a metric exceeds a threshold for 1 minute
1. Add a second scrape target: run a simple Python app that exposes a /metrics endpoint using the prometheus_client library, add it to prometheus.yml
1. You know you are done when you have a working Grafana dashboard pulling live data from Prometheus

---

## Check-in Questions

Write your answers down before your next session.

1. What is the difference between metrics, logs, and traces?
2. What is a Prometheus scrape target?
3. Why does a security team care about application metrics?
4. What is an alert fatigue problem and how do you avoid it?
