# AWS Fargate in Amazon EKS – Architecture & Production Guide

## What is AWS Fargate?

AWS Fargate is a **serverless compute engine** that allows you to run containers **without managing EC2 worker nodes**.

When used with Amazon EKS:

* You run **Kubernetes pods**
* AWS manages **servers, scaling, OS, patching**
* You pay **per pod resources (CPU & memory)**

---

## Core Idea (Simple Explanation)

> With Fargate, you deploy pods directly, and AWS runs them on managed infrastructure without exposing or requiring EC2 nodes.

---

## EKS + Fargate Architecture

### High-Level Flow

```
User / Controller
 ↓
Kubernetes Pod
 ↓
Fargate Scheduler
 ↓
AWS Managed Compute
```

Key points:

* No worker nodes
* No node groups
* No SSH access

---

## How Fargate Works in EKS (Step-by-Step)

1. You create an EKS cluster
2. You define a **Fargate Profile**
3. Pod matches namespace + labels
4. Pod is scheduled onto Fargate
5. AWS provisions isolated compute
6. Pod runs and scales automatically

---

## What is a Fargate Profile?

A Fargate profile defines:

* Namespace
* Optional pod labels

Pods matching the profile:

* Automatically run on Fargate

Example use:

* `kube-system` namespace
* Batch workloads namespace

---

## Supported Kubernetes Features on Fargate

### Supported

* Pods
* Services
* ConfigMaps
* Secrets
* IRSA
* ALB / NLB integration

### Not Supported

* DaemonSets
* Privileged containers
* Host networking
* Custom CNI plugins
* kube-proxy replacement

---

## Networking in Fargate

* Uses AWS VPC networking
* Each pod gets a **dedicated ENI**
* Pod has its own **private IP**

Security:

* Security groups per pod

---

## Storage with Fargate

### Supported

* Amazon EFS (persistent storage)

### Not Supported

* Amazon EBS
* HostPath volumes

Production implication:

* Fargate is **not suitable for databases**

---

## Security Model in Fargate

### Strong Isolation

* One pod per VM
* No noisy neighbor

### IAM Integration

* Mandatory use of IRSA
* Fine-grained IAM permissions

---

## Observability in Fargate

### Logs

* Fluent Bit → CloudWatch

### Metrics

* CloudWatch Container Insights

### Limitations

* No node-level metrics

---

## Production Use Cases for Fargate

### 1️⃣ System Components

* CoreDNS
* Controllers

---

### 2️⃣ Batch & Event-Driven Jobs

* CronJobs
* Data processing

---

### 3️⃣ Multi-Tenant Platforms

* Strong isolation between workloads

---

### 4️⃣ Bursty or Unpredictable Workloads

* Scale to zero
* Pay only when running

---

## When NOT to Use Fargate

❌ High-performance networking workloads
❌ DaemonSet-based tooling (Cilium, monitoring agents)
❌ Stateful databases
❌ Custom kernel features

---

## Fargate vs EC2 Nodes (Production Comparison)

| Feature         | Fargate     | EC2 Nodes    |
| --------------- | ----------- | ------------ |
| Node management | None        | Required     |
| Scaling         | Automatic   | Manual / ASG |
| Cost model      | Per pod     | Per instance |
| DaemonSets      | ❌           | ✅            |
| EBS             | ❌           | ✅            |
| Isolation       | Very strong | Shared       |

---

## Common Production Patterns

### Hybrid Cluster (Most Common)

* EC2 nodes → main workloads
* Fargate → system & batch pods

---

## Cost Considerations

* Fargate is more expensive per unit
* No cost for idle capacity
* Best for spiky workloads

---

## Common Mistakes

* Trying to run databases on Fargate
* Expecting DaemonSets to work
* Ignoring cost modeling

---

## Interview One-Liner (Very Important)

"AWS Fargate with EKS allows us to run Kubernetes pods without managing worker nodes. In production, it’s used for system components, batch jobs, and bursty workloads where operational simplicity and isolation are more important than raw cost efficiency."
