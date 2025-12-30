# EKS Upgrade Cycles & Kubernetes Version Upgrades (Production Guide)

## Why Upgrade Cycles Matter in EKS

Kubernetes evolves rapidly. In production, **keeping EKS clusters updated is mandatory** for:

* Security patches
* Bug fixes
* Performance improvements
* AWS support compliance

Running unsupported versions leads to:

* Security risks
* No AWS support
* Forced upgrades later (high risk)

---

## Kubernetes Version Lifecycle in EKS

### Supported Versions

* EKS supports **multiple Kubernetes minor versions** at a time
* Each version is supported for **~14 months**

Example:

* Kubernetes 1.28
* Kubernetes 1.29
* Kubernetes 1.30

Older versions are gradually deprecated.

---

## EKS Upgrade Responsibility Model

| Component                    | Who Manages |
| ---------------------------- | ----------- |
| Control plane infrastructure | AWS         |
| Kubernetes version selection | You         |
| Upgrade timing               | You         |
| Worker node upgrades         | You         |
| Add-on upgrades              | You         |

AWS **never auto-upgrades** your cluster without notice.

---

## Recommended Upgrade Frequency (Production)

Best practice:

* Upgrade **every 6–9 months**
* Never skip more than **one minor version**

Example:

* 1.27 → 1.28 → 1.29

---

## Correct EKS Upgrade Order (Very Important)

1️⃣ Upgrade control plane
2️⃣ Upgrade managed node groups
3️⃣ Upgrade cluster add-ons

⚠️ Upgrading in wrong order can break workloads.

---

## Step 1: Control Plane Upgrade

### What Happens

* AWS upgrades kube-apiserver, scheduler, controller-manager, etcd
* Control plane is highly available (multi-AZ)
* No downtime for API server

### What You Do

* Trigger upgrade via Console / CLI / IaC

---

## Step 2: Node Group Upgrade

### Managed Node Groups

Upgrade process:

1. New nodes launched with new AMI
2. Pods drained from old nodes
3. Old nodes terminated

Best practices:

* Use PodDisruptionBudgets
* Upgrade one node group at a time

---

## Step 3: Add-on Upgrades

Critical add-ons:

* VPC CNI
* CoreDNS
* kube-proxy

Why this matters:

* Version mismatch can cause networking or DNS failures

---

## Upgrade Strategies in Production

### Rolling Upgrade (Most Common)

* Gradual replacement
* Zero downtime

### Blue-Green Cluster Upgrade (Advanced)

* Create new cluster
* Migrate workloads
* Cut traffic

Used for:

* Mission-critical systems

---

## What to Check BEFORE Upgrading

* Deprecated APIs (kubectl api-resources)
* Add-on compatibility
* Application readiness
* Backup & rollback plan

---

## What to Check AFTER Upgrading

* Node readiness
* Pod health
* Application logs
* Autoscaling behavior

---

## Common Upgrade Mistakes

* Skipping minor versions
* Not upgrading add-ons
* No PDBs
* Upgrading during peak traffic

---

## EKS Add-on Version Skew Rules

* kubelet ≤ API server version
* Add-ons must be compatible with control plane

AWS validates supported combinations.

---

## Production Upgrade Timeline Example

```
Month 0   → Cluster created (v1.28)
Month 6   → Upgrade to v1.29
Month 12  → Upgrade to v1.30
```

---

## Interview One-Liner (Very Important)

"EKS upgrades follow a controlled cycle where customers choose the Kubernetes version and upgrade timing. In production, we first upgrade the control plane, then node groups, and finally cluster add-ons, following Kubernetes version skew rules to ensure zero downtime."
