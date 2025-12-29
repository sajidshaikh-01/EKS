# Ingress vs Gateway API vs Service Mesh (Production Guide)

## Why This Topic Matters

In Kubernetes (and EKS), **traffic management is layered**. Many engineers get confused because **Ingress, Gateway API, and Service Mesh solve different problems**, even though they touch networking.

Understanding **when to use what in production** is a **senior DevOps / SRE skill**.

---

## Big Picture (Mental Model)

```
User Traffic (North-South)
 ├─ Ingress / Gateway API
 │
Internal Traffic (East-West)
 ├─ Service Mesh
```

* **Ingress / Gateway API** → Entry point into the cluster
* **Service Mesh** → Traffic control *inside* the cluster

They are **not replacements** for each other.

---

## 1️⃣ Kubernetes Ingress

### What is Ingress?

Ingress is a Kubernetes API object that defines **HTTP/HTTPS routing rules** to expose services outside the cluster.

Ingress itself is **not a load balancer**.
It requires an **Ingress Controller** (NGINX, ALB, Traefik, etc.).

---

### What Ingress Solves

* Basic HTTP/HTTPS routing
* Path-based routing
* Host-based routing
* TLS termination

---

### Production Usage of Ingress

Used when:

* Simple web applications
* Basic REST APIs
* Minimal traffic control needs

Very common in:

* Small to mid-size clusters
* Early-stage platforms

---

### Limitations of Ingress

❌ HTTP only (no TCP/UDP standard)
❌ Limited traffic shaping
❌ Annotations-based configuration
❌ Vendor-specific behavior

---

## 2️⃣ Kubernetes Gateway API

### What is Gateway API?

Gateway API is the **next-generation replacement for Ingress**.

It is:

* More expressive
* More extensible
* Role-oriented (infra vs app teams)

---

### Key Gateway API Resources

* **GatewayClass** – defines implementation (ALB, Istio, etc.)
* **Gateway** – defines entry point (LB, listeners)
* **HTTPRoute / TCPRoute** – routing rules

---

### What Gateway API Solves

* HTTP, HTTPS, TCP, UDP routing
* Cleaner API (no annotations)
* Better multi-team ownership
* Cloud-native & portable

---

### Production Usage of Gateway API

Used when:

* Large organizations
* Platform teams
* Multiple app teams
* Need long-term standard

Increasing adoption in:

* EKS
* GKE
* AKS

---

### Why Gateway API Is Replacing Ingress

Ingress problems solved:

* Poor extensibility
* Annotation overload
* Hard ownership separation

Gateway API fixes all of these.

---

## 3️⃣ Service Mesh (Istio / Linkerd / Cilium Mesh)

### What is a Service Mesh?

A service mesh manages **east–west traffic (service-to-service)** inside the cluster.

It works using:

* Sidecar proxies (Istio, Linkerd)
* eBPF (Cilium service mesh)

---

### What Service Mesh Solves

* mTLS between services
* Traffic shifting (canary, blue/green)
* Retries, timeouts, circuit breaking
* Deep observability (metrics, traces)
* Zero-trust security

---

### Production Usage of Service Mesh

Used when:

* Large microservice architectures
* Strong security requirements
* SRE-level observability needed
* Complex traffic management

Common in:

* Fintech
* SaaS
* Large-scale platforms

---

### Cost & Complexity Tradeoff

⚠️ Service mesh adds:

* Latency
* Operational complexity
* Resource overhead

Not every cluster needs it.

---

## Comparison Table (Very Important)

| Feature           | Ingress     | Gateway API    | Service Mesh |
| ----------------- | ----------- | -------------- | ------------ |
| Traffic direction | North-South | North-South    | East-West    |
| Protocols         | HTTP/HTTPS  | HTTP, TCP, UDP | Any          |
| TLS               | Basic       | Advanced       | mTLS         |
| Traffic splitting | Limited     | Yes            | Advanced     |
| Observability     | Basic       | Basic          | Deep         |
| Security          | Edge only   | Edge only      | Zero-trust   |
| Complexity        | Low         | Medium         | High         |

---

## How They Are Used Together in Production

### Most Common Real-World Setup

```
Internet
 ↓
ALB / Gateway API
 ↓
Ingress / Gateway
 ↓
Services
 ↓
Service Mesh (Istio / Cilium)
```

* **Ingress or Gateway API** → cluster entry
* **Service Mesh** → internal traffic control

---

## What Most Companies Choose Today

### Small / Medium Teams

* Ingress (ALB / NGINX)

### Growing Platforms

* Gateway API (future-proof)

### Large / Regulated Systems

* Gateway API + Service Mesh

---

## Common Misconceptions ❌

* ❌ Ingress replaces service mesh
* ❌ Service mesh replaces ingress
* ❌ Gateway API is service mesh

All are **wrong**.

---

## Interview One-Liners (Memorize 🔥)

**Ingress:**

> "Ingress is used to expose HTTP services outside the cluster using an ingress controller."

**Gateway API:**

> "Gateway API is the evolution of Ingress, providing a more expressive and role-oriented way to manage north-south traffic."

**Service Mesh:**

> "A service mesh manages secure, observable, and resilient service-to-service communication inside the cluster."

---

