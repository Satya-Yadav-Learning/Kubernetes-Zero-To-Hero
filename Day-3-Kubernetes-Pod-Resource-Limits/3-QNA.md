# Kubernetes Scenario-Based Interview Questions & Answers (L1 → L4)

---

This document contains **scenario-based Kubernetes interview questions and answers**, structured from **L1 (Junior)** to **L4 (Architect)**. Each scenario is based on **real production issues** commonly seen in Kubernetes environments, especially around **resource management, performance, and stability**.

---

## 🟢 L1 – Junior Level (Foundational Understanding)

### Scenario 1: Application Pod is slow but not crashing

**Question:**
Your application Pod is responding slowly, but it is not restarting or failing. What could be the reason?

**Answer:**
The Pod is likely experiencing **CPU throttling** due to restrictive CPU limits.

**Explanation:**

* CPU is a **compressible resource** in Kubernetes
* When a container exceeds its CPU limit, Kubernetes throttles CPU usage instead of killing the container
* Throttling causes degraded performance without crashes

**Validation Command:**

```bash
kubectl describe pod <pod-name>
kubectl top pod <pod-name>
```

---

### Scenario 2: Pod restarts automatically

**Question:**
A Pod restarts frequently and shows `OOMKilled`. Why does this happen?

**Answer:**
The container exceeded its **configured memory limit**.

**Explanation:**

* Memory is **non-compressible**
* If a container exceeds its memory limit, the kernel kills it
* Kubernetes restarts the container automatically

---

### Scenario 3: Pod stuck in Pending state

**Question:**
A Pod remains in the `Pending` state and does not start. What is the most likely cause?

**Answer:**
The Pod’s **resource requests cannot be satisfied** by any node.

**Explanation:**

* Scheduler uses **requests**, not limits
* If requested CPU or memory is unavailable, the Pod stays Pending

---

## 🟡 L2 – Mid-Level (Operational Knowledge)

### Scenario 4: Application works in dev but fails in production

**Question:**
The same application works fine in development but crashes in production. What should you investigate first?

**Answer:**
Review **resource limits versus real production workload usage**.

**Explanation:**

* Production traffic is significantly higher
* Memory limits may be insufficient
* Results in frequent `OOMKilled` events

**Validation Command:**

```bash
kubectl describe pod <pod-name>
kubectl top pod <pod-name>
```

---

### Scenario 5: Horizontal Pod Autoscaler not working

**Question:**
HPA is configured, but Pods are not scaling. Why?

**Answer:**
CPU **requests are not defined** in the Pod spec.

**Explanation:**

* HPA calculates utilization based on CPU requests
* Without requests, HPA cannot determine scaling thresholds

---

### Scenario 6: Pods evicted during node pressure

**Question:**
During memory pressure, some Pods were evicted. Why these Pods?

**Answer:**
Pods with **lower QoS classes** were evicted first.

**Explanation:**
Eviction order:

1. BestEffort
2. Burstable
3. Guaranteed

---

## 🔵 L3 – Senior Level (Production & Troubleshooting)

### Scenario 7: One application impacts the entire node

**Question:**
A single application causes node instability and affects other services. How do you prevent this?

**Answer:**
Apply **resource limits, LimitRange, and ResourceQuota**.

**Explanation:**

* Prevents noisy-neighbor problems
* Enforces fair resource usage
* Improves cluster reliability

---

### Scenario 8: High latency with low CPU usage

**Question:**
Monitoring shows low CPU usage, but application latency is high. Why?

**Answer:**
The Pod is experiencing **CPU throttling due to low CPU limits**.

**Explanation:**

* Short CPU bursts hit the limit
* Kernel throttles CPU despite low average usage

---

### Scenario 9: Pod restarts without application logs

**Question:**
A Pod restarts but logs show no application error. What happened?

**Answer:**
The container was **OOMKilled** by the kernel.

**Explanation:**

* Process is terminated instantly
* Application does not get time to log the failure

---

## 🔴 L4 – Architect Level (Design & Strategy)

### Scenario 10: Designing a multi-tenant Kubernetes cluster

**Question:**
How do you design a Kubernetes cluster for multiple teams while preventing resource abuse?

**Answer:**
Use **Namespaces, ResourceQuota, LimitRange, and RBAC**.

**Explanation:**

* Namespaces isolate workloads
* ResourceQuota limits total consumption
* LimitRange enforces per-Pod defaults
* RBAC controls access and privileges

---

### Scenario 11: Ensuring guaranteed performance for critical workloads

**Question:**
How do you ensure predictable performance for a mission-critical application?

**Answer:**
Deploy Pods with **Guaranteed QoS**.

**Explanation:**

* Set CPU and memory requests equal to limits
* These Pods are last to be evicted
* Ensures stable performance under pressure

---

### Scenario 12: Cost optimization without performance degradation

**Question:**
How do you optimize cluster costs without impacting application performance?

**Answer:**
Right-size resource requests using **metrics-driven analysis**.

**Explanation:**

* Over-provisioned requests waste resources
* Use monitoring data to tune requests
* Balance efficiency and reliability

---

## 🔑 Key Interview Takeaways

* Scheduling is based on **requests**
* Enforcement is based on **limits**
* CPU is throttled; memory causes OOMKills
* QoS determines eviction priority
* Proper resource management is critical for stability and cost control

---

**Document Purpose:** Interview preparation, production readiness review, and long-term Kubernetes reference

