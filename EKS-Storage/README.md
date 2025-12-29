# Amazon EKS Storage – Architecture & Production Guide

## What is EKS Storage?

EKS storage defines **how Kubernetes workloads running on EKS persist data** beyond pod lifecycle. Since pods are ephemeral, **external storage is mandatory in production**.

EKS storage is implemented using **Kubernetes CSI (Container Storage Interface)** drivers integrated with AWS storage services.

---

## High-Level EKS Storage Architecture

EKS storage has four main layers:

1. Application / Pod
2. PersistentVolumeClaim (PVC)
3. PersistentVolume (PV)
4. AWS Storage Backend (EBS / EFS / S3 via CSI)

---

## Why Storage is Critical in Production

Without persistent storage:

* Pod restart = data loss
* Node failure = data loss
* Scaling = inconsistent state

Production workloads that require storage:

* Databases
* Message queues
* CI/CD tools
* Logs & artifacts

---

## Kubernetes Storage Primitives (Core Concepts)

### 1️⃣ Volume

* Mounted inside a pod
* Pod lifecycle dependent

---

### 2️⃣ PersistentVolume (PV)

* Actual storage resource
* Backed by AWS storage

---

### 3️⃣ PersistentVolumeClaim (PVC)

* Request for storage
* Abstracts storage details from developers

---

### 4️⃣ StorageClass

* Defines **how storage is provisioned**
* Enables dynamic provisioning

---

## CSI (Container Storage Interface)

### What is CSI?

CSI is a standard interface that allows Kubernetes to use **external storage systems**.

EKS uses **AWS-managed CSI drivers**:

* EBS CSI Driver
* EFS CSI Driver

---

## 1️⃣ Amazon EBS Storage in EKS

### What is EBS?

Elastic Block Store provides **block-level storage** attached to EC2 instances.

---

### How EBS Works in EKS

1. Pod requests PVC
2. StorageClass uses EBS CSI Driver
3. EBS volume is dynamically created
4. Volume attaches to node
5. Volume is mounted into pod

---

### EBS Access Mode

* ReadWriteOnce (RWO)
* One node at a time

---

### Production Use Cases for EBS

* Databases (MySQL, PostgreSQL)
* StatefulSets
* Message brokers

---

### EBS Production Architecture

* One EBS volume per pod
* Pod scheduled in same AZ as volume
* Volume detached/attached during rescheduling

---

## 2️⃣ Amazon EFS Storage in EKS

### What is EFS?

Elastic File System provides **shared file storage** accessible by multiple nodes.

---

### How EFS Works in EKS

1. EFS filesystem created
2. EFS CSI Driver installed
3. PVC requests shared storage
4. Pods mount EFS via NFS

---

### EFS Access Mode

* ReadWriteMany (RWX)

---

### Production Use Cases for EFS

* Shared configuration
* Media storage
* CI/CD workspaces
* Multiple replicas writing data

---

## 3️⃣ Amazon S3 in EKS (Object Storage)

### Important Note

S3 is **NOT a POSIX filesystem**.

Used via:

* Application SDK
* CSI drivers (special cases)

---

### Production Use Cases for S3

* Logs
* Backups
* Artifacts
* Data lakes

---

## StorageClass – Dynamic Provisioning

### Why StorageClass Matters

* No manual volume creation
* Standardized storage behavior

Example parameters:

* Volume type (gp3)
* Encryption
* Performance

---

## StatefulSets & Storage

### Why StatefulSets?

* Stable network identity
* Stable storage association

Each replica:

* Gets its own PVC
* PVC survives pod restart

---

## Multi-AZ & High Availability

### EBS

* AZ-bound
* Pod must run in same AZ

### EFS

* Regional
* Multi-AZ access

Production decision:

* Databases → EBS
* Shared workloads → EFS

---

## Backup & Disaster Recovery

### Common Strategies

* EBS snapshots
* Velero backups
* S3-based backups

---

## Security Best Practices

* Encrypt volumes at rest
* Use IAM Roles for Service Accounts (IRSA)
* Restrict access using security groups

---

## Common Production Mistakes

* Using emptyDir for stateful apps
* Using EBS for multi-writer apps
* Ignoring AZ constraints

---

## Production Storage Decision Matrix

| Workload     | Recommended Storage |
| ------------ | ------------------- |
| Database     | EBS                 |
| CI/CD        | EFS                 |
| Logs         | S3                  |
| Shared files | EFS                 |

---

## Interview One-Liner (Very Important)

"In EKS, persistent storage is implemented using CSI drivers like EBS and EFS. EBS is used for single-writer stateful workloads, while EFS provides shared storage across multiple pods and AZs. Storage is dynamically provisioned using StorageClasses in production."

