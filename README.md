<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/6f503e5e-008f-475b-a672-ed8c654914e3" />

# Amazon EKS – Architecture, Working & Use Cases

## What is Amazon EKS?

Amazon Elastic Kubernetes Service (EKS) is a **managed Kubernetes service** by AWS that allows you to run Kubernetes clusters without managing the Kubernetes control plane yourself.

AWS manages the **control plane**, and you manage the **worker nodes (data plane)**.

---

## EKS High-Level Architecture

### Control Plane (Managed by AWS)

The EKS control plane is fully managed, highly available, and runs across **multiple Availability Zones**.

Components managed by AWS:

* kube-apiserver
* etcd (distributed key-value store)
* kube-scheduler
* kube-controller-manager

Key points:

* Runs in an AWS-owned VPC
* Automatically patched and upgraded
* Highly available by default
* You do NOT get SSH or OS access

---

### Data Plane (Managed by You)

This is where your workloads actually run.

You can choose:

#### 1. EC2 Worker Nodes

* Managed Node Groups
* Self-managed Node Groups

#### 2. AWS Fargate

* Serverless compute for pods
* No EC2 management
* Per-pod pricing

Worker node components:

* kubelet
* kube-proxy
* Container runtime (containerd)
* CNI plugin

---

## Networking Architecture in EKS

### VPC Integration

* EKS uses **AWS VPC CNI plugin** by default
* Each Pod gets a **real VPC IP address**
* Pods communicate directly with AWS services

### Networking Flow

* Pod → Pod (same node): via local networking
* Pod → Pod (different node): via VPC routing
* Pod → Service: kube-proxy handles traffic
* External → Service: via ALB/NLB

### Core Networking Components

* AWS VPC
* Subnets (Public & Private)
* Security Groups
* Route Tables

---

## Authentication & Authorization

### Authentication

* IAM is used for cluster authentication
* aws-auth ConfigMap maps IAM roles/users to Kubernetes RBAC

### Authorization

* Kubernetes RBAC
* IAM Roles for Service Accounts (IRSA)

IRSA flow:

1. Pod uses Kubernetes ServiceAccount
2. ServiceAccount is mapped to IAM Role
3. AWS STS issues temporary credentials
4. Pod accesses AWS services securely

---

## Storage Architecture

### Block Storage (EBS)

* EBS CSI Driver
* ReadWriteOnce
* Used for databases and stateful workloads

### File Storage (EFS)

* EFS CSI Driver
* ReadWriteMany
* Used for shared file systems

Storage is provisioned using:

* StorageClass
* PersistentVolumeClaim (PVC)

---

## Load Balancing & Ingress

### Service Types

* ClusterIP (internal)
* NodePort (testing/debugging)
* LoadBalancer (AWS NLB)

### AWS Load Balancer Controller

* Provisions ALB for Ingress
* Provisions NLB for Services
* Supports TLS, path-based routing, host-based routing

---

## Observability in EKS

### Metrics

* Prometheus
* CloudWatch Container Insights

### Logs

* Fluent Bit → CloudWatch Logs

### Tracing

* AWS X-Ray
* Jaeger (with service mesh)

---

## How EKS Works (Step-by-Step)

1. User creates EKS cluster using eksctl / Terraform / AWS Console
2. AWS provisions Kubernetes control plane
3. Worker nodes join cluster using bootstrap script
4. aws-auth ConfigMap allows nodes to register
5. Pods are scheduled on worker nodes
6. CNI assigns VPC IPs to pods
7. Services and Ingress expose applications
8. IAM + RBAC secure access
9. Monitoring and logging collect telemetry

---

## High Availability & Scaling

* Control plane is multi-AZ by default
* Worker nodes spread across multiple AZs
* Auto Scaling Groups scale nodes
* Cluster Autoscaler scales nodes
* HPA scales pods

---

## EKS Upgrade Strategy

Recommended order:

1. Upgrade EKS control plane
2. Upgrade add-ons (VPC CNI, CoreDNS, kube-proxy)
3. Upgrade worker node groups

Best practices:

* Use rolling updates
* Use PodDisruptionBudgets
* Test upgrades in staging

---

## Common EKS Use Cases

### 1. Microservices Architecture

* Multiple services
* Independent scaling
* Service mesh integration

### 2. CI/CD Platforms

* Jenkins
* GitHub Actions runners
* ArgoCD

### 3. Cloud-Native Applications

* Kubernetes-native workloads
* Autoscaling and self-healing

### 4. Hybrid & Multi-Cloud

* Integrates with on-prem clusters
* GitOps-driven deployments

### 5. Secure Enterprise Workloads

* IRSA for AWS access
* Network Policies
* Policy engines (Kyverno / OPA)

---

## Why Companies Choose EKS

* No control plane management
* Deep AWS integration
* High availability by default
* Strong security model
* Scales for production workloads

---

