# Karpenter – Why We Use It in EKS (Production Guide)

## What is Karpenter?

Karpenter is an **open-source, Kubernetes-native node autoscaler** developed by AWS. It dynamically provisions **EC2 worker nodes** in response to **real-time pod scheduling needs**.

In simple terms:

> Karpenter automatically creates the *right* EC2 instances at the *right time* for pending pods, without pre-defining node groups.

---

## Why Karpenter Was Created

### Problems with Traditional Node Groups + Cluster Autoscaler

Using Managed Node Groups with Cluster Autoscaler:

* Nodes are pre-defined (instance type, size)
* Scaling is slower
* Over-provisioning is common
* Inefficient for bursty workloads

AWS created Karpenter to solve:

* Slow scaling
* Poor bin-packing
* Rigid node definitions

---

## How Karpenter Works (High-Level)

```
Pending Pod
 ↓
Karpenter watches scheduler
 ↓
Chooses best EC2 instance
 ↓
Launches node directly
 ↓
Pod scheduled immediately
```

Key difference:

> Karpenter reacts to **pods**, not **node groups**.

---

## Karpenter Architecture

### Core Components

1. **Karpenter Controller** (runs as a pod)
2. **AWS APIs** (EC2, IAM)
3. **Kubernetes Scheduler**

---

## Core Concepts in Karpenter

### 1️⃣ NodePool (or Provisioner – older term)

Defines:

* Instance requirements
* Capacity type (On-Demand / Spot)
* Limits (CPU, memory)

---

### 2️⃣ EC2NodeClass

Defines:

* AMI
* Subnets
* Security groups
* IAM role

---

## How Karpenter Scales Nodes

### Scale-Up

1. Pod cannot be scheduled
2. Karpenter analyzes pod requirements
3. Selects best EC2 instance type
4. Creates node
5. Pod is scheduled

---

### Scale-Down

* Identifies underutilized nodes
* Evicts pods safely
* Terminates nodes

---

## Karpenter vs Cluster Autoscaler

| Feature            | Cluster Autoscaler | Karpenter  |
| ------------------ | ------------------ | ---------- |
| Scaling trigger    | Node group size    | Pod demand |
| Instance selection | Fixed              | Dynamic    |
| Scaling speed      | Slow               | Very fast  |
| Cost efficiency    | Medium             | High       |
| Spot handling      | Limited            | Excellent  |

---

## Why Karpenter Is Preferred in Production

### 1️⃣ Faster Scaling

* Seconds instead of minutes

### 2️⃣ Better Cost Optimization

* Uses right-sized instances
* Excellent Spot support

### 3️⃣ No Node Group Management

* No pre-defined ASGs
* Less operational overhead

### 4️⃣ Better Bin Packing

* Higher utilization

---

## Production Use Cases for Karpenter

### 1️⃣ Microservices Platforms

* Variable traffic
* Many small pods

---

### 2️⃣ CI/CD & Batch Workloads

* Burst traffic
* Short-lived pods

---

### 3️⃣ Spot-Heavy Clusters

* Aggressive cost savings

---

## When NOT to Use Karpenter

❌ Very small clusters
❌ Strictly fixed infrastructure
❌ Legacy tooling dependent on node groups

---

## Common Production Pattern

```
EKS Cluster
 ├─ System Node Group (small, stable)
 └─ Karpenter (dynamic workloads)
```

---

## Security & IAM Considerations

* Uses IRSA
* Requires EC2 permissions
* Nodes use dedicated IAM roles

---

## Common Mistakes

* No limits on NodePool
* Ignoring pod disruption budgets
* Using Karpenter for system pods

---

## Interview One-Liner (Very Important)

"Karpenter is a Kubernetes-native autoscaler for EKS that provisions EC2 nodes dynamically based on pod requirements. In production, it’s used for fast scaling, better cost optimization, and efficient Spot instance usage compared to traditional node groups."

