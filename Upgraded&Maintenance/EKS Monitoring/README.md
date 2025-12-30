# Amazon EKS Monitoring – Architecture & Production Guide

## What is Monitoring in EKS?

Monitoring in Amazon EKS means **observing the health, performance, and behavior** of:

* Kubernetes cluster
* Worker nodes
* Pods & containers
* Applications

Monitoring answers questions like:

* Is my cluster healthy?
* Are pods crashing?
* Is CPU/memory sufficient?
* Is traffic behaving normally?

---

## Why Monitoring Is Critical in Production

Without monitoring:

* Failures go unnoticed
* Performance issues affect users
* Autoscaling does not work correctly
* Troubleshooting becomes guesswork

In production, **monitoring is mandatory**, not optional.

---

## EKS Monitoring Layers (Very Important)

EKS monitoring is done in **multiple layers**:

1. Control plane monitoring
2. Node monitoring
3. Pod & container monitoring
4. Application monitoring
5. Alerting

---

## 1️⃣ Control Plane Monitoring

### What is Monitored

* API server latency & errors
* etcd health (AWS-managed)
* Scheduler & controller behavior

### How It Works in EKS

* AWS manages control plane
* Metrics are sent to **Amazon CloudWatch** automatically

### Production Notes

* No access to control plane nodes
* You only consume metrics & logs

---

## 2️⃣ Node Monitoring (Worker Nodes)

### What Is Monitored

* CPU utilization
* Memory usage
* Disk usage
* Network I/O

### Tools Used

* CloudWatch Agent
* Node Exporter (Prometheus)

### Production Use Case

* Detect node pressure
* Prevent pod eviction

---

## 3️⃣ Pod & Container Monitoring

### What Is Monitored

* Pod CPU & memory
* Restart count
* OOMKilled events
* Container status

### Key Kubernetes Metrics

* `container_cpu_usage_seconds_total`
* `container_memory_working_set_bytes`

### Tools Used

* Prometheus
* kube-state-metrics

---

## 4️⃣ Application Monitoring

### What Is Monitored

* Request latency
* Error rate
* Throughput

### Tools Used

* Prometheus
* Application metrics (/metrics endpoint)
* Grafana dashboards

### Production Use Case

* SLO / SLA tracking
* Performance optimization

---

## 5️⃣ Logging vs Monitoring (Important Difference)

| Monitoring        | Logging             |
| ----------------- | ------------------- |
| Metrics over time | Detailed events     |
| Trend analysis    | Root cause analysis |
| Low storage       | High storage        |

Both are required in production.

---

## Common Monitoring Tooling in EKS

### 1️⃣ Amazon CloudWatch Container Insights

* AWS-native solution
* Collects:

  * Node metrics
  * Pod metrics
  * Control plane metrics

Best for:

* Quick setup
* AWS-only environments

---

### 2️⃣ Prometheus + Grafana (Most Popular)

Prometheus:

* Collects metrics
* Scrapes endpoints

Grafana:

* Visualization
* Dashboards

Used for:

* Deep Kubernetes metrics
* Custom alerts

---

### 3️⃣ Metrics Server (Autoscaling)

* Provides CPU/memory metrics
* Used by HPA

⚠️ Not for long-term monitoring

---

## Alerting in EKS

### Why Alerts Matter

* Detect issues early
* Reduce downtime

### Common Alert Types

* Pod crash loops
* High CPU/memory
* Node not ready
* API server errors

### Tools Used

* Alertmanager
* CloudWatch Alarms

---

## Observability vs Monitoring

Monitoring answers:

* What is wrong?

Observability answers:

* Why is it wrong?

Observability includes:

* Metrics
* Logs
* Traces

---

## Production Monitoring Architecture (Typical)

```
Pods / Nodes
 ↓
Prometheus / CloudWatch
 ↓
Grafana / CloudWatch Dashboards
 ↓
Alertmanager / SNS / Slack
```

---

## Best Practices for EKS Monitoring

* Monitor cluster, node, pod, and app separately
* Always enable alerts
* Correlate metrics with logs
* Monitor autoscaling behavior
* Avoid alert fatigue

---

## Common Production Mistakes

* Only monitoring nodes
* No application metrics
* No alerts configured
* Relying only on Metrics Server

---

## Interview One-Liner (Very Important)

"EKS monitoring is implemented across multiple layers using CloudWatch or Prometheus to track control plane, nodes, pods, and application metrics, with alerting to proactively detect and resolve production issues."
