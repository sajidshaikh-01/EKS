<img width="1407" height="550" alt="image" src="https://github.com/user-attachments/assets/aba7bce3-a5fe-4abd-bf56-b62b49ce08f9" />


# IRSA (IAM Roles for Service Accounts) – Architecture & Production Guide

## What is IRSA?

IRSA (IAM Roles for Service Accounts) is an AWS feature that allows **Kubernetes pods running in EKS to securely access AWS services using IAM roles**, without storing AWS access keys inside pods.

In simple terms:

> IRSA lets a pod assume an IAM role **based on its Kubernetes ServiceAccount**.

---

## Why IRSA Is Required in Production

Before IRSA, pods accessed AWS using:

* IAM user access keys (hardcoded ❌)
* Node IAM role (over-privileged ❌)

Problems:

* Security risk
* No least privilege
* Hard rotation

IRSA solves this by:

* Eliminating static credentials
* Enforcing least privilege per pod
* Using short-lived credentials

---

## High-Level IRSA Architecture

```
Pod
 ↓ (uses ServiceAccount)
Kubernetes API
 ↓
OIDC Token (JWT)
 ↓
AWS STS AssumeRoleWithWebIdentity
 ↓
Temporary IAM Credentials
 ↓
AWS Service (S3, DynamoDB, etc.)
```

---

## Core Components of IRSA

### 1️⃣ EKS OIDC Provider

* EKS exposes an OIDC issuer URL
* Trusted by AWS IAM

---

### 2️⃣ IAM Role

* Has trust policy for OIDC provider
* Allows specific ServiceAccount

---

### 3️⃣ Kubernetes ServiceAccount

* Annotated with IAM role ARN

---

### 4️⃣ AWS STS

* Issues short-lived credentials

---

## How IRSA Works (Step-by-Step)

1. Pod starts using a ServiceAccount
2. Kubernetes mounts a projected OIDC token into the pod
3. Pod SDK calls AWS STS
4. STS validates OIDC token
5. IAM role is assumed
6. Temporary credentials returned
7. Pod accesses AWS service

---

## IRSA vs Node IAM Role (Very Important)

| Feature         | Node IAM Role    | IRSA               |
| --------------- | ---------------- | ------------------ |
| Scope           | All pods         | Per ServiceAccount |
| Security        | Over-privileged  | Least privilege    |
| Credential type | Instance profile | Web identity       |
| Production use  | ❌ Avoid          | ✅ Recommended      |

---

## How to Create IRSA (Conceptual Steps)

### Step 1: Enable OIDC Provider

* EKS cluster exposes OIDC URL
* IAM trusts this provider

---

### Step 2: Create IAM Role

Trust policy allows:

* OIDC provider
* Specific ServiceAccount

---

### Step 3: Create Kubernetes ServiceAccount

Annotate with:

* IAM role ARN

---

### Step 4: Use ServiceAccount in Pod

Pod automatically receives IAM credentials.

---

## Example Trust Relationship (Conceptual)

Conditions:

* aud = sts.amazonaws.com
* sub = system:serviceaccount:<namespace>:<serviceaccount>

---

## Production Use Cases for IRSA

### 1️⃣ Accessing AWS Services

* S3
* DynamoDB
* SQS
* Secrets Manager

---

### 2️⃣ External Secrets Operator

* Secure secret retrieval

---

### 3️⃣ CI/CD Controllers

* ArgoCD
* GitHub runners

---

## Security Best Practices

* One IAM role per ServiceAccount
* Least privilege IAM policies
* Avoid wildcard permissions
* Rotate automatically via STS

---

## Common Mistakes

* Using node IAM role for pods
* Sharing one IRSA role across many apps
* Incorrect trust policy conditions

---

## Interview One-Liner (Very Important)

"IRSA allows EKS pods to assume IAM roles using Kubernetes ServiceAccounts via OIDC and AWS STS, enabling secure, least-privilege access to AWS services without storing static credentials."
