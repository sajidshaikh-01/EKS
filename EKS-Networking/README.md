# Amazon EKS Networking – Deep Dive (Production Focus)

## What is EKS Networking?

EKS networking defines **how pods, services, nodes, and external users communicate** with each other inside and outside the Kubernetes cluster running on AWS.

EKS networking is tightly integrated with **AWS VPC**, which is the biggest difference compared to self-managed Kubernetes.

---

## High-Level EKS Networking Architecture

EKS networking is built on these layers:

1. AWS VPC (foundation)
2. Subnets (IP allocation)
3. AWS VPC CNI plugin (pod networking)
4. Kubernetes Services (traffic abstraction)
5. kube-proxy (service routing)
6. Load balancers (ALB / NLB)
7. DNS (CoreDNS)
8. Security layers (Security Groups, NACLs, Network Policies)

---

## 1️⃣ Amazon VPC (Foundation Layer)

### What it does

* Provides isolated networking environment
* Defines IP address range using CIDR

### Production setup

* One VPC per environment (dev / stage / prod)
* CIDR sized properly (e.g., /16 or /18)

### Why important in EKS

* Pods and nodes live inside the VPC
* Native AWS service connectivity

---

## 2️⃣ Subnets (IP Management)

### Types of subnets

* Public subnets → Load balancers
* Private subnets → Worker nodes & pods

### Production best practice

* Use **private subnets for nodes**
* Spread subnets across **multiple AZs**

### Why this matters

* High availability
* Better security

---

## 3️⃣ AWS VPC CNI Plugin (Core of EKS Networking)

### What it is

The AWS VPC CNI plugin is responsible for assigning **VPC IP addresses directly to pods**.

### How it works

* Each node gets one or more ENIs
* Each ENI has multiple secondary IPs
* Each pod gets one secondary IP

### Result

* Pod = First-class citizen in VPC
* No NAT or overlay networking

### Production use case

* Pods can directly access:

  * RDS
  * S3
  * DynamoDB

---

## 4️⃣ Elastic Network Interface (ENI)

### What ENI does

* Network interface attached to EC2
* Controls IP capacity per node

### Important limitation

* Pod count per node depends on instance type

### Production impact

* Choose instance types carefully
* Avoid IP exhaustion

---

## 5️⃣ Pod-to-Pod Networking

### Same Node

* Traffic stays on the node
* No VPC routing involved

### Different Node

* Routed via VPC
* Uses pod IPs directly

### Production example

* Microservices communication
* Low latency

---

## 6️⃣ Kubernetes Services (Traffic Abstraction)

### Service Types

#### ClusterIP

* Internal-only access
* Default service type

Production use:

* Backend APIs
* Internal microservices

---

#### NodePort

* Exposes service on node IP
* Mostly for debugging

Production use:

* Rarely used

---

#### LoadBalancer

* Creates AWS NLB

Production use:

* TCP/UDP workloads
* High performance traffic

---

## 7️⃣ kube-proxy (Service Routing)

### What it does

* Implements service abstraction
* Uses iptables or IPVS

### Traffic flow

* Service IP → Pod IP

### Production importance

* Enables stable service access

---

## 8️⃣ Ingress & AWS Load Balancer Controller

### Ingress

* L7 HTTP/HTTPS routing
* Path-based and host-based routing

### AWS Load Balancer Controller

* Creates ALB automatically
* Integrates with ACM for TLS

### Production use case

* Public-facing applications
* Multi-domain routing

---

## 9️⃣ DNS – CoreDNS

### What CoreDNS does

* Resolves service names to IPs

### Example

```text
service-name.namespace.svc.cluster.local
```

### Production use

* Service discovery
* Decoupled microservices

---

## 🔟 Security Layers in EKS Networking

### Security Groups

* Node-level firewall
* Controls inbound/outbound traffic

### NACLs

* Subnet-level security

### Network Policies

* Pod-level traffic control
* Implemented using CNI like Cilium

### Production use

* Zero-trust networking
* Restrict lateral movement

---

## 1️⃣1️⃣ External Traffic Flow (End-to-End)

Example: User accessing web app

1. User → ALB
2. ALB → Node
3. Node → Service
4. Service → Pod
5. Pod → Backend service

---

## 1️⃣2️⃣ Internal Traffic Flow Example

Microservice A → Microservice B

1. Pod A calls service B
2. CoreDNS resolves service
3. kube-proxy routes traffic
4. Pod B receives request

---

## 1️⃣3️⃣ Common Production Networking Problems

* IP exhaustion
* Misconfigured security groups
* DNS resolution failure
* Incorrect ingress annotations

---

## 1️⃣4️⃣ EKS Networking Best Practices

* Use private subnets for nodes
* Plan CIDR carefully
* Monitor IP usage
* Use IRSA instead of NAT where possible
* Use Network Policies

---

## Interview One-Liner

"EKS networking is built on AWS VPC using the VPC CNI plugin, where each pod receives a real VPC IP address, enabling native AWS integration, high performance, and secure production-grade networking."

