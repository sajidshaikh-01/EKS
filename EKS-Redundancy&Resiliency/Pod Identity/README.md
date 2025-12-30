<img width="1414" height="791" alt="image" src="https://github.com/user-attachments/assets/f85c7d6c-8b73-4e6a-bdfd-433c1bc95a5e" />

# EKS Pod Identity vs IRSA – How Pod Identity Works in Production

## What is Pod Identity in EKS?

Pod Identity is a **newer AWS-native way** for Kubernetes pods in EKS to access AWS services **without using OIDC, trust policies, or ServiceAccount annotations**.

In simple words:

> Pod Identity gives a pod an AWS IAM role directly using an AWS-managed identity agent.

---

## Why Pod Identity Was Introduced

IRSA works well but has challenges:

* Complex OIDC trust policies
* Hard to debug
* Annotation-based configuration

AWS introduced **EKS Pod Identity** to:

* Simplify IAM access for pods
* Reduce configuration complexity
* Improve security and isolation

---

## High-Level Pod Identity Architecture

```
Pod
 ↓
EKS Pod Identity Agent (on node)
 ↓
AWS EKS Auth Service
 ↓
AWS STS
 ↓
Temporary IAM Credentials
 ↓
AWS Services
```

---

## Core Components of Pod Identity

### 1️⃣ EKS Pod Identity Agent

* Runs as a daemon on worker nodes
* Intercepts pod credential requests

---

### 2️⃣ Pod Identity Association

* Maps Kubernetes ServiceAccount → IAM Role
* Managed by AWS API

---

### 3️⃣ AWS STS

* Issues short-lived credentials

---

## How Pod Identity Works (Step-by-Step)

1. Pod starts using a ServiceAccount
2. Pod Identity Agent detects the pod
3. Agent requests credentials from AWS
4. AWS validates association
5. STS issues temporary credentials
6. Pod accesses AWS services

---

## How to Create Pod Identity (Conceptual Steps)

1. Enable Pod Identity on EKS cluster
2. Install Pod Identity Agent (AWS-managed)
3. Create IAM role with permissions
4. Create Pod Identity Association
5. Use ServiceAccount in pod

No OIDC provider or trust policy required.

---

## What is IRSA (Quick Recap)

IRSA (IAM Roles for Service Accounts):

* Uses OIDC federation
* Pod assumes role via STS
* Requires trust policy and annotations

---

## IRSA Architecture (Recap)

```
Pod
 ↓ (JWT token)
OIDC Provider
 ↓
AWS STS
 ↓
Temporary Credentials
```

---

## Pod Identity vs IRSA (Key Differences)

| Feature                   | IRSA            | Pod Identity      |
| ------------------------- | --------------- | ----------------- |
| Identity mechanism        | OIDC federation | AWS-managed agent |
| Trust policy              | Required        | Not required      |
| ServiceAccount annotation | Required        | Not required      |
| Complexity                | Higher          | Lower             |
| Debugging                 | Harder          | Easier            |
| AWS support               | Mature          | Newer             |

---

## Which One Is Used in Production?

### Current Reality (Important)

* **IRSA** → Widely used in production today
* **Pod Identity** → Newer, growing adoption

---

## Production Recommendation

### Use IRSA when:

* Existing clusters already use IRSA
* Need proven, battle-tested solution
* Using older EKS versions

### Use Pod Identity when:

* New EKS clusters
* Want simpler setup
* Fewer IAM mistakes

---

## Security Comparison

Both provide:

* No static credentials
* Short-lived STS tokens
* Least privilege

Pod Identity reduces risk of:

* Misconfigured trust policies

---

## Migration Strategy (Real World)

Most companies:

* Continue using IRSA
* Slowly adopt Pod Identity for new services

---

## Interview One-Liner (Very Important)

"IRSA uses OIDC federation to allow ServiceAccounts to assume IAM roles, while EKS Pod Identity uses an AWS-managed agent to assign IAM roles to pods directly. In production today, IRSA is more common, but Pod Identity is preferred for new clusters due to its simpler and safer model."

