<img width="1412" height="789" alt="image" src="https://github.com/user-attachments/assets/2e53eb83-1480-4734-aa69-56fd66000fb5" />


# EKS Cluster Access & kubeconfig – Production Guide

## What is Cluster Access in EKS?

Cluster access in Amazon EKS defines **who can connect to the Kubernetes API server and what actions they are allowed to perform**.

In EKS, access control is a combination of:

1. AWS IAM (Authentication)
2. Kubernetes RBAC (Authorization)
3. kubeconfig (Client configuration)

---

## High-Level Cluster Access Flow

```
User / CI-CD / Tool
 ↓
kubectl
 ↓
kubeconfig
 ↓
AWS IAM Authentication
 ↓
EKS API Server
 ↓
Kubernetes RBAC Authorization
```

---

## 1️⃣ Authentication in EKS (Who You Are)

### How Authentication Works

EKS uses **AWS IAM** for authentication.

* Users authenticate using:

  * IAM User
  * IAM Role
  * STS assumed role

Kubernetes does NOT store users internally.

---

### AWS IAM Authenticator

When you run:

```bash
kubectl get pods
```

Internally:

1. kubectl uses AWS credentials
2. AWS STS generates a temporary token
3. Token is sent to EKS API server
4. API server validates IAM identity

---

## 2️⃣ Authorization in EKS (What You Can Do)

### Kubernetes RBAC

After authentication, EKS checks **RBAC rules**:

* Roles / ClusterRoles
* RoleBindings / ClusterRoleBindings

Example permissions:

* Read-only access
* Namespace admin
* Cluster admin

---

## aws-auth ConfigMap (Very Important)

The **aws-auth ConfigMap** maps:

* IAM users
* IAM roles

To:

* Kubernetes users & groups

Without this mapping:

* Even valid IAM users cannot access the cluster

---

## 3️⃣ kubeconfig (How Clients Connect)

### What is kubeconfig?

kubeconfig is a **client-side configuration file** used by kubectl and other Kubernetes tools to connect to a cluster.

Default location:

```text
~/.kube/config
```

---

## What kubeconfig Contains

A kubeconfig file contains:

### 1. Cluster Information

* API server endpoint
* Certificate authority

### 2. User Information

* Authentication method
* Token or exec plugin

### 3. Context

* Cluster + user + namespace

---

## kubeconfig Structure (Conceptual)

```
clusters:
- cluster:
    server: https://<eks-endpoint>
  name: eks-cluster
users:
- name: eks-user
contexts:
- context:
    cluster: eks-cluster
    user: eks-user
  name: eks-context
current-context: eks-context
```

---

## How kubeconfig Works with EKS

EKS does NOT store credentials in kubeconfig.

Instead:

* kubeconfig uses an **exec plugin**
* Plugin calls AWS CLI
* AWS CLI generates a short-lived token

This ensures:

* No long-lived credentials
* Secure access

---

## Generating kubeconfig for EKS

Typical command:

```bash
aws eks update-kubeconfig --region <region> --name <cluster-name>
```

This command:

* Fetches cluster endpoint
* Configures exec authentication
* Updates kubeconfig file

---

## Cluster Endpoint Access

### Public Endpoint

* Accessible from internet
* Protected by IAM + RBAC

### Private Endpoint (Production Preferred)

* Accessible only inside VPC
* Requires VPN / Bastion / Direct Connect

---

## Production Access Patterns

### Human Access

* Engineers assume IAM roles
* Temporary access via STS

### CI/CD Access

* GitHub Actions / Jenkins
* IAM roles via OIDC

### Application Access

* Uses ServiceAccounts + IRSA

---

## Security Best Practices (Production)

* Use private cluster endpoint
* Avoid IAM users
* Use IAM roles only
* Least privilege RBAC
* Audit access regularly

---

## Common Mistakes

* Giving cluster-admin to everyone
* Using IAM users in production
* Committing kubeconfig to Git
* Leaving public endpoint open

---

## Interview One-Liner (Very Important)

"In EKS, cluster access is controlled using AWS IAM for authentication and Kubernetes RBAC for authorization. kubeconfig is a client-side file that uses AWS exec authentication to securely obtain short-lived tokens to access the EKS API server."
