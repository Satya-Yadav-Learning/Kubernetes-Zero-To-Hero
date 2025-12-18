# AWS-Mapped Kubernetes Scenario-Based Interview Questions & Answers (L1 → L4)

---

This document maps **Kubernetes scenario-based interview questions** to **real AWS (primarily EKS) production scenarios**. It is structured from **L1 (Junior)** to **L4 (Architect)** and focuses on **resource constraints, performance, scaling, reliability, and cost optimization** in AWS environments.

---

## 🟢 L1 – Junior Level (AWS + Kubernetes Basics)

### Scenario 1: Pod is slow on Amazon EKS but not crashing

**AWS Context:** Amazon EKS running on EC2 worker nodes

**Question:**
An application Pod on EKS is responding slowly but is not restarting. What could be the cause?

**Answer:**
The Pod is likely **CPU throttled** due to low CPU limits on a shared EC2 worker node.

**Explanation:**

* EKS worker nodes share EC2 CPU resources
* CPU is compressible; exceeding limits causes throttling
* Throttling impacts latency without crashing the container

**AWS Validation:**

```bash
kubectl top pod <pod-name>
```

CloudWatch can show node-level CPU metrics.

---

### Scenario 2: Pod restarts with OOMKilled on EKS

**Question:**
A Pod running on EKS restarts and shows `OOMKilled`. Why?

**Answer:**
The container exceeded its **memory limit**, triggering a kernel-level kill on the EC2 node.

**Explanation:**

* Memory is non-compressible
* Linux kernel on EC2 enforces cgroup memory limits
* Kubernetes restarts the container automatically

---

### Scenario 3: Pod stuck in Pending state on EKS

**Question:**
A Pod remains in `Pending` state after deployment. What is the most common reason?

**Answer:**
The Pod’s **resource requests exceed available capacity** on the EC2 worker nodes.

**Explanation:**

* Scheduler checks resource requests
* If EC2 nodes lack required CPU/memory, Pod stays Pending

---

## 🟡 L2 – Mid-Level (Operations on AWS)

### Scenario 4: Application works in dev EKS but fails in prod EKS

**Question:**
An application runs fine in dev EKS but frequently crashes in production. What should you check first?

**Answer:**
Check **resource limits and traffic patterns** in production.

**Explanation:**

* Production load is higher
* Memory limits may be too low
* Results in frequent OOMKilled events

**AWS Angle:**
Use CloudWatch metrics for node memory pressure.

---

### Scenario 5: HPA not scaling Pods on EKS

**Question:**
HPA is configured, but Pods are not scaling in EKS. Why?

**Answer:**
CPU requests are missing or incorrectly set.

**Explanation:**

* HPA relies on CPU requests
* Metrics Server must be installed on EKS

---

### Scenario 6: Pods evicted during EC2 node pressure

**Question:**
During high load, some Pods are evicted from EC2 nodes. Why?

**Answer:**
Pods with **lower QoS classes** were evicted first.

**Explanation:**
Eviction priority on EKS nodes:

1. BestEffort
2. Burstable
3. Guaranteed

---

## 🔵 L3 – Senior Level (Reliability & Scaling on AWS)

### Scenario 7: One application affects entire EKS node

**Question:**
A single Pod causes EC2 node instability and impacts other services. How do you prevent this?

**Answer:**
Use **resource limits, LimitRange, ResourceQuota**, and proper **node sizing**.

**Explanation:**

* Prevents noisy-neighbor issues
* Ensures fair resource sharing
* Stabilizes EC2 worker nodes

---

### Scenario 8: EKS Pods Pending despite Cluster Autoscaler

**Question:**
Pods remain Pending even though Cluster Autoscaler is enabled. Why?

**Answer:**
Resource requests exceed **maximum EC2 instance size** in the node group.

**Explanation:**

* Autoscaler adds nodes but cannot exceed instance limits
* Pod requests must fit on a single node

---

### Scenario 9: High latency with low CPU utilization

**Question:**
CloudWatch shows low EC2 CPU usage, but application latency is high. Why?

**Answer:**
The Pod is CPU throttled due to restrictive limits.

**Explanation:**

* Short CPU bursts hit limits
* Throttling occurs despite low average CPU usage

---

## 🔴 L4 – Architect Level (AWS Design & Strategy)

### Scenario 10: Designing a multi-tenant EKS cluster

**Question:**
How do you design an EKS cluster for multiple teams while preventing resource abuse?

**Answer:**
Use **Namespaces, ResourceQuota, LimitRange, RBAC, and IAM Roles for Service Accounts (IRSA)**.

**Explanation:**

* Namespaces isolate teams
* ResourceQuota limits usage
* IRSA provides fine-grained AWS access

---

### Scenario 11: Guaranteed performance for critical AWS workloads

**Question:**
How do you ensure guaranteed performance for mission-critical workloads on EKS?

**Answer:**
Use **Guaranteed QoS Pods** on **dedicated or tainted EC2 nodes**.

**Explanation:**

* Requests = limits
* Use node affinity and taints
* Protects critical services during node pressure

---

### Scenario 12: Cost optimization in EKS without performance loss

**Question:**
How do you reduce AWS costs while maintaining application performance?

**Answer:**
Right-size Pods and nodes using **metrics and autoscaling**.

**Explanation:**

* Over-provisioned requests waste EC2 capacity
* Use Cluster Autoscaler and HPA together
* Leverage Spot Instances for non-critical workloads

---

## 🔑 Key AWS Interview Takeaways

* EKS scheduling depends on Pod requests
* EC2 enforces CPU and memory limits
* QoS impacts eviction during node pressure
* Autoscaling requires correct resource sizing
* Cost optimization is a balance of performance and efficiency

---

**Document Purpose:** AWS-focused Kubernetes interview preparation, EKS production readiness, and architecture reference

