# Network Policies in Amazon EKS & Cilium Network Policies (Production Guide)

## Why Network Policies Matter in EKS

By default, **Kubernetes allows all pod-to-pod communication**. In production, this is **not secure**.

Network policies are used to:

* Restrict pod-to-pod traffic
* Implement zero-trust networking
* Reduce blast radius during security incidents
* Meet compliance requirements

---

## Important Prerequisite (Very Important)

👉 **EKS does NOT enforce network policies by default**.

Reason:

* AWS VPC CNI **does not implement NetworkPolicy enforcement**

To use network policies in EKS, you must install:

* Cilium
* Calico
* Other policy-capable CNI

---

## Kubernetes Network Policies (Standard)

### What is a Kubernetes NetworkPolicy?

A Kubernetes NetworkPolicy is a **Layer 3 / Layer 4 firewall rule** for pods.

It controls:

* Which pods can talk to which pods
* On which ports
* Using labels

---

## How Kubernetes Network Policies Work

1. You define a NetworkPolicy
2. CNI plugin enforces it
3. Traffic is allowed or denied

⚠️ Without a supporting CNI, policies do nothing.

---

## Kubernetes NetworkPolicy Example

### Default Deny All Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: app
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Effect:

* No pod can receive traffic unless explicitly allowed

---

### Allow Traffic from Frontend to Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: app
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## Limitations of Kubernetes Network Policies

❌ No L7 awareness (HTTP/DNS)
❌ No request-based rules
❌ Hard to debug
❌ No observability

---

## Cilium Network Policies (Advanced)

### What is Cilium Network Policy?

Cilium uses **eBPF** to enforce **identity-based L3–L7 network policies**.

It provides:

* Layer 3–7 policies
* DNS-aware rules
* HTTP method/path rules
* Better performance

---

## How Cilium Policies Work

* Pods get a **Cilium identity** based on labels
* Policies are enforced using eBPF
* No iptables overhead

---

## CiliumNetworkPolicy (CNP) Example

### Allow HTTP GET Only from Frontend

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: backend-http-policy
  namespace: app
spec:
  endpointSelector:
    matchLabels:
      app: backend
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "80"
        protocol: TCP
      rules:
        http:
        - method: "GET"
          path: "/health"
```

---

## DNS-Aware Cilium Policy Example

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: allow-dns
  namespace: app
spec:
  endpointSelector: {}
  egress:
  - toEndpoints:
    - matchLabels:
        k8s-app: kube-dns
    toPorts:
    - ports:
      - port: "53"
        protocol: UDP
```

---

## Kubernetes NetworkPolicy vs Cilium Policy

| Feature        | K8s NetworkPolicy | Cilium Policy |
| -------------- | ----------------- | ------------- |
| Layer          | L3 / L4           | L3–L7         |
| DNS rules      | ❌                 | ✅             |
| HTTP rules     | ❌                 | ✅             |
| Identity-based | ❌                 | ✅             |
| Observability  | ❌                 | ✅             |
| Performance    | Medium            | High (eBPF)   |

---

## Production Usage Pattern in EKS

### Most Common Setup

* AWS VPC CNI → IP allocation
* Cilium → Policy enforcement & observability

---

## Real Production Use Cases

### 1️⃣ Zero Trust Microservices

* Only required services can communicate

### 2️⃣ Sensitive Workloads

* Databases accessible only from app pods

### 3️⃣ Compliance Environments

* PCI-DSS, SOC2, HIPAA

### 4️⃣ Multi-Team Clusters

* Namespace-level isolation

---

## Best Practices for Network Policies

* Start with default deny
* Allow explicitly
* Use labels carefully
* Test in staging
* Monitor dropped packets

---

## Common Mistakes

* Assuming EKS enforces policies by default
* Not allowing DNS traffic
* Overly broad selectors

---

## Interview One-Liner (Very Important)

"In EKS, network policies are not enforced by default. We use Cilium to implement eBPF-based L3–L7 network policies, enabling zero-trust pod communication and deep observability in production."

