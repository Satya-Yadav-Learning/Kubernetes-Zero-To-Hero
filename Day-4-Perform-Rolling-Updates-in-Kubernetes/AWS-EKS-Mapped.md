## 🟠 AWS EKS–Mapped Scenario-Based Q&A (Derived from Same Task)

This section maps the **same rolling update task** to **Amazon EKS production environments**, aligning Kubernetes concepts with AWS services and operational realities.

---

### 🟢 L1 – Junior (EKS Basics)

**Scenario:** An nginx application is running on Amazon EKS with three Pods. A new image `nginx:1.19` must be deployed without downtime.

**Question:** How is this handled in EKS?

**Answer:**
EKS uses the **same Kubernetes Deployment rolling update mechanism**.

**Explanation:**

* EKS is a managed Kubernetes control plane
* Rolling updates are handled by Kubernetes, not AWS-specific services
* Worker nodes (EC2) gradually replace Pods

---

### 🟡 L2 – Mid-Level (EKS Operations)

**Scenario:** Image update fails during rollout in EKS.

**Question:** What should you check first?

**Answer:**
Check the **container name in the Deployment spec** and rollout events.

**Explanation:**

* Common errors are incorrect container names
* Use `kubectl describe deployment` and `kubectl rollout status`
* EKS behaves identically to upstream Kubernetes

---

### 🟡 L2 – Mid-Level (Autoscaling Context)

**Scenario:** Pods remain Pending during a rolling update.

**Question:** Why can this happen in EKS?

**Answer:**
Insufficient EC2 node capacity or oversized Pod resource requests.

**Explanation:**

* Scheduler places Pods based on requests
* Cluster Autoscaler may take time to add EC2 nodes
* Pods must fit on a single node

---

### 🔵 L3 – Senior (Reliability & Monitoring)

**Scenario:** Rollout succeeded but users report latency spikes.

**Question:** What AWS-specific checks do you perform?

**Answer:**
Check **CloudWatch metrics**, ALB target health, and Pod readiness.

**Explanation:**

* CPU throttling can occur on EC2 nodes
* ALB routes traffic only to healthy Pods
* Misconfigured readiness probes cause partial outages

---

### 🔵 L3 – Senior (Node-Level Impact)

**Scenario:** One Pod affects other workloads during rollout.

**Question:** How do you mitigate this in EKS?

**Answer:**
Define **resource requests/limits** and use **node groups**.

**Explanation:**

* Prevents noisy-neighbor issues on EC2
* Allows isolation of critical workloads

---

### 🔴 L4 – Architect (Production Design on AWS)

**Scenario:** Design a zero-downtime rollout strategy for EKS at scale.

**Answer:**
Use:

* Rolling updates
* Readiness probes
* HPA + Cluster Autoscaler
* Multiple node groups
* ALB/NLB health checks

**Explanation:**

* Kubernetes handles Pod lifecycle
* AWS handles infrastructure elasticity
* Proper coordination ensures resilience

---

### 🔴 L4 – Architect (Rollback & Governance)

**Scenario:** A bad nginx release impacts production traffic.

**Question:** How do you recover safely on EKS?

**Answer:**
Perform Kubernetes rollback and validate via AWS monitoring.

**Explanation:**

* `kubectl rollout undo` restores previous ReplicaSet
* CloudWatch and ALB metrics confirm recovery
* No infrastructure changes required

---

### 🔑 AWS EKS Interview Takeaways

* EKS uses upstream Kubernetes rollout mechanics
* AWS manages control plane; Kubernetes manages Pods
* Rolling updates are Kubernetes-native
* Autoscaling and monitoring are AWS-enhanced
* Clear separation of concerns improves reliability

---

---

## 🚨 Real AWS EKS Incident RCAs (Production-Grade)

The following **realistic AWS incident-style Root Cause Analyses (RCAs)** are mapped directly to the **nginx rolling update task** and are commonly discussed in **senior DevOps / SRE / Architect interviews**.

---

### Incident 1: ALB 502 Errors During Rolling Update

**Impact:**
Users experienced intermittent `502 Bad Gateway` errors during deployment.

**Timeline:**

* T0: Rolling update started on EKS
* T+2 min: ALB target health checks failed
* T+5 min: Traffic errors reported

**Root Cause:**
Readiness probe was missing or misconfigured.

**Technical Explanation:**

* ALB routed traffic to Pods immediately after container start
* Application was not ready to accept traffic
* Pods were marked Ready too early

**Resolution:**

* Added proper HTTP readiness probe
* Ensured Pods only receive traffic when fully initialized

**Preventive Actions:**

* Mandatory readiness probes for all production services
* Deployment validation in staging before prod

---

### Incident 2: Pods Stuck in Pending During Rollout

**Impact:**
Rolling update stalled; new Pods never started.

**Root Cause:**
Pod resource requests exceeded available EC2 instance capacity.

**Technical Explanation:**

* Scheduler could not place new Pods
* Cluster Autoscaler was slow to scale
* Pods must fit on a single EC2 node

**Resolution:**

* Reduced Pod resource requests
* Added larger EC2 instance type to node group

**Preventive Actions:**

* Right-size requests using CloudWatch metrics
* Pre-scale node groups before large deployments

---

### Incident 3: Node CPU Starvation During Deployment

**Impact:**
High latency across multiple services during rollout.

**Root Cause:**
Pods had no CPU limits (BestEffort QoS).

**Technical Explanation:**

* One Pod consumed excessive CPU
* Other Pods were starved
* EC2 node became CPU saturated

**Resolution:**

* Added CPU requests and limits
* Restarted affected Pods

**Preventive Actions:**

* Enforce LimitRange at namespace level
* Continuous resource tuning

---

### Incident 4: Rollout Completed but Wrong Version Served

**Impact:**
Some users still saw old application behavior.

**Root Cause:**
Old Pods were still registered in ALB target group.

**Technical Explanation:**

* Slow deregistration delay on ALB
* Old Pods continued receiving traffic briefly

**Resolution:**

* Tuned ALB deregistration delay
* Ensured readiness + terminationGracePeriod alignment

**Preventive Actions:**

* Align ALB health checks with Pod lifecycle
* Validate traffic cutover during canary releases

---

### Incident 5: Failed Release Required Immediate Rollback

**Impact:**
New nginx version introduced configuration errors.

**Root Cause:**
Insufficient pre-production testing.

**Technical Explanation:**

* Rolling update deployed faulty image
* Errors surfaced only under production traffic

**Resolution:**

```bash
kubectl rollout undo deployment/nginx-deployment
```

**Preventive Actions:**

* Canary deployments
* Automated smoke tests in CI/CD
* Rollout pause + manual approval gates

---

## 🔑 Interview Takeaway from RCAs

* Most EKS incidents are **configuration-driven**, not platform failures
* Readiness probes and resource limits prevent majority of outages
* Rollbacks are faster and safer than redeployments
* AWS provides infrastructure, Kubernetes controls behavior

---

**Document Purpose:** End-to-end AWS EKS interview preparation with real incident RCAs, operational depth, and architecture insight

