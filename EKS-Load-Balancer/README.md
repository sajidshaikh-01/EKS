<img width="1412" height="790" alt="image" src="https://github.com/user-attachments/assets/6c8c2d8d-e98a-47a3-b166-8504b54c3fd4" />

# Load Balancers in Amazon EKS – Architecture & Production Guide

## Why Load Balancers Are Needed in EKS

Kubernetes pods are **ephemeral and private**. In production, we need a reliable way to:

* Expose applications to users
* Distribute traffic
* Handle scaling and failures

In EKS, **AWS load balancers** are used for this purpose.

---

## Types of Load Balancers Used in EKS

EKS integrates with **AWS-managed load balancers**:

1. Application Load Balancer (ALB)
2. Network Load Balancer (NLB)

Classic Load Balancer is deprecated and not used in modern production.

---

## High-Level EKS Load Balancer Architecture

```
User
 ↓
AWS Load Balancer (ALB / NLB)
 ↓
Kubernetes Service / Ingress
 ↓
Pods (Private IPs)
```

Key rule:

> Pods never get public IPs. Load balancers handle public access.

---

## Kubernetes Service Types and Load Balancers

### 1️⃣ ClusterIP

* Default service type
* Internal-only access
* No AWS load balancer created

Production use:

* Internal microservices

---

### 2️⃣ NodePort

* Exposes service on each node IP
* Uses high port range

Production use:

* Rare (mostly testing)

---

### 3️⃣ LoadBalancer

When you create:

```yaml
kind: Service
type: LoadBalancer
```

AWS creates:

* **Network Load Balancer (NLB)** by default

---

## Network Load Balancer (NLB) in EKS

### How NLB Works

* Layer 4 (TCP/UDP)
* Very high performance
* Static IP support

Traffic flow:

```
Client → NLB → Node → Pod
```

### Production Use Cases

* gRPC
* WebSockets
* TCP-based apps
* High throughput systems

---

## Ingress in EKS (Layer 7 Routing)

### What is Ingress?

Ingress provides:

* HTTP/HTTPS routing
* Path-based routing
* Host-based routing

Ingress itself is **just a resource**, not a load balancer.

---

## AWS Load Balancer Controller (ALB Controller)

### What It Does

* Watches Kubernetes Ingress resources
* Automatically provisions ALB
* Manages listeners, rules, and target groups

This controller is **mandatory** for ALB usage in EKS.

---

## Application Load Balancer (ALB) in EKS

### How ALB Works

* Layer 7 (HTTP/HTTPS)
* Integrates with ACM for TLS
* Supports path & host routing

Traffic flow:

```
Client → ALB → Target Group → Pod
```

---

## ALB Target Types

### Instance Mode

* Traffic goes to NodePort
* Node forwards to pod

### IP Mode (Recommended)

* ALB sends traffic directly to pod IP
* Better performance

---

## Production Use Cases for ALB

* Web applications
* REST APIs
* Microservices
* Multi-domain routing

---

## Internal vs Internet-Facing Load Balancers

### Internet-Facing

* Has public IPs
* Accessible from internet

### Internal

* Private IP only
* Used for internal apps

---

## Security in EKS Load Balancers

### Security Groups

* Control inbound/outbound traffic

### TLS Termination

* ALB + ACM certificates
* End-to-end encryption possible

---

## High Availability & Scaling

* Load balancers span multiple AZs
* Health checks remove unhealthy pods
* Auto-scale with traffic

---

## Production Architecture Example

```
Internet
 ↓
ALB (Public Subnets)
 ↓
Ingress → Service
 ↓
Pods (Private Subnets, Multi-AZ)
```

---

## Common Production Mistakes

* Using NodePort in production
* Exposing pods directly
* Not using health checks
* Not enabling TLS

---

## Load Balancer Selection Guide

| Requirement         | Recommended LB |
| ------------------- | -------------- |
| HTTP/HTTPS routing  | ALB            |
| High throughput TCP | NLB            |
| WebSockets          | NLB            |
| Path-based routing  | ALB            |

---

## Interview One-Liner (Very Important)

"In EKS, external traffic is handled using AWS-managed load balancers. NLB is used with Service type LoadBalancer for L4 traffic, while ALB is provisioned using AWS Load Balancer Controller for Ingress-based L7 routing. Pods remain private, and load balancers provide scalability and high availability."
