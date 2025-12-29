# Prefix Delegation in Amazon EKS (AWS VPC CNI)

## What is Prefix Delegation?

Prefix Delegation is a **feature of the AWS VPC CNI plugin** that allows an EC2 node to receive **CIDR prefixes (/28)** instead of individual secondary IP addresses. These prefixes are then used to assign IPs to pods.

In simple words:

> Instead of assigning pod IPs **one-by-one**, AWS assigns a **block of IPs** to the node, and the node manages pod IP allocation locally.

---

## Why Prefix Delegation Was Introduced

### Problem Without Prefix Delegation

Traditional AWS VPC CNI behavior:

* Pods consume **secondary IPs on ENIs**
* ENI + IP limits depend on instance type
* Large clusters face:

  * IP exhaustion
  * Slow pod scaling
  * Frequent ENI API calls

Example:

* m5.large → limited ENIs and IPs
* Scaling pods quickly = API throttling

---

## What Changes With Prefix Delegation

### With Prefix Delegation Enabled

* Node receives **/28 IPv4 prefix** (16 IPs)
* Prefix is attached to ENI
* Pods get IPs from the prefix
* No AWS API call for each pod

Result:

* Faster pod startup
* Higher pod density per node
* Better scalability

---

## How Prefix Delegation Works (Step-by-Step)

1. EKS node starts
2. AWS VPC CNI requests a **/28 prefix**
3. AWS assigns prefix to ENI
4. CNI stores prefix locally
5. Pods get IPs from the prefix
6. When prefix is exhausted, a new prefix is requested

---

## IP Comparison (Before vs After)

### Without Prefix Delegation

* 1 pod = 1 secondary IP
* Limited by ENI IP limits
* High AWS API usage

### With Prefix Delegation

* 1 prefix = 16 pod IPs
* Fewer ENIs required
* Reduced AWS API calls

---

## Production Benefits of Prefix Delegation

### 1. Higher Pod Density

* Run more pods per node
* Better resource utilization

### 2. Faster Scaling

* Pods start instantly
* No IP assignment delay

### 3. Reduced API Throttling

* Fewer EC2 API calls
* More stable clusters

### 4. Lower Operational Issues

* Avoid IP exhaustion
* Predictable scaling behavior

---

## When You Should Use Prefix Delegation

Recommended for:

* Large EKS clusters
* High pod churn workloads
* CI/CD runners
* Microservices platforms

Not mandatory for:

* Very small clusters
* Low pod density workloads

---

## How to Enable Prefix Delegation in EKS

### Prerequisites

* EKS with AWS VPC CNI
* Supported EC2 instance types

### Configuration (Conceptual)

Prefix delegation is enabled by configuring the VPC CNI add-on with:

* IPv4 prefix delegation enabled

(This is usually done via EKS add-on or Helm values.)

---

## Interaction with Subnet CIDR

Important notes:

* Prefixes are allocated from subnet CIDR
* Subnet must be **large enough**
* Poor CIDR planning can still cause exhaustion

---

## Prefix Delegation vs Custom Networking

| Feature     | Prefix Delegation    | Custom Networking     |
| ----------- | -------------------- | --------------------- |
| Goal        | Increase pod density | Isolate pod traffic   |
| Complexity  | Low                  | Medium–High           |
| Performance | High                 | High                  |
| Use case    | Scale pods           | Security segmentation |

---

## Common Production Pitfalls

* Enabling prefix delegation but using small subnets
* Ignoring IP monitoring
* Mixing incompatible instance types

---

## Monitoring & Troubleshooting

Key indicators:

* Pod scheduling delays
* IP allocation failures
* Subnet free IP count

Tools:

* CloudWatch
* CNI logs

---

## Interview One-Liner (Must Remember)

"Prefix delegation in EKS allows the VPC CNI to assign CIDR prefixes to nodes instead of individual IPs, enabling higher pod density, faster scaling, and reduced AWS API calls in large production clusters."
