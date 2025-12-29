# Secrets Management in Amazon EKS – Production Guide

## What Are Secrets in EKS?

In Amazon EKS, **secrets are used to store sensitive information** required by applications, such as:

* Database passwords
* API keys
* Tokens
* Certificates

Kubernetes Secrets are objects that store this data in **base64-encoded form** and make it available to pods securely.

---

## Why Secrets Are Important in Production

Without proper secret management:

* Secrets may be hardcoded in images or YAMLs ❌
* Accidental leaks via GitHub ❌
* No rotation or audit ❌

Production-grade secret management ensures:

* Security
* Least privilege
* Rotation
* Auditability

---

## Default Kubernetes Secret Types

### 1️⃣ Opaque Secrets

**Most common type** of secret.

Used for:

* DB credentials
* API keys
* Application secrets

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: ZGJ1c2Vy
  password: cGFzc3dvcmQ=
```

---

### 2️⃣ docker-registry Secret

Used for:

* Pulling images from private registries

Example use case:

* Private Docker Hub
* Private ECR (less common, ECR uses IAM)

---

### 3️⃣ TLS Secrets

Used for:

* HTTPS certificates
* Ingress TLS

Example use case:

* ALB Ingress + TLS
* Internal mTLS

---

### 4️⃣ ServiceAccount Token Secrets

Used for:

* Pod authentication to Kubernetes API
* IRSA integration

These are **auto-managed by Kubernetes**.

---

## How Kubernetes Secrets Work Internally

1. Secret is stored in **etcd**
2. Data is base64 encoded (not encrypted by default)
3. Secret is mounted or injected into pod
4. Application reads secret at runtime

---

## ⚠️ Important Security Limitation

> Kubernetes Secrets are **NOT encrypted by default** in etcd.

In production, this is **not sufficient alone**.

---

## Production-Grade Secret Management in EKS

### 1️⃣ Enable Encryption at Rest (Minimum)

* Use AWS KMS to encrypt etcd secrets
* EKS supports envelope encryption

Use case:

* Compliance requirement

---

### 2️⃣ Environment Variable Injection

Secrets can be injected as:

* Environment variables
* Mounted volumes

Production preference:

* Environment variables for small secrets
* Volume mount for certs

---

## External Secret Management (Production Standard)

Most companies **do NOT store secrets directly in Kubernetes**.

Instead they use:

### 1️⃣ AWS Secrets Manager

Used for:

* Database passwords
* API keys
* Rotated secrets

Benefits:

* Automatic rotation
* IAM-based access
* Audit logs

---

### 2️⃣ AWS Parameter Store (SSM)

Used for:

* App configuration
* Non-rotating secrets

Cheaper alternative to Secrets Manager.

---

## External Secrets Operator (ESO)

### What ESO Does

* Syncs secrets from AWS Secrets Manager / SSM
* Creates Kubernetes Secrets dynamically

Flow:

1. App requests secret
2. ESO fetches from AWS
3. Kubernetes Secret is created
4. Pod consumes secret

---

## IAM Roles for Service Accounts (IRSA)

IRSA is **mandatory in production**.

Used for:

* Allowing pods to access secrets securely
* No static AWS credentials in pods

Flow:

1. Pod uses ServiceAccount
2. ServiceAccount mapped to IAM role
3. IAM role grants secret access

---

## Sealed Secrets (Alternative Approach)

### What Are Sealed Secrets?

* Encrypt secrets using public key
* Safe to store in Git

Use case:

* GitOps workflows
* No external secret store

Limitation:

* No automatic rotation

---

## Production Secret Strategy (Real World)

Most common pattern:

* AWS Secrets Manager → source of truth
* IRSA → access control
* External Secrets Operator → sync
* Kubernetes Secrets → runtime consumption

---

## What NOT to Do in Production ❌

* Hardcode secrets in YAML
* Commit secrets to Git
* Share AWS keys
* Use plaintext ConfigMaps

---

## Interview One-Liner (Very Important)

"In EKS production environments, we avoid storing secrets directly in Kubernetes. Instead, we use AWS Secrets Manager or Parameter Store with IRSA and External Secrets Operator to securely inject secrets into pods with rotation and auditability."

