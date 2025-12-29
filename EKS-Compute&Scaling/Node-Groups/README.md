# Amazon EKS Node Groups – Architecture & Production Guide

## What is an EKS Node Group?

An **EKS Node Group** is a managed way to run and manage **worker nodes (EC2 instances)** that join an Amazon EKS cluster.

Worker nodes are where:

* Pods run
* Containers execute
* Application workloads live

In EKS, AWS provides **Managed Node Groups** to simplify node lifecycle management.

---

## Why Node Groups Are Important

Without node groups:

* You must manually create EC2 instances
* Configure kubelet
* Handle upgrades and replacements

Node groups solve:

* Scaling
* Upgrades
* High availability
* Consistent node configuration

---

## Types of Node Groups in EKS

### 1️⃣ Managed Node Groups (Recommended)

AWS manages:

* EC2 lifecycle
* AMI updates
* Node replacement
* Integration with Auto Scaling Groups

You manage:

* Instance type
* Scaling limits
* Labels & taints

---

### 2️⃣ Self-Managed Node Groups

You manage everything:

* EC2 instances
* AMIs
* Scaling logic

Used when:

* Custom OS required
* Special hardware (GPU, FPGA)

---

### 3️⃣ Fargate (Node-less)

* No EC2 nodes
* Pods run on AWS-managed compute

Used for:

* System workloads
* Batch jobs

---

## Managed Node Group Architecture

```
EKS Control Plane (AWS-managed)
 ↓
Managed Node Group
 ↓
Auto Scaling Group
 ↓
EC2 Worker Nodes
 ↓
Pods
```

---

## How Nodes Join the Cluster

1. EC2 instance launches
2. Bootstrap script runs
3. kubelet starts
4. Node authenticates using IAM role
5. Node registers with EKS control plane

---

## Node IAM Role

Each node group has an **IAM role** that allows:

* Node registration
* Pull images from ECR
* Communicate with AWS APIs

---

## Scaling Node Groups

### Horizontal Scaling

* Increase/decrease node count
* Managed by Auto Scaling Group

### Cluster Autoscaler

* Adds/removes nodes automatically
* Based on pending pods

---

## Labels, Taints, and Workload Isolation

### Node Labels

Used to:

* Schedule specific workloads

Example:

```yaml
nodeSelector:
  node-type: frontend
```

---

### Node Taints

Used to:

* Prevent unwanted pods

Example:

```text
key=dedicated:NoSchedule
```

---

## Production Node Group Patterns

### 1️⃣ Multiple Node Groups

* Frontend workloads
* Backend workloads
* Monitoring workloads

---

### 2️⃣ Spot + On-Demand Mix

* Cost optimization
* Fault tolerance

---

### 3️⃣ AZ-Aware Node Groups

* Nodes spread across multiple AZs
* High availability

---

## Node Group Upgrades (Very Important)

### Managed Node Group Upgrade Flow

1. New AMI launched
2. New nodes join cluster
3. Pods drained from old nodes
4. Old nodes terminated

Best practices:

* Use PodDisruptionBudgets
* Use rolling upgrades

---

## Common Production Mistakes

* Single node group only
* No AZ spread
* No autoscaling
* Running all workloads on one node type

---

## Node Group vs Fargate (Quick Comparison)

| Feature        | Node Group | Fargate |
| -------------- | ---------- | ------- |
| EC2 management | Required   | None    |
| DaemonSets     | ✅          | ❌       |
| Cost control   | Flexible   | Higher  |
| Customization  | High       | Low     |

---

## Interview One-Liner (Very Important)

"An EKS node group is a collection of EC2 worker nodes managed as a unit. In production, we use managed node groups to simplify scaling, upgrades, and high availability, often with multiple node groups for workload isolation and cost optimization."
