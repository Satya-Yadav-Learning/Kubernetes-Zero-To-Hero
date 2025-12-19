# AWS EKS–Mapped Scenario-Based Interview Q&A

## Kubernetes Deployment Rollback on Amazon EKS

---

This document maps a **real Kubernetes rollback task** (`kubectl rollout undo deployment/nginx-deployment`) to **Amazon EKS production environments**.

It explains how rollback works when Kubernetes runs on AWS infrastructure and how this is evaluated in **DevOps / SRE / Platform Engineer interviews**.

The questions are structured from **L1 (Junior)** to **L4 (Architect)**.

---

## 🟢 L1 – Junior Level (EKS Fundamentals)

### Scenario 1: Buggy release on Amazon EKS

**Question:**
An application running on Amazon EKS was updated, and users reported a bug. How do you revert to the previous version?

**Answer:**
By performing a **Kubernetes Deployment rollback** using `kubectl rollout undo`.

**Explanation:**

* Amazon EKS provides a managed Kubernetes control plane
* Rollback is a **Kubernetes-native feature**, not an AWS-specific one
* EKS uses the same Deployment and ReplicaSet mechanics as upstream Kubernetes

**Command:**

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

### Scenario 2: Verifying rollback availability

**Question:**
How do you check whether a rollback is possible on EKS?

**Answer:**
By viewing the Deployment rollout history.

**Explanation:**

* Each revision represents a stored ReplicaSet
* If multiple revisions exist, rollback is supported

**Command:**

```bash
kubectl rollout history deployment/nginx-deployment
```

---

## 🟡 L2 – Mid-Level (EKS Operations)

### Scenario 3: Ensuring the correct EKS cluster

**Question:**
Before performing a rollback, how do you confirm you are connected to the correct EKS cluster?

**Answer:**
By verifying the current kubectl context and cluster endpoint.

**Explanation:**

* EKS environments often have multiple clusters (dev, staging, prod)
* Wrong context can cause a production outage

**Commands:**

```bash
kubectl config current-context
kubectl config get-contexts
kubectl cluster-info
```

---

### Scenario 4: Pods recreated during rollback

**Question:**
After rollback, why are new Pods created instead of reusing existing ones?

**Answer:**
Because Kubernetes activates the previous ReplicaSet and creates fresh Pods.

**Explanation:**

* ReplicaSets are immutable snapshots of Pod templates
* Fresh Pods ensure consistency and correctness

---

### Scenario 5: Validating rollback success in EKS

**Question:**
How do you validate that rollback succeeded on Amazon EKS?

**Answer:**
By verifying Pod status and rollout completion.

**Commands:**

```bash
kubectl get pods
kubectl rollout status deployment/nginx-deployment
```

---

## 🔵 L3 – Senior Level (Reliability & AWS Integration)

### Scenario 6: Rollback succeeded but users still see errors

**Question:**
Rollback completed successfully, but users still report errors. What AWS-specific checks do you perform?

**Answer:**
Check **ALB/NLB target health**, CloudWatch metrics, and application dependencies.

**Explanation:**

* Load balancers may still route traffic to terminating Pods
* Errors may originate from downstream AWS services (RDS, S3, external APIs)

---

### Scenario 7: Node capacity impact during rollback

**Question:**
Why is EC2 node health important during a rollback on EKS?

**Answer:**
Rollback still schedules new Pods, which require EC2 capacity.

**Explanation:**

* If nodes are `NotReady` or out of resources, rollback Pods may fail
* Cluster Autoscaler may need time to add nodes

**Command:**

```bash
kubectl get nodes -o wide
```

---

### Scenario 8: BestEffort Pods during rollback

**Question:**
Why are BestEffort Pods risky during rollback in EKS?

**Answer:**
They are evicted first during EC2 node pressure.

**Explanation:**

* No guaranteed CPU or memory
* Increased risk during scaling or traffic spikes

---

## 🔴 L4 – Architect Level (Design & Strategy on AWS)

### Scenario 9: Rollback vs redeploy on EKS

**Question:**
Why is Kubernetes rollback preferred over rebuilding and redeploying an old image on AWS?

**Answer:**
Rollback is faster, safer, and infrastructure-independent.

**Explanation:**

* No EC2, ASG, or ALB changes required
* Uses already-tested ReplicaSets
* Reduces blast radius during incidents

---

### Scenario 10: Zero-downtime rollback architecture

**Question:**
How do you ensure zero downtime during rollback in an EKS production environment?

**Answer:**
Combine Kubernetes and AWS features.

**Explanation:**

* Kubernetes rolling rollback ensures Pod readiness
* ALB health checks prevent traffic to unhealthy Pods
* Proper terminationGracePeriod ensures clean traffic drain

---

### Scenario 11: Production hardening on EKS

**Question:**
What best practices improve rollback safety on Amazon EKS?

**Answer:**

* Readiness and liveness probes
* Resource requests and limits
* Canary or blue-green deployments
* CI/CD-integrated rollback automation

**Explanation:**
These practices reduce MTTR and production risk.

---

## 🔑 Interview One-Liner (AWS EKS)

> “In Amazon EKS, rollback is handled entirely by Kubernetes using Deployment revision history, while AWS provides the underlying compute, networking, and monitoring to ensure the rollback completes safely.”

---

## Final Takeaways

* EKS uses **upstream Kubernetes rollback mechanics**
* AWS manages infrastructure, Kubernetes manages application state
* Rollback is faster and safer than redeploying images
* Strong validation is critical in multi-cluster AWS environments

---

---

## 🚨 Real AWS EKS Rollback Incident RCAs (Production)

The following **real-world rollback-related incident RCAs** are commonly seen in Amazon EKS environments and are directly relevant to the rollback scenario discussed above.

---

### Incident 1: Rollback Completed but Users Still See Errors

**Impact:**
Intermittent 5xx errors continued even after rollback was marked successful.

**Root Cause:**
ALB target group still routing traffic to terminating Pods from the failed release.

**Technical Details:**

* Kubernetes rollback recreated Pods from the previous ReplicaSet
* Old Pods entered `Terminating` state
* ALB deregistration delay was higher than Pod termination timing

**Resolution:**

* Tuned ALB deregistration delay
* Increased `terminationGracePeriodSeconds`
* Ensured readiness probes correctly reflect application readiness

**Preventive Actions:**

* Align ALB health checks with Kubernetes readiness probes
* Validate traffic cutover during rollback testing

---

### Incident 2: Rollback Stuck Due to Insufficient EC2 Capacity

**Impact:**
Rollback stalled; new Pods remained in `Pending` state.

**Root Cause:**
EC2 node group had insufficient capacity to schedule rollback Pods.

**Technical Details:**

* Rollback still requires scheduling new Pods
* Cluster Autoscaler did not scale fast enough
* Pod resource requests exceeded available node resources

**Resolution:**

* Manually scaled EC2 Auto Scaling Group
* Reduced Pod resource requests temporarily

**Preventive Actions:**

* Pre-scale node groups before risky deployments
* Right-size resource requests using CloudWatch metrics

---

### Incident 3: Rollback Triggered Pod Evictions

**Impact:**
Multiple non-related applications were evicted during rollback.

**Root Cause:**
Pods were running with `BestEffort` QoS.

**Technical Details:**

* Rollback created CPU and memory pressure on EC2 nodes
* BestEffort Pods were evicted first

**Resolution:**

* Added CPU and memory requests/limits
* Restarted affected workloads

**Preventive Actions:**

* Enforce `LimitRange` and `ResourceQuota`
* Avoid BestEffort Pods in production EKS clusters

---

### Incident 4: Wrong Revision Rolled Back in Production

**Impact:**
Application reverted to an even older, incompatible version.

**Root Cause:**
Rollback executed without verifying revision history.

**Technical Details:**

* Multiple revisions existed
* Operator assumed previous revision was correct
* Change cause was not reviewed

**Resolution:**

* Rolled forward to the correct revision
* Verified image version explicitly

**Preventive Actions:**

* Always review `kubectl rollout history`
* Use `--record=true` for all production changes

---

### Incident 5: Rollback Succeeded but Config Bug Persisted

**Impact:**
Application errors continued after rollback.

**Root Cause:**
Bug was introduced via ConfigMap, not container image.

**Technical Details:**

* Rollback restored the image but not the configuration
* ConfigMap was updated independently of Deployment

**Resolution:**

* Rolled back ConfigMap
* Restarted Pods to apply correct configuration

**Preventive Actions:**

* Version ConfigMaps and Secrets
* Include config rollback steps in incident runbooks

---

## 🔑 Interview Takeaways from AWS Rollback RCAs

* Rollback success ≠ user impact resolved
* AWS load balancer behavior matters during rollback
* Node capacity and QoS directly affect rollback safety
* Always validate revision history and configuration sources

---

**Document Purpose:** AWS EKS rollback interview preparation with real incident RCAs and production-grade operational insight

